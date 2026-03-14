---
name: trace
description: Deep-trace a single feature's execution paths including entry points, service layers, repositories, jobs, events, and outbound calls. Use after feature discovery.
disable-model-invocation: true
argument-hint: "<feature-name>"
---

You are running the feature-documenter trace skill. This is Phase 1: deep execution path tracing.

## Reference: State File Format

!cat ${CLAUDE_SKILL_DIR}/../run/references/state-file-format.md

## Reference: Service Detection Patterns

!cat ${CLAUDE_SKILL_DIR}/../run/references/service-detection.md

## Instructions

### Step 1: Validate and Load

1. Read `.feature-docs/_settings.md`. If missing, tell user to run `/feature-documenter:init` and stop.
2. Read `.feature-docs/_repos.md` for the repository registry.
3. Read `.feature-docs/index.md` to find features.

### Step 2: Identify Target Feature

Parse `$ARGUMENTS` to identify the feature name. Match against `index.md`:
- Try exact slug match first
- Then case-insensitive partial match on feature name
- If ambiguous, show matches and ask user to clarify using AskUserQuestion
- If no match, tell user to run `/feature-documenter:discover` first and stop

### Step 3: Load State

Read `.feature-docs/_state/<slug>.md`. Verify:
- The state file exists
- `phase` is at least `discovered`
- If `phase` is already `traced` or later, ask user: "This feature has already been traced. Re-trace from scratch?" using AskUserQuestion. If no, stop.

### Step 4: Trace Each Entry Point

For each entry point in the state file's Entry Points table:

1. **Find the handler**: Use the file path and handler name from the entry point to read the handler code.

2. **Trace the service layer**: Follow the handler's execution:
   - Identify what service/class methods the handler calls
   - For each service method, read its implementation
   - Record the call chain: ClassName.methodName and a plain-language description of what each step does
   - Continue following calls until you reach repository/data layer or outbound calls
   - Do NOT record the actual code -- only the method names and their purpose

3. **Identify the data layer**: For each repository/data access call found:
   - Record the entity name, repository class, operations used, and key fields on the entity
   - This populates the Repository/Data Layer section

4. **Identify outbound calls**: Look for:
   - HTTP client calls (RestTemplate, WebClient, axios, fetch, httpx, etc.)
   - gRPC stub calls
   - Message broker publishes (Kafka, RabbitMQ, NATS, etc.)
   - Record: target service name (inferred from URL/topic/queue), HTTP method (if applicable), endpoint path, and mark status as "unresolved"

5. **Identify background jobs and events**: Look for:
   - Event publishing (what events are emitted, what triggers them)
   - Event consuming (what events are listened to, what handler processes them)
   - Scheduled jobs related to this feature
   - Async tasks triggered by the feature flow
   - Record: type (event-published/event-consumed/scheduled-job/async-task), name, trigger, handler

### Step 5: Update State File

Update `.feature-docs/_state/<slug>.md`:

1. **Service Layer Trace section**: Write the traced call chain for each entry point. Use the format from the state file format reference: one H3 per entry point, numbered steps with ClassName.methodName and description.

2. **Repository/Data Layer section**: Write the entity table.

3. **Outbound Calls section**: Write the outbound calls table. All statuses are "unresolved" at this phase.

4. **Background Jobs and Events section**: Write the jobs/events table.

5. **Trace Log**: Append entries for each traced entry point:
   `- [{date} {time}] trace: Traced {METHOD} {path} -- {N} steps, {M} outbound calls`

6. **Next Steps**: Update with remaining work:
   - If outbound calls exist: "Resolve cross-service dependencies"
   - If no outbound calls: "Generate documentation"

7. **Frontmatter**: Update:
   - `phase: traced`
   - `status: complete`
   - `entry_points_count: {actual count}`

### Step 6: Update Index

Read and update `.feature-docs/index.md`: change this feature's phase to `traced`.

### Step 7: Report

Tell the user:

"Traced {feature name}:
- {N} entry points traced
- {M} outbound calls to {K} services found
- {J} background jobs/events identified

Outbound dependencies: {list service names}

Next: run `/feature-documenter:resolve {feature-name}` to resolve cross-service dependencies."

If no outbound calls were found:
"Traced {feature name}:
- {N} entry points traced
- No cross-service dependencies found
- {J} background jobs/events identified

This feature is self-contained. Run `/feature-documenter:generate {feature-name}` to produce documentation. For best results, start a fresh Claude session first."
