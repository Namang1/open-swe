# Files

- [Coding Agent Assembly](agent-graph.md) - How the primary coding Deep Agent is assembled for an executable LangGraph thread run, from configuration and model policy through sandbox, tools, skills, run preparation, and middleware safety gates.
- [Agent Middleware and Failure Boundaries](middleware-stack.md) - Ordering-sensitive middleware around coding-agent and reviewer model and tool loops. Covers preparation, dynamic capabilities, normalization, queued input, deadlines, retries, guards, and the boundary between model-visible and terminal failures.
- [Runtime Architecture](overview.md) - Deployable LangGraph graph entrypoints, FastAPI ingress, durable run dispatch, sandbox lifecycle, and the cloud and desktop dashboard surfaces.
- [Review, PR Chat, and Style Learning Graphs](reviewer-and-analyzer.md) - Read-only PR review, PR chat, review-scout, and repository-style analysis graphs. Explains their distinct context, state, GitHub authority, sandbox boundaries, and the feedback loop from human review outcomes to future reviews.
- [Thread Sandbox Lifecycle](sandbox-lifecycle.md) - Explains how agent threads bind to persistent sandboxes, choose and bootstrap providers, refresh GitHub proxy credentials, and recover safely without losing coding work.
