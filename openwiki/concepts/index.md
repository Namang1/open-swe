# Files

- [Identity, Authorization, and Credential Boundaries](auth-and-security.md) - Authentication and authorization boundaries for dashboard users, inbound integrations, GitHub credentials, repository workspaces, and sandboxed execution.
- [Model, Profile, and Instruction Resolution](models-profiles-instructions.md) - Explains how workspace defaults, profiles, thread snapshots, explicit run selections, adaptive routing, and provider settings determine an agent run. Covers persistence boundaries and the repository, workspace, user, and AGENTS.md instruction layers.
- [Threads, Runs, and Durable State](threads-and-state.md) - How Open SWE gives conversations a durable identity, creates checkpointed LangGraph runs, propagates configuration and metadata, and retains Store and sandbox state across product surfaces.
- [Tool Surface and Capability Gating](tools.md) - How Open SWE selects static, sandbox, personal, MCP, and administrative tool capabilities for each agent run. Covers source and mode filtering, deferred integration loading, credential boundaries, and runtime safety guards.
