# Writing Guide

This guide governs all documentation output from the generate phase. It contains hard rules (always enforced) and tone profiles (selected during init).

## Hard Rules

These apply regardless of tone selection. Violations must be caught and corrected before output.

### No Code

- No code snippets, inline code, or code blocks in final documentation
- No syntax-highlighted blocks
- No file paths in the output (reference components by their class/function name only)
- Exception: configuration property names may appear in prose (e.g., "the max-retry-count setting") but never in code formatting

### Banned Phrases

Never use any of the following:

- "delve into"
- "it's important to note"
- "leverage"
- "utilize"
- "in order to"
- "it should be noted"
- "as previously mentioned"
- "comprehensive"
- "robust"
- "seamless"
- "ensure"
- "facilitate"
- "cutting-edge"
- "game-changing"
- "state-of-the-art"
- "best-in-class"

### No Hedging in Resolved Sections

When a dependency has been traced and resolved, describe it with certainty. Do not use:

- "might"
- "possibly"
- "could potentially"
- "appears to"
- "seems to"
- "likely"
- "probably"

Hedging is acceptable ONLY in the Unresolved Dependencies section where behavior genuinely has not been confirmed.

### Naming

- Use actual codebase names: "OrderCancellationService", not "the cancellation service"
- Use actual endpoint paths: "the /api/orders/{id}/cancel endpoint", not "the cancellation endpoint"
- Use actual event names: "OrderCancelledEvent", not "the cancellation event"
- First mention in a section uses the full name. Subsequent mentions in the same section may use a shortened form if unambiguous.

### Structure

- Document WHY before WHAT. Open each feature section with business context and motivation before describing mechanics.
- Error flows get equal treatment to happy paths. Do not relegate errors to footnotes or appendices.
- Every cross-service call must name both the source and target service explicitly.
- Dependency chains must be described in execution order, not alphabetical or arbitrary order.

### No Emojis

No emojis anywhere in the output. Not in headings, not in callouts, not in lists.

## Tone Profiles

### Professional (default)

Full sentences. Third person ("the system", "the service", not "you" or "we"). Section introductions that provide context before details. Paragraphs of 3-5 sentences. Formal but not stiff -- aim for technical writing, not academic writing.

Calibration paragraph:
> When a user submits a cancellation request, the OrderCancellationService validates the order's current state against the cancellation policy. The service checks three conditions: the order must exist, its status must be in a cancellable state (pending or confirmed, not shipped), and the cancellation window must not have expired. If validation passes, the service coordinates a refund through the payment service and persists the status change before publishing an OrderCancelledEvent for downstream consumers.

### Conversational

Second person ("you", "your"). Shorter paragraphs of 1-3 sentences. Rhetorical questions permitted. Contractions encouraged. Still technically accurate -- this is approachable, not dumbed down.

Calibration paragraph:
> When you hit the cancel endpoint, OrderCancellationService checks three things: does the order exist, can it still be cancelled (only pending or confirmed orders, not shipped ones), and is the cancellation window still open? If everything checks out, it kicks off a refund through the payment service, saves the new status, and fires an OrderCancelledEvent so other services know what happened.

### Terse

No section introductions. Bullet points and fragments are acceptable. Minimal connective tissue. Every sentence carries information -- no filler. Tables preferred over prose where data is structured.

Calibration paragraph:
> OrderCancellationService validates: order exists, status is cancellable (pending/confirmed), cancellation window open. On success: initiates refund via payment service, persists cancelled status, publishes OrderCancelledEvent. On failure: returns specific error (not found, already shipped, window expired).
