<p align="center">
  <img src="assets/logo.svg" alt="feature-documenter" width="520">
</p>

<p align="center">
  <a href="LICENSE"><img src="https://img.shields.io/badge/License-MIT-blue.svg" alt="License: MIT"></a>
  <a href="https://docs.anthropic.com/en/docs/claude-code"><img src="https://img.shields.io/badge/Claude_Code-Plugin-blueviolet" alt="Claude Code Plugin"></a>
  <a href="https://github.com/fatihgune/feature-documenter/releases"><img src="https://img.shields.io/github/v/release/fatihgune/feature-documenter?label=Release&color=brightgreen" alt="GitHub Release"></a>
  <a href="https://github.com/fatihgune/feature-documenter/stargazers"><img src="https://img.shields.io/github/stars/fatihgune/feature-documenter?style=flat&label=Stars&color=yellow" alt="GitHub Stars"></a>
  <a href="https://github.com/fatihgune/feature-documenter/issues"><img src="https://img.shields.io/github/issues/fatihgune/feature-documenter?label=Issues" alt="GitHub Issues"></a>
  <a href="https://github.com/fatihgune/feature-documenter/commits/main"><img src="https://img.shields.io/github/last-commit/fatihgune/feature-documenter?label=Last%20Commit" alt="Last Commit"></a>
  <img src="https://img.shields.io/badge/Tech_Stacks-8-orange" alt="8 Tech Stacks">
  <img src="https://img.shields.io/badge/Eval_Assertions-166-teal" alt="159 Eval Assertions">
</p>

---

**Turn undocumented microservices into clear, readable feature documentation -- without writing a single line of it yourself.**

You just joined a team. There are 15 microservice repos. No documentation. The onboarding doc says "read the code." You open the order service, find a controller, follow it into a service class, hit a REST call to another repo, open *that* repo, find the handler, follow it into *its* service class, and now you're three repos deep and you've forgotten what the original endpoint even did.

feature-documenter does that tracing for you. It scans your repos, discovers features, follows execution paths across service boundaries, and writes documentation that explains what the system does and why -- in plain language, with no code.

## What it produces

Given an order-service with a cancellation endpoint, the plugin generates documentation like this:

> **Order Cancellation** validates three preconditions before proceeding: the order must exist, its status must be cancellable (pending or confirmed, not shipped), and the cancellation window must still be open. OrderCancellationService coordinates the full workflow -- it retrieves the order, validates cancellability, initiates a refund through the payment service, persists the cancelled status, and publishes an OrderCancelledEvent for downstream consumers like notification and analytics.

No code snippets. No file paths. Just business logic that any engineer can read and understand.

Output formats: **Markdown** (GFM), **Notion** (with toggle blocks and callouts), or **Confluence** (with wiki macros).

## Who is this for

- Engineers joining a team and trying to understand how things work
- Teams that need documentation but no one has time to write it
- Architects mapping cross-service data flows and dependencies
- Anyone doing knowledge transfer, incident review, or onboarding prep

## Prerequisites

- [Claude Code](https://docs.anthropic.com/en/docs/claude-code) installed and configured

## Installation

**From GitHub:**
```
claude plugin install fatihgune/feature-documenter
```

**From a local clone:**
```
git clone https://github.com/fatihgune/feature-documenter.git
claude --plugin-dir ./feature-documenter
```

## Getting started

Navigate to your project directory (ideally the parent of your microservice repos) and follow these five steps. Each one is a separate slash command, and the plugin remembers where you left off between sessions.

### 1. Set up

```
/feature-documenter:init
```

Interactive setup that walks you through:
- Picking a documentation format (Markdown, Notion, or Confluence)
- Picking a writing tone (Professional, Conversational, or Terse)
- Auto-detecting repositories in sibling directories and their tech stacks
- Setting how deep to trace cross-service dependencies

### 2. Discover features

```
/feature-documenter:discover
```

Scans all registered repositories for API endpoints and groups them into logical features (e.g., "Order Management", "User Authentication"). You review the list and pick which features to document.

### 3. Trace a feature

```
/feature-documenter:trace order-cancellation
```

Follows the feature's execution path end to end: entry points, service layer calls, data access patterns, outbound calls to other services, events published, and background jobs triggered.

### 4. Resolve cross-service dependencies

```
/feature-documenter:resolve order-cancellation
```

For each outbound call found during tracing, the plugin navigates into the target repository and traces what happens on the receiving end. It handles transitive dependencies too -- if service A calls service B which calls service C, it follows the whole chain (up to a configurable depth).

### 5. Generate documentation

Start a fresh Claude session for the best output quality, then:

```
/feature-documenter:generate order-cancellation
```

Produces the final document at `.feature-docs/features/order-cancellation.md`, ready to paste into your wiki.

## Commands

| Command | Purpose |
|-|-|
| `/feature-documenter:run` | Main entry point. Shows status dashboard, resumes where you left off |
| `/feature-documenter:init` | One-time setup: format, tone, repos, depth settings |
| `/feature-documenter:discover` | Scan repos and catalogue features |
| `/feature-documenter:trace <feature>` | Deep-trace a feature's execution paths |
| `/feature-documenter:resolve <feature>` | Resolve cross-service dependencies |
| `/feature-documenter:generate <feature>` | Generate the final documentation |

### Flags (via `:run`)

| Flag | Effect |
|-|-|
| `--search <query>` | Find a feature by name or search repos for a term |
| `--fresh` | Delete all state and start over (asks for confirmation) |
| `--auto` | Run phases back-to-back without pausing (except before generate) |

## How it works

### Output structure

The plugin creates a `.feature-docs/` directory in your working directory:

```
.feature-docs/
  _settings.md            # Your preferences (format, tone, depth)
  _repos.md               # Registered repositories and their tech stacks
  _state/
    order-cancellation.md  # Trace state -- call chains, deps, progress
    user-auth.md           # Each feature gets its own state file
  features/
    order-cancellation.md  # Final documentation (the deliverable)
    user-auth.md
  index.md                 # Master list of all features and their status
```

### Resumability

Everything is stored in state files. Close your terminal mid-trace, come back tomorrow, run `/feature-documenter:run` -- it picks up exactly where you left off. Each feature progresses through phases independently:

```
discovered -> traced -> resolving -> resolved -> generated
```

### Supported tech stacks

The plugin knows how to find routes, services, and outbound calls in:

- **Spring Boot** (Java/Kotlin) -- controllers, `@FeignClient`, Kafka/Rabbit listeners
- **Express** (Node.js) -- router files, axios/fetch calls, Bull queues
- **NestJS** (Node.js) -- decorated controllers, `ClientProxy`, `@Cron` jobs
- **Django** (Python) -- URL conf, views/viewsets, Celery tasks, signals
- **FastAPI** (Python) -- decorated routes, `httpx`, `BackgroundTasks`
- **Go** (net/http, Gin, Echo, Chi, gRPC) -- handler funcs, HTTP clients, NATS/Kafka
- **Rails** (Ruby) -- `routes.rb`, service objects, ActiveJob, Sidekiq
- **ASP.NET Core** (.NET) -- attributed controllers, `HttpClient`, MassTransit, Hangfire

## Configuration

Settings live in `.feature-docs/_settings.md` and can be changed by re-running `/feature-documenter:init`.

| Setting | Options | Default |
|-|-|-|
| Format | markdown, notion, confluence | chosen during init |
| Tone | professional, conversational, terse | chosen during init |
| Max dependency depth | 1-10 | 3 |
| Confirm at each depth level | yes / no | yes |

## Contributing

Contributions are welcome. Here are some ways to help:

- **Report bugs**: Open an issue at [github.com/fatihgune/feature-documenter/issues](https://github.com/fatihgune/feature-documenter/issues)
- **Add a tech stack**: The detection patterns live in `skills/run/references/service-detection.md`. Add an H2 section following the existing format.
- **Improve writing quality**: The writing guide is at `skills/run/references/writing-guide.md`. If you see the plugin producing awkward phrasing or missing edge cases, improve the rules.
- **Add eval cases**: The `evals/` directory has test scenarios for each skill. More scenarios = better quality.
- **Add a format template**: Templates are in `skills/run/references/format-templates/`. If you use a wiki system we don't support, add a template for it.

### Evals

The `evals/` directory contains 39 test scenarios with assertions that verify each skill produces correct output:

| File | Scenarios | What it tests |
|-|-|-|
| `generate.json` | 14 | Writing guide compliance, format templates, tone profiles, no-code enforcement |
| `discover.json` | 5 | Feature grouping, state file structure, search mode |
| `trace.json` | 5 | Call chain completeness, outbound call detection, background jobs |
| `resolve.json` | 5 | Repository matching, depth limits, transitive dependencies |
| `init.json` | 3 | File creation, tech stack detection, config handling |
| `run.json` | 7 | Dashboard mode, search, flag behavior |

Each assertion is typed as `quality` (substantive correctness) or `format` (structural requirements). If you change a skill, run its evals to make sure nothing regressed.

## Related Concepts

If you've been searching for any of the following, this plugin is what you're looking for:

- Automatic microservice documentation generator
- Cross-service dependency mapper and tracer
- API documentation from source code without code exposure
- Feature-level documentation for distributed systems
- Business logic extractor for microservice architectures
- Confluence / Notion documentation generator from codebase
- Onboarding documentation tool for engineering teams
- Service mesh / call graph documentation
- Claude Code plugin for code documentation
- Reverse engineering microservice behavior into docs

**GitHub Topics:** `claude-code-plugin` `documentation-generator` `microservices` `cross-service-tracing` `feature-documentation` `confluence` `notion` `developer-tools` `onboarding` `code-analysis`

## License

[MIT](LICENSE)
