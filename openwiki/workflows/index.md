# Files

- [Run Context and Prompt Construction](context-engineering.md) - How source events and durable thread state become attributed input messages, configuration-controlled prompt layers, repository instructions, skills, participant context, and reviewer or analyzer context.
- [Follow-ups, Interrupts, and Completion Delivery](follow-up-messages.md) - How existing Open SWE threads accept new work through durable runs or the in-run store queue, including handoff, cancellation, completion recovery, and sandbox continuity.
- [Invocation from Dashboard, Webhooks, and Schedules](invocation.md) - How dashboard commands, signed GitHub, Slack, and Linear webhooks, and schedules are admitted, attributed, routed to a thread, converted to structured input, and dispatched as durable LangGraph runs.
- [Code Delivery and Pull Request Creation](pr-creation.md) - How a coding agent delivers prepared sandbox changes through a guarded GitHub push and attributed pull request, then records, monitors, and follows up on the delivery.
- [Pull Request Review Workflow](pr-review.md) - How GitHub events, dashboard actions, and reviewer tools run pull-request reviews through eligibility, durable state, finding publication, re-review, reply reassessment, and check settlement.
- [Scheduled Work, CI Monitoring, and Baby-sit](scheduling-and-baby-sit.md) - How deterministic scheduler ticks launch recurring automation, recovery and workspace work, deferred enrichment, background-task follow-ups, and opt-in pull-request CI watches.
