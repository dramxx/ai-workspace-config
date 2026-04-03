---
name: code-architecture
description: Guides when to abstract vs duplicate code. Use this skill when creating shared utilities, deciding between DRY/WET approaches, or refactoring existing abstractions.
---

# Code Architecture: Avoiding Wrong Abstractions

## Core Principle

**Prefer duplication over the wrong abstraction. Wait for patterns to emerge before abstracting.**

Premature abstraction creates confusing, hard-to-maintain code. Duplication is far cheaper to fix than unwinding a wrong abstraction.

## The Rule of Three

Don't abstract until code appears in **at least 3 places**. This provides enough context to identify genuine patterns vs coincidental similarities.

```jsx
// ✅ Correct: Wait for the pattern to emerge
// First occurrence - just write it
const userTotal = items.reduce((sum, item) => sum + item.price, 0);

// Second occurrence - still duplicate
const cartTotal = products.reduce((sum, p) => sum + p.price, 0);

// Third occurrence - NOW consider abstraction
const calculateTotal = (items, priceKey = 'price') =>
  items.reduce((sum, item) => sum + item[priceKey], 0);
```

## When to Abstract

### ✅ Abstract When

- Same code appears in **3+ places**
- Pattern has **stabilized** (requirements are clear)
- Abstraction **simplifies** understanding
- Use cases share **identical behavior**, not just similar structure

### ❌ Don't Abstract When

- Code only appears in 1-2 places
- Requirements are still evolving
- Use cases need **different behaviors** (even if structure looks similar)
- Abstraction would require parameters/conditionals for variations

## The Wrong Abstraction Pattern

This is how wrong abstractions evolve:

```jsx
// ❌ Wrong: Premature abstraction with parameters
function fetchData({ type, id, params, transform }) {
  if (type === 'user') {
    return fetchUser(id, params).then(transform);
  }
  if (type === 'post') {
    return fetchPost(id, params).then(transform);
  }
  // More conditionals...
}

// ✅ Better: Separate, clear functions
function fetchUser(id, params) { /* ... */ }
function fetchPost(id, params) { /* ... */ }
```

## Good Abstraction Principles

### 1. Single Responsibility
Each abstraction should do one thing well.

```jsx
// ✅ Good: Focused abstraction
const calculateTotal = (items, priceKey = 'price') =>
  items.reduce((sum, item) => sum + item[priceKey], 0);

// ❌ Bad: Multiple responsibilities
const processOrder = (order, applyTax, applyShipping, formatCurrency) => {
  // Too many concerns
};
```

### 2. Clear Naming
The name should exactly describe what it does.

```jsx
// ✅ Good: Clear intent
const calculateSubtotal = (items) => items.reduce((sum, item) => sum + item.price, 0);
const applyTax = (subtotal, taxRate) => subtotal * (1 + taxRate);

// ❌ Bad: Vague naming
const process = (items, rate) => { /* ... */ };
```

### 3. Minimal Parameters
Fewer parameters mean simpler abstraction.

```jsx
// ✅ Good: Essential parameters only
const formatDate = (date) => date.toLocaleDateString();

// ❌ Bad: Too many options
const formatDate = (date, locale, format, timezone, includeTime, use12Hour) => { /* ... */ };
```

## Refactoring Wrong Abstractions

When you encounter a wrong abstraction:

1. **Identify the concrete use cases**
2. **Extract specific implementations**
3. **Delete the generic abstraction**
4. **Let new patterns emerge naturally**

```jsx
// Before: Wrong abstraction
function renderList({ data, type, renderItem, loading, error }) {
  if (loading) return <Loading />;
  if (error) return <Error message={error} />;
  
  return (
    <div>
      {type === 'users' && data.map(renderItem)}
      {type === 'posts' && data.map(renderItem)}
    </div>
  );
}

// After: Specific, clear components
function UserList({ users }) {
  return <div>{users.map(user => <UserItem key={user.id} user={user} />)}</div>;
}

function PostList({ posts }) {
  return <div>{posts.map(post => <PostItem key={post.id} post={post} />)}</div>;
}
```

## Testing Abstractions

Good abstractions are easy to test:

```jsx
// ✅ Good: Simple, testable
const calculateTotal = (items, priceKey = 'price') =>
  items.reduce((sum, item) => sum + item[priceKey], 0);

// Test: Simple and clear
expect(calculateTotal([{ price: 10 }, { price: 20 }])).toBe(30);
```

Wrong abstractions are hard to test due to complexity and conditionals.

## Summary

- Wait for the Rule of Three
- Prefer duplication over wrong abstraction
- Keep abstractions focused and simple
- Let patterns emerge from real usage
- Delete wrong abstractions aggressively
