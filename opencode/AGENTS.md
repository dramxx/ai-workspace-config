## Environment

- OS: Windows 11
- Shell: PowerShell (`pwsh`) only
- Never reference `bash.exe`, `sh.exe`, or Git Bash
- Run commands directly via `pwsh`

## Language

- English only

## Git

- Do not use git commands, unless explicitly asked
- Ensure proper `.gitignore` exists in project

## Workflow

- When a `PLAN.md` exists in the working directory, read it first before doing anything else
- Treat `PLAN.md` as the source of truth for the task
- Execute the plan fully in one pass unless a real blocker prevents progress
- Do not stop midway to summarize or ask for confirmation unless blocked
- If `PLAN.md` conflicts with the actual codebase, stop and ask one focused question before deviating
- If ambiguity would materially affect implementation, ask one focused question, not a list
- Start with the user's stated scope and goal when no `PLAN.md` is present

## Instruction Precedence

- Follow direct user instructions first
- Then follow `PLAN.md` when it exists
- Then follow repo-specific guidance in the current project
- Then follow this global `AGENTS.md`
- If two sources conflict and the conflict materially affects the work, ask one focused question before proceeding

## Response Style

- Lead with the result, not the explanation
- Keep structure short by default
- Be concrete: name the files, risks, and next actions explicitly
- No preamble, no postamble, no self-congratulatory filler
- Ask only necessary questions

## General Working Rules

- Prefer small, high-confidence changes over broad rewrites
- Expand scope only when required to complete the task correctly
- State assumptions clearly when context is incomplete
- Do not invent files, commands, APIs, tools, or dependencies
- Refer to relevant skills only when they materially improve the work
- Follow existing project conventions first before introducing new patterns

## Code Change Guidelines

- Preserve behavior unless the user explicitly asks for behavior change
- Edit the narrowest reasonable scope first, then expand only if needed
- Do not refactor outside the scope of the current task
- Favor guard clauses, low nesting, and single-purpose functions
- Prefer simple solutions over clever or over-engineered ones
- Prefer simple duplication over premature abstraction
- Keep architecture maintainable and scalable, but stay proportional to the current task
- No comments in code unless explicitly asked
- Verify or recommend verification steps when changes affect behavior

## Verification Rules

- When changes affect behavior, run the smallest relevant verification step if practical
- Prefer targeted tests or checks over broad full-project runs
- If verification is unavailable, too expensive, or blocked, say so plainly
- Do not claim something is verified unless it was actually checked

## Dependency And Tooling Rules

- Do not install or suggest any package version released within the last 7 days, including transitive dependencies where possible. If a newer version exists but is under 7 days old, fall back to the most recent version that meets the age requirement.
- Additionally, ensure the project's package manager config enforces this at install time: min-release-age=7 for npm (.npmrc), minimumReleaseAge=10080 for pnpm (.npmrc), or npmMinimalAgeGate: 10080 for Yarn (.yarnrc.yml). Apply this to all package managers used in the project.
- Never assume a library is available; verify it in the appropriate manifest before using it
- Do not install new dependencies without asking first
- Use file write/edit tools to create or update files; do not use shell redirection for file creation when a proper file tool is available

## Documentation Rules

- Keep `README` and other relevant markdown files in sync when codebase changes make them stale
- Update documentation only when the task or code changes actually require it

## Review Guidelines

- Review changed code first, then expand only to directly affected callers, tests, contracts, and configuration
- Prioritize correctness, risk, maintainability, and security over style nitpicks
- Call out missing context instead of guessing
- Keep recommendations actionable and proportional to the change size

## Skill Usage Guidelines

- Load only the few skills that are directly relevant to the task
- Prefer broad, reusable coding skills before niche strategy or product skills
- If two skills conflict, prefer the one that matches the repo conventions and user goal

## Task-Specific Output

- For implementation tasks, only call out blockers, assumptions, risks, or verification when useful
- For review tasks, rank findings by severity and explain impact
- For planning tasks, provide a short plan with concrete next actions
