# AGENTS.md — Project Agent Instructions

> **Project-specific.** Fill in this file for each project. It defines
> who the agents are, what they can touch, and how the project works.

## Project Identity

- **Project name:** [FILL: e.g. AreteOS]
- **Repository:** [FILL: git URL]
- **Tech stack:** [FILL: e.g. Next.js, TypeScript, PostgreSQL, Tailwind]
- **Deployment:** [FILL: e.g. Vercel, staging → production via DevOps role]

## Agent Roles

Define which agent profiles operate in this project and what they do.
Map each role from AUTONOMOUS.md to a specific agent/CLI.

| Role | Agent Profile | Primary Tool | Responsibilities |
|---|---|---|---|
| Planner | [FILL] | [FILL] | PRD creation from signed scope |
| Architect | [FILL] | [FILL] | Technical decisions, seams, APIs |
| Designer | [FILL] | [FILL] | DESIGN.md, UI consistency |
| PreDocumentation | [FILL] | Spec Kit CLI | Phase breakdown, specs, tickets |
| Developer | [FILL] | Codex CLI | Implementation from tickets |
| Tester | [FILL] | Playwright | E2E tests, evidence collection |
| DevOps | [FILL] | Vercel/gh CLI | Staging and production deploys |
| Operator | [FILL] | Hermes Agent | Orchestration, review gates |

## Workspace Map

[FILL: Describe the repository structure. What lives where?]

```text
project-root/
├── apps/
│   ├── web/           # [FILL: what's here]
│   ├── api/           # [FILL: what's here]
│   └── mobile/        # [FILL: what's here]
├── packages/
│   └── shared/        # [FILL: what's here]
├── docs/              # Architecture, ADRs, runbooks
├── specs/             # Spec Kit output
└── evidence/          # Test evidence
```

## Delivery Model

- **Main branch:** [FILL: e.g. `main`]
- **Delivery branches:** [FILL: naming convention, e.g. `delivery/m3-phase-1`]
- **Staging URL:** [FILL]
- **Production URL:** [FILL]
- **Deploy command:** [FILL]
- **Who can deploy:** DevOps role only

## Boundaries (What NOT to Touch)

- [FILL: files, directories, or systems that are off-limits]
- [FILL: secrets, credentials, environment variables]
- [FILL: third-party integrations that require manual setup]

## Dependencies

- **Package manager:** [FILL: npm, yarn, pnpm, uv, pip]
- **Install command:** [FILL: `npm install`, `uv sync`, etc.]
- **Build command:** [FILL]
- **Test command:** [FILL]
- **Lint command:** [FILL]
- **Format command:** [FILL: prettier, biome, black]

## Git Conventions

- **Branch naming:** [FILL: e.g. `feat/`, `fix/`, `delivery/`]
- **Commit format:** `type(scope): description`
  - Types: `feat`, `fix`, `refactor`, `docs`, `test`, `chore`
  - Scope: [FILL: e.g. `auth`, `api`, `ui`, `db`]
- **PR format:** [FILL: template or conventions]
- **Merging:** [FILL: squash, rebase, merge commit]

## Context Files

These files are read by every agent at session start (see ROLE.md):

| File | Purpose | Location |
|---|---|---|
| PROJECT.md | Project constitution, north star, milestones | `PROJECT.md` |
| ROLE.md | Bootstrap — how to become this agent | `framework/ROLE.md` |
| ARCHITECTURE.md | System design, seams, data flow | `framework/ARCHITECTURE.md` |
| DATABASE.md | Models, relationships, migrations | `framework/DATABASE.md` |
| DESIGN.md | Visual tokens, components | `framework/DESIGN.md` |
| TESTING.md | Testing strategy, evidence, E2E | `framework/TESTING.md` |
| DEBUGGING.md | Structured logging, error tracing | `framework/DEBUGGING.md` |
| VERSION-CONTROL.md | Git strategy, PRs, CI/CD, reviews | `framework/VERSION-CONTROL.md` |
| COMMIT-TEMPLATE.md | Commit message format, learnings extraction | `framework/COMMIT-TEMPLATE.md` |
| SUBAGENT.md | Inter-agent contract | `framework/SUBAGENT.md` |
| HERMES.md | Hermes operations, dispatcher, spawn | `framework/HERMES.md` |
| LEARNINGS.md | Accumulated knowledge | `framework/LEARNINGS.md` |
| AUTONOMOUS.md | Full pipeline contract | `AUTONOMOUS.md` |
