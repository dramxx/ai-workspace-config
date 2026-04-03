---
description: Plans and implements code refactoring with intelligent skill loading
---

1. **Understand Scope**
   - Confirm which files/components/modules are in scope
   - Identify the goal (clarity, patterns, performance)
   - Note constraints (must preserve behavior, no breaking changes)
   - If target files unclear, ask for smallest scope to proceed safely

2. **Analyze Code Structure**
   - Read target files to understand current implementation
   - Identify patterns (languages, frameworks, anti-patterns)
   - Map dependencies and relationships
   - Note complexity hotspots and code smells

3. **Load Relevant Skills**
   - Use skill tool to load 2-5 relevant skills based on detected patterns:
     * Files: .ts, .tsx → typescript-best-practices
     * Files: .tsx, .jsx → react-hooks-patterns
     * Has Tailwind classes → tailwind-v4-best-practices
     * Has CSS (.css, .scss) → css-container-queries
     * Complex abstractions → code-architecture-wrong-abstraction
     * Naming concerns detected → naming-cheatsheet
     * Nested conditionals > 3 → apply guard clause pattern
   - Max 5 skills at a time to avoid overwhelm
   - Prioritize: framework skills → pattern skills → architecture skills

4. **Create Refactoring Plan**
   - List identified issues by severity
   - Create step-by-step plan with safe ordering
   - Identify dependencies between changes
   - Estimate change size (small/medium/large)
   - Note which skills will guide each change

5. **Implement Refactoring**
   - Execute plan incrementally using edit tool
   - Preserve all functionality
   - Apply loaded skill guidance for each change
   - Keep changes small and verifiable
   - Document each step as executed

6. **Verify Changes**
   - Check behavior is unchanged
   - Run tests if available
   - Confirm improvements in code quality
   - Validate that all issues from plan are addressed

7. **Output Format**
   ```
   ## Refactoring Plan

   ### Scope
   - **Files affected:** [list]
   - **Changes estimated:** [small/medium/large]

   ### Identified Issues
   1. [Issue 1] — [Severity]
   2. [Issue 2] — [Severity]

   ### Skills Loaded
   - [skill-1]: [reason loaded]
   - [skill-2]: [reason loaded]

   ### Steps
   1. [First safest change]
   2. [Second change]
   3. [...]

   ### Implementation
   [As each step is executed, note the specific files and changes made]

   ### Verification
   - [ ] All tests pass (if available)
   - [ ] Behavior preserved
   - [ ] Code quality improved
   ```

8. **Quality Principles**

   **Do:**
   - Preserve all behavioral contracts
   - Make incremental, testable changes
   - Apply project-specific conventions
   - Use skills for guidance on complex patterns

   **Don't:**
   - Change behaviors without explicit request
   - Over-refactor for minimal gains
   - Skip verification steps
   - Make assumptions about unclear requirements

9. **Safety Checks**
   - Before each edit: Does this change behavior?
   - After each edit: Can this be verified independently?
   - At completion: Are all goals met?
   - Final check: Is the code more maintainable?
