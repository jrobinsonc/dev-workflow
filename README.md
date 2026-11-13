# dev-workflow

An unattended agent loop that works a repository's GitHub issue backlog one ticket at a time, asking the maintainer on the issue when it is stuck and resuming when they answer. It targets Claude Code and Antigravity CLI.

> [!WARNING]
> **Work in progress.** This README describes the v1 design, which is not implemented yet. The driver, the backlog scripts, the `next` skill, the `reviewer` agent and the plugin manifests do not exist in this repo. Nothing here is installable or runnable today, and details may change before release.
>
> The `agents/`, `skills/` and `plugin.json` files currently in the repo belong to the previous four-agent Gemini pipeline (architect, developer, document writer, QA). v1 replaces that pipeline and those files will be deleted.

The full design lives in [the v1 spec](docs/specs/0001-ticket-loop-v1.md). The vocabulary is defined in [GLOSSARY.md](GLOSSARY.md), and the main decisions are recorded in [docs/adr/](docs/adr/).

## What it will do

You break your work into GitHub tickets with acceptance criteria, a `ready-for-agent` label and native blocking edges. Then you start the driver and do something else.

Each iteration, the driver takes one ticket from the frontier and works it in a fresh context:

1. It claims the ticket and creates a throwaway worktree for it.
2. The agent reads the ticket, its comments, the merged pull requests of its blockers, and your repo's agent docs, glossary and ADRs.
3. If anything blocks a confident implementation, the agent posts all its questions in one comment, parks the ticket and moves on to another.
4. Otherwise it implements the ticket test-first, typechecks and runs the full test suite.
5. A fresh-context reviewer checks the diff against the ticket's acceptance criteria and the repo's conventions.
6. The agent opens one pull request that closes the issue on merge, listing the assumptions it made.

When you reply on a parked ticket, the agent resumes it automatically. When nothing is workable but something is parked or blocked by an unmerged pull request, the driver sleeps and re-checks every five minutes. When the backlog has nothing left, it exits.

Your only duties are answering questions on GitHub and merging pull requests. The agent never merges its own pull requests, never force-pushes and never touches the default branch.

## Prerequisites

- Claude Code or Antigravity CLI.
- `gh`, authenticated.
- A repository set up with `/setup-matt-pocock-skills`, which writes the issue tracker and triage label files under `docs/agents/`. The driver refuses to start if the tracker file is missing or names a tracker other than GitHub.
- GitHub native issue dependencies on the target repository. A "Blocked by" line in the issue body is not parsed in v1.

Matt Pocock's `tdd`, `code-review` and `pr` skills are used when installed. Without them, the plugin falls back to its own test-first rules, its own reviewer and a fixed pull request template.

## Installation

Not available yet. The plan is:

- **Claude Code:** install from a marketplace manifest shipped in this repo.
- **Antigravity CLI:** install from a local clone.

## Running the driver

The driver will be a bash script under `scripts/`, started from your repository. Planned options:

| Option | Default | Purpose |
| --- | --- | --- |
| `--runtime claude\|agy` | auto-detected | Required only when both runtimes are on the PATH. |
| `--interval` | `5m` | How long to sleep when every remaining ticket is parked or blocked. |
| `--max-iterations` | `20` | Cap on iterations per run. |
| `--max-idle` | `8h` | Exit after this long without an iteration. |
| `--model` | none | Passed through to the runtime. The plugin sets no model itself. |
| `--notify <command>` | log line on stdout | Runs when a ticket is parked, failed or has a pull request. Receives the event kind, the issue number and the issue or PR URL. |
| `--safe` | off | Uses the platform's allowlist mode instead of skip-permissions. |
| `--dry-run` | off | Prints the decision and the ticket it would pick. Runs nothing and changes nothing. |

Exit codes will distinguish "backlog empty" from "cap reached" from "error".

## Ticket states

A ticket keeps its `ready-for-agent` label for its whole life under the agent. Agent progress is a second set of labels, with at most one on a ticket at a time:

| Label | Meaning |
| --- | --- |
| none | In the frontier (if it has no open blockers and no assignee), or blocked. |
| `agent:working` | Claimed. An iteration is working on it. |
| `agent:waiting` | Parked. The agent asked questions and is waiting for your reply. |
| `agent:in-review` | A pull request is open and waiting for you to merge it. |
| `agent:failed` | The agent gave up and explained what it tried. Waiting for your decision. |

How you control it:

- **Answer a question:** reply on the issue. The ticket resumes on its own.
- **Retry a failed ticket:** remove `agent:failed`.
- **Take a ticket away from the agent:** remove `ready-for-agent`. A ticket without it is never touched, whatever `agent:*` label it carries.

The `agent:*` labels are created automatically on first run and can be renamed with environment variables. The `ready-for-agent` name is read from your repo's `docs/agents/triage-labels.md` when present.

Every agent comment starts with the line `> *Posted by dev-workflow (agent)*`, so you can tell it apart from your own comments under the same login.

## Safety

Headless mode means a denied tool call fails silently, so the driver defaults to the platform's skip-permissions flag (plus the sandbox flag on Antigravity). **Run it in an isolated environment.** Use `--safe` for the allowlist mode instead.

Other limits on what the agent can do:

- It works only in a throwaway worktree under `.worktrees/`, so your own checkout and uncommitted changes are never touched.
- It pushes all work in progress before parking or failing a ticket.
- Nothing lands without your merge.
- Iteration and idle caps bound the cost of an overnight run.

## Notifications

The agent runs under your own GitHub login, so GitHub sends you no notifications for its comments. Use `--notify` to learn when something is waiting for you. GitHub's email setting "Include your own updates" is an alternative, but it is noisy.

## Not in v1

- A separate GitHub identity for the agent, and therefore @mentions and assignee-based claims.
- Reacting to pull request review comments or failing CI on an open pull request.
- Running several tickets in parallel.
- Trackers other than GitHub, and runtimes other than Claude Code and Antigravity CLI.
- Breaking work into tickets, writing specs and triage.

## License

[MIT](LICENSE)
