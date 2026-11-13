---
status: accepted
---

# Agent files carry only `name` and `description` so one file serves both platforms

Claude Code and Antigravity CLI both load `agents/<name>.md`, but their `tools:` names and `model:` values are incompatible (`Read`/`Bash` and `sonnet` versus `view_file`/`run_command` and `flash`/`pro`). We keep a single shared `agents/` folder and restrict every agent's frontmatter to the two fields both platforms require.

## Consequences

Tool restrictions (for example, the reviewer must not edit files) live in the agent's prose instead of being enforced by the runtime. Model choice is a runtime flag on the driver, not a plugin setting. If a future agent genuinely needs enforced restrictions, it will need one file per platform and this decision should be revisited.
