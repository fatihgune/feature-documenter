---
name: run
description: Main entry point for feature-documenter. Checks existing state and resumes where you left off. Use to start or continue feature documentation work.
disable-model-invocation: true
argument-hint: "[--search <query>] [--fresh] [--auto] [feature-name]"
---

You are running the feature-documenter run skill. This is the main orchestrator that checks state and routes to the appropriate phase.

## Reference: State File Format

!cat ${CLAUDE_SKILL_DIR}/../run/references/state-file-format.md

## Instructions

### Step 1: Parse Flags

Parse `$ARGUMENTS` for:
- `--fresh` : reset everything
- `--auto` : skip inter-phase confirmations (except before generate)
- `--search <query>` : search for a feature
- Remaining text after flags: treated as a feature name

### Step 2: Handle --fresh

If `--fresh` is present:
1. Ask user for confirmation using AskUserQuestion: "This will delete all feature documentation state in `.feature-docs/`. Are you sure? (yes/no)"
2. If confirmed: delete the `.feature-docs/` directory using Bash (`rm -rf .feature-docs/`)
3. Tell user: "Cleared. Run `/feature-documenter:init` to start fresh."
4. Stop.

### Step 3: Check Configuration

Read `.feature-docs/_settings.md`.

If it does not exist, tell user: "No configuration found. Run `/feature-documenter:init` to set up." and stop.

### Step 4: Handle --search

If `--search <query>` is present:
1. Read `.feature-docs/index.md`
2. Search for the query in feature names and slugs (case-insensitive partial match)
3. If matches found:
   - Show the matching features with their current phase
   - For each match, show what the next action would be based on phase:
     - `discovered` -> "Run `/feature-documenter:trace {slug}` to trace"
     - `traced` -> "Run `/feature-documenter:resolve {slug}` to resolve dependencies"
     - `resolving` -> "Run `/feature-documenter:resolve {slug}` to continue resolution"
     - `resolved` -> "Run `/feature-documenter:generate {slug}` to generate docs (fresh session recommended)"
     - `generated` -> "Documentation complete at `.feature-docs/features/{slug}.md`"
4. If no matches:
   - Tell user: "No existing features match '{query}'. Run `/feature-documenter:discover --search {query}` to search repositories."
5. Stop.

### Step 5: Handle Specific Feature

If a feature name was provided (after removing flags):
1. Read `.feature-docs/index.md`
2. Match the feature name (exact slug, then partial name match)
3. If found:
   - Read `.feature-docs/_state/<slug>.md`
   - Show current state: feature name, phase, status, key stats
   - Determine next action based on phase (same routing as Step 4)
   - If `--auto` is set and next action is NOT generate: invoke the next skill automatically using the Skill tool
   - If `--auto` is set and next action IS generate: tell user "Auto mode pauses before generation. Start a fresh session and run `/feature-documenter:generate {slug}` for best results."
   - If `--auto` is not set: tell user what to run next
4. If not found:
   - Tell user: "Feature '{name}' not found. Available features:" and list from index
5. Stop.

### Step 6: No Feature Specified (Dashboard Mode)

If no feature name and no --search:
1. Read `.feature-docs/index.md`
2. If index is empty or has no features:
   - Tell user: "No features discovered yet. Run `/feature-documenter:discover` to scan your repositories."
   - Stop.
3. If features exist, show a dashboard:

   "Feature Documentation Status:

   | Feature | Source Repo | Phase | Next Action |
   |-|-|-|-|
   | {name} | {repo} | {phase} | {next action description} |
   | ... | ... | ... | ... |

   {Summary: N total, X discovered, Y traced, Z resolved, W generated}

   Which feature would you like to work on? Enter a feature name, or:
   - `/feature-documenter:discover` to find more features
   - `/feature-documenter:trace <feature>` to trace a specific feature
   - `/feature-documenter:resolve <feature>` to resolve dependencies
   - `/feature-documenter:generate <feature>` to generate docs"

4. Wait for user response using AskUserQuestion.
5. If user names a feature:
   - Route to the appropriate next skill based on phase
   - If `--auto` and next is not generate: invoke automatically
   - Otherwise: tell user what to run

### Phase Transition Notes

At every transition point:
- If the next phase is `:generate`, always say: "For best output quality, start a fresh Claude session before running `/feature-documenter:generate {slug}`."
- If `--auto` is set and next is not generate, invoke the Skill tool with the appropriate skill name and arguments. For example: use the Skill tool with skill "feature-documenter:trace" and args "{slug}".
- If `--auto` is not set, tell the user what command to run next and stop.
