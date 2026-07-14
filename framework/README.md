# Autonomous Development Framework

A templated framework for running autonomous AI agent software development
pipelines. One ROLE.md file transforms a general-purpose agent into a
specialized project agent in a single session.

## How It Works

```
Clean session → read ROLE.md → read AGENTS.md → read ARCHITECTURE.md
→ read DATABASE.md → read DESIGN.md → read SUBAGENT.md → agent is ready
```

## Files

| File | Type | Purpose |
|---|---|---|
| **ROLE.md** | Template (project-agnostic) | Entry point. Teaches agent how to read the other files and become project-aware. |
| **AGENTS.md** | Template (fill per project) | Agent roles, workspace map, delivery model, boundaries, conventions. |
| **ARCHITECTURE.md** | Template (fill per project) | Module boundaries, data flow, API contracts, seams, ADRs. |
| **DATABASE.md** | Template (fill per project) | Models, relationships, migrations, invariants, query patterns. |
| **DESIGN.md** | Template (fill per project) | Google DESIGN.md token spec — colors, typography, spacing, components. |
| **TESTING.md** | Ready-to-use | Testing strategy — pyramid, states, evidence, E2E, learnings extraction. |
| **VERSION-CONTROL.md** | Ready-to-use (adapt project) | Branching strategy, commits, PRs, CI/CD, code review, agent identity. |
| **COMMIT-TEMPLATE.md** | Ready-to-use | Commit message format, types, scopes, learnings extraction protocol. |
| **SUBAGENT.md** | Ready-to-use | Inter-agent contract — vocabulary, checklist, handoff protocol, rules. |
| **HERMES.md** | Ready-to-use | Hermes operations — kanban dispatcher, agent spawning, worktrees, cron, recovery. |
| **LEARNINGS.md** | Living document | Accumulated knowledge from retrospectives. Updated every phase. |

## Setup for a New Project

1. Copy the `framework/` folder to your project.
2. Fill in the `[FILL]` markers in AGENTS.md, ARCHITECTURE.md, DATABASE.md, DESIGN.md.
3. Customize SUBAGENT.md if needed (defaults are opinionated but universal).
4. Start every agent session with: "read framework/ROLE.md"

## Prerequisites

- Hermes Agent (for kanban, delegation, skills)
- Codex CLI or Claude Code (for implementation)
- Skills installed (see AUTONOMOUS.md References section)
- Spec Kit (for PreDocumentation role)

## Origin

Extracted from the AUTONOMOUS.md pipeline contract. See `../AUTONOMOUS.md`
for the full pipeline specification — roles, principles, gates, artifacts,
and anti-patterns.

## Related

- **AUTONOMOUS.md** — full pipeline contract (in parent directory)
- **skills.sh** — agent skills hub (https://skills.sh)
- **Matt Pocock skills** — engineering skill pack (https://github.com/mattpocock/skills)
- **DESIGN.md spec** — Google's token spec (https://github.com/google-labs-code/design.md)
- **Clean Code** — Robert C. Martin, extracted as Hermes skill
