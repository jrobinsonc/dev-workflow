# Dev Workflow

An unattended agent loop that works a repository's GitHub issue backlog one ticket at a time, asking the maintainer on the issue when it is stuck and resuming when they answer.

## Language

**Ticket**:
A GitHub issue that describes one vertical slice of work, with acceptance criteria and blocking edges to other tickets.
_Avoid_: Task, issue (when meaning a unit of agent work), story

**Maintainer**:
The human who owns the backlog, answers the agent's questions, and merges its pull requests.
_Avoid_: User, owner, reporter

**Blocker**:
A ticket that must be done before another ticket can start, expressed as a GitHub native dependency.
_Avoid_: Dependency, prerequisite

**Done**:
The state of a ticket whose issue is closed, which happens when the maintainer merges its pull request.
_Avoid_: Finished, complete, PR opened

**Frontier**:
The set of tickets that are open, marked ready for an agent, have no open blockers, and are neither claimed nor parked.
_Avoid_: Queue, backlog, available tickets

**Claim**:
The act of marking a frontier ticket as taken by the agent so no other session picks it up.
_Avoid_: Assign, lock, checkout

**Park**:
The act of suspending work on a claimed ticket because the agent needs an answer from the maintainer, with the questions posted on the ticket.
_Avoid_: Block, pause, needs-info

**Resume**:
Picking a parked ticket back up after the maintainer has replied on it.
_Avoid_: Unpark, continue, retry

**Failed**:
The state of a claimed ticket the agent gave up on, with an explanation posted on the ticket, awaiting a maintainer decision.
_Avoid_: Blocked, abandoned, errored

**Iteration**:
One pass of the loop: a single ticket is worked in a fresh context until it is parked, failed, or has a pull request.
_Avoid_: Run, cycle, turn

**Sleep**:
Waiting a fixed interval between iterations because every remaining ticket is parked or blocked.
_Avoid_: Poll, wait, idle

**Driver**:
The process that decides whether to run an iteration, sleep, or exit, and that picks, claims and prepares the ticket each iteration works on.
_Avoid_: Loop, runner, scheduler, orchestrator

**Ticket worktree**:
A throwaway git worktree created for one ticket's branch, where an iteration does its work, and removed when the iteration ends.
_Avoid_: Clone, checkout, sandbox

**Reviewer**:
A fresh-context agent that checks a ticket's diff against the ticket's acceptance criteria and the repo's conventions before the pull request opens.
_Avoid_: QA, QA engineer, code reviewer

**Notify command**:
A user-supplied command the driver runs when an iteration parks, fails, or opens a pull request, so the maintainer learns there is something to look at.
_Avoid_: Webhook, alert, mention
