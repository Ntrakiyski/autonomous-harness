# Autonomous Harness

> Turn agent work into a disciplined, evidence-backed delivery flow inside an
> existing software project.

Autonomous Harness is a project-local skill package, not a project template to
clone. It adds reusable delivery procedures while preserving the product's own
architecture, design, documentation, and source code.

## Install

From the root of an existing project:

```bash
npx skills add Ntrakiyski/autonomous-harness -a codex
```

This installs the four skills in `.agents/skills/` for Codex:

- `autonomous-init` — adopt the Harness and create project state.
- `autonomous-spec` — turn approved scope into a Spec Kit feature.
- `autonomous-deliver` — implement one approved vertical slice with evidence.
- `autonomous-gate` — run an evidence-backed PRD, spec, code, or ship gate.

## First Use

Ask Codex to initialize the project:

```text
Use $autonomous-init to set up autonomous delivery in this project.
```

The initializer inspects existing instructions and documentation, then creates
only Harness-owned state:

```text
.autonomous/
├── PROJECT.md
├── CONTEXT.md
├── GUARDRAILS.md
├── state.json
├── phases/
├── evidence/
└── retrospectives/
```

It never overwrites an existing `AGENTS.md` or project document without asking.

## Spec Kit

`autonomous-spec` requires Spec Kit. Initialize it once in the project before
creating a governed feature:

```bash
specify init --here --force --integration codex --integration-options="--skills"
```

Spec Kit owns `.specify/` and creates feature artifacts under
`specs/<number>-<slug>/`. Harness does not copy or relocate those files; its
phase manifest and lifecycle state link to them.

`--force` permits Spec Kit to merge into the existing project after Harness
skills have been installed. Review the merge output; the setup preserves the
installed Harness skills alongside the Spec Kit skills.

```text
.agents/skills/                 # Autonomous Harness and Spec Kit skills
.autonomous/                    # Harness-owned project and lifecycle state
.specify/                       # Spec Kit configuration, templates, memory
specs/001-feature-slug/         # Spec Kit feature: spec.md, plan.md, tasks.md
```

## Lifecycle State

`.autonomous/state.json` is the current machine-readable lifecycle record. It
contains the Harness initialization state, Spec Kit availability, active
milestone, milestone statuses, feature directories, and latest gate result.

It is current state, not an activity log. Product context remains in
`PROJECT.md`; feature requirements remain in Spec Kit; raw verification output
remains in `.autonomous/evidence/`.

## Delivery Flow

1. Initialize Harness with `autonomous-init`.
2. Approve scope and initialize Spec Kit.
3. Run `autonomous-spec` for one vertical feature.
4. Run `autonomous-deliver` for an approved task or feature slice.
5. Run `autonomous-gate` before progressing it.

The skills maintain durable files so the next agent can continue from evidence
and state rather than chat history.

## Contributing

The public package source lives in `skills/`. Every skill has a concise
`SKILL.md` and generated `agents/openai.yaml`; only `autonomous-init` includes
state-template assets. `autonomous-gate/references/` contains its gate
methodology and mode examples; each real gate still derives checks from the
current project and evidence.

## Framework Reference

[`docs/framework/`](docs/framework/) holds the project-document concepts that
informed the Harness: product context, guardrails, architecture, data, design,
testing, and learnings. They explain the framework; they are not installed by
the skills or required in a consuming project.
