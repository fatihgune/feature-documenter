# Notion Format Template

Notion imports markdown with some extensions. This template uses `<details>` for toggle blocks and blockquotes with special prefixes for callouts. Structure mirrors the standard template with Notion-specific formatting.

## Document Structure

```
# {Feature Name}

> **What this feature does:** One-sentence summary for scanning.

## Overview

One paragraph summarizing what this feature does from a business perspective. Who uses it, when, and why it exists.

## Business Context

> [!NOTE]
> Key business context that engineers should understand before reading further.

Why this feature was built. What business problem it solves. What happens if it breaks. Who are the stakeholders.

## Feature Flow

Walk through the feature's execution path in order. One toggle block per entry point.

<details>
<summary>{HTTP Method} {Path} -- one-line summary of what this endpoint does</summary>

Describe the full flow from request to response. Cover:
- What triggers this path
- Validation and preconditions
- Core business logic steps
- Data that gets read or written
- Side effects (events published, notifications sent)
- What the caller receives back

</details>

<details>
<summary>{Additional Entry Points...}</summary>

...

</details>

## Data Flow

What data this feature reads, writes, and transforms.

<details>
<summary>Entities and their relationships</summary>

- Primary entities involved and their key fields
- How data moves between services
- What gets persisted vs. what is transient
- Important state transitions

</details>

## Cross-Service Dependencies

One toggle per dependent service.

<details>
<summary>{Service Name} -- one-line summary of the dependency</summary>

- What this feature sends to the dependency
- What the dependency does with it
- What comes back
- What happens if the dependency is unavailable

</details>

## Error Handling and Edge Cases

> [!WARNING]
> Highlight the most critical failure mode here.

Cover:
- Validation failures and their responses
- Downstream service failures and fallback behavior
- Race conditions or timing issues
- Retry behavior and idempotency

## Related Features

Brief mentions of features that interact with this one. What events does this feature produce that others consume? What shared data does it modify?
```

## Companion Database Schema

Suggest to the user that they create a Notion database alongside the documents with these properties:

| Property | Type | Purpose |
|-|-|-|
| Feature Name | Title | Primary identifier |
| Status | Select: Discovered / Traced / Resolved / Documented | Current documentation state |
| Source Service | Select | Primary repository |
| Dependencies | Multi-select | Services this feature calls |
| Entry Points | Number | Count of API endpoints |
| Last Updated | Date | When documentation was last generated |

## Formatting Rules

- Use `<details><summary>` for any section a reader might want to collapse
- Use blockquote callouts (`> [!NOTE]`, `> [!WARNING]`, `> [!TIP]`) for important context
- Keep toggle summaries to one line with a brief description after a double dash
- Tables for structured data
- Bold for first mention of a component name in a section
- No code blocks, inline code, or syntax highlighting
