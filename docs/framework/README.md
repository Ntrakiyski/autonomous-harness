# Framework Reference

This is the full Hermes-led operating contract for Autonomous Harness. It
teaches the operator and coding agents how the delivery loop works; Skills.sh
installs only the agent bridge under `skills/`.

## Operating Layer

| Document | Purpose |
|---|---|
| [AUTONOMOUS.md](AUTONOMOUS.md) | Delivery pipeline, roles, artifacts, gates, and principles. |
| [HERMES.md](HERMES.md) | Operator model, delegation, worktrees, and recovery. |
| [ROLE.md](ROLE.md) | Coding-agent startup sequence and working boundaries. |
| [AGENTS.md](AGENTS.md) | Project roles, workspace, dependencies, and constraints. |
| [SUBAGENT.md](SUBAGENT.md) | Shared vocabulary, pre-flight checks, and handoffs. |
| [VERSION-CONTROL.md](VERSION-CONTROL.md) | Branch, review, CI, and release conventions. |
| [COMMIT-TEMPLATE.md](COMMIT-TEMPLATE.md) | Commit and learning-capture conventions. |
| [DEBUGGING.md](DEBUGGING.md) | Evidence, errors, and incident investigation. |

## Project Truth Templates

| Document | Purpose |
|---|---|
| [PROJECT.md](PROJECT.md) | Product intent, outcomes, and explicit unknowns. |
| [GUARDRAILS.md](GUARDRAILS.md) | Safety and scope boundaries. |
| [ARCHITECTURE.md](ARCHITECTURE.md) | Modules, seams, contracts, and technical decisions. |
| [DATABASE.md](DATABASE.md) | Models, relationships, invariants, and migrations. |
| [DESIGN.md](DESIGN.md) | Visual tokens and component conventions. |
| [TESTING.md](TESTING.md) | Test strategy and evidence collection. |
| [LEARNINGS.md](LEARNINGS.md) | Reusable lessons from completed work. |

## Agent Bridge

`autonomous-init` maps equivalent project documents into
`.autonomous/CONTEXT.md`; it does not copy this framework into a consuming
repository. `autonomous-gate` owns its dynamic gate prompts under
[`skills/autonomous-gate/references/`](../../skills/autonomous-gate/references/).
