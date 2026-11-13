---
status: accepted
---

# Track agent progress with `agent:*` labels layered under `ready-for-agent`, not with assignees or `needs-info`

The loop needs to know which tickets are claimed, parked, in review or failed. Matt Pocock's triage vocabulary requires exactly one state role per issue (`needs-triage`, `needs-info`, `ready-for-agent`, `ready-for-human`, `wontfix`). We add a second axis, `agent:working`, `agent:waiting`, `agent:in-review` and `agent:failed`, and a ticket keeps `ready-for-agent` for its whole life under the agent. The frontier is `ready-for-agent` with no `agent:*` label; the state role always wins, so removing `ready-for-agent` stops the agent regardless of any `agent:*` label.

## Considered options

- **Assignee as the claim** (the `wayfinder` convention). Rejected for v1: the agent runs under the maintainer's own GitHub login, so "assigned to me" cannot distinguish the agent from the human.
- **Reuse `needs-info` for parked tickets.** Rejected: the `triage` skill moves `needs-info` back to `needs-triage` on any reply, which would pull the ticket out of the agent's queue.
- **A second label axis.** Chosen: it leaves Matt's state machine intact, is visible in GitHub's UI without opening anything, and makes the frontier query label-only.

## Consequences

Retrying a failed ticket means removing `agent:failed`, not re-adding `ready-for-agent`. Label strings are fixed names with environment-variable overrides, and the `ready-for-agent` string is resolved from the repo's `docs/agents/triage-labels.md` when present.
