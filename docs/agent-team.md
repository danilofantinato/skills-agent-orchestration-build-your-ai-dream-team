# Agent team

I am using GitHub Copilot CLI in a Codespace to orchestrate four custom agents that build Mona's Project Pulse dashboard. All definitions live in `.github/agents/`.

| Agent | Model | Responsibility | Definition |
| --- | --- | --- | --- |
| Orchestrator | Claude Opus 4.7 | Breaks down requests, delegates to specialists with explicit file scopes, runs work in parallel only when scopes don't overlap, and verifies the integrated result. Does not implement. | `.github/agents/orchestrator.agent.md` |
| Planner | Claude Opus 4.7 | Researches the repo and docs, then returns an ordered plan with file assignments, dependencies, edge cases, and validation expectations. Does not write code. | `.github/agents/planner.agent.md` |
| Coder | GPT-5.5 | Implements logic, fixes bugs, and creates assigned support files such as `.vscode/launch.json` (cwd `${workspaceFolder}/app`, opens `index.html`). | `.github/agents/coder.agent.md` |
| Designer | Gemini 3.1 Pro | Handles UX, accessibility, and styling: project cards, status badges, priority treatment, responsive layout, and CSS hooks like `.dashboard` and `.project-card`. | `.github/agents/designer.agent.md` |

## Workflow

1. The Orchestrator asks the Planner for a plan.
1. The plan is split into phases with file assignments.
1. The Coder and Designer work within their assigned files.
1. The Orchestrator verifies the integrated result.

None of the agents stage, commit, or push changes. I control all git operations through Copilot CLI prompts.
