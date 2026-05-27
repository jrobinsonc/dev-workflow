---
name: dev-workflow
description: Coordinates a multi-agent workflow from Architecture to QA.
---

# Architect Workflow Orchestration

When this skill is activated, you must guide the session through the following phases:

1.  **Architecture Phase**: Call the `software_architect` sub-agent to analyze the user's task and break it down into documentation and development requirements.
2.  **Documentation & Development Phase**: In parallel (if possible) or sequentially, invoke the following:
    - Call the `document_writer` sub-agent with the documentation requirements from the Architect.
    - Call the `software_developer` sub-agent with the development requirements from the Architect.
3.  **QA Phase**: Once the `software_developer` finishes, call the `qa_engineer` sub-agent to review the code and ensure it meets the Architect's requirements.
4.  **Final Summary**: Provide a high-level summary of the entire process, including the output from the QA phase.

Always ensure the Architect's requirements are clearly communicated to the sub-agents.
