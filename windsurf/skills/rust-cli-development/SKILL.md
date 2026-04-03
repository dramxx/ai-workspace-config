---
name: rust-cli-development
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

## Project Structure

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

use clap::Parser;
use anyhow::Result;

fn main() -> Result<()> {
    let cli = cli::Cli::parse();
    commands::run(cli)
}
```

## Dependencies

### Core Dependencies
```toml
[dependencies]
clap = { version = "4.0", features = ["derive"] }
anyhow = "1.0"          # For simple error handling
# OR
thiserror = "1.0"       # For custom error types
serde = { version = "1.0", features = ["derive"] }
toml = "0.8"            # Config file parsing
```

### Optional Dependencies
```toml
tokio = { version = "1.0", features = ["full"] }  # Async
reqwest = { version = "0.11", features = ["json"] }  # HTTP
dirs = "5.0"             # Cross-platform config dirs
console = "0.15"        # Terminal styling
dialoguer = "0.10"      # Interactive prompts
indicatif = "0.17"      # Progress bars
```

## CLI Design with Clap

### Basic Structure
```rust
// src/cli.rs
use clap::{Parser, Subcommand};

#[derive(Parser)]
#[command(name = "my-tool")]
#[command(about = "A brief description")]
#[command(version = "1.0")]
pub struct Cli {
    #[command(subcommand)]
    pub command: Commands,

    /// Verbose output
    #[arg(short, long)]
    pub verbose: bool,

    /// Config file path
    #[arg(short, long, default_value = "config.toml")]
    pub config: PathBuf,
}

#[derive(Subcommand)]
pub enum Commands {
    /// Add a new item
    Add {
        /// Item name
        name: String,
        /// Item description
        #[arg(short, long)]
        description: Option<String>,
    },
    /// Remove an item
    Remove {
        /// Item ID
        id: u32,
    },
    /// List all items
    List {
        /// Filter by pattern
        #[arg(short, long)]
        filter: Option<String>,
    },
}
```

### Advanced Argument Types
```rust
use std::path::PathBuf;

#[derive(Parser)]
pub struct Cli {
    /// Input file (supports multiple)
    #[arg(short, long, num_args = 1..)]
    pub input: Vec<PathBuf>,

    /// Output directory
    #[arg(short, long)]
    pub output: PathBuf,

    /// Log level
    #[arg(short, long, default_value = "info", value_parser = clap::value_parser!(LogLevel))]
    pub log_level: LogLevel,

    /// Custom format
    #[arg(short = 'F', long, value_parser = parse_format)]
    pub format: Format,
}

#[derive(Clone, Debug)]
pub enum LogLevel {
    Debug,
    Info,
    Warn,
    Error,
}

fn parse_format(s: &str) -> Result<Format, String> {
    match s {
        "json" => Ok(Format::Json),
        "yaml" => Ok(Format::Yaml),
        _ => Err("format must be json or yaml".to_string()),
    }
}
```

## Error Handling

### Using Anyhow (Simple)
```rust
use anyhow::{Result, Context, anyhow};

fn read_file(path: &Path) -> Result<String> {
    std::fs::read_to_string(path)
        .with_context(|| format!("Failed to read file: {}", path.display()))?;
    
    // Process file...
    Ok(content)
}

fn process_data() -> Result<()> {
    let data = fetch_data().context("Failed to fetch data")?;
    validate_data(&data).context("Data validation failed")?;
    Ok(())
}
```

### Using ThisError (Custom Types)
```rust
// src/error.rs
use thiserror::Error;

#[derive(Error, Debug)]
pub enum AppError {
    #[error("IO error: {0}")]
    Io(#[from] std::io::Error),

    #[error("Config error: {0}")]
    Config(String),

    #[error("Network error: {0}")]
    Network(#[from] reqwest::Error),

    #[error("Parse error at line {line}: {message}")]
    Parse { line: usize, message: String },

    #[error("Validation failed: {field} cannot be {value}")]
    Validation { field: String, value: String },
}

// Usage
impl From<toml::de::Error> for AppError {
    fn from(err: toml::de::Error) -> Self {
        AppError::Config(format!("TOML parsing failed: {}", err))
    }
}
```

## Configuration Management

### Config Structure
```rust
// src/config.rs
use serde::{Deserialize, Serialize};
use std::path::PathBuf;

#[derive(Debug, Deserialize, Serialize)]
pub struct Config {
    pub database: DatabaseConfig,
    pub api: ApiConfig,
    pub logging: LoggingConfig,
}

#[derive(Debug, Deserialize, Serialize)]
pub struct DatabaseConfig {
    pub url: String,
    pub max_connections: u32,
}

#[derive(Debug, Deserialize, Serialize)]
pub struct ApiConfig {
    pub base_url: String,
    pub timeout: u64,
    pub api_key: Option<String>,
}

#[derive(Debug, Deserialize, Serialize)]
pub struct LoggingConfig {
    pub level: String,
    pub file: Option<PathBuf>,
}

impl Default for Config {
    fn default() -> Self {
        Self {
            database: DatabaseConfig {
                url: "sqlite://app.db".to_string(),
                max_connections: 10,
            },
            api: ApiConfig {
                base_url: "https://api.example.com".to_string(),
                timeout: 30,
                api_key: None,
            },
            logging: LoggingConfig {
                level: "info".to_string(),
                file: None,
            },
        }
    }
}
```

### Config Loading
```rust
use dirs::config_dir;
use anyhow::Result;

impl Config {
    pub fn load(config_path: &Path) -> Result<Self> {
        if config_path.exists() {
            let content = std::fs::read_to_string(config_path)?;
            let config: Config = toml::from_str(&content)?;
            Ok(config)
        } else {
            // Create default config
            let default_config = Config::default();
            default_config.save(config_path)?;
            Ok(default_config)
        }
    }

    pub fn save(&self, config_path: &Path) -> Result<()> {
        let content = toml::to_string_pretty(self)?;
        std::fs::write(config_path, content)?;
        Ok(())
    }

    pub fn get_default_config_path() -> PathBuf {
        config_dir()
            .unwrap_or_else(|| PathBuf::from("."))
            .join("my-tool")
            .join("config.toml")
    }
}
```

## File I/O and Paths

### Cross-Platform Path Handling
```rust
use std::path::{Path, PathBuf};
use anyhow::Result;

fn expand_home(path: &Path) -> PathBuf {
    if let Some(home) = dirs::home_dir() {
        if path.starts_with("~") {
            return home.join(path.strip_prefix("~").unwrap());
        }
    }
    path.to_path_buf()
}

fn ensure_dir_exists(path: &Path) -> Result<()> {
    if !path.exists() {
        std::fs::create_dir_all(path)?;
    }
    Ok(())
}

fn safe_file_write(path: &Path, content: &str) -> Result<()> {
    // Create parent directories if needed
    if let Some(parent) = path.parent() {
        ensure_dir_exists(parent)?;
    }
    
    // Write to temporary file first, then rename
    let temp_path = path.with_extension("tmp");
    std::fs::write(&temp_path, content)?;
    std::fs::rename(&temp_path, path)?;
    
    Ok(())
}
```

### File Operations with Error Context
```rust
use std::fs;
use anyhow::Result;

fn process_files(input_dir: &Path) -> Result<Vec<String>> {
    let mut results = Vec::new();
    
    for entry in fs::read_dir(input_dir)
        .with_context(|| format!("Failed to read directory: {}", input_dir.display()))? 
    {
        let entry = entry
            .with_context(|| "Failed to read directory entry")?;
        
        let path = entry.path();
        if path.extension().and_then(|s| s.to_str()) == Some("txt") {
            let content = fs::read_to_string(&path)
                .with_context(|| format!("Failed to read file: {}", path.display()))?;
            results.push(content);
        }
    }
    
    Ok(results)
}
```

## HTTP Client

### Basic HTTP Operations
```rust
use reqwest;
use serde::{Deserialize, Serialize};
use anyhow::Result;

#[derive(Debug, Deserialize)]
pub struct User {
    pub id: u32,
    pub name: String,
    pub email: String,
}

#[derive(Debug, Serialize)]
pub struct CreateUserRequest {
    pub name: String,
    pub email: String,
}

pub struct ApiClient {
    client: reqwest::Client,
    base_url: String,
}

impl ApiClient {
    pub fn new(base_url: String) -> Self {
        Self {
            client: reqwest::Client::new(),
            base_url,
        }
    }

    pub async fn get_user(&self, id: u32) -> Result<User> {
        let url = format!("{}/users/{}", self.base_url, id);
        let user = self.client
            .get(&url)
            .send()
            .await?
            .json::<User>()
            .await?;
        Ok(user)
    }

    pub async fn create_user(&self, request: CreateUserRequest) -> Result<User> {
        let url = format!("{}/users", self.base_url);
        let user = self.client
            .post(&url)
            .json(&request)
            .send()
            .await?
            .json::<User>()
            .await?;
        Ok(user)
    }
}
```

## Terminal UI

### Progress Bars
```rust
use indicatif::{ProgressBar, ProgressStyle};
use std::time::Duration;

fn process_with_progress(items: Vec<String>) -> Result<()> {
    let pb = ProgressBar::new(items.len() as u64);
    pb.set_style(
        ProgressStyle::default_bar()
            .template("{spinner:.green} [{elapsed_precise}] [{bar:40.cyan/blue}] {pos}/{len} ({eta})")
            .unwrap()
            .progress_chars("#>-")
    );

    for item in items {
        // Process item
        std::thread::sleep(Duration::from_millis(100));
        pb.inc(1);
    }

    pb.finish_with_message("Done!");
    Ok(())
}
```

### Interactive Prompts
```rust
use dialoguer::{Confirm, Input, Select};

fn interactive_mode() -> Result<()> {
    // Confirmation
    let should_continue = Confirm::new()
        .with_prompt("Do you want to continue?")
        .default(true)
        .interact()?;

    if !should_continue {
        return Ok(());
    }

    // Text input
    let name: String = Input::new()
        .with_prompt("Enter your name")
        .default("Anonymous".to_string())
        .interact()?;

    // Selection
    let items = vec!["Option 1", "Option 2", "Option 3"];
    let selection = Select::new()
        .with_prompt("Choose an option")
        .items(&items)
        .interact()?;

    println!("Selected: {}", items[selection]);
    Ok(())
}
```

## Testing

### Unit Tests
```rust
#[cfg(test)]
mod tests {
    use super::*;

    #[test]
    fn test_config_default() {
        let config = Config::default();
        assert_eq!(config.database.url, "sqlite://app.db");
        assert_eq!(config.database.max_connections, 10);
    }

    #[test]
    fn test_config_parsing() {
        let toml_content = r#"
[database]
url = "postgresql://localhost/mydb"
max_connections = 20

[api]
base_url = "https://api.test.com"
timeout = 60
"#;
        
        let config: Config = toml::from_str(toml_content).unwrap();
        assert_eq!(config.database.url, "postgresql://localhost/mydb");
        assert_eq!(config.database.max_connections, 20);
    }
}
```

### Integration Tests
```rust
// tests/integration_test.rs
use assert_cmd::Command;
use tempfile::NamedTempFile;
use std::fs;

#[test]
fn test_cli_add_command() -> Result<()> {
    let mut cmd = Command::cargo_bin("my-tool")?;
    let temp_file = NamedTempFile::new()?;
    
    cmd.arg("--config")
       .arg(temp_file.path())
       .arg("add")
       .arg("test-item")
       .assert()
       .success();

    let content = fs::read_to_string(temp_file.path())?;
    assert!(content.contains("test-item"));
    
    Ok(())
}
```

## Building and Distribution

### Cargo.toml Configuration
```toml
[package]
name = "my-tool"
version = "1.0.0"
edition = "2021"
authors = ["Your Name <your.email@example.com>"]
description = "A brief description"
license = "MIT OR Apache-2.0"
repository = "https://github.com/yourusername/my-tool"

[profile.release]
strip = true
lto = true
codegen-units = 1
panic = "abort"
```

### GitHub Actions for Releases
```yaml
# .github/workflows/release.yml
name: Release

on:
  push:
    tags:
      - 'v*'

jobs:
  release:
    runs-on: ${{ matrix.os }}
    strategy:
      matrix:
        os: [ubuntu-latest, windows-latest, macos-latest]
    
    steps:
    - uses: actions/checkout@v3
    
    - name: Build
      run: cargo build --release
    
    - name: Upload binary
      uses: actions/upload-artifact@v3
      with:
        name: binary-${{ matrix.os }}
        path: target/release/my-tool${{ matrix.os == 'windows-latest' && '.exe' || '' }}
```

## Best Practices Summary

1. **Minimal main.rs** - parse args, delegate to commands
2. **Strong typing** - use clap derive macros
3. **Error context** - use anyhow/thiserror with context()
4. **Config separation** - load/save config separately
5. **Cross-platform paths** - use dirs crate, handle ~ expansion
6. **Async when needed** - tokio for HTTP/network operations
7. **Testing** - unit tests for logic, integration tests for CLI
8. **Release optimization** - strip symbols, LTO, single binary
9. **User experience** - progress bars, interactive prompts, clear errors
10. **Documentation** - help text, examples, README with installation

This approach will create professional, maintainable Rust CLI tools that follow idiomatic patterns and provide excellent user experience.
