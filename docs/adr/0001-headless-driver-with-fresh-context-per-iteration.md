---
status: accepted
---

# Drive the loop from a bash script that runs one headless iteration per ticket

The loop must run unattended on both Claude Code and Antigravity CLI. The obvious choice was the Ralph Loop plugin's Stop hook, which keeps re-feeding a prompt inside one session. We chose instead a bash driver (`scripts/loop.sh`) that computes the backlog state deterministically with `gh`, then runs one headless `-p` invocation per ticket, sleeps, or exits.

## Considered options

- **In-session Stop hook (Ralph Loop plugin style).** Rejected: the hook file, schema and stdin differ between the two platforms, so it needs two implementations; one context grows until compaction; and a hook cannot sleep for five minutes.
- **Platform schedulers** (Claude Code `/loop` and cron routines, Antigravity sidecars). Rejected: none is portable, and Antigravity documents sidecars as App-only.
- **Bash driver with headless iterations.** Chosen: it is the original Ralph shape, it is the only portable option, every ticket starts with a fresh context, and the decision to sleep or exit costs no tokens because the script makes it.

## Consequences

The model never decides to wait. The driver owns picking, claiming, worktree setup and notifications; the skill owns understanding, asking, implementing, reviewing and opening the PR. Headless mode means a denied tool call fails silently, so the driver defaults to the platform's skip-permissions flag and the README tells users to run it in an isolated environment.
