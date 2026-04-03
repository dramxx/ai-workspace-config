---
name: hypothesis-tree
description: Structure complex questions into testable hypotheses. Use when validating product ideas, debugging problems, planning experiments, or breaking down ambiguous challenges into actionable research.
---

# Hypothesis Tree - Structured Problem Decomposition

A Hypothesis Tree is a structured approach to breaking down complex questions into testable hypotheses. Originally from management consulting (McKinsey), it ensures MECE (Mutually Exclusive, Collectively Exhaustive) coverage of a problem space.

## When to Use

- Validating new product or feature ideas
- Investigating why metrics are underperforming
- Planning user research or experiments
- Breaking down ambiguous strategic questions
- Prioritizing what to test first
- Communicating analysis structure to stakeholders

## Structure of a Hypothesis Tree

```
                    Main Question
                    "Why is X happening?"
                          |
          +---------------+---------------+
          |               |               |
     Hypothesis A    Hypothesis B    Hypothesis C
          |               |               |
       +--+--+         +--+--+         +--+--+
       |     |         |     |         |     |
     Sub-   Sub-     Sub-   Sub-     Sub-   Sub-
     hyp    hyp      hyp    hyp      hyp    hyp
```

## MECE Principle

**Mutually Exclusive**: No overlap between branches
**Collectively Exhaustive**: All possibilities covered

```
Good MECE:                    Bad (not MECE):
+----------------+            +----------------+
| Revenue        |            | Revenue        |
+----------------+            +----------------+
| New customers  |            | New customers  |
| Existing       |            | Marketing      |
+----------------+            +----------------+
```

## Step-by-Step Process

### 1. Define the Core Question
Make it specific and measurable:

```
❌ Bad: "How can we grow?"
✅ Good: "How can we increase monthly active users by 20% in Q3?"
```

### 2. Brainstorm Hypotheses
Generate potential answers that could explain the question:

```
Question: "Why is user engagement declining?"
Hypotheses:
- Users don't find new features valuable
- Technical issues are frustrating users
- Competitors have better offerings
- Users don't understand how to use features
```

### 3. Organize into MECE Structure
Group hypotheses logically:

```
User Engagement Declining
├── Product Issues
│   ├── Features don't solve user problems
│   ├── User experience is confusing
│   └── Performance/bugs frustrate users
├── Market Factors
│   ├── Competitors have better features
│   ├── User needs have changed
│   └── Market saturation
└── User Understanding
    ├── Onboarding is ineffective
    ├── Feature discovery is poor
    └── Value proposition unclear
```

### 4. Develop Sub-Hypotheses
Break each hypothesis into testable statements:

```
Features don't solve user problems
├── Users don't need collaborative editing
├── Reporting features don't match workflows
└── Integration with existing tools is missing
```

### 5. Prioritize by Impact and Testability
Rank hypotheses by:
- **Potential impact** on the main question
- **Ease of testing** (data availability, resources)
- **Speed of validation**

## Example: Declining Conversion Rate

### Main Question
"Why has our e-commerce conversion rate dropped from 3% to 2%?"

### Hypothesis Tree

```
Conversion Rate Decline (3% → 2%)
├── Product Issues
│   ├── Product quality has declined
│   │   ├── More defective items
│   │   ├── Materials changed to cheaper
│   │   └── Manufacturing issues
│   ├── Product selection is less appealing
│   │   ├── Popular items out of stock
│   │   ├── New items don't match trends
│   │   └── Price increases on key items
│   └── Product information is inadequate
│       ├── Poor photos/descriptions
│       ├── Missing size/spec details
│       └── No customer reviews visible
├── Website/Technical Issues
│   ├── Site performance problems
│   │   ├── Slower page load times
│   │   ├── Mobile experience degraded
│   │   └── Checkout process issues
│   ├── Search/browse functionality
│   │   ├── Search results less relevant
│   │   ├── Filters not working properly
│   │   └── Navigation confusing
│   └── Trust signals reduced
│       ├── Security badges removed
│       ├── Return policy changed
│       └── Customer support less visible
└── External Factors
    ├── Competitive landscape
    │   ├── Competitors lowered prices
    │   ├── New competitors entered market
    │   └── Competitors improved UX
    ├── Market conditions
    │   ├── Economic downturn
    │   ├── Seasonal trends
    │   └── Industry-wide decline
    └── Customer expectations
        ├── Free shipping standards changed
        ├── Delivery speed expectations
        └── Social proof importance
```

## Testing the Hypotheses

### 1. Data Collection
For each hypothesis, identify needed data:

```
Hypothesis: "Site performance problems"
Data needed:
- Page load times by device
- Mobile conversion rates
- Checkout funnel drop-off points
- Error rates in analytics
```

### 2. Validation Methods
Choose appropriate testing methods:

| Hypothesis Type | Validation Method |
|-----------------|-------------------|
| User behavior | Analytics, user testing |
| Technical issues | Performance monitoring, error tracking |
| Market factors | Competitor analysis, market research |
| Product quality | Customer feedback, return data |

### 3. Prioritization Matrix

```
Impact: High/Low  |  Testability: High/Low
--------------------------------------------
High/High: Test immediately
High/Low: Invest in testing capability
Low/High: Quick wins, test if resources allow
Low/Low: Deprioritize or skip
```

## Common Pitfalls

### ❌ Non-MECE Structure
```
Bad: Overlapping categories
├── Marketing issues
├── Social media problems  (overlaps with marketing)
├── Email campaigns        (overlaps with marketing)
```

### ✅ MECE Structure
```
Good: Clear separation
├── Acquisition issues
│   ├── Marketing channels
│   ├── Social media
│   └── Email campaigns
├── Conversion issues
│   ├── Website experience
│   ├── Product presentation
│   └── Checkout process
```

### ❌ Too Broad Hypotheses
```
❌ "Marketing is ineffective"
✅ "Facebook ads have 50% higher CPA than industry average"
```

### ❌ Untestable Hypotheses
```
❌ "Users don't like our brand"
✅ "Brand sentiment score decreased from 8.2 to 6.5 in Q3"
```

## Template for Documentation

```
## Hypothesis Tree - [Project/Question]

### Main Question
[Specific, measurable question]

### Hypothesis Structure
[Tree diagram or nested list]

### Testing Plan
| Hypothesis | Priority | Data Needed | Test Method | Owner | Due Date |
|------------|----------|-------------|-------------|-------|----------|
| [Hypothesis 1] | [High/Med/Low] | [Data] | [Method] | [Name] | [Date] |

### Validation Results
| Hypothesis | Result | Confidence | Next Steps |
|------------|--------|-------------|------------|
| [Hypothesis 1] | [Supported/Rejected] | [%] | [Action] |
```

## Advanced Techniques

### 1. Hypothesis Scoring
Rate each hypothesis on:
- **Likelihood** (how likely it's true)
- **Impact** (if true, how much it explains)
- **Testability** (how easy to validate)

### 2. Iterative Refinement
Start with a broad tree, then:
- Test high-impact hypotheses first
- Refine tree based on early results
- Add new hypotheses as you learn

### 3. Cross-Functional Input
Include perspectives from:
- **Product**: User needs and value proposition
- **Engineering**: Technical constraints and capabilities
- **Marketing**: Acquisition and positioning
- **Support**: Customer feedback and pain points

This ensures comprehensive coverage and buy-in for the analysis.
