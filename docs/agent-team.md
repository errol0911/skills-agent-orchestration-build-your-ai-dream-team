# Agent team

I am using **GitHub Copilot CLI in a Codespace** to orchestrate four custom agents that will build Mona's Project Pulse dashboard. All definitions live in `.github/agents/`.

| Agent | Target model | Responsibility | Definition |
| --- | --- | --- | --- |
| Orchestrator | Claude Opus 5.5 (copilot) | Breaks requests into tasks, delegates to Planner, Coder, and Designer with explicit file scopes, runs work in parallel only when scopes don't overlap, verifies the integrated result, and reports. It does not implement. | `.github/agents/orchestrator.agent.md` |
| Planner | Claude Opus 5.5 (copilot) | Researches the repo and docs, then returns a plan: ordered steps, file assignments, dependencies, parallel vs. sequential work, edge cases, validation, and open questions. It writes no code. | `.github/agents/planner.agent.md` |
| Coder | GPT-5.5 (copilot) | Implements code and fixes bugs within its assigned files, and creates runnable-app support such as `.vscode/launch.json` (`cwd` of `${workspaceFolder}/app`, opening `index.html`). It validates before reporting. | `.github/agents/coder.agent.md` |
| Designer | Gemini 3.7 Flash (copilot) | Handles UI/UX, accessibility, and visual design. For Project Pulse, it delivers a polished, responsive dashboard with project cards, status badges, priority treatment, and `.dashboard` / `.project-card` hooks. | `.github/agents/designer.agent.md` |

## Workflow

1. Orchestrator asks Planner for a plan.
1. Orchestrator splits the plan into phases with non-overlapping file scopes.
1. Coder and Designer implement their assigned files.
1. Orchestrator verifies the result and reports.

No agent stages, commits, or pushes. I control all git operations through Copilot CLI prompts.
