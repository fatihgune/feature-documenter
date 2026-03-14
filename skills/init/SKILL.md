---
name: init
description: Set up feature-documenter preferences including documentation format, tone, tech stack detection, and repository registry. Use when setting up feature documentation for the first time or reconfiguring settings.
disable-model-invocation: true
---

You are running the feature-documenter init skill. This is an interactive onboarding flow that configures user preferences and discovers repositories.

## Reference: Service Detection Patterns

!cat ${CLAUDE_SKILL_DIR}/../run/references/service-detection.md

## Instructions

Follow these steps exactly. Do not skip steps. Ask questions one at a time using AskUserQuestion.

### Step 1: Check for Existing Configuration

Check if `.feature-docs/_settings.md` exists by reading it.

- If it exists: show the user their current settings and ask "Reconfigure from scratch or keep existing settings?" using AskUserQuestion. If they want to keep, exit with "Settings unchanged."
- If it does not exist: continue to step 2.

### Step 2: Documentation Format

Ask the user using AskUserQuestion:

"Which documentation format do you want to generate?
1. Markdown (GitHub-Flavored Markdown)
2. Notion (Markdown with toggle blocks and callouts)
3. Confluence (Wiki markup with macros)

Enter 1, 2, or 3:"

Store their choice as `format`: `markdown`, `notion`, or `confluence`.

### Step 3: Writing Tone

Ask the user using AskUserQuestion:

"Which writing tone?
1. Professional -- full sentences, third person, section introductions
2. Conversational -- second person, shorter paragraphs, approachable
3. Terse -- bullet points, fragments, no filler

Enter 1, 2, or 3:"

Store their choice as `tone`: `professional`, `conversational`, or `terse`.

### Step 4: Repository Discovery

Run `ls` on the parent directory of the current working directory. For each sibling directory (and the current directory itself), check for project marker files:

- `pom.xml` or `build.gradle` or `build.gradle.kts` -> Java/Kotlin (likely Spring Boot)
- `package.json` -> Node.js (check for express, @nestjs/core, or other frameworks)
- `go.mod` -> Go
- `requirements.txt` or `pyproject.toml` or `Pipfile` -> Python (check for django, fastapi, flask)
- `*.csproj` or `*.sln` -> .NET
- `Gemfile` -> Ruby (check for rails)
- `Cargo.toml` -> Rust

For Node.js repos, read `package.json` dependencies to distinguish Express vs NestJS vs other.
For Python repos, check dependency files to distinguish Django vs FastAPI vs other.
For Ruby repos, check Gemfile for rails.

Present the results to the user as a table using AskUserQuestion:

"I found these repositories:

| Repository | Path | Tech Stack |
|-|-|-|
| order-service | ../order-service | Spring Boot (Java) |
| user-service | ../user-service | NestJS (Node.js) |
| ... | ... | ... |

Confirm this list, or tell me what to add, remove, or change. You can also add repositories from other locations by providing their full path."

Iterate with the user until they confirm the list.

### Step 5: Dependency Settings

Ask the user using AskUserQuestion:

"Dependency resolution settings:
- Maximum dependency depth (how many levels deep to trace cross-service calls). Default: 3
- Ask for confirmation at each depth level? Default: yes

Enter as: depth,ask (e.g., '3,yes' or '5,no') or press enter for defaults:"

Parse their response. Store as `max_depth` (integer) and `confirm_each_level` (boolean).

### Step 6: Write Configuration Files

Create the `.feature-docs/` directory structure and write all configuration files.

**Write `.feature-docs/_settings.md`:**

```markdown
---
format: {format}
tone: {tone}
max_depth: {max_depth}
confirm_each_level: {confirm_each_level}
created: {today's date}
---

# Feature Documenter Settings

## Documentation Format
{format}

## Writing Tone
{tone}

## Dependency Resolution
- Maximum depth: {max_depth}
- Confirm at each level: {confirm_each_level}
```

**Write `.feature-docs/_repos.md`:**

```markdown
---
repo_count: {count}
last_updated: {today's date}
---

# Repository Registry

| Repository | Path | Tech Stack | Status |
|-|-|-|-|
| {name} | {path} | {stack} | active |
```

**Create directories:**
- `.feature-docs/_state/`
- `.feature-docs/features/`

**Write `.feature-docs/index.md`:**

```markdown
---
feature_count: 0
last_updated: {today's date}
---

# Feature Index

No features discovered yet. Run `/feature-documenter:discover` to scan repositories.
```

### Step 7: Confirm

Tell the user:

"Setup complete. Created:
- `.feature-docs/_settings.md` -- your preferences
- `.feature-docs/_repos.md` -- {N} repositories registered
- `.feature-docs/index.md` -- feature index (empty)

Next: run `/feature-documenter:discover` to scan your repositories for features."
