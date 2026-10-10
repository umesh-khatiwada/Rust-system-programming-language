# Learning Rust on Linux: Beginner to Intermediate

A 12-week study plan, setup guide, and ready-to-use README for your learning repository.


Rust Book: https://doc.rust-lang.org/book/
Rust Install Docs: https://doc.rust-lang.org/book/ch01-0...
Rust Installer Download Page: https://www.rust-lang.org/tools/install


## 1. Overview

**Goal:** Go from zero Rust knowledge to writing intermediate-level programs: ownership and borrowing, traits and generics, error handling, iterators, concurrency, testing, and publishing a real CLI tool.

**Time commitment:** about 8 to 10 hours per week for 12 weeks. If you can do more, compress it; if less, stretch each week to two.

**Learning approach:**

- Read a little, code a lot. Every concept gets a small program.
- Fight the compiler on purpose. Rust's error messages are a teacher.
- Build one project every two to three weeks.
- Keep a learning journal of mistakes and "aha" moments.

**What "intermediate" means here:** you can design small to medium programs without constant lookups, read other people's Rust, use common crates, write tests, handle errors idiomatically, and reason about ownership, lifetimes, and threads.

## 2. Linux Environment Setup

### 2.1 Install system prerequisites

Rust needs a C linker and build tools.

```bash
# Debian / Ubuntu / Mint
sudo apt update && sudo apt install -y build-essential curl git pkg-config libssl-dev

# Fedora
sudo dnf groupinstall -y "Development Tools" && sudo dnf install -y curl git openssl-devel

# Arch / Manjaro
sudo pacman -S --needed base-devel curl git openssl
```

### 2.2 Install Rust with rustup

Use the official installer rather than your distro's package, so you can update easily and switch toolchains.

```bash
curl --proto '=https' --tlsv1.2 -sSf https://sh.rustup.rs | sh
# choose option 1 (default), then:
source "$HOME/.cargo/env"

rustc --version
cargo --version
rustup --version
```

rustup adds `~/.cargo/bin` to your PATH through your shell profile. Open a new terminal if the commands are not found.

### 2.3 Add essential components

```bash
rustup component add rustfmt clippy rust-analyzer
rustup update            # run this every few weeks
```

- **rustfmt**: automatic code formatting (`cargo fmt`)
- **clippy**: linter with excellent learning hints (`cargo clippy`)
- **rust-analyzer**: language server for your editor

### 2.4 Editor setup

- **VS Code:** install the `rust-analyzer` extension (and optionally `CodeLLDB` for debugging, `Even Better TOML` for Cargo.toml).
- **Neovim:** use `nvim-lspconfig` with rust-analyzer, or `rustaceanvim`.
- **JetBrains RustRover:** free for non-commercial use, works out of the box.
- **Helix / Zed:** rust-analyzer support is built in.

Enable "format on save" and "clippy on save" early. It builds good habits.

### 2.4 Useful cargo tools (install as you need them)

```bash
cargo install cargo-edit        # cargo add / cargo rm
cargo install cargo-watch       # re-run on file change: cargo watch -x run
cargo install cargo-nextest     # faster test runner
cargo install cargo-expand      # see macro expansions (needs nightly for expand)
cargo install cargo-audit       # security audit of dependencies
```

### 2.5 Verify your setup

```bash
cargo new hello_rust
cd hello_rust
cargo run
```

You should see `Hello, world!`. Environment is ready.

### 2.6 Core cargo commands

| Command | Purpose |
| --- | --- |
| `cargo new name` | Create a binary project (`--lib` for a library) |
| `cargo check` | Fast compile check without producing a binary |
| `cargo build` / `cargo build --release` | Debug / optimized build |
| `cargo run` | Build and run |
| `cargo test` | Run unit, integration, and doc tests |
| `cargo fmt` | Format code |
| `cargo clippy` | Lint code |
| `cargo doc --open` | Build and open documentation |
| `cargo add crate_name` | Add a dependency |
| `cargo tree` | Show dependency tree |

## 3. Resources

**Primary (free):**

1. **The Rust Programming Language** ("the Book"), the main text: `doc.rust-lang.org/book`. Also available offline with `rustup doc --book`.
2. **Rustlings**, small broken exercises you fix: `github.com/rust-lang/rustlings`
3. **Rust by Example**, short runnable snippets: `doc.rust-lang.org/rust-by-example`
4. **Comprehensive Rust** (Google), a course with strong exercises: `google.github.io/comprehensive-rust`
5. **Standard library docs**, work offline with `rustup doc --std`
6. **Exercism Rust track**, practice problems with mentoring: `exercism.org/tracks/rust`

**Secondary:**

- *Command-Line Rust* (Ken Youens-Clark), great Linux-flavored projects
- *Programming Rust* (Blandy, Orendorff, Tindall), deeper reference
- *Rust for Rustaceans* (Jon Gjengset), for the intermediate-to-advanced step
- *Crust of Rust* video series by Jon Gjengset on YouTube
- The Rust Cookbook: `rust-lang-nursery.github.io/rust-cookbook`
- The Rust users forum (`users.rust-lang.org`) and r/rust for questions

**Tip:** `rustup doc` opens the whole documentation set offline, which is handy on slow connections.

## 4. The 12-Week Plan

Each week lists topics, reading, practice, and a mini-goal. Do the exercises before moving on.

### Phase 1: Foundations (Weeks 1 to 4)

#### Week 1: Getting started and basic syntax

- **Topics:** cargo workflow, variables and mutability, shadowing, scalar and compound types, functions, comments, control flow (`if`, `loop`, `while`, `for`), expressions vs statements.
- **Read:** Book chapters 1 to 3.
- **Practice:** Rustlings `intro`, `variables`, `functions`, `if`, `primitive_types`.
- **Mini-goals:** temperature converter, Fibonacci generator, FizzBuzz.
- **Linux touch:** read command-line arguments with `std::env::args()` and print them.

#### Week 2: Ownership, borrowing, and slices

This is the heart of Rust. Take your time.

- **Topics:** stack vs heap, ownership rules, move vs copy, `clone`, references, mutable references, the borrowing rules, slices, `String` vs `&str`.
- **Read:** Book chapter 4 (twice if needed).
- **Practice:** Rustlings `move_semantics`, `strings`.
- **Mini-goals:** write functions that take ownership, borrow, and borrow mutably; deliberately trigger every borrow-checker error and read the messages.
- **Journal:** write the three borrowing rules in your own words.

#### Week 3: Structs, enums, and pattern matching

- **Topics:** structs, methods and associated functions (`impl`), tuple structs, enums with data, `Option<T>`, `Result<T, E>` basics, `match`, `if let`, `while let`, `let else`.
- **Read:** Book chapters 5 and 6, and the first part of chapter 18 (patterns).
- **Practice:** Rustlings `structs`, `enums`, `options`.
- **Mini-goal:** model a simple state machine (for example a traffic light or an order status) with an enum.

#### Week 4: Collections, errors, and your first project

- **Topics:** `Vec`, `String`, `HashMap`, `HashSet`, `BTreeMap`; `panic!` vs `Result`; the `?` operator; reading input.
- **Read:** Book chapters 8 and 9.
- **Practice:** Rustlings `vecs`, `hashmaps`, `error_handling`.
- **Project 1: Guessing game + word-frequency counter.** Read a text file passed as an argument, count word frequencies in a `HashMap`, print the top 10.

### Phase 2: Building real programs (Weeks 5 to 8)

#### Week 5: Modules, packages, and testing

- **Topics:** packages, crates, modules, `pub`, `use`, file layout (`mod.rs` vs `foo.rs`), `lib.rs` and `main.rs` together, unit tests, integration tests (`tests/`), doc tests, `#[cfg(test)]`.
- **Read:** Book chapters 7 and 11.
- **Practice:** Rustlings `modules`, `tests`.
- **Project 2: `minigrep`**, Book chapter 12. Then extend it with case-insensitive search, regex via the `regex` crate, and recursive directory search.
- **Habit:** run `cargo fmt && cargo clippy && cargo test` before every commit.

#### Week 6: Generics and traits

- **Topics:** generic functions and structs, trait definitions, trait implementations, default methods, trait bounds, `impl Trait`, `where` clauses, common derives (`Debug`, `Clone`, `PartialEq`, `Default`), standard traits (`Display`, `From`, `Into`, `FromStr`).
- **Read:** Book chapter 10 (generics and traits parts) and chapter 17 on trait objects.
- **Practice:** Rustlings `generics`, `traits`.
- **Mini-goal:** build a `Shape` trait with several implementors, then use it both statically (generics) and dynamically (`Box<dyn Shape>`). Note the trade-offs.

#### Week 7: Lifetimes and idiomatic error handling

- **Topics:** lifetime annotations, elision rules, structs holding references, `'static`; custom error types, `std::error::Error`, `From` conversions for `?`, `Box<dyn Error>`; the `thiserror` and `anyhow` crates.
- **Read:** Book chapter 10 (lifetimes) and re-read chapter 9.
- **Practice:** Rustlings `lifetimes`.
- **Project 3: Todo CLI.** Store tasks in a JSON file using `serde` and `serde_json`, parse commands with `clap` (derive API), use `anyhow` in `main` and `thiserror` for library errors. Store data under `~/.local/share/yourapp/` using the `dirs` crate to follow Linux conventions.

#### Week 8: Closures and iterators

- **Topics:** closures and capture modes (`Fn`, `FnMut`, `FnOnce`), the `Iterator` trait, adapters (`map`, `filter`, `enumerate`, `zip`, `take`, `skip`, `flat_map`, `fold`), consumers (`collect`, `sum`, `count`), writing your own iterator, iterator vs loop performance.
- **Read:** Book chapter 13.
- **Practice:** Rustlings `iterators`, Exercism problems using iterator chains.
- **Mini-goal:** refactor Project 2 and Project 3 to use iterator chains where it improves clarity.

### Phase 3: Intermediate Rust (Weeks 9 to 12)

#### Week 9: Smart pointers and interior mutability

- **Topics:** `Box<T>`, `Rc<T>`, `Arc<T>`, `RefCell<T>`, `Cell<T>`, `Weak<T>`, the `Deref` and `Drop` traits, reference cycles.
- **Read:** Book chapter 15.
- **Practice:** Rustlings `smart_pointers`.
- **Mini-goal:** implement a singly linked list, then a tree with parent pointers using `Rc<RefCell<_>>` and `Weak`.

#### Week 10: Concurrency

- **Topics:** `std::thread`, `move` closures with threads, message passing (`mpsc`), shared state (`Mutex`, `RwLock`, `Arc`), `Send` and `Sync`, scoped threads (`thread::scope`), atomics basics, data parallelism with `rayon`.
- **Read:** Book chapter 16 and 20 (the web server).
- **Practice:** Rustlings `threads`.
- **Project 4: Multithreaded tool.** Build a parallel file hasher or log analyzer that walks a directory tree and processes files with a thread pool. Compare results with and without `rayon`.

#### Week 11: Async basics, the ecosystem, and Linux system programming

- **Topics:** `async`/`await` concepts, futures, the `tokio` runtime, spawning tasks, async I/O, `reqwest` for HTTP; plus Linux specifics: `std::fs`, `std::process::Command`, environment variables, exit codes, signals (via `signal-hook` or `tokio::signal`), reading `/proc`, and the `nix` crate for POSIX calls.
- **Read:** the Tokio tutorial (`tokio.rs/tokio/tutorial`), Rust async book overview.
- **Mini-goals:** a TCP echo server with tokio; a program that reads `/proc/meminfo` and `/proc/loadavg` and prints a summary; graceful shutdown on `SIGINT`.

#### Week 12: Capstone, polish, and what's next

- **Topics:** documentation comments, `cargo doc`, benchmarking basics (`criterion`), logging (`tracing` or `log` with `env_logger`), release builds, cross-compiling basics, publishing to crates.io, CI with GitHub Actions.
- **Capstone project:** choose one (details in section 5).
- **Review:** re-read your journal, redo the three hardest Rustlings exercises from memory, and write a short retrospective.
- **Next steps:** macros (`macro_rules!`), `unsafe` Rust and FFI, `Rust for Rustaceans`, contributing to an open-source crate.

## 5. Project Ladder

| # | Project | Skills practiced | Difficulty |
| --- | --- | --- | --- |
| 1 | Word-frequency counter | files, `HashMap`, sorting | Easy |
| 2 | `minigrep` | modules, errors, tests, iterators | Easy to medium |
| 3 | Todo CLI with JSON storage | `serde`, `clap`, error types, XDG paths | Medium |
| 4 | Parallel file hasher / log analyzer | threads, channels, `rayon` | Medium |
| 5 | Capstone (pick one below) | everything | Medium to hard |

**Capstone options:**

- **`mytree`**: a `tree`-like directory lister with colors, depth limits, and ignore patterns.
- **`sysmon`**: a terminal system monitor reading `/proc` (CPU, memory, processes), refreshing every second, using `ratatui` for a TUI.
- **Mini HTTP server**: tokio-based static file server with request logging and graceful shutdown.
- **Pomodoro or notes CLI**: with persistent storage, config file (`toml`), and shell completions generated by `clap_complete`.

**Capstone requirements:** unit and integration tests, `clippy` clean, a good README, at least one custom error type, and a tagged release.

## 6. Weekly Routine

A sustainable schedule for roughly 9 hours:

| Day | Activity | Time |
| --- | --- | --- |
| Mon | Read the week's chapter(s), take notes | 1.5 h |
| Tue | Rustlings / exercises on the topic | 1.5 h |
| Wed | Code your own examples without copying | 1.5 h |
| Thu | Project work | 1.5 h |
| Fri | Project work + write tests | 1.5 h |
| Sat | Review, refactor, clippy, journal | 1.5 h |
| Sun | Rest or light reading (docs, blog posts) | optional |

**Checkpoint at the end of each week:** can you explain the week's main idea to a beginner without notes? If not, repeat the exercises before moving on.

## 7. Progress Checklist

**Beginner level (end of week 4)**

- [ ] I can create, build, run, and test projects with cargo
- [ ] I can explain ownership, moves, and borrowing
- [ ] I use `struct`, `enum`, and `match` fluently
- [ ] I handle errors with `Result` and `?`
- [ ] I have finished Project 1

**Lower-intermediate (end of week 8)**

- [ ] I organize code into modules and write tests
- [ ] I write generic functions and implement traits
- [ ] I can read and fix lifetime errors
- [ ] I use iterators and closures naturally
- [ ] I have finished Projects 2 and 3

**Intermediate (end of week 12)**

- [ ] I choose the right smart pointer for a situation
- [ ] I write safe multithreaded code
- [ ] I understand the basics of `async`/`await`
- [ ] I use common crates (`serde`, `clap`, `tokio`, `anyhow`)
- [ ] I have shipped a capstone with tests and docs

## 8. Common Pitfalls and Tips

- **Fighting the borrow checker:** when stuck, simplify. Clone temporarily to get working code, then remove clones later. Cloning is not a sin while learning.
- **Reading error messages:** read the whole message, including the `help:` lines. They often contain the exact fix.
- **Overusing `unwrap()`:** fine for experiments, but replace with `?` or `expect("reason")` in real code.
- **`String` vs `&str` confusion:** use `&str` for function parameters, `String` when you need ownership.
- **Skipping tests:** tests make refactoring safe, and refactoring is how you learn idiomatic Rust.
- **Tutorial hell:** after each chapter, build something small without looking at the text.
- **Debug builds are slow:** use `cargo build --release` before judging performance.
- **Ask for help well:** share a minimal example and the full compiler error on the Rust users forum or Discord.

## 9. Quick Reference

```rust
// Variables and types
let x = 5;                 // immutable
let mut y = 10;            // mutable
let v: Vec<i32> = vec![1, 2, 3];

// Ownership and borrowing
fn len(s: &String) -> usize { s.len() }        // shared borrow
fn push(s: &mut String) { s.push('!'); }       // mutable borrow

// Enums and match
enum Shape { Circle(f64), Rect { w: f64, h: f64 } }
fn area(s: &Shape) -> f64 {
    match s {
        Shape::Circle(r) => std::f64::consts::PI * r * r,
        Shape::Rect { w, h } => w * h,
    }
}

// Error handling
use std::{fs, io};
fn read(path: &str) -> Result<String, io::Error> {
    let text = fs::read_to_string(path)?;
    Ok(text)
}

// Iterators
let total: i32 = (1..=10).filter(|n| n % 2 == 0).map(|n| n * n).sum();
```

## 10. README.md for Your Learning Repository

Create a repository (for example `rust-learning`) and use the README below as its `README.md`. Copy it, then edit the details.

### Suggested repository layout

```text
rust-learning/
├── README.md
├── .gitignore
├── journal/
│   ├── week-01.md
│   └── ...
├── notes/
│   ├── ownership.md
│   └── traits.md
├── exercises/
│   └── rustlings/            # your rustlings solutions
├── projects/
│   ├── 01-word-freq/
│   ├── 02-minigrep/
│   ├── 03-todo-cli/
│   ├── 04-parallel-hasher/
│   └── 05-capstone/
└── scripts/
    └── setup.sh              # optional: automates Linux setup
```

A Cargo workspace is optional. Start with independent projects, and move to a workspace in the intermediate phase if you want shared dependencies.

### README template

````markdown
# Rust Learning Journey (Linux)

My 12-week path from Rust beginner to intermediate, with notes, exercises,
and projects. Developed on Linux (distro: ______).

![status](https://img.shields.io/badge/status-in%20progress-yellow)

## Goals

- Understand ownership, borrowing, and lifetimes
- Write idiomatic, tested, lint-clean Rust
- Build CLI tools that follow Linux conventions
- Ship a capstone project

## Environment

| Item | Version |
|---|---|
| OS | e.g. Ubuntu 24.04 |
| rustc | run `rustc --version` |
| cargo | run `cargo --version` |
| Editor | e.g. VS Code + rust-analyzer |

### Setup

```bash
sudo apt update && sudo apt install -y build-essential curl git pkg-config libssl-dev
curl --proto '=https' --tlsv1.2 -sSf https://sh.rustup.rs | sh
source "$HOME/.cargo/env"
rustup component add rustfmt clippy rust-analyzer
```

## Repository Structure

| Path | Contents |
|---|---|
| `journal/` | Weekly reflections and mistakes |
| `notes/` | Topic notes in my own words |
| `exercises/` | Rustlings and Exercism solutions |
| `projects/` | Numbered projects, one cargo project each |

## Running Things

```bash
cd projects/02-minigrep
cargo run -- needle haystack.txt
cargo test
cargo fmt --check && cargo clippy -- -D warnings
```

## Progress

| Week | Topic | Status |
|---|---|---|
| 1 | Basics and cargo | [ ] |
| 2 | Ownership and borrowing | [ ] |
| 3 | Structs, enums, patterns | [ ] |
| 4 | Collections and errors | [ ] |
| 5 | Modules and testing | [ ] |
| 6 | Generics and traits | [ ] |
| 7 | Lifetimes and error types | [ ] |
| 8 | Closures and iterators | [ ] |
| 9 | Smart pointers | [ ] |
| 10 | Concurrency | [ ] |
| 11 | Async and Linux systems | [ ] |
| 12 | Capstone and polish | [ ] |

## Projects

| # | Project | Status | Notes |
|---|---|---|---|
| 1 | Word-frequency counter | [ ] | |
| 2 | minigrep | [ ] | |
| 3 | Todo CLI | [ ] | |
| 4 | Parallel hasher | [ ] | |
| 5 | Capstone | [ ] | |

## Resources

- The Rust Programming Language (the Book)
- Rustlings, Rust by Example, Exercism
- Comprehensive Rust, Rust Cookbook
- Command-Line Rust, Rust for Rustaceans

## Workflow Rules

1. One commit per exercise or meaningful step.
2. Run `cargo fmt`, `cargo clippy`, and `cargo test` before committing.
3. Write a journal entry every week: what I learned, what confused me.
4. Never copy-paste solutions; retype and understand.

## License

MIT
````

### Recommended `.gitignore`

```text
target/
**/*.rs.bk
.idea/
.vscode/
*.swp
```

Commit `Cargo.lock` for binaries and applications; it is optional for libraries.

### Weekly journal template

```markdown
# Week N: <topic>

## What I learned
-

## What confused me
-

## Compiler errors I hit (and the fix)
-

## Code I'm proud of
-

## Next week's focus
-
```

## 11. After Week 12: Where to Go Next

- **Deepen the language:** macros, `unsafe`, FFI with C libraries, advanced trait features (associated types, GATs), pinning.
- **Systems on Linux:** `epoll` and `io_uring` through `mio` and `tokio-uring`, writing a shell, a simple container runtime using namespaces via `nix`, eBPF with `aya`.
- **Web and services:** `axum` or `actix-web`, `sqlx`, `Zero To Production In Rust`.
- **Contribute:** look for `good first issue` labels on projects such as `ripgrep`, `bat`, `fd`, and `clap`.
- **Keep practicing:** Advent of Code in Rust each December is an excellent way to build fluency.

*Good luck, and enjoy the compiler's strictness. It is the best code reviewer you will ever have.*
