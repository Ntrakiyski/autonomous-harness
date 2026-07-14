# Autonomous Harness

> **Turn a swarm of AI agents into a disciplined software delivery pipeline.**
> One ROLE.md file transforms a general-purpose agent into a specialized
> team member. The harness keeps them aligned on architecture, design,
> quality, and consistency — from ticket to production.

---

## What It Is

A templated framework for running autonomous AI agent software development
pipelines. It defines **roles, principles, gates, checklists, and contracts**
that make a team of independent AI agents behave like a single engineering
organization.

**Not a tool. Not a SaaS.** It's a set of files you copy into your project.
Every agent reads them at session start and they become project-aware.

```
Clean session → read ROLE.md → read 12 framework files → agent is ready
```

---

## How It Works

```
┌──────────┐    ┌───────────┐    ┌────────────┐    ┌──────────┐
│  Manager │───▶│  Planner  │───▶│  PRD Gate  │───▶│   Spec   │
│ (CEO)    │    │ Architect │    │ Checklist  │    │   Kit    │
│          │    │ Designer  │    │  22 items  │    │          │
└──────────┘    └───────────┘    └────────────┘    └──────────┘
                                                        │
                                                        ▼
┌──────────┐    ┌───────────┐    ┌────────────┐    ┌──────────┐
│  Deploy  │◀───│ Ship Gate │◀───│   Tester   │◀───│ Developer│
│ (Vercel) │    │ Checklist │    │  + CI/CD   │    │ + Codex  │
│          │    │           │    │            │    │ + Husky  │
└──────────┘    └───────────┘    └────────────┘    └──────────┘
      │
      ▼
┌──────────┐
│Retro-    │
│spective  │
│+ Learn   │
└──────────┘
```

---

## File Map

| File | Purpose |
|---|---|
| `AUTONOMOUS.md` | Full pipeline contract — 12 sections, 540 lines |
| `framework/ROLE.md` | Agent bootstrap — 12-step startup sequence |
| `framework/AGENTS.md` | Project roles, workspace, delivery model |
| `framework/ARCHITECTURE.md` | Module boundaries, data flow, APIs, seams |
| `framework/DATABASE.md` | Models, relationships, migrations |
| `framework/DESIGN.md` | Visual token spec (Google DESIGN.md format) |
| `framework/TESTING.md` | Testing strategy — pyramid, states, evidence, E2E |
| `framework/DEBUGGING.md` | Structured logging, error traceability, quick fixes |
| `framework/VERSION-CONTROL.md` | Git workflow, branches, PRs, CI/CD, code review |
| `framework/COMMIT-TEMPLATE.md` | Commit format + learnings extraction |
| `framework/SUBAGENT.md` | Inter-agent contract — vocabulary, checklist, handoff |
| `framework/HERMES.md` | Hermes operations — dispatcher, spawning, worktrees |
| `framework/LEARNINGS.md` | Accumulated knowledge from retrospectives |
| `checklists/` | Dynamic per-phase checklists (PRD, Spec, Code, Ship) |

---

## Principles

- **The deletion test.** If removing a module spreads complexity, it was
  earning its keep.
- **Tracer bullets, not horizontal layers.** Every ticket is a vertical
  slice — demoable on its own.
- **Consistency over novelty.** The codebase reads like one person wrote it.
- **Evidence, not verdicts.** Every test result is backed by artifacts.
- **The reviewer is adversarial.** Assume it's wrong until proven right.
- **Durable state over chat memory.** Everything lives in files.

---

## Tech Stack

| Layer | Tool | Link |
|---|---|---|
| **Orchestrator** | Hermes Agent | [github.com/NousResearch/hermes-agent](https://github.com/NousResearch/hermes-agent) |
| **Implementation** | Codex CLI | [github.com/openai/codex](https://github.com/openai/codex) |
| **Quality gates** | Husky | [github.com/typicode/husky](https://github.com/typicode/husky) |
| **Testing** | Playwright | [playwright.dev](https://playwright.dev) |
| **Design** | Google DESIGN.md | [github.com/google-labs-code/design.md](https://github.com/google-labs-code/design.md) |
| **Specs** | Spec Kit | [github.com/github/spec-kit](https://github.com/github/spec-kit) |
| **Deploy** | Vercel | [vercel.com](https://vercel.com) |
| **Research** | Feynman | [github.com/companion-inc/feynman](https://github.com/companion-inc/feynman) |
| **Book extraction** | book-to-skill | [github.com/virgiliojr94/book-to-skill](https://github.com/virgiliojr94/book-to-skill) |
| **Backlog mgmt** | ClawSweeper | [github.com/openclaw/clawsweeper](https://github.com/openclaw/clawsweeper) |
| **Auto-review** | ClawPatch | [github.com/openclaw/clawpatch](https://github.com/openclaw/clawpatch) |

---

## Quick Start

```bash
# 1. Clone the harness
git clone https://github.com/Ntrakiyski/autonomous-harness.git

# 2. Copy it into your project
cp -r autonomous-harness/ your-project/autonomous-harness/

# 3. Ask your AI agent:
"What is autonomous-harness and what are my first steps?"
```

The agent will read ROLE.md, understand the framework, and guide you
through filling in the project-specific templates.

---

## Startup Sequence

Every agent reads these files in order at session start:

```
1. ROLE.md           — How to become this agent
2. AGENTS.md         — Who I am, what I can touch
3. ARCHITECTURE.md   — How the system is built
4. DATABASE.md       — How data is stored
5. DESIGN.md         — How things look
6. TESTING.md        — How we test
7. VERSION-CONTROL.md — How we use git
8. COMMIT-TEMPLATE.md — How we write commits
9. SUBAGENT.md       — How we work together
10. DEBUGGING.md     — How we debug
11. LEARNINGS.md     — What we've learned
12. AUTONOMOUS.md    — The full pipeline contract
```

---

## Academic Foundation

Pipeline design validated through Feynman PaperRank against 50+ papers.
Key findings: CI/CD empirically improves software quality; multi-agent
architectures are an emerging field with no established git workflows —
this harness pioneers agent-specific version control patterns.

---

## Inspired By

- [Matt Pocock's skills](https://github.com/mattpocock/skills) (169K ⭐) — `codebase-design`, `to-spec`, `to-tickets`, `implement`, `tdd`, `code-review`, `grill-me`
- [Google DESIGN.md](https://github.com/google-labs-code/design.md) — Machine-readable visual token spec
- [Clean Code](https://www.amazon.com/Clean-Code-Handbook-Software-Craftsmanship/dp/0132350882) (Robert C. Martin) — Extracted as a Hermes skill via [book-to-skill](https://github.com/virgiliojr94/book-to-skill)
- [Feynman](https://github.com/companion-inc/feynman) (Companion Inc.) — Research agent for paper ranking and synthesis
- [ClawSweeper](https://github.com/openclaw/clawsweeper) / [ClawPatch](https://github.com/openclaw/clawpatch) (OpenClaw) — Automated backlog cleanup and PR management
- [Skills Hub](https://skills.sh) — 872K+ agent skills, by Vercel
- [ClawHub](https://clawhub.ai) — Skills and plugins marketplace, by OpenClaw
- [Husky](https://github.com/typicode/husky) — Git hooks for pre-commit quality gates
- [Playwright](https://playwright.dev) — E2E testing for the modern web
- [50 Developer Acronyms](https://dev.to/eleftheriabatsou/50-acronyms-developers-use-6kl) — Vocabulary that shaped the subagent contract

---

## License

MIT — use it, modify it, ship with it.
