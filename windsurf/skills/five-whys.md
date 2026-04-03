---
description: Root cause analysis using the Five Whys technique for problem investigation
---

# Five Whys - Root Cause Analysis

Systematic guide to uncovering root causes through iterative questioning, originally developed by Sakichi Toyoda for Toyota Motor Corporation.

## When to Use

- Investigating recurring problems
- Debugging system failures
- Understanding customer churn
- Analyzing project delays or budget overruns
- Post-mortem analysis
- Process improvement initiatives

## The Method

```
Problem Statement
    ↓
Why? → Answer 1
    ↓
Why? → Answer 2
    ↓
Why? → Answer 3
    ↓
Why? → Answer 4
    ↓
Why? → Answer 5
    ↓
Root Cause Identified
    ↓
Solution Implementation
```

## Key Principles

| Principle | Description |
|-----------|-------------|
| **Facts over assumptions** | Base questions on data, not guesses |
| **Systems over individuals** | Focus on process failures, not blame |
| **Iterative depth** | Each "why" should dig deeper |
| **Actionable root cause** | Must lead to concrete solutions |

## Step-by-Step Process

### 1. Define the Problem
Be specific and measurable:

```
❌ Bad: "The website is slow"
✅ Good: "Page load time increased from 2s to 8s over the past week"
```

### 2. Ask the First Why
Focus on the immediate cause:

```
Problem: Page load time increased from 2s to 8s
Why 1: Because database queries are taking longer
```

### 3. Continue Asking Why
Each answer should explain the previous why:

```
Why 2: Because we added new indexes but didn't analyze query performance
Why 3: Because the deployment process doesn't include performance testing
Why 4: Because the team doesn't have performance monitoring tools
Why 5: Because performance monitoring wasn't prioritized in the tech budget
```

### 4. Identify Root Cause
The final why should reveal a systemic issue:

**Root Cause:** Performance monitoring tools and processes are not integrated into the development workflow

### 5. Implement Solutions
Address the root cause, not symptoms:

```
✅ Solutions:
- Add performance monitoring to CI/CD pipeline
- Budget for monitoring tools
- Include performance testing in definition of done
- Train team on performance optimization

❌ Wrong approach (treating symptoms):
- Just optimize the slow queries
- Add more servers
```

## Example: Customer Churn

### Problem
"Customer churn increased by 30% in Q3"

### Five Whys Analysis
```
Why 1: Customers are canceling after the first month
Why 2: Because they're not finding value in the product quickly
Why 3: Because our onboarding process is unclear and overwhelming
Why 4: Because we added too many features without improving guidance
Why 5: Because product decisions were made without user feedback loops
```

### Root Cause & Solution
**Root Cause:** Product development lacks user feedback integration

**Solutions:**
- Implement regular user feedback sessions
- Create a simplified onboarding flow
- Add progress tracking for new users
- Establish user success metrics

## Common Pitfalls

### ❌ Don't Stop Too Early
```
Problem: Bug in production
Why 1: Code wasn't tested properly
Why 2: Developer was rushed
❌ Stop here - blame individual
```

### ✅ Continue to System Level
```
Why 3: Because sprint planning doesn't include testing time
Why 4: Because velocity is measured over quality
Why 5: Because performance metrics don't track bug rates
```

### ❌ Don't Ask "Why" About People
```
❌ "Why did the developer make this mistake?"
✅ "Why did the process allow this mistake to reach production?"
```

### ❌ Don't Use Vague Answers
```
❌ "Why did it fail?" → "Because of technical issues"
✅ "Why did it fail?" → "Because the API timeout was set to 1s but average response time is 3s"
```

## Tips for Effective Five Whys

### 1. Involve the Right People
- Include people close to the problem
- Get different perspectives (engineering, product, support)
- Avoid blame by focusing on systems

### 2. Use Data When Possible
```
❌ "The system is slow"
✅ "Database query time increased from 100ms to 2s"
```

### 3. Verify Each Answer
Before moving to the next "why", confirm:
- Is this answer accurate?
- Do we have evidence?
- Could there be other causes?

### 4. Look for Patterns
If the same root cause appears in multiple analyses, it's a systemic issue that needs priority attention.

## Template for Documentation

```
## Five Whys Analysis - [Date]

### Problem Statement
[Specific, measurable problem]

### Analysis
1. **Why 1:** [Answer]
2. **Why 2:** [Answer]
3. **Why 3:** [Answer]
4. **Why 4:** [Answer]
5. **Why 5:** [Answer]

### Root Cause
[Systemic issue identified]

### Action Items
- [ ] [Specific action 1] - Owner: [Name] - Due: [Date]
- [ ] [Specific action 2] - Owner: [Name] - Due: [Date]

### Success Metrics
- [ ] [How we'll measure success]
```

## When Five Whys Isn't Enough

For complex problems with multiple causes, consider:
- **Fishbone diagrams** for visual cause mapping
- **Hypothesis trees** for testing multiple theories
- **Systems thinking** for understanding feedback loops

Five Whys works best for linear cause-effect relationships. For complex systems, combine with other analysis methods.
