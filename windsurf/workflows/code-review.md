---
description: Review changed code for correctness, risk, maintainability, and security
---

1. **Initial Triage**
   - Review the changed code first
   - Expand scope only to directly affected callers, tests, contracts, and configuration
   - Scale depth by change size:
     * <200 lines: Deep, comprehensive
     * 200-500 lines: Standard
     * >500 lines: Flag for human oversight, focus on critical paths
   - Classify: feature | bug fix | refactoring | breaking change
   - Keep findings proportional to the risk and size of the change

2. **Security Review (OWASP Top 10)**
   - Check for broken access control and IDOR vulnerabilities
   - Verify cryptographic implementations and secure random generation
   - Look for SQL/NoSQL/command injection via user input
   - Assess insecure design and missing threat modeling
   - Identify security misconfigurations and default credentials
   - Check for vulnerable components with known CVEs
   - Review authentication failures and session management
   - Verify data integrity checks and JWT signing
   - Ensure proper audit logging and no sensitive data in logs
   - Check for SSRF via unvalidated user-controlled URLs
   - Also check: Input validation, secret exposure, timing attacks

3. **Performance & Scalability Analysis**
   - Identify N+1 queries (DB calls in loops)
   - Check for missing indexes on queried columns
   - Look for synchronous external API calls blocking threads
   - Assess in-memory state that won't scale horizontally
   - Find unbounded collections without pagination
   - Verify connection pooling and rate limiting
   - Analyze algorithm complexity (time/space)
   - Check memory allocation patterns
   - Identify caching opportunities
   - Assess lazy loading implementation

4. **Architecture & Design Review**
   - Check SOLID violations:
     * Single Responsibility: Multiple reasons to change
     * Open/Closed: Requires modification to extend
     * Liskov Substitution: Subtypes not substitutable
     * Interface Segregation: Forced dependencies on unused methods
     * Dependency Inversion: Depends on concretions
   - Identify anti-patterns:
     * God objects (>500 lines or >20 methods)
     * Anemic domain models
     * Shotgun surgery, inappropriate intimacy, feature envy
   - For distributed systems: Service cohesion, data ownership, API versioning, circuit breakers, idempotency

5. **Code Quality Assessment**
   - Check for proper error handling and propagation
   - Verify consistent naming conventions
   - Assess code complexity and readability
   - Look for dead code and unused imports
   - Check for proper documentation and comments
   - Verify consistent code style and formatting
   - Assess test coverage and test quality
   - Check for proper separation of concerns

6. **Testing & Validation**
   - Review test coverage for changed code
   - Check for proper unit tests, integration tests
   - Verify edge cases and error scenarios are tested
   - Assess test quality and maintainability
   - Check for proper mocking and test isolation

7. **Final Assessment**
   - Summarize findings by severity (Critical, High, Medium, Low)
   - Provide actionable recommendations
   - Estimate effort required for fixes
   - Flag any blockers for merge/deployment
   - Suggest additional review if needed

8. **Output Format**
   - Lead with most critical findings first
   - Group by category (Security, Performance, Architecture, Code Quality)
   - Provide specific line references where applicable
   - Include concrete examples of issues found
   - Suggest specific fixes or improvements
   - Keep recommendations proportional to change size
