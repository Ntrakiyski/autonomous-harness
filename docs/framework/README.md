# Framework Reference

These documents explain the project truths that the Harness helps an agent
discover and preserve. They are examples and prompts, not files that the
skills install or require.

| Document | Explains |
|---|---|
| [PROJECT.md](PROJECT.md) | Product intent, outcomes, and explicit unknowns. |
| [GUARDRAILS.md](GUARDRAILS.md) | Safety and scope boundaries. |
| [ARCHITECTURE.md](ARCHITECTURE.md) | Modules, seams, contracts, and technical decisions. |
| [DATABASE.md](DATABASE.md) | Models, relationships, invariants, and migrations. |
| [DESIGN.md](DESIGN.md) | Visual tokens and component conventions. |
| [TESTING.md](TESTING.md) | Test strategy and evidence collection. |
| [LEARNINGS.md](LEARNINGS.md) | Reusable lessons from completed work. |

`autonomous-init` maps whatever equivalent documents a consuming project
already has into `.autonomous/CONTEXT.md`. `autonomous-gate` derives its
checks from those project documents and its own references.
