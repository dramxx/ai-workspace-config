---
name: rust-cli
description: Expert Rust CLI application development covering project structure, error handling, argument parsing with clap, config management, file I/O, HTTP clients, cross-platform concerns, and distribution. Always trigger for any Rust CLI task — .rs files, Cargo.toml questions, clap usage, anyhow/thiserror, building terminal tools, single-binary apps, or any mention of writing a Rust command-line tool.
---

# Rust CLI Development

Expert guide to building clean, idiomatic, single-binary Rust CLI tools.

## When to Use

- Starting a new Rust CLI project
- Designing argument/subcommand structure with clap
- Error handling strategy (anyhow vs thiserror)
- Config file parsing (TOML)
- File I/O and path handling
- HTTP requests from CLI tools
- Cross-platform path/OS concerns
- Distribution and release builds

---

## 1. Project Structure

```
my-tool/
├── Cargo.toml
├── Cargo.lock              # commit for binaries, gitignore for libs
├── README.md
├── src/
│   ├── main.rs             # entry point only — parse args, call run()
│   ├── cli.rs              # clap structs (Args, Subcommands)
│   ├── config.rs           # config loading/saving
│   ├── error.rs            # custom error types (if using thiserror)
│   └── commands/
│       ├── mod.rs
│       ├── add.rs
│       └── remove.rs
└── tests/
    └── integration_test.rs
```

**`main.rs` should be minimal:**

```rust
mod cli;
mod config;
mod commands;
mod error;

use anyhow::Result;

fn main() {
    if let Err(e) = run() {
        eprintln!("Error: {e:#}");
        std::process::exit(1);
    }
}

fn run() -> Result<()> {
    let args = cli::parse();
    match args.command {
        cli::Command::Add(cmd) => commands::add::run(cmd),
        cli::Command::Remove(cmd) => commands::remove::run(cmd),
        cli::Command::List => commands::list::run(),
    }
}
```

---

## 2. Cargo.toml

Essential dependencies for a CLI tool:

```toml
[package]
name = "my-tool"
version = "0.1.0"
edition = "2021"

[[bin]]
name = "my-tool"
path = "src/main.rs"

[dependencies]
# Argument parsing
clap = { version = "4", features = ["derive"] }

# Error handling
anyhow = "1"
thiserror = "2"

# Serialization (config, API responses)
serde = { version = "1", features = ["derive"] }
toml = "0.8"
serde_json = "1"

# HTTP (pick one)
ureq = { version = "3", features = ["json"] }      # simple, sync
# reqwest = { version = "0.12", features = ["blocking", "json"] }  # async option

# Paths and dirs
dirs = "5"

# Terminal output
colored = "2"

[profile.release]
opt-level = 3
lto = true
codegen-units = 1
strip = true          # strip debug symbols — smaller binary
```

---

## 3. Argument Parsing with Clap

Always use the `derive` feature — never build Args manually:

```rust
// src/cli.rs
use clap::{Parser, Subcommand, Args};

#[derive(Parser)]
#[command(
    name = "my-tool",
    version,
    about = "Short description of the tool",
    long_about = None,
)]
pub struct Cli {
    /// Enable verbose output
    #[arg(short, long, global = true)]
    pub verbose: bool,

    #[command(subcommand)]
    pub command: Command,
}

#[derive(Subcommand)]
pub enum Command {
    /// Add a new entry
    Add(AddArgs),
    /// Remove an entry by name
    Remove(RemoveArgs),
    /// List all entries
    List,
}

#[derive(Args)]
pub struct AddArgs {
    /// Name of the entry
    pub name: String,

    /// Optional target path
    #[arg(short, long)]
    pub path: Option<std::path::PathBuf>,

    /// Force overwrite if exists
    #[arg(short, long)]
    pub force: bool,
}

#[derive(Args)]
pub struct RemoveArgs {
    /// Name to remove
    pub name: String,
}

pub fn parse() -> Cli {
    Cli::parse()
}
```

**Key clap patterns:**

```rust
// Mutually exclusive flags
#[arg(short, long, conflicts_with = "quiet")]
pub verbose: bool,

// Required unless other flag present
#[arg(required_unless_present = "list")]
pub name: Option<String>,

// Multiple values
#[arg(short, long, num_args = 1..)]
pub tags: Vec<String>,

// Enum argument with automatic parsing
#[arg(value_enum)]
pub format: OutputFormat,

#[derive(clap::ValueEnum, Clone)]
pub enum OutputFormat { Json, Table, Plain }
```

---

## 4. Error Handling

### Use `anyhow` for application code, `thiserror` for library-style errors

```rust
// For most CLI code — anyhow is enough
use anyhow::{Context, Result, bail, anyhow};

fn load_file(path: &Path) -> Result<String> {
    std::fs::read_to_string(path)
        .with_context(|| format!("Failed to read {}", path.display()))
}

fn validate(name: &str) -> Result<()> {
    if name.is_empty() {
        bail!("Name cannot be empty");
    }
    if name.len() > 64 {
        return Err(anyhow!("Name too long: {} chars (max 64)", name.len()));
    }
    Ok(())
}
```

```rust
// src/error.rs — for structured error variants you match on
use thiserror::Error;

#[derive(Error, Debug)]
pub enum AppError {
    #[error("Entry not found: {0}")]
    NotFound(String),

    #[error("Entry already exists: {0}")]
    AlreadyExists(String),

    #[error("Config error: {0}")]
    Config(#[from] toml::de::Error),

    #[error("IO error: {0}")]
    Io(#[from] std::io::Error),
}
```

**Never `unwrap()` in production paths. Never `expect()` unless it's truly invariant.**

---

## 5. Config Management

```rust
// src/config.rs
use anyhow::{Context, Result};
use serde::{Deserialize, Serialize};
use std::path::PathBuf;

#[derive(Debug, Serialize, Deserialize, Default)]
pub struct Config {
    pub entries: Vec<Entry>,
    #[serde(default)]
    pub settings: Settings,
}

#[derive(Debug, Serialize, Deserialize, Default)]
pub struct Settings {
    pub verbose: bool,
    pub default_format: String,
}

#[derive(Debug, Serialize, Deserialize)]
pub struct Entry {
    pub name: String,
    pub value: String,
}

impl Config {
    pub fn path() -> PathBuf {
        dirs::config_dir()
            .expect("No config dir available")
            .join("my-tool")
            .join("config.toml")
    }

    pub fn load() -> Result<Self> {
        let path = Self::path();
        if !path.exists() {
            return Ok(Self::default());
        }
        let content = std::fs::read_to_string(&path)
            .with_context(|| format!("Failed to read config: {}", path.display()))?;
        toml::from_str(&content)
            .with_context(|| "Invalid config format")
    }

    pub fn save(&self) -> Result<()> {
        let path = Self::path();
        if let Some(parent) = path.parent() {
            std::fs::create_dir_all(parent)
                .with_context(|| format!("Failed to create config dir: {}", parent.display()))?;
        }
        let content = toml::to_string_pretty(self)
            .context("Failed to serialize config")?;
        std::fs::write(&path, content)
            .with_context(|| format!("Failed to write config: {}", path.display()))?;
        Ok(())
    }
}
```

---

## 6. HTTP Requests (ureq)

For CLI tools, prefer `ureq` — sync, no tokio runtime, tiny binary:

```rust
use anyhow::{Context, Result};
use serde::{Deserialize, Serialize};

#[derive(Deserialize)]
struct ApiResponse {
    id: String,
    content: String,
}

#[derive(Serialize)]
struct CreateRequest<'a> {
    name: &'a str,
    value: &'a str,
}

fn fetch_entry(id: &str, token: &str) -> Result<ApiResponse> {
    let url = format!("https://api.example.com/entries/{id}");
    let response = ureq::get(&url)
        .header("Authorization", &format!("Bearer {token}"))
        .call()
        .with_context(|| format!("Failed to fetch entry: {id}"))?;

    response
        .body_mut()
        .read_json::<ApiResponse>()
        .context("Failed to parse response")
}

fn create_entry(name: &str, value: &str, token: &str) -> Result<ApiResponse> {
    let body = CreateRequest { name, value };
    let response = ureq::post("https://api.example.com/entries")
        .header("Authorization", &format!("Bearer {token}"))
        .send_json(&body)
        .context("Failed to create entry")?;

    response
        .body_mut()
        .read_json::<ApiResponse>()
        .context("Failed to parse response")
}
```

---

## 7. Output and User Feedback

```rust
use colored::Colorize;

// Consistent output helpers
pub fn success(msg: &str) {
    println!("{} {msg}", "✓".green().bold());
}

pub fn info(msg: &str) {
    println!("{} {msg}", "→".cyan());
}

pub fn warn(msg: &str) {
    eprintln!("{} {msg}", "⚠".yellow().bold());
}

pub fn error(msg: &str) {
    eprintln!("{} {msg}", "✗".red().bold());
}

// Always write errors to stderr, data to stdout
// This allows: my-tool list | grep foo
println!("{}", data);        // stdout — pipeable
eprintln!("{}", status_msg); // stderr — user-facing only
```

---

## 8. Cross-Platform Path Handling

```rust
use std::path::{Path, PathBuf};

// Never hardcode separators
// Bad
let path = format!("{}\\config\\tool.toml", home);

// Good
let path = dirs::config_dir()
    .unwrap()
    .join("tool")
    .join("config.toml");

// Symlinks
std::os::unix::fs::symlink(src, dst)?;  // Unix only
// For cross-platform:
#[cfg(unix)]
std::os::unix::fs::symlink(&src, &dst)?;
#[cfg(windows)]
std::os::windows::fs::symlink_file(&src, &dst)?;

// Canonicalize (resolves symlinks, makes absolute)
let real_path = path.canonicalize()
    .with_context(|| format!("Path does not exist: {}", path.display()))?;
```

---

## 9. Common Patterns

### Reading stdin or file (flexible input)

```rust
use std::io::{self, Read};

fn read_input(file: Option<&Path>) -> Result<String> {
    match file {
        Some(path) => std::fs::read_to_string(path)
            .with_context(|| format!("Cannot read {}", path.display())),
        None => {
            let mut buf = String::new();
            io::stdin().read_to_string(&mut buf)
                .context("Failed to read stdin")?;
            Ok(buf)
        }
    }
}
```

### Confirmation prompt

```rust
fn confirm(prompt: &str) -> Result<bool> {
    print!("{prompt} [y/N] ");
    std::io::Write::flush(&mut std::io::stdout())?;
    let mut input = String::new();
    std::io::stdin().read_line(&mut input)?;
    Ok(matches!(input.trim().to_lowercase().as_str(), "y" | "yes"))
}
```

### Environment variable fallback

```rust
fn get_token() -> Result<String> {
    std::env::var("MY_TOOL_TOKEN")
        .context("MY_TOOL_TOKEN not set. Run `my-tool auth` first.")
}
```

---

## 10. Testing

```rust
// Unit test config loading
#[cfg(test)]
mod tests {
    use super::*;
    use tempfile::TempDir;

    #[test]
    fn test_config_roundtrip() {
        let dir = TempDir::new().unwrap();
        let config = Config {
            entries: vec![Entry {
                name: "test".into(),
                value: "value".into(),
            }],
            ..Default::default()
        };
        // test serialize/deserialize
        let toml = toml::to_string_pretty(&config).unwrap();
        let parsed: Config = toml::from_str(&toml).unwrap();
        assert_eq!(parsed.entries[0].name, "test");
    }
}
```

Add to `Cargo.toml` for tests:

```toml
[dev-dependencies]
tempfile = "3"
assert_cmd = "2"    # for integration tests
predicates = "3"    # for assert_cmd matchers
```

---

## 11. Release Builds

```bash
# Optimized release
cargo build --release

# Cross-compile (install cross first: cargo install cross)
cross build --release --target x86_64-unknown-linux-musl   # static Linux binary
cross build --release --target x86_64-pc-windows-gnu
cross build --release --target aarch64-apple-darwin
```

**Binary ends up at:** `target/release/my-tool` (or `.exe` on Windows)

---

## Common Mistakes to Avoid

| Mistake                         | Problem                       | Solution                       |
| ------------------------------- | ----------------------------- | ------------------------------ |
| `unwrap()` on user-facing paths | Panics with no context        | Use `?` + `.context()`         |
| Hardcoded paths with `\` or `/` | Breaks on other OS            | `Path::join()` always          |
| Writing status to stdout        | Breaks piping                 | Status → stderr, data → stdout |
| `reqwest` in simple CLIs        | Pulls in tokio, bloats binary | Use `ureq` for sync HTTP       |
| Deep nesting in match arms      | Hard to read                  | Extract to functions           |
| No `--force` on destructive ops | Surprises users               | Add confirmation prompt        |
| `panic!` for user errors        | Bad UX                        | `bail!` or `return Err(...)`   |

---

## Quick Reference

```rust
// Entry point
fn main() { if let Err(e) = run() { eprintln!("{e:#}"); exit(1); } }

// Clap derive
#[derive(Parser)] struct Cli { #[command(subcommand)] cmd: Cmd }

// Error with context
thing().with_context(|| format!("doing X with {path:?}"))?;

// Early exit with message
bail!("Something went wrong: {reason}");

// Config dir (cross-platform)
dirs::config_dir().unwrap().join("my-tool").join("config.toml")

// Colored output
println!("{} done", "✓".green().bold());

// Env var
std::env::var("TOKEN").context("TOKEN not set")?;
```

---

## References

- [Rust CLI Book](https://rust-cli.github.io/book/)
- [clap docs](https://docs.rs/clap/latest/clap/)
- [anyhow docs](https://docs.rs/anyhow/)
- [thiserror docs](https://docs.rs/thiserror/)
- [ureq docs](https://docs.rs/ureq/)
- [dirs docs](https://docs.rs/dirs/)
