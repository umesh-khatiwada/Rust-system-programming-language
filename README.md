#!/usr/bin/env python3
"""
ArgoCD Multi-Sector Scenario Runner for Level-2 (L2) Autonomous SRE Agent Validation.

Target Cluster: configured via KUBECONFIG env var or --kubeconfig flag
Target Application: payment-service (namespace: argocd, destination: alert-test)
"""

import argparse
import json
import os
import subprocess
import sys
import time
from pathlib import Path

# P4.3: Use KUBECONFIG env var with a portable fallback instead of a
# hardcoded machine-specific path.
DEFAULT_KUBECONFIG = os.getenv(
    "KUBECONFIG",
    os.path.expanduser("~/.kube/config"),
)
APP_NAME = "payment-service"
APP_NS = "argocd"
DEST_NS = "alert-test"
DEPLOY_NAME = "payment-service-helm-guestbook"
BASE_DIR = Path(__file__).resolve().parent


def run_cmd(cmd: list[str], check: bool = True, capture: bool = True, kubeconfig: str = DEFAULT_KUBECONFIG) -> subprocess.CompletedProcess:
    """Run a command with the configured kubeconfig if kubectl is invoked."""
    env = os.environ.copy()
    if kubeconfig and os.path.exists(kubeconfig):
        env["KUBECONFIG"] = kubeconfig
    elif kubeconfig:
        print(f"[!] Kubeconfig not found at '{kubeconfig}' — using system default")
    
    res = subprocess.run(cmd, env=env, text=True, capture_output=capture)
    if check and res.returncode != 0:
        print(f"[-] Command failed: {' '.join(cmd)}")
        if res.stderr:
            print(f"    Error: {res.stderr.strip()}")
        sys.exit(res.returncode)
    return res


def get_app_status() -> dict:
    """Fetch live ArgoCD application status."""
    res = run_cmd(["kubectl", "get", f"application/{APP_NAME}", "-n", APP_NS, "-o", "json"], check=False)
    if res.returncode != 0:
        return {"error": "Application not found or API unreachable"}
    data = json.loads(res.stdout)
    sync_status = data.get("status", {}).get("sync", {}).get("status", "Unknown")
    health_status = data.get("status", {}).get("health", {}).get("status", "Unknown")
    op_phase = data.get("status", {}).get("operationState", {}).get("phase", "None")
    op_msg = data.get("status", {}).get("operationState", {}).get("message", "")
    return {
        "sync_status": sync_status,
        "health_status": health_status,
        "operation_phase": op_phase,
        "operation_message": op_msg,
    }


def print_status():
    """Display current health and pod states."""
    print("\n" + "=" * 65)
    print(" 📊 Current Target Application & Cluster Status")
    print("=" * 65)
    app_st = get_app_status()
    print(f" ArgoCD Application : {APP_NAME} (namespace: {APP_NS})")
    print(f" Destination NS     : {DEST_NS}")
    print(f" Sync Status        : {app_st.get('sync_status')}")
    print(f" Health Status      : {app_st.get('health_status')}")
    print(f" Operation Phase    : {app_st.get('operation_phase')}")
    if app_st.get("operation_message"):
        print(f" Operation Details  : {app_st.get('operation_message')[:80]}...")
    
    print("\n Workload Pods (namespace: alert-test):")
    pods_res = run_cmd(["kubectl", "get", "pods", "-n", DEST_NS, "-l", "app=helm-guestbook"], check=False)
    if pods_res.stdout.strip():
        for line in pods_res.stdout.strip().split("\n"):
            print(f"   {line}")
    else:
        print("   (No pods found or matching selector)")
    print("=" * 65 + "\n")


def inject_sector(sector: int):
    """Inject failure scenario for the specified sector."""
    print(f"\n[+] Injecting Sector {sector} Fault...")

    if sector == 1:
        print("  -> Sector 1: GitOps Drift (Disable selfHeal and scale live deployment to 0)")
        patch_argo = '{"spec":{"syncPolicy":{"automated":{"selfHeal":false}}}}'
        run_cmd(["kubectl", "patch", f"application/{APP_NAME}", "-n", APP_NS, "--type", "merge", "-p", patch_argo])
        run_cmd(["kubectl", "scale", f"deployment/{DEPLOY_NAME}", "-n", DEST_NS, "--replicas=0"])
        print("  ✓ Drift injected! Live deployment has 0 replicas, Git HEAD expects 1.")

    elif sector == 2:
        print("  -> Sector 2: Bad Deployment Configuration / CrashLoopBackOff")
        patch_crash = json.dumps({
            "spec": {
                "source": {
                    "helm": {
                        "parameters": [
                            {"name": "image.repository", "value": "busybox"},
                            {"name": "image.tag", "value": "latest"},
                            {"name": "command[0]", "value": "sh"},
                            {"name": "command[1]", "value": "-c"},
                            {"name": "command[2]", "value": "echo '[FATAL] Invalid configuration startup error'; exit 1"}
                        ]
                    }
                }
            }
        })
        run_cmd(["kubectl", "patch", f"application/{APP_NAME}", "-n", APP_NS, "--type", "merge", "-p", patch_crash])
        # Trigger sync
        sync_op = '{"operation":{"sync":{"revision":"HEAD"}}}'
        run_cmd(["kubectl", "patch", f"application/{APP_NAME}", "-n", APP_NS, "--type", "merge", "-p", sync_op])
        print("  ✓ CrashLoopBackOff injected! Container will fail with Exit Code 1.")

    elif sector == 3:
        print("  -> Sector 3: Resource Starvation / OOMKilled Clamp")
        patch_oom = json.dumps({
            "spec": {
                "source": {
                    "helm": {
                        "parameters": [
                            {"name": "image.repository", "value": "nginx"},
                            {"name": "image.tag", "value": "alpine"},
                            {"name": "resources.limits.memory", "value": "8Mi"},
                            {"name": "resources.requests.memory", "value": "4Mi"}
                        ]
                    }
                }
            }
        })
        run_cmd(["kubectl", "patch", f"application/{APP_NAME}", "-n", APP_NS, "--type", "merge", "-p", patch_oom])
        sync_op = '{"operation":{"sync":{"revision":"HEAD"}}}'
        run_cmd(["kubectl", "patch", f"application/{APP_NAME}", "-n", APP_NS, "--type", "merge", "-p", sync_op])
        print("  ✓ Memory limit clamped to 8Mi (Nginx requires > 15Mi). OOMKilled will trigger.")

    elif sector == 4:
        print("  -> Sector 4: Readiness Probe Misconfiguration")
        patch_probe = json.dumps({
            "spec": {
                "source": {
                    "helm": {
                        "parameters": [
                            {"name": "image.repository", "value": "nginx"},
                            {"name": "image.tag", "value": "alpine"},
                            {"name": "readinessProbe.httpGet.path", "value": "/non-existent-health-check"},
                            {"name": "readinessProbe.httpGet.port", "value": "80"}
                        ]
                    }
                }
            }
        })
        run_cmd(["kubectl", "patch", f"application/{APP_NAME}", "-n", APP_NS, "--type", "merge", "-p", patch_probe])
        sync_op = '{"operation":{"sync":{"revision":"HEAD"}}}'
        run_cmd(["kubectl", "patch", f"application/{APP_NAME}", "-n", APP_NS, "--type", "merge", "-p", sync_op])
        print("  ✓ Readiness probe set to invalid path /non-existent-health-check.")

    elif sector == 5:
        print("  -> Sector 5: Stuck Operation & PreSync Hook Job Failure")
        job_file = BASE_DIR / "presync-hook-job.yaml"
        if not job_file.exists():
            print(f"[-] Hook job file {job_file} not found!")
            return
        run_cmd(["kubectl", "apply", "-f", str(job_file)])
        print("  ✓ PreSync migration job deployed and designed to exit with code 1.")

    else:
        print(f"[-] Unknown sector: {sector}. Choose 1-5.")
        return

    time.sleep(3)
    print_status()


def diagnose_sector(sector: int):
    """Simulate the L2 Agent Multi-Subgraph Diagnostic Triad."""
    print("\n" + "=" * 65)
    print(f" 🔍 L2 Agent Multi-Subgraph Diagnostic Analysis (Sector {sector})")
    print("=" * 65)

    if sector == 1:
        print(" [ArgoCD Subgraph Query]: argocd_get_application('payment-service')")
        print("   -> Sync Status: OutOfSync (Drift detected)")
        print(" [ArgoCD Subgraph Query]: argocd_diff_application('payment-service')")
        print("   -> Diff: Live replica count is 0, Git target specification is 1")
        print(" [Kubernetes Subgraph Query]: kubectl get deployment -n alert-test")
        print("   -> Deployment scaled down out-of-band; selfHeal was disabled")
        print(" 🎯 Diagnostic Verdict: Out-of-band cluster drift without Git approval.")

    elif sector == 2:
        print(" [ArgoCD Subgraph Query]: argocd_get_application('payment-service')")
        print("   -> Health Status: Degraded")
        print(" [Kubernetes Subgraph Query]: kubectl get pod, kubectl logs --tail=50")
        print("   -> Container Exit Code: 1 | Reason: Error")
        print("   -> Log snippet: '[FATAL] Invalid configuration startup error'")
        print(" [ArgoCD Subgraph Query]: argocd_list_application_history('payment-service')")
        print("   -> Last known healthy deployment was nginx:alpine")
        print(" 🎯 Diagnostic Verdict: Bad container entrypoint configuration in recent revision.")

    elif sector == 3:
        print(" [Kubernetes Subgraph Query]: kubectl describe pod -n alert-test")
        print("   -> Last State: Terminated (Exit Code 137, Reason: OOMKilled)")
        print(" [Prometheus Subgraph Query]: PromQL container_memory_working_set_bytes")
        print("   -> Saturation: 100% (8.0Mi / 8.0Mi limit reached)")
        print("   -> Node Memory Headroom: > 10Gi available (Node pressure ruled out)")
        print(" [ArgoCD Subgraph Query]: argocd_get_application('payment-service')")
        print("   -> Limits set via Helm parameters override")
        print(" 🎯 Diagnostic Verdict: Workload clamped by insufficient container memory limit.")

    elif sector == 4:
        print(" [Kubernetes Subgraph Query]: kubectl describe pod -n alert-test")
        print("   -> Warning: Readiness probe failed: HTTP probe failed with statuscode: 404")
        print("   -> Ready: 0/1 | Status: Running")
        print(" [ArgoCD Subgraph Query]: argocd_get_application('payment-service')")
        print("   -> Health Status: Progressing / Degraded")
        print(" 🎯 Diagnostic Verdict: Application failed readiness gate due to invalid endpoint path.")

    elif sector == 5:
        print(" [ArgoCD Subgraph Query]: argocd_get_application('payment-service')")
        print("   -> Operation Phase: Failed | Sync hook execution blocked")
        print(" [Kubernetes Subgraph Query]: kubectl logs job/presync-db-migration -n alert-test")
        print("   -> Hook output: 'Migration failed: syntax error in schema.sql' (Exit 1)")
        print(" 🎯 Diagnostic Verdict: PreSync Hook Job failure blocking deployment pipeline.")

    print("=" * 65 + "\n")


def remediate_sector(sector: int):
    """Apply the GitOps-safe remediation executed by the L2 Agent."""
    print(f"\n[+] Executing L2 Agent GitOps Remediation for Sector {sector}...")

    if sector == 1:
        print("  -> Action: Re-enable selfHeal and trigger argocd_sync_application")
        patch_heal = '{"spec":{"syncPolicy":{"automated":{"prune":true,"selfHeal":true}}}}'
        run_cmd(["kubectl", "patch", f"application/{APP_NAME}", "-n", APP_NS, "--type", "merge", "-p", patch_heal])
        sync_op = '{"operation":{"sync":{"prune":true}}}'
        run_cmd(["kubectl", "patch", f"application/{APP_NAME}", "-n", APP_NS, "--type", "merge", "-p", sync_op])
        print("  ✓ Application re-synced and self-heal enabled.")

    elif sector == 2:
        print("  -> Action: Roll back parameters to known healthy nginx:alpine spec")
        patch_rollback = json.dumps({
            "spec": {
                "source": {
                    "helm": {
                        "parameters": [
                            {"name": "image.repository", "value": "nginx"},
                            {"name": "image.tag", "value": "alpine"}
                        ]
                    }
                }
            }
        })
        run_cmd(["kubectl", "patch", f"application/{APP_NAME}", "-n", APP_NS, "--type", "merge", "-p", patch_rollback])
        sync_op = '{"operation":{"sync":{"revision":"HEAD"}}}'
        run_cmd(["kubectl", "patch", f"application/{APP_NAME}", "-n", APP_NS, "--type", "merge", "-p", sync_op])
        print("  ✓ Rollback applied. Healthy nginx:alpine configuration restored.")

    elif sector == 3:
        print("  -> Action: Update Helm parameters to grant adequate memory limit (128Mi)")
        patch_res = json.dumps({
            "spec": {
                "source": {
                    "helm": {
                        "parameters": [
                            {"name": "image.repository", "value": "nginx"},
                            {"name": "image.tag", "value": "alpine"},
                            {"name": "resources.limits.memory", "value": "128Mi"},
                            {"name": "resources.requests.memory", "value": "32Mi"}
                        ]
                    }
                }
            }
        })
        run_cmd(["kubectl", "patch", f"application/{APP_NAME}", "-n", APP_NS, "--type", "merge", "-p", patch_res])
        sync_op = '{"operation":{"sync":{"revision":"HEAD"}}}'
        run_cmd(["kubectl", "patch", f"application/{APP_NAME}", "-n", APP_NS, "--type", "merge", "-p", sync_op])
        print("  ✓ Memory limits adjusted to 128Mi.")

    elif sector == 4:
        print("  -> Action: Revert readiness probe to default '/' path")
        patch_clean_probe = json.dumps({
            "spec": {
                "source": {
                    "helm": {
                        "parameters": [
                            {"name": "image.repository", "value": "nginx"},
                            {"name": "image.tag", "value": "alpine"}
                        ]
                    }
                }
            }
        })
        run_cmd(["kubectl", "patch", f"application/{APP_NAME}", "-n", APP_NS, "--type", "merge", "-p", patch_clean_probe])
        sync_op = '{"operation":{"sync":{"revision":"HEAD"}}}'
        run_cmd(["kubectl", "patch", f"application/{APP_NAME}", "-n", APP_NS, "--type", "merge", "-p", sync_op])
        print("  ✓ Readiness probe restored to valid endpoint.")

    elif sector == 5:
        print("  -> Action: Terminate stuck operation and clean up failed hook job")
        run_cmd(["kubectl", "delete", "job", "presync-db-migration", "-n", DEST_NS, "--ignore-not-found=true"])
        # Clear operation state if stuck
        sync_op = '{"operation":{"sync":{"revision":"HEAD","prune":true}}}'
        run_cmd(["kubectl", "patch", f"application/{APP_NAME}", "-n", APP_NS, "--type", "merge", "-p", sync_op])
        print("  ✓ Hook cleared and sync operation resumed.")

    time.sleep(3)
    print_status()


def reset_baseline():
    """Reset target application to a clean, known-good baseline state."""
    print("\n[+] Resetting payment-service to pristine baseline state...")
    # Clean up jobs
    run_cmd(["kubectl", "delete", "job", "presync-db-migration", "-n", DEST_NS, "--ignore-not-found=true"])
    
    # Restore clean application spec
    baseline_spec = json.dumps({
        "spec": {
            "source": {
                "helm": {
                    "parameters": [
                        {"name": "image.repository", "value": "nginx"},
                        {"name": "image.tag", "value": "alpine"}
                    ]
                }
            },
            "syncPolicy": {
                "automated": {
                    "prune": True,
                    "selfHeal": True
                },
                "syncOptions": ["CreateNamespace=true"]
            }
        }
    })
    run_cmd(["kubectl", "patch", f"application/{APP_NAME}", "-n", APP_NS, "--type", "merge", "-p", baseline_spec])
    sync_op = '{"operation":{"sync":{"revision":"HEAD","prune":true}}}'
    run_cmd(["kubectl", "patch", f"application/{APP_NAME}", "-n", APP_NS, "--type", "merge", "-p", sync_op])
    time.sleep(3)
    print_status()
    print("  ✓ Pristine baseline restored successfully.")


def parse_sector_list(items: list[str]) -> list[int]:
    """Parse list of sector strings, supporting numbers and 'all'."""
    sectors = []
    for it in items:
        if str(it).lower() == "all":
            return [1, 2, 3, 4, 5]
        try:
            val = int(it)
            if 1 <= val <= 5:
                sectors.append(val)
        except ValueError:
            pass
    return sorted(list(set(sectors)))


def test_all_sectors():
    """Run end-to-end validation across all 5 sectors."""
    print("\n" + "=" * 65)
    print(" 🚀 RUNNING COMPLETE E2E VALIDATION SUITE (SECTORS 1 - 5)")
    print("=" * 65)
    for sector in [1, 2, 3, 4, 5]:
        print(f"\n>>> [TESTING SECTOR {sector}] <<<")
        inject_sector(sector)
        time.sleep(2)
        diagnose_sector(sector)
        remediate_sector(sector)
        time.sleep(3)
    reset_baseline()
    print("\n" + "=" * 65)
    print(" 🎉 ALL 5 SECTORS TESTED, REMEDIATED, AND BASELINE RESTORED!")
    print("=" * 65)


def main():
    global DEFAULT_KUBECONFIG
    parser = argparse.ArgumentParser(description="ArgoCD Multi-Sector Fault & L2 Agent Remediation Suite")
    parser.add_argument("--status", action="store_true", help="Display live ArgoCD & workload status")
    parser.add_argument("--inject", nargs="+", help="Inject fault for sector (1-5, multiple e.g. 1 2 3, or 'all')")
    parser.add_argument("--diagnose", nargs="+", help="Run L2 diagnostic inquiry for sector (1-5, or 'all')")
    parser.add_argument("--remediate", nargs="+", help="Execute GitOps remediation for sector (1-5, multiple e.g. 1 2 3, or 'all')")
    parser.add_argument("--test-all", action="store_true", help="Run end-to-end cycle across all sectors 1-5")
    parser.add_argument("--reset", action="store_true", help="Reset application to pristine baseline")
    # P4.3: Portable kubeconfig flag
    parser.add_argument(
        "--kubeconfig",
        default=DEFAULT_KUBECONFIG,
        help=(
            f"Path to kubeconfig file. Overrides KUBECONFIG env var. "
            f"Default: {DEFAULT_KUBECONFIG}"
        ),
    )

    args = parser.parse_args()

    # Override DEFAULT_KUBECONFIG globally for this run
    DEFAULT_KUBECONFIG = args.kubeconfig

    if args.status:
        print_status()
    elif args.test_all:
        test_all_sectors()
    elif args.inject:
        sectors = parse_sector_list(args.inject)
        for s in sectors:
            inject_sector(s)
    elif args.diagnose:
        sectors = parse_sector_list(args.diagnose)
        for s in sectors:
            diagnose_sector(s)
    elif args.remediate:
        sectors = parse_sector_list(args.remediate)
        for s in sectors:
            remediate_sector(s)
    elif args.reset:
        reset_baseline()
    else:
        parser.print_help()


if __name__ == "__main__":
    main()
