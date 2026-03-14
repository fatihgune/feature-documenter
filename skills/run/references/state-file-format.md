# State File Format

State files live at `.feature-docs/_state/<feature-slug>.md`. They are the communication layer between phases and sessions. Every skill reads and writes these files. Getting this format wrong breaks everything.

## Structure

Each state file has two parts: YAML frontmatter (machine-readable) and a markdown body (human/LLM-readable).

### Frontmatter

```yaml
---
feature: "Order Cancellation"
slug: "order-cancellation"
source_repo: "order-service"
discovered: "2026-03-14"
phase: discovered
status: complete
entry_points_count: 3
outbound_calls_resolved: 2
outbound_calls_unresolved: 1
dependency_depth: 2
---
```

**Field definitions:**

| Field | Type | Values | Set By |
|-|-|-|-|
| feature | string | Human-readable feature name | discover |
| slug | string | Kebab-case identifier | discover |
| source_repo | string | Primary repo where feature originates | discover |
| discovered | string | ISO date of discovery | discover |
| phase | enum | `discovered` / `traced` / `resolving` / `resolved` / `generated` | Each phase on completion |
| status | enum | `in-progress` / `blocked` / `complete` | Any phase |
| entry_points_count | integer | Number of entry points found | trace |
| outbound_calls_resolved | integer | Count of resolved outbound deps | resolve |
| outbound_calls_unresolved | integer | Count of unresolved outbound deps | resolve |
| dependency_depth | integer | How many levels deep deps were traced | resolve |

### Body Sections

The body uses H2 headings in strict order. All sections must exist even if empty (use "None identified." as placeholder). No section may be deleted once created.

## Section Definitions

### 1. Entry Points

Table format:

```
| Method | Path | Handler | File |
|-|-|-|-|
| POST | /api/orders/{id}/cancel | OrderController.cancelOrder | order-service/src/.../OrderController.java |
```

Populated by: discover (initial), trace (refined)

### 2. Service Layer Trace

Ordered flow of method calls from entry point through business logic. No code -- only class.method names and what each step does in plain language.

Format:
```
### POST /api/orders/{id}/cancel

1. OrderController.cancelOrder -- receives the cancellation request and validates the order ID format
2. OrderCancellationService.cancel -- orchestrates the full cancellation workflow
3. OrderRepository.findById -- retrieves the order entity
4. OrderCancellationService.validateCancellable -- checks order status, time window, and fulfillment state
5. PaymentService.initiateRefund -- calls the payment service to start the refund process (outbound)
6. OrderRepository.save -- persists the updated order with cancelled status
7. EventPublisher.publish -- emits OrderCancelledEvent to the event bus
```

Populated by: trace

### 3. Repository/Data Layer

Table format:

```
| Entity | Repository | Operations | Key Fields |
|-|-|-|-|
| Order | OrderRepository | findById, save | id, status, userId, createdAt, cancelledAt |
```

Populated by: trace

### 4. Outbound Calls

Table format:

```
| Target Service | Method | Endpoint | Status | Target Repo |
|-|-|-|-|-|
| payment-service | POST | /api/refunds | resolved | payment-service |
| notification-service | POST | /api/notifications/send | unresolved | -- |
```

Status values: `resolved` / `unresolved` / `not-in-registry`

Populated by: trace (initial), resolve (status updates)

### 5. Background Jobs and Events

Table format:

```
| Type | Name | Trigger | Handler |
|-|-|-|-|
| event-published | OrderCancelledEvent | Order cancellation completes | -- |
| event-consumed | RefundCompletedEvent | Payment service confirms refund | OrderRefundHandler.handle |
| scheduled-job | CancellationCleanup | Cron: daily 2am | CancellationCleanupJob.run |
```

Populated by: trace

### 6. Resolved Dependencies

One H3 subsection per resolved service. Each contains a mini-trace of the target endpoint.

Format:
```
### payment-service: POST /api/refunds

**Handler:** RefundController.createRefund

1. RefundController.createRefund -- validates refund request
2. RefundService.processRefund -- coordinates with payment gateway
3. RefundRepository.save -- persists refund record
4. EventPublisher.publish -- emits RefundInitiatedEvent

**Data:** Refund entity (id, orderId, amount, status, gatewayReference)

**Further outbound:** Calls payment-gateway (external, not traced)
```

Populated by: resolve

### 7. Unresolved Dependencies

List of what could not be resolved and why.

Format:
```
- notification-service POST /api/notifications/send -- not in repository registry
- analytics-service POST /api/events -- repo found but endpoint not located
```

Populated by: resolve

### 8. Trace Log

Append-only timestamped log. Every phase and subagent appends entries here. Never edit or delete existing entries.

Format:
```
- [2026-03-14 10:23] discover: Feature discovered with 3 entry points in order-service
- [2026-03-14 10:45] trace: Traced POST /api/orders/{id}/cancel -- 7 steps, 2 outbound calls
- [2026-03-14 10:47] trace: Traced GET /api/orders/{id}/cancel-status -- 3 steps, 0 outbound calls
- [2026-03-14 11:02] resolve: Resolved payment-service POST /api/refunds -- depth 1
- [2026-03-14 11:05] resolve: Could not resolve notification-service -- not in registry
```

Populated by: all phases (append only)

### 9. Next Steps

Remaining work items. Updated by each phase on completion.

Format:
```
- [ ] Trace remaining entry point: DELETE /api/orders/{id}/cancel
- [ ] Resolve notification-service dependency
- [ ] Generate documentation
```

Populated by: all phases

## Rules

1. Subagents ONLY append to the Trace Log section. They do not modify any other section directly -- they return data to the parent skill which updates sections.
2. Phase and status updates go in frontmatter only, never in body text.
3. No section may be deleted, even if empty.
4. The section order is fixed. Do not reorder sections.
5. When resuming, always read the full state file before making changes.
6. Frontmatter counts (entry_points_count, outbound_calls_resolved, etc.) must be kept in sync with body content.
