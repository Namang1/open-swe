---
type: contributor guide
title: Open SWE Codebase Guide
description: A task-oriented route from local setup and runtime entrypoints to the source, focused tests, and detailed guides that own a safe Open SWE change.
tags: [open-swe, contributor-guide, development, langgraph, testing]
verified:
  - by: openwiki/0.4.2
    at: 2026-09-24T08:16:16.200Z
sources:
  - id: openwiki-source-328bde9e94017848bb09ba23
    resource: repo://agent/api/app.py
  - id: openwiki-source-921ec88ab63280d28b3dddb5
    resource: repo://agent/chat.py
  - id: openwiki-source-c48b309c5ca416cf623f0866
    resource: repo://agent/dispatch.py
  - id: openwiki-source-f8665996049065d2172f68e2
    resource: repo://agent/graphs/agent.py
  - id: openwiki-source-f2c7a9cbc0f7af0b4db77658
    resource: repo://agent/graphs/analyzer.py
  - id: openwiki-source-368e3a3da2c40119aead4316
    resource: repo://agent/graphs/chat.py
  - id: openwiki-source-6edf3a3d0424db652805727f
    resource: repo://agent/graphs/review_scout.py
  - id: openwiki-source-73db7609f2a24f4a0ff5c32c
    resource: repo://agent/graphs/reviewer.py
  - id: openwiki-source-1116ea2d477f08cf0f5b2ef0
    resource: repo://agent/graphs/scheduler.py
  - id: openwiki-source-276ab38291eb5741b4c2141c
    resource: repo://agent/reviewer.py
  - id: openwiki-source-6fd11c8bb15f5eb94b765440
    resource: repo://agent/sandboxes/lifecycle.py
  - id: openwiki-source-3e15117ace082a39e1f130d8
    resource: repo://agent/scheduler.py
  - id: openwiki-source-856ade03ef31ac38e1347f7c
    resource: repo://agent/server.py
  - id: openwiki-source-3096620cfd0eb1bae6d9e78c
    resource: repo://agent/webapp.py
  - id: openwiki-source-8037e2358a2c4f9b2c722a11
    resource: repo://AGENTS.md
  - id: openwiki-source-19973c87ca458faa5d03fecc
    resource: repo://docs/DEVELOPMENT.md
  - id: openwiki-source-5bbba7b2a8ea8360ff233d63
    resource: repo://langgraph.json
  - id: openwiki-source-012f2c78e3b1446dfc35803f
    resource: repo://Makefile
  - id: openwiki-source-5b54a58d1b51cd490b0e7162
    resource: repo://package.json
  - id: openwiki-source-05ccef8d4cf1698187f20464
    resource: repo://pyproject.toml
  - id: openwiki-source-23775c3de52f3ab95a13cb8b
    resource: repo://README.md
  - id: openwiki-source-f0a6e7dc03522b2682f88655
    resource: repo://tests/conftest.py
  - id: openwiki-source-859f98720585f4648f0f7b2e
    resource: repo://tests/e2e/playwright.config.ts
  - id: openwiki-source-4b944ec14a3d793a6f771403
    resource: repo://tests/e2e/playwright.desktop.config.ts
  - id: openwiki-source-7ef60dc4372e1a33c7728fe6
    resource: repo://tests/e2e/README.md
generated: { by: "openwiki/0.4.2", at: "2026-09-24T08:16:16.200Z" }
---

# Open SWE Codebase Guide

Open SWE is an asynchronous software factory built on Deep Agents and LangGraph. Dashboard, GitHub, Slack, Linear, and scheduled work can create durable agent work; coding threads use isolated sandboxes, while pull-request review and review-style analysis are separate concerns. This is a navigation page, not a substitute for implementation reading: **source code and focused tests are authoritative**. Start with the owning code and its nearest tests, then use the linked pages for cross-cutting context.

## Start the appropriate local loop

The Python backend requires Python 3.14+ and uses `uv`; `ui`, `desktop`, and `tests/e2e` are a pnpm workspace. For a first local environment, follow [the development guide](../docs/DEVELOPMENT.md), including its PostgreSQL, credentials, dashboard, and webhook-tunnel requirements.

```bash
make install            # uv sync --extra dev
make dev                # LangGraph graphs and FastAPI on :2024
make run                # FastAPI only on :8000
make dev-ui             # Vite plus the backend fronting it
make web                # dashboard Vite server
make desktop            # Electron wrapper; backend runs separately
```

Use `make dev` for any change that creates or executes LangGraph runs. It starts the local PostgreSQL container unless `POSTGRES_URI` is set and serves the registered graphs and FastAPI app. `make run` is useful for FastAPI-only work, but has no LangGraph runtime, so run-creating flows cannot work there. For dashboard changes, `make dev-ui` keeps the browser on `:2024` while the backend fronts Vite on `:3000`.

For local GitHub or Slack webhooks, use the documented tunnel setup. `make tunnel NGROK_DOMAIN=<name>.ngrok-free.dev` exposes only `/webhooks/*`; do not expose the unauthenticated local LangGraph API through a whole-port tunnel.

### Contributor guardrails

- Keep implementations async-only. If an interface requires a synchronous method, make that method raise `NotImplementedError` rather than maintaining two implementations.
- Use strong Python and TypeScript types; do not widen values to `Any` or `any` to suppress a type error.
- Put model-facing prompts in `agent/resources/prompts/` and load or render them rather than inlining them in Python.
- Put a dashboard endpoint in its feature package; `agent/dashboard/routes.py` aggregates feature routers under `/dashboard/api` rather than owning feature behavior.
- Never discard an exception silently: propagate it or log it with structured context.

## Locate the runtime owner

`langgraph.json` is the deployable registration boundary. It selects Python 3.14, registers six graph entrypoints, attaches `agent.webapp:app`, and configures delete-based checkpoint cleanup (a 60-minute sweep and a 43,200-minute default TTL). The graph modules in `agent/graphs/` are public, thin re-export shims; normally change the owning implementation instead of the shim.

| Entrypoint | Owning implementation | Use it when changing… |
| --- | --- | --- |
| `agent.graphs.agent:traced_agent` | `agent/server.py` | coding-agent assembly, prompts, models, tools, middleware, or coding sandbox preparation |
| `agent.graphs.reviewer:traced_reviewer_agent` | `agent/reviewer.py` | diff-based findings, review publication, or reviewer sandbox behavior |
| `agent.graphs.analyzer:traced_analyzer` | `agent/analyzer.py` | repository review-style analysis and guidance |
| `agent.graphs.review_scout:traced_review_scout` | `agent/review_scout/graph.py` | historical review scouting and walkthrough inputs |
| `agent.graphs.chat:traced_chat_agent` | `agent/chat.py` | dashboard PR chat and its read-only context/tools |
| `agent.graphs.scheduler:get_scheduler` | `agent/scheduler.py` | cron routing, scheduled work, CI watches, refreshes, or background tasks |
| `agent.webapp:app` | `agent/api/app.py` | dashboard/API composition, health, webhook ingress, CORS, or static UI mounting |

```mermaid
flowchart LR
    Surface["Dashboard and webhook surfaces"] --> Dispatch["dispatch_agent_run"]
    Dispatch --> Durable["Durable LangGraph run"]
    Durable --> Graph["Agent or reviewer graph"]
    Cron["Cron tick"] --> Scheduler["Scheduler graph"]
    Scheduler --> Durable
```

This diagram shows the main run-routing boundary: interactive surfaces dispatch agent or reviewer runs, while scheduler ticks either perform maintenance work or launch scheduled agent work.

### Boundaries that prevent unsafe changes

- The coding-agent factory is stateless; per-thread continuity belongs to LangGraph thread metadata and the sandbox. Do not add state to a long-lived graph instance to solve a thread problem.
- A main-agent sandbox that is merely unreachable is not replaced automatically, because a replacement could discard uncommitted work. The reviewer may replace one because it recreates its checkout for every review.
- The reviewer is non-mutating: its review-specific tools manage findings and publication, not commits, pushes, or pull-request creation. PR chat has no sandbox; it reads seeded `/pr/` virtual files and repository data through a repository-scoped GitHub App token, while excluding shell and file-mutation tools.
- Dashboard, Slack, Linear, and GitHub callers use `dispatch_agent_run` to create durable `agent` or `reviewer` runs. Its normal strategy interrupts competing work on the thread, and a caller must use either a prebuilt input or content/context/source identities, never both.
- FastAPI validates allowlists, sandbox configuration, local-development model configuration, and database setup during startup; it includes dashboard, plan, workflow-approval, Linear, Slack, health, GitHub, and sandbox-tool routes. Credentialed CORS is enabled only for configured dashboard origins and rejects `*`.

## Pick the task guide

### Change an agent, review, state, or tool capability

- [Runtime Architecture](architecture/overview.md) — deployment boundaries, graph registration, FastAPI composition, and product surfaces.
- [Coding Agent Assembly](architecture/agent-graph.md) — agent factory, model/profile resolution, prompt context, tools, and sandbox attachment.
- [Agent Middleware and Failure Boundaries](architecture/middleware-stack.md) — ordering-sensitive retries, queue delivery, timeouts, fallbacks, guards, and tool errors.
- [Thread Sandbox Lifecycle](architecture/sandbox-lifecycle.md) and [Sandbox Provider Integration](integrations/sandbox-providers.md) — ownership, recovery policy, snapshots, proxy credentials, and provider extensions.
- [Review, PR Chat, and Style Learning Graphs](architecture/reviewer-and-analyzer.md) — reviewer, analyzer, scout, findings, and read-only boundaries.
- [Threads, Runs, and Durable State](concepts/threads-and-state.md) — thread identity, checkpoint/store state, metadata, and queued messages.
- [Tool Surface and Capability Gating](concepts/tools.md) — static and dynamic tools, MCP, approvals, and authorization.
- [Model, Profile, and Instruction Resolution](concepts/models-profiles-instructions.md) — settings precedence, providers, and layered instructions.

### Change ingress, UI, integration, or delivery behavior

- [Invocation from Dashboard, Webhooks, and Schedules](workflows/invocation.md) — admission, identity/context construction, thread selection, and dispatch.
- [Follow-ups, Interrupts, and Completion Delivery](workflows/follow-up-messages.md) — continuing, stopping, and delivering work on an existing thread.
- [Run Context and Prompt Construction](workflows/context-engineering.md) — conversion from event/state into model messages and specialized context.
- [Code Delivery and Pull Request Creation](workflows/pr-creation.md) — commits, pushes, approval gates, PR creation, and CI hooks.
- [Pull Request Review Workflow](workflows/pr-review.md) — automatic/manual review, findings, publication, and replies.
- [Scheduled Work, CI Monitoring, and Baby-sit](workflows/scheduling-and-baby-sit.md) — schedule persistence, watches, reconciliation, and safe follow-up dispatch.
- [Dashboard, Web UI, and Desktop Clients](integrations/dashboard-ui.md) — browser/Electron behavior, APIs, streams, and approvals.
- [Identity, Authorization, and Credential Boundaries](concepts/auth-and-security.md) — sessions, webhook verification, repository access, and credentials.
- [Observability, External Tools, and MCP](integrations/observability-and-mcp.md) — tracing, connected services, optional tool loading, and failure handling.

### Change configuration or deployment

- [Configuration and Workspace Administration](operations/configuration.md) — environment registry, startup validation, workspace routing, settings, and credentials.
- [Local Development, Packaging, and Deployment](operations/deployment.md) — local topology, static UI builds, Docker/LangGraph deployment, and desktop packaging.

## Validate the changed boundary

**Do not run the full local test suite.** Choose the smallest deterministic test that owns the observable behavior, then add the relevant static check. `make test` accepts a test file or directory; use a direct pytest node id for one behavior.

```bash
make test TEST_FILE=tests/github/test_open_pull_request.py
uv run pytest -vvv tests/path/to_test.py::test_name
make lint
make format-check
make typecheck
```

Pytest runs async tests automatically. Shared fixtures route `agent.store` through an in-memory implementation while preserving its serialization path, reset process-global TTL and sandbox registries around tests, and enable automatic review by default; a test of those gates must override the fixture deliberately. Tests requiring PostgreSQL use a migrated, throwaway schema only when `TEST_ANALYTICS_POSTGRES_URI` is configured, otherwise they skip.

Use [Testing Strategy and Focused Validation](testing/overview.md) to select the nearest unit/integration family for an agent, sandbox, webhook, dashboard, review, or tool change. Add tests only for observable behavior and meaningful edge cases—not source-structure, constant, prompt-text, or incidental-call-order checks.

Use Playwright only when a real cross-boundary contract requires it. The harness runs real agent code, local sandbox, local git, dashboard, and Electron paths, while faking the model and external SaaS HTTP boundaries. Install Chromium once, then select a single relevant spec; browser runs are serial and the desktop configuration selects only the Electron spec.

```bash
pnpm install --frozen-lockfile
pnpm run test:e2e:install
pnpm exec playwright test tests/full_flow.spec.ts
```

See [Testing Strategy and Focused Validation](testing/overview.md) for E2E fakes, artifacts, and targeted frontend or desktop commands.
