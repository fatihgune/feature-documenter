---
name: resolve
description: Resolve cross-service dependencies for a traced feature. Matches outbound calls to local repositories and traces the target endpoints. Use after deep tracing.
disable-model-invocation: true
argument-hint: "<feature-name>"
---

You are running the feature-documenter resolve skill. This is Phase 2: cross-service dependency resolution. This is the most architecturally complex skill -- it matches outbound calls to repositories and traces target endpoints.

## Reference: State File Format

!cat ${CLAUDE_SKILL_DIR}/../run/references/state-file-format.md

## Instructions

### Step 1: Validate and Load

1. Read `.feature-docs/_settings.md`. If missing, tell user to run `/feature-documenter:init` and stop.
2. Read `.feature-docs/_repos.md` for the repository registry.
3. Read `.feature-docs/index.md` to find features.

### Step 2: Identify Target Feature

Parse `$ARGUMENTS` to identify the feature. Same matching logic as the trace skill:
- Exact slug match, then case-insensitive partial match
- If ambiguous, ask user to clarify
- If no match, stop

### Step 3: Load and Validate State

Read `.feature-docs/_state/<slug>.md`. Verify:
- `phase` is at least `traced`
- If `phase` is `resolved` or `generated`, ask: "Dependencies already resolved. Re-resolve?" using AskUserQuestion
- Parse the Outbound Calls table to get the list of unresolved dependencies

If there are no unresolved outbound calls, tell the user: "No unresolved dependencies. Run `/feature-documenter:generate {feature-name}` to produce documentation." and stop.

### Step 4: Match Dependencies to Repos

Read `_repos.md` to get the repository registry.

For each unresolved outbound call:

1. **Match by service name**: Compare the target service name from the outbound call against repository names in the registry. Try:
   - Exact match (target "payment-service" matches repo "payment-service")
   - Partial match (target "payment" matches repo "payment-service")
   - If the outbound call has a URL, extract the service name from it

2. **If no match found**: Mark as unresolved with reason "not in repository registry" in the Unresolved Dependencies section. Update the Outbound Calls table status to `not-in-registry`.

3. **If match found**: Proceed to Step 5 for this dependency.

### Step 5: Trace Resolved Dependencies

Read `_settings.md` to get `max_depth` and `confirm_each_level`.

For each matched dependency, use the Agent tool to spawn an Explore subagent. The subagent should:

**Subagent prompt template:**

"You are tracing a cross-service dependency for feature documentation. Final response under 2000 characters. List outcomes, not process.

Target: {repo_name} at {repo_path}
Tech stack: {tech_stack}
Looking for: {HTTP method} {endpoint_path}

Tasks:
1. Find the route/endpoint handler for {HTTP method} {endpoint_path} in the target repository
2. Trace its execution: handler -> service layer -> data layer
3. Record the call chain as: ClassName.methodName -- plain language description of what it does
4. Identify the entities/data this endpoint reads or writes
5. Identify any further outbound calls this endpoint makes (transitive dependencies)

Return:
- Handler: {ClassName.methodName}
- Trace (numbered steps with ClassName.methodName and description)
- Data: entities involved with key fields
- Further outbound calls: list of {target service, method, endpoint} or 'none'"

Use `subagent_type: "Explore"` for each subagent.

After each subagent returns:

1. Update the **Resolved Dependencies** section: add an H3 subsection for this service+endpoint with the trace data
2. Update the **Outbound Calls** table: set status to `resolved`, set target_repo
3. Append to **Trace Log**: `[{date} {time}] resolve: Resolved {service} {method} {path} -- depth {N}`

### Step 6: Handle Transitive Dependencies

If any resolved dependency revealed further outbound calls:

1. Check current depth against `max_depth` from settings
2. If depth >= max_depth: record transitive deps in Unresolved Dependencies with reason "max depth reached"
3. If depth < max_depth and `confirm_each_level` is true:
   - Ask user using AskUserQuestion: "Found {N} more dependencies at depth {D}: {list}. Continue resolving? (yes/no)"
   - If no: record as unresolved with reason "user skipped"
4. If depth < max_depth and `confirm_each_level` is false:
   - Continue resolving automatically
5. For continued resolution: add the transitive deps to the Outbound Calls table and loop back to Step 4

### Step 7: Update State File

Update `.feature-docs/_state/<slug>.md` frontmatter:
- `phase: resolved` (if all resolvable deps are resolved)
- `phase: resolving` (if stopped early due to depth limit or user choice)
- `status: complete`
- `outbound_calls_resolved: {count}`
- `outbound_calls_unresolved: {count}`
- `dependency_depth: {max depth reached}`

Update **Next Steps**:
- "Generate documentation"

Append to **Trace Log**:
- `[{date} {time}] resolve: Resolution complete. {resolved} resolved, {unresolved} unresolved.`

### Step 8: Update Index

Read and update `.feature-docs/index.md`: change this feature's phase to `resolved` (or `resolving`).

### Step 9: Report

Tell the user:

"Dependency resolution complete for {feature name}:
- {resolved} dependencies resolved
- {unresolved} dependencies unresolved
- Maximum depth reached: {depth}

{If unresolved > 0: list the unresolved deps and reasons}

All tracing is complete. Your feature state file contains everything needed for documentation generation. **Start a fresh Claude session and run `/feature-documenter:generate {feature-name}`** for the best output quality -- a clean context window produces significantly better documentation."
