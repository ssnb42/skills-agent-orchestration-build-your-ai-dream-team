# Project Pulse Agent Team

This document describes the custom agents used to build Mona's Project Pulse
dashboard. The agent definitions referenced here live in the repository's
`.github/agents/` folder. The work is orchestrated through GitHub Copilot CLI
running in a Codespace.

## Agent roles

### Orchestrator

**Definition:** `.github/agents/orchestrator.agent.md`

The Orchestrator coordinates the other specialist agents. It breaks complex
requests into tasks, assigns explicit file scopes, manages dependencies and
parallel or sequential phases, and verifies that the integrated result works
together. The Orchestrator coordinates the work but does not implement it
itself.

### Planner

**Definition:** `.github/agents/planner.agent.md`

The Planner researches the repository, documentation, dependencies, risks, and
edge cases, then produces a practical implementation plan. The plan includes
ordered steps, file assignments, dependencies, parallelizable work,
validation expectations, and open questions. The Planner does not write code.

### Designer

**Definition:** `.github/agents/designer.agent.md`

The Designer handles UI/UX, accessibility, information hierarchy,
interaction flow, visual clarity, responsive behavior, and consistency with
existing product patterns. For Project Pulse, the Designer defines a polished
dashboard experience with project cards, status badges, priority treatment,
readable spacing, responsive behavior, and deterministic CSS hooks such as
`.dashboard` and `.project-card`.

### Coder

**Definition:** `.github/agents/coder.agent.md`

The Coder implements code-oriented tasks within the file scope assigned by the
Orchestrator. The Coder writes clear, explicit, deterministic, and testable
code, follows repository conventions, handles errors visibly, and validates
the changes. For runnable Project Pulse work, the Coder can also create the
assigned `.vscode/launch.json` support configuration.

## Assigned models

The following model assignments are taken directly from the `model` field in
each agent definition under `.github/agents/`.

| Agent | Model |
|---|---|
| Orchestrator | Claude Opus 4.7 (copilot) |
| Planner | Claude Opus 4.7 (copilot) |
| Designer | Gemini 3.1 Pro (copilot) |
| Coder | GPT-5.5 (copilot) |

## Collaboration flow

1. The Orchestrator receives the dashboard request and delegates repository
   research and planning to the Planner.
2. The Planner defines the implementation phases, file ownership, data
   structure, dependencies, and validation criteria.
3. The Designer defines the visual, responsive, and accessibility contract.
4. The Coder implements the assigned dashboard files using the plan and design
   contract.
5. The Orchestrator reviews the integrated result and reports the outcome.

All agents are configured not to stage, commit, or push changes. Git
operations remain under the learner's control through GitHub Copilot CLI.
