# dev-workflow

Gemini agents and skills for orchestrating a multi-agent software development workflow.

## Installation

Clone this repo into your Gemini plugins directory:

```sh
git clone https://github.com/jrobinsonc/dev-workflow ~/.gemini/config/plugins/dev-workflow
```

Restart your IDE. The agents and skill will be available automatically.

---

## Skills

### `dev-workflow`

Runs a full pipeline: the Architect breaks down your task, the Developer and Document Writer work in parallel, and the QA Engineer reviews the result.

**How to use it:**

> "Use the dev-workflow skill to implement a CSV export feature for the orders module."

> "Trigger the dev-workflow to add JWT authentication to the API."

The assistant will automatically sequence the sub-agents — you just describe what you want built.

---

## Agents

Agents are called by the assistant as part of a workflow. You can also address one directly if you only need that specific role.

### `software-architect`

Breaks a feature request into documentation requirements and development specifications.

> "Ask the software-architect agent to analyze what it would take to migrate from REST to GraphQL."

### `software-developer`

Implements code based on a set of requirements.

> "Have the software-developer agent implement the authentication module based on these specs: ..."

### `document-writer`

Produces READMEs, API docs, diagrams, or tutorials.

> "Ask the document-writer agent to write API documentation for the `/orders` endpoints."

### `qa-engineer`

Reviews code for correctness, quality, test coverage, and edge cases.

> "Have the qa-engineer review the changes in `src/auth/` against these requirements: ..."
