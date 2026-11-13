# Spec: dev-workflow v1, the ticket loop

Vocabulary follows `GLOSSARY.md`. Decisions reference `docs/adr/`.

## Problem Statement

A maintainer who has already broken work into GitHub tickets (with acceptance criteria, `ready-for-agent` labels and native blocking edges, as produced by Matt Pocock's `to-tickets`) still has to start an agent by hand for every ticket, watch it, answer its questions live, and start the next one. The agent cannot run overnight because the first open question stops everything, and when it is blocked on one ticket it does not move to another.

## Solution

A plugin, installable on Claude Code and Antigravity CLI, whose driver works the backlog unattended. Each iteration takes one ticket from the frontier in a fresh context, implements it test-first, has a fresh-context reviewer check it, and opens a pull request for the maintainer to merge. When the agent needs an answer it posts one batched question on the ticket, parks it, and moves on to the next ticket. When nothing is workable but something is parked or blocked by an unmerged pull request, the driver sleeps and re-checks every five minutes. When the backlog has nothing left, it exits. The maintainer's only duties are answering questions on GitHub and merging pull requests; a notify command tells them when either is waiting.

## User Stories

**Maintainer**

1. As a maintainer, I want to start the driver from my repository with one command, so that the backlog gets worked while I do something else.
2. As a maintainer, I want the driver to pick the next unblocked `ready-for-agent` ticket by itself, so that I never have to choose what the agent works on.
3. As a maintainer, I want the agent to ask me all its questions about a ticket in a single comment, so that I answer once instead of in a drip.
4. As a maintainer, I want to answer by replying on the GitHub issue, so that the conversation lives next to the ticket.
5. As a maintainer, I want the agent to resume a parked ticket automatically after I reply, so that I never have to touch labels to get it going again.
6. As a maintainer, I want the agent to move to another ticket while it waits for my answer, so that one question does not stall the whole backlog.
7. As a maintainer, I want the driver to sleep and re-check every five minutes when everything is parked or blocked, so that it continues on its own once I reply or merge.
8. As a maintainer, I want the driver to exit when nothing is ready, parked or blocked, so that it does not poll GitHub forever.
9. As a maintainer, I want one pull request per ticket that closes the issue on merge, so that my merge is the quality gate and the state of the backlog follows from GitHub itself.
10. As a maintainer, I want the agent never to merge its own pull requests or push to the default branch, so that nothing lands without my review.
11. As a maintainer, I want dependent tickets to wait until their blocker's issue is closed, so that no pull request builds on unreviewed work.
12. As a maintainer, I want to be notified through a command of my choosing when a ticket is parked, failed or has a pull request, so that I learn about it without watching the terminal.
13. As a maintainer, I want to see every ticket's agent state from its labels alone, so that a GitHub label filter is my dashboard.
14. As a maintainer, I want the agent to give up on a ticket it cannot finish, explain what it tried, and move on, so that one hard ticket does not burn my budget.
15. As a maintainer, I want to retry a failed ticket by removing one label, so that recovery is a click.
16. As a maintainer, I want to take a ticket away from the agent by changing its state label, so that I stay in control mid-flight.
17. As a maintainer, I want the agent to state the assumptions it made in the pull request body, so that I can correct them at review instead of being asked mid-way.
18. As a maintainer, I want every agent comment to be visibly marked as agent-written, so that I can tell them apart from my own comments under the same login.
19. As a maintainer, I want caps on iterations and idle time, so that an overnight run has a bounded cost.
20. As a maintainer, I want the agent to work in a throwaway worktree per ticket, so that my own checkout and uncommitted changes are never touched.
21. As a maintainer, I want all work-in-progress pushed before a ticket is parked or failed, so that nothing is lost if the machine or session dies.
22. As a maintainer, I want the agent to read my repository's agent docs, glossary and ADRs, so that its code follows my conventions without extra configuration.

**Plugin user (another developer)**

23. As a Claude Code user, I want to install the plugin from a marketplace with two commands, so that setup takes a minute.
24. As an Antigravity CLI user, I want to install the same plugin from a local clone, so that I am not forced onto a different tool.
25. As a plugin user, I want the driver to pick my installed runtime automatically when only one is present, and to require an explicit choice when both are, so that it never spends the wrong budget.
26. As a plugin user, I want the driver to default to skip-permissions in a sandbox with a clear warning, and to offer a safe allowlist mode, so that I can choose my risk level.
27. As a plugin user, I want the plugin to use the issue tracker and label vocabulary my repo already configured through `/setup-matt-pocock-skills`, so that there is no second configuration.
28. As a plugin user whose repo uses different label names, I want to override the agent labels with environment variables, so that the plugin fits my existing vocabulary.
29. As a plugin user, I want the plugin's own labels created automatically on first run, so that there is no setup step for them.
30. As a plugin user, I want the README to tell me the prerequisites, the label state machine, the safety model and what v1 does not do, so that I know what I am pointing at my repository.
31. As a plugin user, I want a dry-run mode that shows what the driver would pick and do without running an agent, so that I can trust it before spending anything.

## Implementation Decisions

### Platforms and layout

- Target Claude Code and Antigravity CLI from a single shared core: the skills, agents and scripts live once at the repo root. Two thin manifests, one under `.claude-plugin/` and one under `.antigravity-plugin/`, point at the same components. There is no root `plugin.json`, no `commands/` folder and no hooks (ADR-0001, ADR-0002).
- A Claude Code marketplace manifest ships in the repo so Claude users install from GitHub. Antigravity users install from a local clone in v1.
- The plugin keeps the name `dev-workflow`.
- Agent files carry only `name` and `description`; restrictions are prose (ADR-0002).

### Components

- **Driver** (a bash script under `scripts/`): the only long-running process. It selects the runtime, resolves label strings, ensures labels exist, computes the backlog state, picks and claims a ticket, prepares the ticket worktree, runs one headless iteration, cleans up, notifies, and applies the sleep or exit rules.
- **Backlog scripts** (bash over `gh`, under `scripts/`): list the frontier, list resumable parked tickets, list waiting and blocked tickets, claim, park, fail, mark in review, post a marked comment. Each is a small single-purpose script so it can be tested with a fake `gh` and reused by the skill.
- **`next` skill**: the body of one iteration, invoked headlessly with the ticket number as its argument. It is the implementer.
- **`reviewer` agent**: a fresh-context subagent given the diff and the ticket. Returns findings; never edits.
- **README**: installation for both platforms, prerequisites, the label state machine, running the driver, safety, v1 non-goals.

### Dependencies

- Hard: `gh` authenticated; the files `/setup-matt-pocock-skills` writes under `docs/agents/` (issue tracker and triage-labels). The driver refuses to start if the tracker file is missing or names a tracker other than GitHub.
- Soft: Matt Pocock's `tdd`, `code-review` and `pr` skills are used when installed; otherwise the `next` skill applies its own test-first rules, the `reviewer` agent alone, and a fixed PR template.

### Labels and ticket state (ADR-0003)

- A ticket keeps its `ready-for-agent` state role for its whole life under the agent. Agent progress is a second axis: `agent:working`, `agent:waiting`, `agent:in-review`, `agent:failed`. At most one `agent:*` label at a time.
- Frontier: open, carries the `ready-for-agent` role, no open blockers per GitHub native dependencies, no `agent:*` label, no assignee.
- Resumable: carries `agent:waiting` and its newest comment lacks the hidden agent marker.
- Waiting: carries `agent:waiting` and is not resumable. Blocked: carries `ready-for-agent`, no `agent:*` label, and has at least one open blocker.
- The state role wins: a ticket without the `ready-for-agent` role is never touched, whatever `agent:*` label it carries. The skill re-checks the role before opening a PR; if it was removed, it pushes the branch, comments, removes the `agent:*` label and stops.
- Retry of a failed ticket: the maintainer removes `agent:failed`. Declining it: the maintainer changes the state role and removes `agent:failed`.
- The `ready-for-agent` string is resolved from the repo's `docs/agents/triage-labels.md` table, defaulting to the literal. The four `agent:*` names default to those literals and are overridable by environment variables. Labels are created idempotently on first run.
- The agent never uses `needs-info`, assignees or @mentions in v1.

### Driver rules

- Runtime: auto-detect when exactly one of `claude` or `agy` is on the PATH; require `--runtime claude|agy` when both are. The runtime decides the skill invocation form (`/dev-workflow:next <n>` versus `/next <n>`) and the headless flags.
- Decision each cycle: if a resumable ticket exists, iterate on the oldest one; else if the frontier is non-empty, iterate on the lowest issue number; else if any ticket is waiting or blocked, sleep for the interval; else exit.
- Caps: `--interval` (default 5m), `--max-iterations` (default 20), `--max-idle` (default 8h, exit after that long without an iteration). Exit codes distinguish "backlog empty" from "cap reached" from "error".
- Permissions: default to the platform's skip-permissions flag, plus the sandbox flag on Antigravity. `--safe` uses the platform's allowlist mode instead. The README carries the warning to run in an isolated environment.
- Model: `--model` is passed through to the runtime; the plugin sets no model itself.
- Per iteration: claim (label plus marked comment), create the ticket worktree under `.worktrees/<n>` from the existing `ticket/<n>` branch on the remote if present, otherwise from the default branch; exclude `.worktrees/` via the repo's git exclude file, not a tracked `.gitignore`; run the runtime headlessly with the worktree as working directory and a timeout; afterwards verify the ticket carries exactly one of `agent:waiting`, `agent:in-review`, `agent:failed` (if it still carries `agent:working`, the iteration crashed: push whatever exists, fail the ticket with a comment); remove the worktree; run the notify command.
- Notify: `--notify <command>` receives the event kind (`parked`, `failed`, `pr-opened`), the issue number and the issue or PR URL as arguments. Default: one log line on stdout.
- `--dry-run` prints the decision and the ticket it would pick, runs nothing, changes nothing.

### The `next` skill (one iteration)

Given a ticket number, in this order:

1. Confirm the ticket carries `agent:working` (the driver claimed it). Detect prior work: an existing `ticket/<n>` branch and earlier agent comments mean this is a resume, so read the maintainer's answer and continue; never restart from scratch.
2. Read the ticket body, every comment, and the merged pull requests of its blockers.
3. Read the repo's agent docs (`CLAUDE.md` or `AGENTS.md`, `GLOSSARY.md`, ADRs) for conventions and for how to typecheck and run tests. The plugin adds no configuration of its own.
4. Explore the code the ticket touches.
5. Decision point. If anything blocks a confident implementation, post one comment containing all questions, swap `agent:working` for `agent:waiting`, push any work, and stop. Doubts that arise later become stated assumptions in the PR, unless they change the acceptance criteria, in which case the ticket is parked at that point.
6. Implement test-first on the ticket branch only, with small commits following the repo's detectable commit convention. Use the `tdd` skill when installed.
7. Typecheck and run the full test suite.
8. Hand the diff and the ticket to the `reviewer` agent (or the `code-review` skill when installed); fix findings; re-review. At most two review rounds, then proceed listing unresolved findings in the PR.
9. Re-check the ticket's state role. Push. Open a PR whose body comes from the `pr` skill when installed, else a fixed template: what and why, `Closes #<n>`, assumptions made, how it was tested, unresolved review findings. Swap `agent:working` for `agent:in-review` and post a marked comment linking the PR.
10. If at any point the ticket cannot be finished (tests cannot pass, ticket contradicts the code, blocker's PR did not deliver what the ticket assumes), push, post a marked comment explaining what was tried, swap to `agent:failed`, and stop.

The skill never merges, never force-pushes, never touches the default branch, never retries a failed ticket.

### Comment convention

Every agent comment begins with the visible line `> *Posted by dev-workflow (agent)*` followed by a hidden HTML comment naming the kind: `claimed`, `question`, `failed`, `pr-opened`. Scripts detect "the maintainer replied" by the absence of the marker on the newest comment.

### Reviewer agent

Input: the ticket and the diff of the ticket branch against the default branch. Checks acceptance criteria coverage, tests present and meaningful, repo conventions and glossary vocabulary, ADR compliance. Output: a numbered list of findings, each with severity and location, or an explicit "no findings". It does not edit files.

## Testing Decisions

- Good tests here exercise external behaviour at the shell boundary: given a fake `gh` that returns canned issue JSON and records the commands it received, a backlog script must emit the right ticket numbers and issue the right `gh` calls. Implementation details of the scripts are not asserted.
- The seam is the `gh` executable on the PATH. Tests prepend a directory containing a fake `gh` (and fake `claude`/`agy` for driver tests) that replays fixtures and logs calls. One seam covers every script and the driver.
- Driver tests cover each decision outcome (iterate on resumable, iterate on frontier, sleep, exit, cap reached, dual-runtime refusal, dry-run) and the crash path where a ticket still carries `agent:working` after an iteration.
- Manifests and component files are validated with `agy plugin validate --strict` and Claude Code's plugin validation; both must pass in CI.
- Skill and agent prose is not unit-tested; the acceptance test for the `next` skill is an end-to-end run against a scratch GitHub repository with three tickets (one parkable, one with a blocker, one straightforward), performed manually before release.
- Prior art: none in this repo. Use `bats-core` for the shell tests.

## Out of Scope

- A separate GitHub identity for the agent, and therefore @mention notifications and assignee-based claims (v2 idea, still to be fleshed out).
- Reacting to pull request review comments or red CI on an open PR (v2, same park and resume mechanism applied to PRs).
- Running several tickets in parallel worktrees.
- Trackers other than GitHub, and any runtime other than Claude Code and Antigravity CLI.
- Breaking work into tickets, writing specs, triage. Those are Matt Pocock's skills.
- The previous four-agent pipeline (architect, developer, document writer, QA). Its files are deleted.

## Further Notes

- Because the agent runs under the maintainer's own login, GitHub generates no notifications for its comments. The notify command is the v1 answer; the README mentions GitHub's email setting "Include your own updates" as an optional alternative, with its noise caveat.
- GitHub native issue dependencies and sub-issues must be available on the target repository; where they are not, `to-tickets` falls back to a "Blocked by" line in the body, which v1 does not parse. The README states this.
- Antigravity's documentation says skills expand in headless print mode (there is a flag to disable it), and the Antigravity validator rejects the old agent files' model tier, confirming the ADR-0002 approach. Remote plugin install on Antigravity is unverified and left as a README note.
