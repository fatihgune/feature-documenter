---
name: discover
description: Scan repositories and catalogue all features found. Use when discovering features in a microservice codebase for documentation.
disable-model-invocation: true
argument-hint: "[--search <query>] [repo-name]"
---

You are running the feature-documenter discover skill. This is Phase 0: feature discovery.

## Reference: Service Detection Patterns

!cat ${CLAUDE_SKILL_DIR}/../run/references/service-detection.md

## Reference: State File Format

!cat ${CLAUDE_SKILL_DIR}/../run/references/state-file-format.md

## Instructions

### Step 1: Validate Setup

Read `.feature-docs/_settings.md`. If it does not exist, tell the user: "No configuration found. Run `/feature-documenter:init` first." and stop.

Read `.feature-docs/_repos.md` to get the repository registry.

### Step 2: Parse Arguments

Check `$ARGUMENTS` for:
- `--search <query>`: targeted search mode
- A repo name: limit scan to that specific repo
- Nothing: full scan of all repos

### Step 3: Scan for Features

#### Targeted Search Mode (--search)

If `--search <query>` was provided:
1. Use Grep to search across all registered repos for the query term in file names, class names, route paths, and method names.
2. For each match, identify the feature context: what route/endpoint does this belong to? What controller/handler?
3. Group matches into logical features.

#### Full Scan Mode

For each repository in `_repos.md`:

1. Identify the tech stack from the registry.
2. Using the service detection patterns reference, find all route/endpoint definitions:
   - Use Glob to find controller/route files matching the patterns for that tech stack
   - Use Grep to extract route definitions (HTTP method + path) from those files
   - Use Grep to find the handler function/method name for each route
3. Group endpoints into logical features. Grouping heuristics:
   - Same controller/router file = same feature area
   - Shared URL prefix (e.g., /api/orders/*) = same feature area
   - Use the controller/resource name as the feature name (e.g., OrderController -> "Order Management")
4. For each feature group, identify entry points (method, path, handler, file).

#### Single Repo Mode

Same as full scan but limited to the specified repo.

### Step 4: Present Results

After scanning, present the discovered features to the user using AskUserQuestion:

"Discovered {N} features across {M} repositories:

| Feature | Source Repo | Entry Points | Example Endpoints |
|-|-|-|-|
| Order Management | order-service | 5 | GET /api/orders, POST /api/orders, ... |
| User Authentication | user-service | 3 | POST /api/auth/login, POST /api/auth/register, ... |
| ... | ... | ... | ... |

Which features should I catalogue? Enter:
- 'all' to catalogue everything
- Comma-separated numbers to select specific features
- 'skip N' to exclude specific features
- Or describe how you'd like to adjust the grouping"

Iterate with the user until they confirm a set of features.

### Step 5: Create State Files

For each confirmed feature:

1. Generate a slug from the feature name (lowercase, hyphens, no special characters).
2. Create `.feature-docs/_state/<slug>.md` with:
   - Frontmatter: feature name, slug, source_repo, today's date, phase=discovered, status=complete
   - Entry Points section populated with the discovered endpoints
   - All other sections present but with "None identified." placeholder
   - Trace Log with initial entry: "[{date} {time}] discover: Feature discovered with N entry points in {repo}"
   - Next Steps with: "Trace feature execution paths"

### Step 6: Update Index

Read `.feature-docs/index.md` and replace its contents with an updated feature table:

```markdown
---
feature_count: {count}
last_updated: {today's date}
---

# Feature Index

| Feature | Slug | Source Repo | Phase | Entry Points |
|-|-|-|-|-|
| Order Management | order-management | order-service | discovered | 5 |
| ... | ... | ... | ... | ... |
```

Preserve any existing features that were already in the index (from prior discovery runs). Update counts for features that were re-discovered.

### Step 7: Report

Tell the user:

"Catalogued {N} features. State files created in `.feature-docs/_state/`.

To trace a specific feature: `/feature-documenter:trace <feature-name>`
To continue with the orchestrator: `/feature-documenter:run`"
