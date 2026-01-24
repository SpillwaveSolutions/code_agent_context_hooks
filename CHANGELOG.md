# Changelog

All notable changes to Claude Context Hooks (CCH) will be documented in this file.

The format is based on [Keep a Changelog](https://keepachangelog.com/en/1.0.0/),
and this project adheres to [Semantic Versioning](https://semver.org/spec/v2.0.0.html).

## [1.0.0] - 2026-01-23

### Added

#### Core Features
- **Block Dangerous Operations** - Prevent destructive commands like `git push --force`
- **Inject Context** - Automatically inject context files based on directory patterns
- **Run Custom Validators** - Execute Python/shell scripts to validate tool inputs
- **Permission Explanations** - Provide structured explanations for permission requests

#### CLI Commands
- `cch init` - Create default hooks.yaml with example rules and context files
- `cch install` - Register CCH with Claude Code settings.json
- `cch uninstall` - Remove CCH from Claude Code settings
- `cch validate` - Validate hooks.yaml configuration syntax and schema
- `cch logs` - Query and filter JSON Lines log entries
- `cch explain` - Explain which rules matched an event
- `cch debug` - Simulate events to test rule matching
- `cch repl` - Interactive debug mode for testing rules

#### Configuration
- YAML-based rule configuration in `.claude/hooks.yaml`
- Support for global (`~/.claude/hooks.yaml`) and project-level configs
- Rule matchers: `tools`, `extensions`, `directories`, `operations`, `command_patterns`
- Rule actions: `block`, `block_if_match`, `inject`, `run`

#### Logging & Observability
- JSON Lines format for machine-readable logs
- Structured event details for all tool types
- Response summary logging (continue, reason, context_length)
- Debug mode with raw event and rule evaluation details

#### Performance
- Sub-10ms event processing (<3ms actual)
- Cold start under 5ms p95
- Minimal memory footprint (<50MB resident)

### Technical Details

- **Language**: Rust 2024 edition
- **Runtime**: Tokio async (current_thread flavor)
- **Zero unsafe code**: Memory safety guaranteed by compiler
- **Cross-platform**: Linux (x86_64, aarch64), macOS (x86_64, aarch64), Windows (x86_64)

### Testing

- 64 tests covering all user stories
- Unit tests for core logic
- Integration tests for CLI commands
- Performance tests for latency requirements

## Links

- [Documentation](docs/README.md)
- [User Guide - CLI](docs/USER_GUIDE_CLI.md)
- [User Guide - Skill](docs/USER_GUIDE_SKILL.md)
