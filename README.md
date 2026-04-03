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

- **windows** - ~\.codeium\windsurf\global_workflows
- **linux** - ~/.codeium/windsurf/global_workflows

#### Windows Installation

```powershell
# Create global windsurf directories
mkdir "$env:USERPROFILE\.codeium\windsurf\global_workflows"
mkdir "$env:USERPROFILE\.codeium\windsurf\skills"

# Copy skills and workflows
Copy-Item -Recurse "windsurf\skills\*" "$env:USERPROFILE\.codeium\windsurf\skills\"
Copy-Item "windsurf\workflows\*" "$env:USERPROFILE\.codeium\windsurf\global_workflows\"

# Copy global rules
Copy-Item "windsurf\global_rules.md" "$env:USERPROFILE\.codeium\windsurf\"
```

#### Linux Installation

```bash
# Create global windsurf directories
mkdir -p ~/.codeium/windsurf/global_workflows
mkdir -p ~/.codeium/windsurf/skills

# Copy skills and workflows
cp -r windsurf/skills/* ~/.codeium/windsurf/skills/
cp windsurf/workflows/* ~/.codeium/windsurf/global_workflows/

# Copy global rules
cp windsurf/global_rules.md ~/.codeium/windsurf/
```

## Project-Level Usage

For project-specific configuration, place files in:

### Opencode

```
your-project/
├── .opencode/
│   ├── skills/
│   └── agents/
```

### Windsurf

```
your-project/
├── .windsurf/
│   ├── workflows/
│   ├── skills/
│   └── global_rules.md
```
