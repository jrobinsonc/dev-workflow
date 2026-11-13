# dev-workflow

A plugin for Claude Code and Antigravity CLI that works a repository's GitHub ticket backlog unattended, one ticket at a time. Read `GLOSSARY.md` for the project vocabulary and `docs/adr/` for the decisions behind the design. The v1 spec is `docs/specs/0001-ticket-loop-v1.md`.

## Agent skills

### Issue tracker

Issues live in this repo's GitHub Issues, managed with the `gh` CLI. See `docs/agents/issue-tracker.md`.

### Triage labels

Default vocabulary: `needs-triage`, `needs-info`, `ready-for-agent`, `ready-for-human`, `wontfix`. See `docs/agents/triage-labels.md`.

### Domain docs

Single-context: `GLOSSARY.md` and `docs/adr/` at the repo root. See `docs/agents/domain.md`.
