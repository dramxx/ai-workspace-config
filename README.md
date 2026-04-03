# AI Workspace Config

Meant for agentic coding with `PLAN.md`driven execution. Agents should load only the smallest useful set of skills for the task.

## Structure

- **ai-workspace-config.json** - Config file in a very generic state
- **AGENTS.md** - Global execution contract for workflow, scope, verification, and output behavior
- **opencode/** - Original Opencode configuration (legacy)
- **windsurf/** - Windsurf-specific configuration and workflows

## Agents (opencode) / Workflows (windsurf)

- **code-reviewer** - Diff-first code review with scoped findings proportional to change size
- **deep-thinker** - Structured thinking partner for ambiguity, trade-offs, and problem framing
- **refactoring** - Safe, incremental refactoring with context-aware skill loading

## Skills (both)

- `typescript-best-practices` - Full-stack TypeScript guidance
- `naming-cheatsheet` - A/HC/LC naming conventions
- `code-architecture` - Avoiding wrong abstractions
- `project-structure` - Feature-based architecture
- `five-whys` - Root cause analysis
- `hypothesis-tree` - Structured problem decomposition
- `godot-best-practices` - Godot 4 development patterns
- `rust-cli-development` - Rust CLI application development

## Global Installation

### Opencode

- **windows** - ~\.config\opencode
- **linux** - ~/.config/opencode

### Windsurf

- **windows** - ~\.codeium/windsurf
- **linux** - ~/.codeium/windsurf
