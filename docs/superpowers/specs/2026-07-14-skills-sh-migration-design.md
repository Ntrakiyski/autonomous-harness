# Autonomous Harness Skills.sh Migration Design

## Goal

Turn Autonomous Harness from a clone-first documentation template into a
project-local, installable skill package. A user adds the skills to an existing
repository, initializes durable operating state, and runs a disciplined delivery
workflow without copying the harness source tree into their product.

## Chosen Approach

Use a hybrid model.

- **Skills** hold reusable procedures: they decide what to read, what to check,
  and which artifact to produce.
- **Project state** holds facts that vary by product: scope, architecture,
  design, phase records, evidence, and retrospectives.
- **The public repository** remains the canonical source and is installed with
  `npx skills add Ntrakiyski/autonomous-harness -a codex`.

This avoids both rejected extremes: a clone-first template that leaves agents
with static instructions, and a skill-only package that loses auditable
project-specific context.

## User Experience

From an existing product repository:

```text
npx skills add Ntrakiyski/autonomous-harness -a codex
# Ask the agent: initialize autonomous delivery for this project
```

`autonomous-init` inspects the project and creates its initial operating state.
It never overwrites existing project instructions or documentation without
confirmation. It also detects whether Spec Kit has been initialized. Before a
project can use `autonomous-spec`, it must run Spec Kit initialization; that
creates the project-root `.specify/` directory and its managed configuration.
Subsequent skills operate against that state and the project's normal source
tree.

## Target Layout

### Public source repository

```text
autonomous-harness/
├── skills/
│   ├── autonomous-init/
│   │   ├── SKILL.md
│   │   ├── agents/openai.yaml
│   │   └── assets/project-state/
│   ├── autonomous-spec/
│   │   ├── SKILL.md
│   │   └── agents/openai.yaml
│   ├── autonomous-deliver/
│   │   ├── SKILL.md
│   │   └── agents/openai.yaml
│   └── autonomous-gate/
│       ├── SKILL.md
│       └── agents/openai.yaml
├── docs/                         # contributor and user documentation
└── README.md                     # install-first entrypoint
```

The skills contain no redundant README files. The initializer's assets contain
only the small templates it must create.

### A consuming product repository

```text
client-product/
├── AGENTS.md                     # thin project entrypoint and local rules
├── .agents/skills/               # project-local installed skills
├── .specify/                     # Spec Kit configuration, templates, and memory
├── specs/                        # Spec Kit-owned feature artifacts
│   └── 001-feature-slug/
│       ├── spec.md
│       ├── plan.md
│       └── tasks.md
├── .autonomous/
│   ├── PROJECT.md                # product intent and scope boundaries
│   ├── CONTEXT.md                # domain vocabulary and source-document map
│   ├── GUARDRAILS.md             # shared delivery safety rules
│   ├── phases/
│   │   └── phase-N.md            # status and pointer to specs/001-feature-slug/
│   ├── evidence/
│   └── retrospectives/
└── docs/                         # existing architecture/design/product docs
```

The initializer references existing architecture, design, and database docs
where they already live. It does not copy them into `.autonomous/`.
`.specify/` and `specs/` are owned by Spec Kit; Autonomous Harness never
duplicates or relocates their files.

## V1 Skills

### `autonomous-init`

- **Trigger:** Set up or adopt Autonomous Harness in an existing repository.
- **Inputs:** Current repository instructions, existing documentation, and user
  answers for missing project facts.
- **Output:** `.autonomous/` state, a source-document map, and a thin `AGENTS.md`
  only when one does not already exist.
- **Spec Kit boundary:** Detect `.specify/`. If it is absent, explain that
  `autonomous-spec` is unavailable until the user initializes Spec Kit in the
  project; never create or edit `.specify/` directly.
- **Safety:** Detect conflicts; ask before overwriting; mark unknown facts as
  unknown.
- **Resources:** A minimal project-state asset template.

### `autonomous-spec`

- **Trigger:** Turn approved scope into one independently shippable phase.
- **Inputs:** Approved scope, project state, and referenced architecture/design
  constraints.
- **Output:** A Spec Kit feature directory at `specs/<number>-<slug>/` with its
  `spec.md`, `plan.md`, and `tasks.md`; a `.autonomous/phases/phase-N.md`
  manifest links to that directory and records status and blockers.
- **Safety:** Reject unresolved scope; do not expand a phase beyond the approved
  scope. Require Spec Kit for this skill. If the project is not initialized,
  report the exact setup blocker rather than generating a competing artifact
  format.

### `autonomous-deliver`

- **Trigger:** Execute one approved phase or ticket.
- **Inputs:** The phase tasks, project instructions, existing patterns, and
  previous blocking handoffs.
- **Output:** The requested implementation, verification evidence, and
  `handoff.json`.
- **Safety:** Read before writing, preserve project conventions, test at seams,
  stop on blocked or unsafe work.

### `autonomous-gate`

- **Trigger:** Review a phase at PRD/spec/code/ship boundaries.
- **Inputs:** The artifact under review, project constraints, learnings, and
  evidence.
- **Output:** A dynamic, phase-specific checklist and an evidence-backed
  pass/fail result.
- **Safety:** A missing artifact or missing evidence is not a pass. The skill
  reports gaps; it does not self-waive them.

## Migration Map

| Current material | V1 destination |
|---|---|
| `framework/ROLE.md` | Split into the initialization and delivery procedures. |
| `AUTONOMOUS.md` | Split across the four skill workflows; retain a concise contributor reference. |
| `framework/PROJECT.md`, `AGENTS.md` | Initializer assets that become product-specific state. |
| Architecture, database, design, and testing templates | Referenced project documents; do not make them always-loaded skill content. |
| Spec Kit phase instructions | Keep `.specify/` and `specs/<number>-<slug>/` as Spec Kit-owned state; record only a link and delivery status in `.autonomous/phases/`. |
| `checklists/` | `autonomous-gate` generation and audit procedure. |
| `SUBAGENT.md` handoff format | `autonomous-deliver` output contract. |
| `GUARDRAILS.md` | Shared guardrail reference loaded by every V1 skill. |
| `README.md` | Replace with install, initialize, and first-phase walkthrough. |

## Validation and Release

1. Create each skill with `skill-creator`'s initializer and generate its
   `agents/openai.yaml` metadata from the final `SKILL.md`.
2. Run `quick_validate.py` for every skill.
3. Install the package in a clean fixture repository using the Skills CLI,
   targeting Codex project scope.
4. Initialize Spec Kit in the fixture and confirm a new feature writes under
   `specs/<number>-<slug>/`, while `.autonomous/phases/` contains only a
   manifest that links to it.
5. Run the initializer without pre-existing instructions and with pre-existing
   instructions; verify no silent overwrite occurs.
6. Forward-test each skill against a small realistic task; check its files,
   handoff, and gate result rather than accepting prose claims.
7. Update the install-first README, then publish a tagged GitHub release.

## Non-Goals

- Do not create one skill per human role.
- Do not require Hermes, Vercel, Playwright, or Spec Kit merely to install.
- Do not emulate or replace Spec Kit's feature-artifact format.
- Do not move or duplicate a consuming project's existing documentation.
- Do not automate deployment in V1.

## Risks and Responses

| Risk | Response |
|---|---|
| Skills become giant copies of the current framework docs. | Keep each `SKILL.md` trigger-specific and concise; use references only when needed. |
| Initializer overwrites existing project conventions. | Inspect first, preserve existing files, and require confirmation for conflicting writes. |
| Generic state conflicts with real repositories. | Treat unknown as unknown and use a source-document map instead of duplicate templates. |
| Tool availability varies by project. | Detect optional tools at runtime and state the fallback or blocker explicitly; `autonomous-spec` blocks when its required Spec Kit dependency is absent. |
| Harness and Spec Kit both create specs. | Spec Kit owns `.specify/` and `specs/`; Harness keeps only phase manifests that point to those files. |

## Implementation Sequence

1. Restructure the repository around `skills/` without changing the existing
   reference material yet.
2. Implement and validate `autonomous-init` plus its minimal asset template.
3. Implement and validate `autonomous-spec`, `autonomous-deliver`, and
   `autonomous-gate` one at a time.
4. Replace clone-first documentation with the Skills.sh install flow.
5. Run the clean-fixture acceptance flow and refine before publishing.
