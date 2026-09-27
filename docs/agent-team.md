# Agent team

Mona's Project Pulse dashboard will be built by a four-agent team coordinated through GitHub Copilot CLI in a Codespace.

| Agent | Target model | Responsibility | Definition |
|---|---|---|---|
| Orchestrator | Claude Opus 4.7 (copilot) | Breaks the project into phases, delegates work with explicit file scopes, coordinates dependencies, and verifies the integrated result. | `.github/agents/orchestrator.agent.md` |
| Planner | Claude Opus 4.7 (copilot) | Researches the repository and relevant documentation, identifies risks and edge cases, and produces an ordered implementation plan with dependencies and validation expectations. | `.github/agents/planner.agent.md` |
| Coder | GPT-5.5 (copilot) | Implements application logic and runnable-app support, keeps behavior explicit and testable, and validates code changes. | `.github/agents/coder.agent.md` |
| Designer | Gemini 3.1 Pro (copilot) | Owns the dashboard's UI/UX, accessibility, information hierarchy, responsive behavior, and polished Project Pulse visual design. | `.github/agents/designer.agent.md` |

The Orchestrator will use the Planner's implementation strategy to sequence or parallelize the Coder's and Designer's assignments while avoiding overlapping file ownership. Git operations remain under the learner's control through GitHub Copilot CLI prompts.
