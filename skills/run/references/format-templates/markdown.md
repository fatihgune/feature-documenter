# Markdown (GFM) Format Template

Use standard GitHub-Flavored Markdown. This template defines the section structure for the final feature document.

## Document Structure

```
# {Feature Name}

## Overview

One paragraph summarizing what this feature does from a business perspective. Who uses it, when, and why it exists.

## Business Context

Why this feature was built. What business problem it solves. What happens if it breaks. Who are the stakeholders.

## Feature Flow

Walk through the feature's execution path in order. One H3 subsection per entry point.

### {HTTP Method} {Path}

Describe the full flow from request to response. Cover:
- What triggers this path
- Validation and preconditions
- Core business logic steps
- Data that gets read or written
- Side effects (events published, notifications sent)
- What the caller receives back

### {Additional Entry Points...}

## Data Flow

What data this feature reads, writes, and transforms. Cover:
- Primary entities involved and their key fields
- How data moves between services
- What gets persisted vs. what is transient
- Important state transitions

## Cross-Service Dependencies

One H3 per dependent service. For each:
- What this feature sends to the dependency
- What the dependency does with it
- What comes back
- What happens if the dependency is unavailable

### {Service Name}

## Error Handling and Edge Cases

Equal treatment to the happy path. Cover:
- Validation failures and their responses
- Downstream service failures and fallback behavior
- Race conditions or timing issues
- Retry behavior and idempotency

## Related Features

Brief mentions of features that interact with this one. What events does this feature produce that others consume? What shared data does it modify?
```

## Formatting Rules

- Use H1 for the feature name only
- Use H2 for major sections
- Use H3 for subsections (entry points, dependent services)
- Tables for structured data (entities, field lists, status mappings)
- Ordered lists for sequential flows
- Unordered lists for non-sequential items
- Bold for first mention of a component name in a section
- No code blocks, inline code, or syntax highlighting
