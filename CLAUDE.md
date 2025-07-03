# Technical Specification: 'n' CLI Tool

## **Language: Rust**
**Rationale**: Single binary distribution, fast execution, excellent CLI ecosystem, cross-platform compatibility.

## **Project Structure**
```
n/
├── src/
│   ├── main.rs              # CLI entry point
│   ├── commands/
│   │   ├── mod.rs
│   │   ├── create.rs        # File creation (n)
│   │   ├── workspace.rs     # Folder creation (nw)
│   │   └── prompt.rs        # AI-assisted creation (nwc/new)
│   ├── storage/
│   │   ├── mod.rs
│   │   ├── manager.rs       # Path management
│   │   └── templates.rs     # Template handling
│   ├── config/
│   │   ├── mod.rs
│   │   └── settings.rs      # Configuration management
│   └── utils/
│       ├── mod.rs
│       └── helpers.rs       # Common utilities
├── templates/               # Built-in templates
├── tests/
└── Cargo.toml
```

## **Dependencies**
```toml
[dependencies]
clap = { version = "4.0", features = ["derive"] }
tokio = { version = "1.0", features = ["full"] }
serde = { version = "1.0", features = ["derive"] }
serde_json = "1.0"
dirs = "5.0"
anyhow = "1.0"
reqwest = { version = "0.11", features = ["json"] }
```

## **CLI Interface**
```rust
#[derive(Parser)]
#[command(name = "n", version, about = "Fast file and workspace creation")]
struct Cli {
    #[command(subcommand)]
    command: Option<Commands>,
    
    /// Create file in current directory
    #[arg(help = "File name to create")]
    filename: Option<String>,
}

#[derive(Subcommand)]
enum Commands {
    /// Create workspace folder
    #[command(name = "nw")]
    Workspace {
        name: String,
        #[arg(short, long)]
        template: Option<String>,
    },
    /// Create workspace with AI prompt
    #[command(name = "nwc")]
    WorkspacePrompt {
        prompt: String,
    },
    /// Create folder with prompt
    #[command(name = "new")]
    NewPrompt {
        prompt: String,
    },
}
```

## **Core Components**

### **1. Storage Manager**
```rust
pub struct StorageManager {
    pub data_dir: PathBuf,      // ~/.local/share/n/
    pub config_dir: PathBuf,    // ~/.config/n/
    pub templates_dir: PathBuf, // ~/.local/share/n/templates/
}

impl StorageManager {
    pub fn new() -> Result<Self>;
    pub fn ensure_dirs(&self) -> Result<()>;
    pub fn get_template_path(&self, name: &str) -> PathBuf;
    pub fn list_templates(&self) -> Result<Vec<String>>;
}
```

### **2. File Creation**
```rust
pub struct FileCreator {
    storage: StorageManager,
}

impl FileCreator {
    pub async fn create_file(&self, name: &str, template: Option<&str>) -> Result<()>;
    pub async fn create_workspace(&self, name: &str, template: Option<&str>) -> Result<()>;
    pub fn resolve_path(&self, name: &str) -> PathBuf;
}
```

### **3. Template System**
```rust
pub struct Template {
    pub name: String,
    pub files: Vec<TemplateFile>,
    pub variables: HashMap<String, String>,
}

pub struct TemplateFile {
    pub path: String,
    pub content: String,
    pub executable: bool,
}

impl Template {
    pub fn load(path: &Path) -> Result<Self>;
    pub fn render(&self, variables: &HashMap<String, String>) -> Result<Vec<TemplateFile>>;
}
```

### **4. AI Integration**
```rust
pub struct PromptHandler {
    client: reqwest::Client,
    api_key: Option<String>,
}

impl PromptHandler {
    pub async fn generate_structure(&self, prompt: &str) -> Result<ProjectStructure>;
    pub async fn suggest_filename(&self, context: &str) -> Result<String>;
}

pub struct ProjectStructure {
    pub name: String,
    pub files: Vec<FileSpec>,
    pub folders: Vec<String>,
}
```

## **Configuration**
```rust
#[derive(Serialize, Deserialize)]
pub struct Config {
    pub default_template: Option<String>,
    pub ai_provider: Option<String>,
    pub api_key: Option<String>,
    pub storage_location: Option<PathBuf>,
}
```

## **Command Flow**

### **File Creation (`n filename`)**
1. Parse filename
2. Check if template specified
3. Resolve full path
4. Create file with template content
5. Set permissions if needed

### **Workspace Creation (`nw name`)**
1. Create directory structure
2. Apply template if specified
3. Initialize git repo (optional)
4. Create default files

### **AI-Assisted Creation (`nwc/new prompt`)**
1. Send prompt to AI service
2. Parse response for structure
3. Create files and folders
4. Apply generated content

## **Error Handling**
```rust
#[derive(Debug, thiserror::Error)]
pub enum NError {
    #[error("File already exists: {0}")]
    FileExists(String),
    #[error("Template not found: {0}")]
    TemplateNotFound(String),
    #[error("IO error: {0}")]
    Io(#[from] std::io::Error),
    #[error("Config error: {0}")]
    Config(String),
}
```

## **Templates Format**
```json
{
  "name": "rust-project",
  "description": "Basic Rust project structure",
  "files": [
    {
      "path": "Cargo.toml",
      "content": "[package]\nname = \"{{project_name}}\"\nversion = \"0.1.0\"\n",
      "executable": false
    },
    {
      "path": "src/main.rs",
      "content": "fn main() {\n    println!(\"Hello, world!\");\n}\n",
      "executable": false
    }
  ],
  "variables": {
    "project_name": "default_name"
  }
}
```

## **Build & Distribution**
```bash
# Development
cargo build --release

# Cross-compilation
cargo install cross
cross build --target x86_64-unknown-linux-gnu --release

# Installation
cargo install --path .
```

# important-instruction-reminders
Do what has been asked; nothing more, nothing less.
NEVER create files unless they're absolutely necessary for achieving your goal.
ALWAYS prefer editing an existing file to creating a new one.
NEVER proactively create documentation files (*.md) or README files. Only create documentation files if explicitly requested by the User.
Respond briefly to any questions.