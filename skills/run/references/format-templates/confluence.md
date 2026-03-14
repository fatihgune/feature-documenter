# Confluence Format Template

Confluence uses wiki markup macros. This template includes macro placeholders that render correctly when pasted into Confluence's markdown-to-wiki converter or imported directly.

## Document Structure

```
# {Feature Name}

{toc:minLevel=2|maxLevel=3}

{info:title=What this feature does}
One-sentence summary for scanning.
{info}

## Overview

One paragraph summarizing what this feature does from a business perspective. Who uses it, when, and why it exists.

## Business Context

{info:title=Key Context}
Key business context that engineers should understand before reading further.
{info}

Why this feature was built. What business problem it solves. What happens if it breaks. Who are the stakeholders.

## Feature Flow

Walk through the feature's execution path in order. One expand block per entry point.

{expand:title={HTTP Method} {Path} -- one-line summary}
Describe the full flow from request to response. Cover:
- What triggers this path
- Validation and preconditions
- Core business logic steps
- Data that gets read or written
- Side effects (events published, notifications sent)
- What the caller receives back
{expand}

{expand:title={Additional Entry Points...}}
...
{expand}

## Data Flow

What data this feature reads, writes, and transforms.

| Entity | Key Fields | Operations | Notes |
|-|-|-|-|
| {EntityName} | {fields} | {read/write/update} | {context} |

## Cross-Service Dependencies

One expand block per dependent service.

{expand:title={Service Name} -- one-line summary of the dependency}
- What this feature sends to the dependency
- What the dependency does with it
- What comes back
- What happens if the dependency is unavailable
{expand}

{status:colour=Green|title=Resolved}  or  {status:colour=Red|title=Unresolved}

Use status macros inline next to each dependency name to indicate resolution state.

## Error Handling and Edge Cases

{warning:title=Critical Failure Mode}
Highlight the most critical failure mode here.
{warning}

Cover:
- Validation failures and their responses
- Downstream service failures and fallback behavior
- Race conditions or timing issues
- Retry behavior and idempotency

## Related Features

Brief mentions of features that interact with this one. What events does this feature produce that others consume? What shared data does it modify?
```

## Macro Reference

| Macro | Purpose | Usage |
|-|-|-|
| `{toc}` | Table of contents | Top of document, once |
| `{info:title=X}...{info}` | Blue info panel | Business context, summaries |
| `{warning:title=X}...{warning}` | Yellow warning panel | Critical failure modes, caveats |
| `{expand:title=X}...{expand}` | Collapsible section | Entry points, dependencies |
| `{status:colour=X\|title=Y}` | Colored status lozenge | Dependency resolution state |

## Formatting Rules

- Use `{toc}` at the top of every document
- Use `{expand}` blocks for sections readers may want to collapse
- Use `{info}` and `{warning}` macros for callouts
- Use `{status}` macros for dependency resolution state (Green=Resolved, Red=Unresolved, Yellow=Partial)
- Tables for structured data
- Bold for first mention of a component name in a section
- No code blocks, inline code, or syntax highlighting
