# RuleZ UI - Development Plan

**Version:** 1.1  
**Date:** 2026-01-23  
**Status:** Draft  
**Author:** Claude Code Assistant

---

## Table of Contents

1. [Executive Summary](#executive-summary)
2. [Project Goals](#project-goals)
3. [Technology Stack](#technology-stack)
4. [Architecture](#architecture)
5. [Directory Structure](#directory-structure)
6. [Wireframes](#wireframes)
7. [Component Specifications](#component-specifications)
8. [Tauri IPC Commands](#tauri-ipc-commands)
9. [JSON Schema](#json-schema)
10. [Testing Strategy](#testing-strategy)
11. [Phase 1 Milestones](#phase-1-milestones)
12. [Future Phases](#future-phases)
13. [Development Workflow](#development-workflow)
14. [Risk Assessment](#risk-assessment)

---

## Executive Summary

**Project:** RuleZ UI (`rulez_ui`)  
**Purpose:** Desktop/Web application for visualizing, editing, validating, and debugging Claude Context Hooks (CCH) configurations  
**Stack:** Tauri 2.0 + React 18 + TypeScript + Monaco Editor + **Bun**  
**Dual-Mode:** Web mode for Playwright testing; **Native desktop app is the primary target**

### Key Decisions

| Decision | Choice | Rationale |
|----------|--------|-----------|
| File Scope | Multi-file | Support both global (~/.claude/hooks.yaml) and project-level configs |
| Log Streaming | Real-time | File watching with live UI updates |
| Debug Backend | CCH Binary | Call actual `cch debug` command for guaranteed behavior parity |
| Validation | Monaco + Schema | Full editor with JSON Schema validation, autocomplete |
| Theme | Dual (Dark/Light) | System preference detection with manual override |
| **Package Manager** | **Bun** | All TypeScript/React builds, testing, and tooling use Bun |
| **Primary Target** | **Native Desktop** | Web browser mode is solely for automated Playwright testing |

---

## Project Goals

### Phase 1 Goals (MVP)

1. **Visualize** - Display hooks.yaml configuration in a structured, readable format
2. **Validate** - Real-time schema validation with inline error indicators
3. **Edit** - Full-featured YAML editor with syntax highlighting and autocomplete
4. **Debug** - Simulate events and see which rules match with full evaluation trace
5. **Multi-file** - Support both global and project-level configuration files

### Success Criteria

- [ ] User can open and edit hooks.yaml files with syntax highlighting
- [ ] Validation errors appear inline with red squiggly underlines
- [ ] User can simulate PreToolUse/PostToolUse events and see matching rules
- [ ] Application runs in both web browser (for testing) and as native desktop app
- [ ] **Native desktop app is the primary deliverable**
- [ ] **Web browser mode exists solely for Playwright automated testing**
- [ ] **All builds, deploys, and packaging for TypeScript/React use Bun**
- [ ] Playwright E2E tests pass in CI

---

## Technology Stack

### Runtime & Package Manager

| Tool | Purpose | Version |
|------|---------|---------|
| **Bun** | JavaScript runtime, package manager, bundler, test runner | ^1.1 |

> **Note:** Bun replaces Node.js, npm/pnpm/yarn, and Vite for all TypeScript/React operations. Bun provides faster installs, builds, and native TypeScript execution.

### Frontend (React)

| Package | Purpose | Version |
|---------|---------|---------|
| `react` | UI framework | ^18.3 |
| `react-dom` | DOM rendering | ^18.3 |
| `typescript` | Type safety | ^5.4 |
| `@monaco-editor/react` | Code editor component | ^4.6 |
| `monaco-yaml` | YAML language support + schema validation | ^5.2 |
| `zustand` | Lightweight state management | ^5.0 |
| `@tanstack/react-query` | Async state management for file ops | ^5.0 |
| `tailwindcss` | Utility-first CSS | ^4.0 |
| `@radix-ui/react-*` | Headless UI primitives | latest |
| `lucide-react` | Icons | ^0.300 |
| `js-yaml` | YAML parsing (fallback for web mode) | ^4.1 |
| `react-virtuoso` | Virtual scrolling for logs (Phase 2) | ^4.7 |
| `clsx` | Conditional classnames | ^2.1 |

### Backend (Tauri/Rust)

| Crate | Purpose |
|-------|---------|
| `tauri` | Desktop application framework v2.0 |
| `tauri-plugin-fs` | File system read/write/watch |
| `tauri-plugin-shell` | Execute CCH binary commands |
| `serde` | Serialization |
| `serde_yaml` | YAML parsing |
| `serde_json` | JSON handling |
| `notify` | Cross-platform file watching |
| `tokio` | Async runtime |
| `thiserror` | Error handling |

### Build Tools (All Bun-based)

| Tool | Purpose | Command |
|------|---------|---------|
| **Bun** | Package installation | `bun install` |
| **Bun** | Development server with HMR | `bun run dev` |
| **Bun** | Production build | `bun run build` |
| **Bun** | Unit testing | `bun test` |
| **Bun** | Linting (via biome) | `bun run lint` |
| **Bun** | Formatting (via biome) | `bun run format` |
| **Bun** | Type checking | `bun run typecheck` |
| `@tauri-apps/cli` | Tauri CLI for desktop builds | `bunx tauri dev/build` |
| `playwright` | E2E testing framework | `bun run test:e2e` |
| `@biomejs/biome` | Fast linter + formatter (replaces ESLint + Prettier) | via Bun |

---

## Architecture

### Dual-Mode Architecture

```
+------------------------------------------------------------------+
|                        RuleZ UI Architecture                      |
+------------------------------------------------------------------+
|                                                                    |
|   +------------------------+    +-----------------------------+   |
|   |    Web Mode (Test)     |    |    Desktop Mode (Primary)   |   |
|   +------------------------+    +-----------------------------+   |
|   |                        |    |                             |   |
|   |  Browser (localhost)   |    |  Tauri WebView Window       |   |
|   |         |              |    |         |                   |   |
|   |         v              |    |         v                   |   |
|   |  React App (Bun HMR)   |    |  React App (bundled)        |   |
|   |         |              |    |         |                   |   |
|   |         v              |    |         v                   |   |
|   |  Web Fallbacks:        |    |  Tauri IPC:                 |   |
|   |  - localStorage        |    |  - invoke('read_config')    |   |
|   |  - Mock data           |    |  - invoke('run_debug')      |   |
|   |  - Sample configs      |    |  - File watchers            |   |
|   |                        |    |         |                   |   |
|   +------------------------+    |         v                   |   |
|                                 |  Rust Backend:              |   |
|   Used ONLY for:                |  - File I/O                 |   |
|   - Playwright E2E testing      |  - CCH binary execution     |   |
|   - Automated CI testing        |  - Real-time file watching  |   |
|                                 |                             |   |
|                                 +-----------------------------+   |
|                                          ▲                        |
|                                          |                        |
|                                 PRIMARY DELIVERABLE               |
+------------------------------------------------------------------+
```

### Data Flow

```
+------------------------------------------------------------------+
|                         Data Flow                                 |
+------------------------------------------------------------------+
|                                                                    |
|  User Action          React Component        Tauri/Backend         |
|  ───────────          ───────────────        ────────────          |
|                                                                    |
|  Edit YAML    ──────► YamlEditor     ──────► (real-time)           |
|                       │                      │                     |
|                       │ onChange             │                     |
|                       v                      v                     |
|               Monaco validates ◄──── JSON Schema                   |
|                       │                                            |
|                       │ onSave                                     |
|                       v                                            |
|               configStore.save() ──────► write_config()            |
|                                          │                         |
|                                          v                         |
|                                   ~/.claude/hooks.yaml             |
|                                                                    |
|  ─────────────────────────────────────────────────────────────     |
|                                                                    |
|  Run Debug    ──────► DebugSimulator ──────► run_debug()           |
|                       │                      │                     |
|                       │                      v                     |
|                       │              cch debug PreToolUse          |
|                       │              --tool Bash                   |
|                       │              --command "git push"          |
|                       │                      │                     |
|                       v                      v                     |
|               ResultView     ◄────── JSON output parsed            |
|                                                                    |
+------------------------------------------------------------------+
```

---

## Directory Structure

```
rulez_ui/
├── src/                              # React frontend source
│   ├── components/
│   │   ├── editor/
│   │   │   ├── YamlEditor.tsx          # Monaco editor wrapper
│   │   │   ├── RuleTreeView.tsx        # Visual tree of rules
│   │   │   ├── ValidationPanel.tsx     # Error/warning display
│   │   │   └── EditorToolbar.tsx       # Save, format, undo buttons
│   │   ├── simulator/
│   │   │   ├── DebugSimulator.tsx      # Main simulator container
│   │   │   ├── EventForm.tsx           # Event type/tool/command inputs
│   │   │   ├── ResultView.tsx          # Match results display
│   │   │   └── EvaluationTrace.tsx     # Step-by-step rule evaluation
│   │   ├── files/
│   │   │   ├── FileSidebar.tsx         # File tree navigation
│   │   │   ├── FileTab.tsx             # Tab for open file
│   │   │   └── FileTabBar.tsx          # Tab bar container
│   │   ├── layout/
│   │   │   ├── AppShell.tsx            # Main application shell
│   │   │   ├── Header.tsx              # App header with theme toggle
│   │   │   ├── Sidebar.tsx             # Left sidebar container
│   │   │   └── StatusBar.tsx           # Bottom status bar
│   │   └── ui/                         # Reusable UI primitives
│   │       ├── Button.tsx
│   │       ├── Input.tsx
│   │       ├── Select.tsx
│   │       ├── Tabs.tsx
│   │       ├── Badge.tsx
│   │       ├── Tooltip.tsx
│   │       └── ThemeToggle.tsx
│   ├── hooks/
│   │   ├── useTauri.ts                 # Tauri detection + invoke wrapper
│   │   ├── useConfig.ts                # Config file loading/saving
│   │   ├── useValidation.ts            # Schema validation hook
│   │   ├── useDebugSimulator.ts        # Debug simulation logic
│   │   ├── useTheme.ts                 # Theme management
│   │   └── useFileWatcher.ts           # Real-time file watching
│   ├── lib/
│   │   ├── tauri.ts                    # isTauri() + API layer
│   │   ├── schema.ts                   # JSON Schema loader
│   │   ├── yaml-utils.ts               # YAML parse/stringify helpers
│   │   ├── mock-data.ts                # Sample configs for web mode
│   │   └── constants.ts                # App constants
│   ├── stores/
│   │   ├── configStore.ts              # Zustand: config file state
│   │   ├── editorStore.ts              # Zustand: editor state (tabs, cursor)
│   │   └── uiStore.ts                  # Zustand: UI state (panels, theme)
│   ├── types/
│   │   ├── hooks.ts                    # TypeScript types for CCH models
│   │   ├── tauri.ts                    # Tauri command types
│   │   └── editor.ts                   # Editor-related types
│   ├── styles/
│   │   ├── globals.css                 # Global styles + Tailwind
│   │   └── monaco-theme.ts             # Custom Monaco themes
│   ├── App.tsx                         # Root component
│   ├── main.tsx                        # Entry point
│   └── env.d.ts                        # Environment type declarations
│
├── src-tauri/                        # Rust backend
│   ├── src/
│   │   ├── main.rs                     # Tauri entry point
│   │   ├── lib.rs                      # Library root
│   │   ├── commands/
│   │   │   ├── mod.rs                  # Commands module
│   │   │   ├── config.rs               # Config read/write/list
│   │   │   ├── debug.rs                # CCH debug invocation
│   │   │   └── validate.rs             # CCH validate invocation
│   │   ├── watchers/
│   │   │   ├── mod.rs                  # Watchers module
│   │   │   └── file_watcher.rs         # File change notifications
│   │   └── error.rs                    # Error types
│   ├── Cargo.toml
│   ├── tauri.conf.json                 # Tauri configuration
│   ├── capabilities/
│   │   └── default.json                # Permission capabilities
│   └── icons/                          # App icons
│
├── tests/                            # Playwright E2E tests
│   ├── editor.spec.ts                  # Editor tests
│   ├── simulator.spec.ts               # Simulator tests
│   ├── validation.spec.ts              # Validation tests
│   ├── files.spec.ts                   # File operations tests
│   └── fixtures/
│       ├── valid-config.yaml           # Test fixture
│       └── invalid-config.yaml         # Test fixture
│
├── public/
│   ├── schema/
│   │   └── hooks-schema.json           # JSON Schema for validation
│   └── favicon.svg
│
├── .github/
│   └── workflows/
│       └── ci.yml                      # CI pipeline (uses Bun)
│
├── package.json                      # Bun package manifest
├── bun.lockb                         # Bun lockfile (binary)
├── bunfig.toml                       # Bun configuration
├── biome.json                        # Biome linter/formatter config
├── tsconfig.json
├── tsconfig.node.json
├── tailwind.config.ts
├── playwright.config.ts
├── index.html                        # HTML entry point
├── .gitignore
└── README.md
```

---

## Wireframes

### Main Application Layout

```
+------------------------------------------------------------------+
|  [logo] RuleZ UI                    [?] [sun/moon] [_] [x]       |
+------------------------------------------------------------------+
|        |                                              |           |
| FILES  |  [hooks.yaml] [project/hooks.yaml]  [+]     | SIMULATOR |
|        +---------------------------------------------+           |
| v Global                                             |           |
|   hooks.yaml *                                       | Event Type|
|                                                      | [PreTool~]|
| v Project                                            |           |
|   .claude/                                           | Tool      |
|     hooks.yaml                                       | [Bash    ]|
|                                                      |           |
|        +---------------------------------------------+ Command   |
|        |  1  version: "1.0"                          | [git push]|
|        |  2                                          |           |
|        |  3  settings:                               | Path      |
|        |  4    log_level: "info"                     | [        ]|
|        |  5    fail_open: true                       |           |
|        |  6                                          | [Simulate]|
|        |  7  rules:                                  +-----------+
|        |  8    - name: block-force-push              |           |
|        |  9      description: "Block force push"     | RESULTS   |
|        | 10      matchers:                           |           |
|        | 11        tools: ["Bash"]                   | Outcome:  |
|        | 12        command_match: "git push.*-f"     | [BLOCKED] |
|        | 13      actions:                            |           |
|        | 14        block: true                       | Matched:  |
|        | 15~~~~~~~~~~~~~~~~~~~~~~~~~~~~             | > block-  |
|        |         ^                                   |   force-  |
|        |         |                                   |   push    |
|        |    [red squiggle = validation error]        |           |
|        |                                             | Eval Time:|
|        |                                             | 2.3ms     |
+--------+---------------------------------------------+-----------+
| Ln 14, Col 8 | YAML | UTF-8 | 2 errors, 1 warning   | Connected |
+------------------------------------------------------------------+
```

### Rule Tree View Panel (Alternative to Code View)

```
+------------------------------------------+
| RULES                              [code]|
+------------------------------------------+
| v Settings                               |
|   log_level: info                        |
|   fail_open: true                        |
|   max_context_size: 1MB                  |
|                                          |
| v Rules (3)                              |
|   +--------------------------------------+
|   | [!] block-force-push          [on]  |
|   |     Tools: Bash                      |
|   |     Match: git push.*--force         |
|   |     Action: BLOCK                    |
|   +--------------------------------------+
|   | [i] inject-python-context     [on]  |
|   |     Tools: Write, Edit               |
|   |     Extensions: .py                  |
|   |     Action: INJECT                   |
|   +--------------------------------------+
|   | [>] run-secret-scanner        [off] |
|   |     Tools: Bash                      |
|   |     Match: git commit                |
|   |     Action: RUN SCRIPT               |
|   +--------------------------------------+
+------------------------------------------+
```

### Debug Simulator - Expanded Results

```
+------------------------------------------------------------------+
| DEBUG SIMULATOR                                                   |
+------------------------------------------------------------------+
|                                                                   |
| Event Configuration                                               |
| +---------------------------------------------------------------+ |
| | Event Type    | Tool          | Command                       | |
| | [PreToolUse v]| [Bash      v] | [git push --force origin main]| |
| +---------------------------------------------------------------+ |
| | Path (optional)                                                | |
| | [                                                            ] | |
| +---------------------------------------------------------------+ |
|                                                                   |
| [ Run Simulation ]                                                |
|                                                                   |
+------------------------------------------------------------------+
| RESULTS                                                           |
+------------------------------------------------------------------+
|                                                                   |
| Outcome: BLOCKED                                        [2.3ms]   |
| Reason: "Force push to main/master is prohibited"                 |
|                                                                   |
| +---------------------------------------------------------------+ |
| | EVALUATION TRACE                                              | |
| +---------------------------------------------------------------+ |
| | #  Rule Name           Match?  Time   Details                 | |
| | -- -------------------- ------- ------ ----------------------- | |
| | 1  block-force-push    YES     0.8ms  command_match matched   | |
| |    > Pattern: git push.*(--force|-f).*(main|master)           | |
| |    > Input: git push --force origin main                      | |
| |    > Action: BLOCK                                            | |
| | -- -------------------- ------- ------ ----------------------- | |
| | 2  inject-python       NO      0.1ms  tool mismatch           | |
| |    > Expected: Write, Edit                                    | |
| |    > Got: Bash                                                | |
| | -- -------------------- ------- ------ ----------------------- | |
| | 3  log-all-commands    SKIP    -      disabled                | |
| +---------------------------------------------------------------+ |
|                                                                   |
+------------------------------------------------------------------+
```

### Validation Error Display

```
+------------------------------------------------------------------+
| PROBLEMS                                                    [x]   |
+------------------------------------------------------------------+
| Errors (2)                                                        |
| +---------------------------------------------------------------+ |
| | [!] Line 14: Invalid value for 'log_level'                    | |
| |     Expected: "debug" | "info" | "warn" | "error"             | |
| |     Got: "verbose"                                            | |
| +---------------------------------------------------------------+ |
| | [!] Line 22: Missing required field 'name' in rule            | |
| +---------------------------------------------------------------+ |
|                                                                   |
| Warnings (1)                                                      |
| +---------------------------------------------------------------+ |
| | [i] Line 8: Rule 'block-force-push' has no description        | |
| +---------------------------------------------------------------+ |
+------------------------------------------------------------------+
```

### Theme Toggle States

```
Light Mode:                          Dark Mode:
+------------------+                 +------------------+
| [sun]  Light     |                 | [moon] Dark      |
| Background: #fff |                 | Background: #1a1a|
| Text: #1a1a1a    |                 | Text: #f5f5f5    |
| Editor: vs       |                 | Editor: vs-dark  |
+------------------+                 +------------------+
```

---

## Component Specifications

### 1. YamlEditor Component

**File:** `src/components/editor/YamlEditor.tsx`

**Purpose:** Monaco-based YAML editor with schema validation

**Props:**
```typescript
interface YamlEditorProps {
  value: string;
  onChange: (value: string) => void;
  onSave?: () => void;
  schema?: object;
  readOnly?: boolean;
  height?: string | number;
}
```

**Features:**
- Monaco Editor with YAML language support
- JSON Schema validation via monaco-yaml
- Inline error markers (red squiggles)
- Autocomplete for rule fields
- Custom dark/light themes
- Keyboard shortcuts (Cmd+S to save, Cmd+Z undo)
- Line numbers with error indicators

**Implementation Notes:**
```typescript
// Key setup for monaco-yaml
import { configureMonacoYaml } from 'monaco-yaml';

useEffect(() => {
  if (monaco) {
    configureMonacoYaml(monaco, {
      enableSchemaRequest: false,
      schemas: [{
        uri: 'inmemory://hooks-schema.json',
        fileMatch: ['*'],
        schema: hooksSchema,
      }],
    });
  }
}, [monaco]);
```

---

### 2. DebugSimulator Component

**File:** `src/components/simulator/DebugSimulator.tsx`

**Purpose:** Simulate events and display rule evaluation results

**State:**
```typescript
interface SimulatorState {
  eventType: EventType;
  tool: string;
  command: string;
  path: string;
  isLoading: boolean;
  result: DebugResult | null;
  error: string | null;
}
```

**Event Types:**
```typescript
type EventType = 
  | 'PreToolUse'
  | 'PostToolUse'
  | 'PermissionRequest'
  | 'UserPromptSubmit'
  | 'SessionStart'
  | 'SessionEnd'
  | 'PreCompact';
```

**Backend Integration:**
```typescript
async function runSimulation(params: SimulatorState) {
  if (isTauri()) {
    // Call CCH binary via Tauri
    return invoke('run_debug', {
      eventType: params.eventType,
      tool: params.tool,
      command: params.command,
      path: params.path,
    });
  } else {
    // Web fallback: mock response
    return mockDebugResult(params);
  }
}
```

---

### 3. RuleTreeView Component

**File:** `src/components/editor/RuleTreeView.tsx`

**Purpose:** Visual tree representation of configured rules

**Props:**
```typescript
interface RuleTreeViewProps {
  config: HooksConfig;
  onRuleClick: (ruleName: string, lineNumber: number) => void;
  onRuleToggle: (ruleName: string, enabled: boolean) => void;
}
```

**Features:**
- Collapsible sections (Settings, Rules)
- Click rule to jump to line in editor
- Toggle switch to enable/disable rules
- Badge showing action type (Block, Inject, Run)
- Icons for rule status

---

### 4. FileSidebar Component

**File:** `src/components/files/FileSidebar.tsx`

**Purpose:** Display and manage configuration files

**Features:**
- Tree view of config file locations
- Global config: `~/.claude/hooks.yaml`
- Project config: `.claude/hooks.yaml`
- File status indicators (modified, errors)
- Create new config file
- Refresh file list

**State:**
```typescript
interface FileState {
  globalConfig: ConfigFile | null;
  projectConfig: ConfigFile | null;
  activeFile: string | null;
}

interface ConfigFile {
  path: string;
  exists: boolean;
  modified: boolean;
  hasErrors: boolean;
  content?: string;
}
```

---

## Tauri IPC Commands

### Rust Command Definitions

**File:** `src-tauri/src/commands/config.rs`

```rust
use serde::{Deserialize, Serialize};
use std::path::PathBuf;
use tauri::command;

#[derive(Serialize, Deserialize)]
pub struct ConfigFile {
    pub path: String,
    pub exists: bool,
    pub content: Option<String>,
}

/// List available config files (global and project)
#[command]
pub async fn list_config_files(project_dir: Option<String>) -> Result<Vec<ConfigFile>, String> {
    let mut files = Vec::new();
    
    // Global config
    if let Some(home) = dirs::home_dir() {
        let global_path = home.join(".claude").join("hooks.yaml");
        files.push(ConfigFile {
            path: global_path.to_string_lossy().to_string(),
            exists: global_path.exists(),
            content: None,
        });
    }
    
    // Project config
    if let Some(dir) = project_dir {
        let project_path = PathBuf::from(dir).join(".claude").join("hooks.yaml");
        files.push(ConfigFile {
            path: project_path.to_string_lossy().to_string(),
            exists: project_path.exists(),
            content: None,
        });
    }
    
    Ok(files)
}

/// Read config file content
#[command]
pub async fn read_config(path: String) -> Result<String, String> {
    std::fs::read_to_string(&path)
        .map_err(|e| format!("Failed to read config: {}", e))
}

/// Write config file content
#[command]
pub async fn write_config(path: String, content: String) -> Result<(), String> {
    // Ensure parent directory exists
    if let Some(parent) = std::path::Path::new(&path).parent() {
        std::fs::create_dir_all(parent)
            .map_err(|e| format!("Failed to create directory: {}", e))?;
    }
    
    std::fs::write(&path, content)
        .map_err(|e| format!("Failed to write config: {}", e))
}
```

**File:** `src-tauri/src/commands/debug.rs`

```rust
use serde::{Deserialize, Serialize};
use std::process::Command;
use tauri::command;

#[derive(Serialize, Deserialize)]
pub struct DebugResult {
    pub outcome: String,        // "Allow", "Block", "Inject"
    pub reason: Option<String>,
    pub matched_rules: Vec<String>,
    pub evaluation_time_ms: f64,
    pub evaluations: Vec<RuleEvaluation>,
}

#[derive(Serialize, Deserialize)]
pub struct RuleEvaluation {
    pub rule_name: String,
    pub matched: bool,
    pub time_ms: f64,
    pub details: String,
}

/// Run CCH debug command and parse output
#[command]
pub async fn run_debug(
    event_type: String,
    tool: Option<String>,
    command: Option<String>,
    path: Option<String>,
) -> Result<DebugResult, String> {
    let mut cmd = Command::new("cch");
    cmd.arg("debug").arg(&event_type);
    
    if let Some(t) = tool {
        cmd.arg("--tool").arg(t);
    }
    if let Some(c) = command {
        cmd.arg("--command").arg(c);
    }
    if let Some(p) = path {
        cmd.arg("--path").arg(p);
    }
    cmd.arg("--verbose");
    cmd.arg("--json"); // Request JSON output for parsing
    
    let output = cmd.output()
        .map_err(|e| format!("Failed to run cch: {}", e))?;
    
    if !output.status.success() {
        return Err(String::from_utf8_lossy(&output.stderr).to_string());
    }
    
    let result: DebugResult = serde_json::from_slice(&output.stdout)
        .map_err(|e| format!("Failed to parse output: {}", e))?;
    
    Ok(result)
}
```

**File:** `src-tauri/src/commands/validate.rs`

```rust
use serde::{Deserialize, Serialize};
use std::process::Command;
use tauri::command;

#[derive(Serialize, Deserialize)]
pub struct ValidationResult {
    pub valid: bool,
    pub errors: Vec<ValidationError>,
    pub warnings: Vec<ValidationWarning>,
}

#[derive(Serialize, Deserialize)]
pub struct ValidationError {
    pub line: usize,
    pub column: usize,
    pub message: String,
}

#[derive(Serialize, Deserialize)]
pub struct ValidationWarning {
    pub line: usize,
    pub message: String,
}

/// Validate config file using CCH
#[command]
pub async fn validate_config(path: String) -> Result<ValidationResult, String> {
    let output = Command::new("cch")
        .arg("validate")
        .arg("--config")
        .arg(&path)
        .arg("--json")
        .output()
        .map_err(|e| format!("Failed to run cch validate: {}", e))?;
    
    let result: ValidationResult = serde_json::from_slice(&output.stdout)
        .map_err(|e| format!("Failed to parse validation output: {}", e))?;
    
    Ok(result)
}
```

### Frontend API Layer

**File:** `src/lib/tauri.ts`

```typescript
// Dual-mode API: works in both Tauri desktop and web browser

declare global {
  interface Window {
    __TAURI__?: {
      invoke: <T>(cmd: string, args?: Record<string, unknown>) => Promise<T>;
    };
  }
}

export const isTauri = (): boolean => {
  return typeof window !== 'undefined' && !!window.__TAURI__;
};

export async function invoke<T>(
  cmd: string,
  args?: Record<string, unknown>
): Promise<T> {
  if (isTauri() && window.__TAURI__) {
    return window.__TAURI__.invoke<T>(cmd, args);
  }
  throw new Error(`Tauri not available for command: ${cmd}`);
}

// Config commands
export async function listConfigFiles(projectDir?: string): Promise<ConfigFile[]> {
  if (isTauri()) {
    return invoke('list_config_files', { projectDir });
  }
  // Web fallback
  return getMockConfigFiles();
}

export async function readConfig(path: string): Promise<string> {
  if (isTauri()) {
    return invoke('read_config', { path });
  }
  // Web fallback
  return localStorage.getItem(`config:${path}`) ?? getMockConfig(path);
}

export async function writeConfig(path: string, content: string): Promise<void> {
  if (isTauri()) {
    return invoke('write_config', { path, content });
  }
  // Web fallback
  localStorage.setItem(`config:${path}`, content);
}

// Debug commands
export async function runDebug(params: DebugParams): Promise<DebugResult> {
  if (isTauri()) {
    return invoke('run_debug', params);
  }
  // Web fallback
  return getMockDebugResult(params);
}
```

---

## JSON Schema

**File:** `public/schema/hooks-schema.json`

```json
{
  "$schema": "http://json-schema.org/draft-07/schema#",
  "$id": "https://cch.dev/schemas/hooks.json",
  "title": "CCH Hooks Configuration",
  "description": "Configuration schema for Claude Context Hooks (CCH)",
  "type": "object",
  "required": ["version"],
  "additionalProperties": false,
  "properties": {
    "version": {
      "type": "string",
      "description": "Configuration version (format: X.Y)",
      "pattern": "^\\d+(\\.\\d+)?$",
      "examples": ["1.0", "1"]
    },
    "settings": {
      "$ref": "#/definitions/Settings"
    },
    "rules": {
      "type": "array",
      "description": "List of hook rules",
      "items": {
        "$ref": "#/definitions/Rule"
      }
    },
    "hooks": {
      "type": "array",
      "description": "Alternative name for rules (legacy support)",
      "items": {
        "$ref": "#/definitions/Rule"
      }
    }
  },
  "definitions": {
    "Settings": {
      "type": "object",
      "description": "Global settings for CCH",
      "additionalProperties": false,
      "properties": {
        "log_level": {
          "type": "string",
          "description": "Logging verbosity level",
          "enum": ["debug", "info", "warn", "error"],
          "default": "info"
        },
        "max_context_size": {
          "type": "integer",
          "description": "Maximum injected context size in bytes",
          "minimum": 0,
          "default": 1048576
        },
        "script_timeout": {
          "type": "integer",
          "description": "Default script timeout in seconds",
          "minimum": 1,
          "default": 5
        },
        "fail_open": {
          "type": "boolean",
          "description": "Continue on errors (true) or fail (false)",
          "default": true
        },
        "debug_logs": {
          "type": "boolean",
          "description": "Enable detailed debug logging",
          "default": false
        }
      }
    },
    "Rule": {
      "type": "object",
      "description": "A single hook rule",
      "required": ["name"],
      "additionalProperties": false,
      "properties": {
        "name": {
          "type": "string",
          "description": "Unique rule identifier",
          "pattern": "^[a-zA-Z0-9_-]+$",
          "minLength": 1,
          "maxLength": 64
        },
        "description": {
          "type": "string",
          "description": "Human-readable explanation of the rule"
        },
        "event": {
          "type": "string",
          "description": "Event type to match (alternative to matchers.operations)",
          "enum": [
            "PreToolUse",
            "PostToolUse",
            "PermissionRequest",
            "UserPromptSubmit",
            "SessionStart",
            "SessionEnd",
            "PreCompact"
          ]
        },
        "priority": {
          "type": "integer",
          "description": "Rule priority (higher = evaluated first)",
          "default": 0
        },
        "matchers": {
          "$ref": "#/definitions/Matchers"
        },
        "match": {
          "$ref": "#/definitions/Matchers",
          "description": "Alternative name for matchers"
        },
        "actions": {
          "$ref": "#/definitions/Actions"
        },
        "action": {
          "$ref": "#/definitions/Action",
          "description": "Alternative single action format"
        },
        "metadata": {
          "$ref": "#/definitions/RuleMetadata"
        }
      }
    },
    "Matchers": {
      "type": "object",
      "description": "Conditions that trigger the rule",
      "additionalProperties": false,
      "properties": {
        "tools": {
          "type": "array",
          "description": "Tool names to match",
          "items": {
            "type": "string",
            "enum": [
              "Bash",
              "Read",
              "Write",
              "Edit",
              "Glob",
              "Grep",
              "WebFetch",
              "TodoRead",
              "TodoWrite",
              "Task"
            ]
          }
        },
        "extensions": {
          "type": "array",
          "description": "File extensions to match",
          "items": {
            "type": "string",
            "pattern": "^\\.[a-zA-Z0-9]+$"
          },
          "examples": [[".py", ".rs", ".ts"]]
        },
        "directories": {
          "type": "array",
          "description": "Directory patterns to match (glob-like)",
          "items": {
            "type": "string"
          },
          "examples": [["src/**", "tests/**"]]
        },
        "operations": {
          "type": "array",
          "description": "Event types to match",
          "items": {
            "type": "string",
            "enum": [
              "PreToolUse",
              "PostToolUse",
              "PermissionRequest",
              "UserPromptSubmit",
              "SessionStart",
              "SessionEnd",
              "PreCompact"
            ]
          }
        },
        "command_match": {
          "type": "string",
          "description": "Regex pattern for command matching"
        }
      }
    },
    "Actions": {
      "type": "object",
      "description": "Actions to perform when rule matches",
      "additionalProperties": false,
      "properties": {
        "block": {
          "type": "boolean",
          "description": "Block the operation"
        },
        "block_if_match": {
          "type": "string",
          "description": "Conditional block based on content regex"
        },
        "inject": {
          "type": "string",
          "description": "Path to file to inject as context"
        },
        "run": {
          "type": "string",
          "description": "Path to validator script to execute"
        }
      }
    },
    "Action": {
      "type": "object",
      "description": "Single action format (alternative)",
      "additionalProperties": false,
      "properties": {
        "type": {
          "type": "string",
          "enum": ["block", "inject", "run"],
          "description": "Action type"
        },
        "reason": {
          "type": "string",
          "description": "Explanation for blocking"
        },
        "source": {
          "type": "string",
          "enum": ["file", "inline"],
          "description": "Source type for inject action"
        },
        "path": {
          "type": "string",
          "description": "File path for inject action"
        },
        "content": {
          "type": "string",
          "description": "Inline content for inject action"
        },
        "command": {
          "type": "string",
          "description": "Command to run for run action"
        },
        "timeout": {
          "type": "integer",
          "description": "Timeout for run action in seconds"
        }
      }
    },
    "RuleMetadata": {
      "type": "object",
      "description": "Optional rule metadata",
      "additionalProperties": false,
      "properties": {
        "priority": {
          "type": "integer",
          "description": "Rule priority (higher = evaluated first)",
          "default": 0
        },
        "timeout": {
          "type": "integer",
          "description": "Script timeout override in seconds"
        },
        "enabled": {
          "type": "boolean",
          "description": "Enable/disable the rule",
          "default": true
        }
      }
    }
  }
}
```

---

## Testing Strategy

### Test Categories

| Category | Tool | Coverage Target |
|----------|------|-----------------|
| Unit Tests | **Bun test** | 80%+ for utilities |
| Component Tests | **Bun test** + Testing Library | Key components |
| E2E Tests | Playwright (via Bun) | Critical user flows |
| Integration Tests | Playwright | Tauri IPC (manual) |

### Bun Unit Tests

**File:** `src/lib/yaml-utils.test.ts`

```typescript
import { describe, expect, test } from 'bun:test';
import { parseYaml, stringifyYaml, validateYaml } from './yaml-utils';

describe('YAML Utils', () => {
  test('parseYaml should parse valid YAML', () => {
    const yaml = 'version: "1.0"\nrules: []';
    const result = parseYaml(yaml);
    expect(result.version).toBe('1.0');
    expect(result.rules).toEqual([]);
  });

  test('parseYaml should throw on invalid YAML', () => {
    const yaml = 'invalid: yaml: syntax:';
    expect(() => parseYaml(yaml)).toThrow();
  });

  test('stringifyYaml should produce valid YAML', () => {
    const obj = { version: '1.0', rules: [] };
    const result = stringifyYaml(obj);
    expect(result).toContain('version:');
  });
});
```

### Playwright E2E Tests

**File:** `tests/editor.spec.ts`

```typescript
import { test, expect } from '@playwright/test';

test.describe('YAML Editor', () => {
  test.beforeEach(async ({ page }) => {
    await page.goto('/');
  });

  test('should load with default content', async ({ page }) => {
    const editor = page.locator('.monaco-editor');
    await expect(editor).toBeVisible();
  });

  test('should show validation error for invalid YAML', async ({ page }) => {
    // Focus editor and type invalid YAML
    await page.click('.monaco-editor');
    await page.keyboard.type('invalid: yaml: syntax:');
    
    // Wait for validation
    await page.waitForTimeout(500);
    
    // Check for error marker
    const errorMarker = page.locator('.squiggly-error');
    await expect(errorMarker).toBeVisible();
  });

  test('should show autocomplete for rule fields', async ({ page }) => {
    await page.click('.monaco-editor');
    await page.keyboard.type('version: "1.0"\nrules:\n  - ');
    await page.keyboard.press('Control+Space');
    
    const suggestions = page.locator('.monaco-list-row');
    await expect(suggestions.first()).toBeVisible();
  });

  test('should save with keyboard shortcut', async ({ page }) => {
    await page.click('.monaco-editor');
    await page.keyboard.type('version: "1.0"');
    await page.keyboard.press('Meta+s');
    
    // Check for save confirmation
    await expect(page.locator('text=Saved')).toBeVisible();
  });
});
```

**File:** `tests/simulator.spec.ts`

```typescript
import { test, expect } from '@playwright/test';

test.describe('Debug Simulator', () => {
  test.beforeEach(async ({ page }) => {
    await page.goto('/');
  });

  test('should simulate PreToolUse event', async ({ page }) => {
    // Select event type
    await page.selectOption('[data-testid="event-type"]', 'PreToolUse');
    
    // Enter tool and command
    await page.fill('[data-testid="tool-input"]', 'Bash');
    await page.fill('[data-testid="command-input"]', 'git push --force');
    
    // Run simulation
    await page.click('[data-testid="simulate-button"]');
    
    // Wait for results
    await expect(page.locator('[data-testid="result-outcome"]')).toBeVisible();
  });

  test('should display matched rules', async ({ page }) => {
    // Load a config with known rules first
    await page.evaluate(() => {
      localStorage.setItem('config:test', `
version: "1.0"
rules:
  - name: block-force-push
    matchers:
      tools: ["Bash"]
      command_match: "git push.*--force"
    actions:
      block: true
      `);
    });
    
    await page.reload();
    
    // Simulate event that matches
    await page.selectOption('[data-testid="event-type"]', 'PreToolUse');
    await page.fill('[data-testid="tool-input"]', 'Bash');
    await page.fill('[data-testid="command-input"]', 'git push --force origin main');
    await page.click('[data-testid="simulate-button"]');
    
    // Check for matched rule
    await expect(page.locator('text=block-force-push')).toBeVisible();
    await expect(page.locator('[data-testid="result-outcome"]')).toContainText('BLOCKED');
  });
});
```

### Playwright Configuration

**File:** `playwright.config.ts`

```typescript
import { defineConfig, devices } from '@playwright/test';

export default defineConfig({
  testDir: './tests',
  fullyParallel: true,
  forbidOnly: !!process.env.CI,
  retries: process.env.CI ? 2 : 0,
  workers: process.env.CI ? 1 : undefined,
  reporter: 'html',
  
  use: {
    baseURL: 'http://localhost:1420',
    trace: 'on-first-retry',
    screenshot: 'only-on-failure',
  },

  projects: [
    {
      name: 'chromium',
      use: { ...devices['Desktop Chrome'] },
    },
    {
      name: 'firefox',
      use: { ...devices['Desktop Firefox'] },
    },
    {
      name: 'webkit',
      use: { ...devices['Desktop Safari'] },
    },
  ],

  webServer: {
    command: 'bun run dev',
    url: 'http://localhost:1420',
    reuseExistingServer: !process.env.CI,
    timeout: 120 * 1000,
  },
});
```

---

## Phase 1 Milestones

| Milestone | Description | Deliverables | Est. Days |
|-----------|-------------|--------------|-----------|
| **M1** | Project Setup | Tauri + React scaffold with **Bun**, dual-mode architecture | 1 |
| **M2** | Monaco Editor | YAML editor with syntax highlighting, basic editing | 1 |
| **M3** | Schema Validation | JSON Schema integration, inline errors, autocomplete | 2 |
| **M4** | File Operations | Read/write hooks.yaml, global + project support | 1 |
| **M5** | Rule Tree View | Visual tree component, rule navigation | 1 |
| **M6** | Debug Simulator | Event form, CCH integration, results display | 2 |
| **M7** | Theming | Dark/light mode, system preference, Monaco themes | 0.5 |
| **M8** | Playwright Tests | E2E test suite, CI setup with Bun | 1 |

**Total Estimate:** 9.5 days

### Milestone Dependencies

```
M1 (Setup with Bun)
  │
  ├──► M2 (Monaco) ──► M3 (Schema)
  │                        │
  │                        v
  ├──► M4 (Files) ────────►├──► M5 (Tree View)
  │                        │
  │                        v
  └──► M7 (Theming) ──────►└──► M6 (Simulator)
                                    │
                                    v
                               M8 (Tests)
```

---

## Future Phases

### Phase 2: Log Viewer (Est. 5-7 days)

**Features:**
- Real-time log streaming with file watcher
- Virtual scrolling for large log files
- Filter by log level (debug, info, warn, error)
- Filter by rule name
- Filter by timestamp range
- Full-text search
- Export logs to file
- Log entry detail view

**Components:**
- `LogViewer.tsx` - Main log viewer container
- `LogEntry.tsx` - Single log entry display
- `LogFilters.tsx` - Filter controls
- `LogSearch.tsx` - Search input

### Phase 3: Advanced Features (Est. 7-10 days)

**Features:**
- Rule templates library (pre-built rules)
- Import/export configurations
- Diff view between global and project configs
- Performance metrics dashboard
- Integration with Claude Code settings.json
- Rule creation wizard
- Regex tester for command_match patterns
- Context file preview (for inject actions)

### Phase 4: Polish & Distribution (Est. 3-5 days)

**Features:**
- Application installer (DMG, MSI, DEB)
- Auto-update mechanism
- Onboarding tutorial
- Documentation integration
- Error reporting/telemetry (opt-in)

---

## Development Workflow

### Daily Development Cycle

```
┌─────────────────────────────────────────────────────────────────┐
│                    Development Workflow (Bun)                    │
├─────────────────────────────────────────────────────────────────┤
│                                                                  │
│  Morning: UI Development                                         │
│  ─────────────────────────                                       │
│  $ bun run dev                                                   │
│  └── Opens browser at localhost:1420                             │
│  └── Hot reload for React changes                                │
│  └── Use browser DevTools for debugging                          │
│                                                                  │
│  Afternoon: Integration Testing                                  │
│  ────────────────────────────                                    │
│  $ bun run dev:tauri                                             │
│  └── Opens Tauri window                                          │
│  └── Test file system operations                                 │
│  └── Test CCH binary integration                                 │
│                                                                  │
│  Evening: Automated Testing                                      │
│  ──────────────────────────                                      │
│  $ bun run test:e2e                                              │
│  └── Runs Playwright E2E tests                                   │
│  └── Generates test report                                       │
│                                                                  │
└─────────────────────────────────────────────────────────────────┘
```

### Available Scripts (package.json)

```json
{
  "scripts": {
    "dev": "bunx --bun vite",
    "dev:tauri": "bunx tauri dev",
    "build": "bunx --bun vite build",
    "build:tauri": "bunx tauri build",
    "preview": "bunx --bun vite preview",
    "test": "bun test",
    "test:e2e": "bunx playwright test",
    "test:e2e:ui": "bunx playwright test --ui",
    "lint": "bunx @biomejs/biome check src",
    "lint:fix": "bunx @biomejs/biome check --apply src",
    "format": "bunx @biomejs/biome format --write src",
    "typecheck": "bunx tsc --noEmit"
  }
}
```

### Bun Configuration

**File:** `bunfig.toml`

```toml
[install]
# Use exact versions for reproducible builds
exact = true

[install.lockfile]
# Use binary lockfile for faster installs
save = true

[run]
# Enable bun's native TypeScript support
bun = true
```

### Biome Configuration (replaces ESLint + Prettier)

**File:** `biome.json`

```json
{
  "$schema": "https://biomejs.dev/schemas/1.5.0/schema.json",
  "organizeImports": {
    "enabled": true
  },
  "linter": {
    "enabled": true,
    "rules": {
      "recommended": true,
      "correctness": {
        "useExhaustiveDependencies": "warn"
      },
      "style": {
        "noNonNullAssertion": "off"
      }
    }
  },
  "formatter": {
    "enabled": true,
    "indentStyle": "space",
    "indentWidth": 2,
    "lineWidth": 100
  },
  "javascript": {
    "formatter": {
      "quoteStyle": "single",
      "semicolons": "always"
    }
  }
}
```

---

## Risk Assessment

| Risk | Likelihood | Impact | Mitigation |
|------|------------|--------|------------|
| Monaco + YAML integration issues | Medium | High | Early spike on monaco-yaml setup |
| CCH binary not found in PATH | Medium | Medium | Add binary path configuration in settings |
| Tauri 2.0 breaking changes | Low | Medium | Pin Tauri version, monitor release notes |
| File permission issues (global config) | Medium | Medium | Graceful error handling, user guidance |
| Large log files causing performance issues | Medium | High | Virtual scrolling, pagination (Phase 2) |
| Cross-platform file path differences | Medium | Medium | Use Tauri's path resolution APIs |
| **Bun compatibility issues** | Low | Medium | Bun has excellent React/TS support; fallback to Node if needed |

---

## Appendix A: CCH Binary Requirements

The RuleZ UI depends on the CCH binary being installed and accessible. The following commands must be available:

| Command | Purpose | Required Flags |
|---------|---------|----------------|
| `cch debug` | Simulate events | `--json` for structured output |
| `cch validate` | Validate config | `--json` for structured output |
| `cch --version` | Version check | - |

**Note:** If CCH binary is not in PATH, the UI should:
1. Show a warning message
2. Allow user to specify binary path in settings
3. Disable debug simulator (but still allow editing/validation)

---

## Appendix B: Color Palette

### Light Theme

| Element | Color | Hex |
|---------|-------|-----|
| Background | White | `#FFFFFF` |
| Surface | Light Gray | `#F5F5F5` |
| Text Primary | Dark Gray | `#1A1A1A` |
| Text Secondary | Medium Gray | `#666666` |
| Accent | Blue | `#3B82F6` |
| Error | Red | `#EF4444` |
| Warning | Amber | `#F59E0B` |
| Success | Green | `#10B981` |

### Dark Theme

| Element | Color | Hex |
|---------|-------|-----|
| Background | Dark | `#1A1A1A` |
| Surface | Darker | `#252525` |
| Text Primary | Light | `#F5F5F5` |
| Text Secondary | Gray | `#A0A0A0` |
| Accent | Blue | `#60A5FA` |
| Error | Red | `#F87171` |
| Warning | Amber | `#FBBF24` |
| Success | Green | `#34D399` |

---

## Appendix C: Keyboard Shortcuts

| Shortcut | Action |
|----------|--------|
| `Cmd/Ctrl + S` | Save current file |
| `Cmd/Ctrl + Z` | Undo |
| `Cmd/Ctrl + Shift + Z` | Redo |
| `Cmd/Ctrl + F` | Find in editor |
| `Cmd/Ctrl + Shift + F` | Find in all files |
| `Cmd/Ctrl + /` | Toggle comment |
| `Cmd/Ctrl + D` | Run debug simulation |
| `Cmd/Ctrl + ,` | Open settings |
| `Cmd/Ctrl + 1` | Switch to Editor tab |
| `Cmd/Ctrl + 2` | Switch to Simulator tab |
| `Cmd/Ctrl + 3` | Switch to Logs tab (Phase 2) |

---

## Appendix D: Bun vs Node.js Comparison

| Feature | Bun | Node.js |
|---------|-----|---------|
| Package Install | ~10x faster | Baseline |
| TypeScript | Native support | Requires ts-node/tsx |
| Test Runner | Built-in (`bun test`) | Requires vitest/jest |
| Bundler | Built-in | Requires vite/webpack |
| Start Time | ~4x faster | Baseline |
| npm Compatibility | 100% | Native |

**Why Bun for RuleZ UI:**
- Faster development iteration with instant TypeScript execution
- Single tool for install, run, test, and bundle
- Full npm compatibility (all React/Monaco packages work)
- Native Tauri CLI works via `bunx`

---

## Document History

| Version | Date | Author | Changes |
|---------|------|--------|---------|
| 1.0 | 2026-01-23 | Claude Code | Initial draft |
| 1.1 | 2026-01-23 | Claude Code | Updated to use Bun for all TypeScript/React tooling |

---

*This document serves as the foundation for the RuleZ UI PRD and SDD workflow initialization.*
