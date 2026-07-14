# Autonomous Harness Skills.sh Migration Implementation Plan

> **For agentic workers:** REQUIRED SUB-SKILL: Use superpowers:subagent-driven-development (recommended) or superpowers:executing-plans to implement this plan task-by-task. Steps use checkbox (`- [ ]`) syntax for tracking.

**Goal:** Publish Autonomous Harness as four project-local skills that create and operate durable project state without duplicating Spec Kit artifacts.

**Architecture:** Procedural knowledge lives in four `skills/*/SKILL.md` folders. Harness owns `.autonomous/`, including the current lifecycle record in `state.json`; Spec Kit owns `.specify/` and `specs/<number>-<slug>/`; phase manifests link the two without copying artifacts.

**Tech Stack:** Markdown, YAML, `skill-creator`, Skills CLI, GitHub Spec Kit.

## Global Constraints

- Install from an existing repository with `npx skills add Ntrakiyski/autonomous-harness -a codex`.
- Keep each skill concise, trigger-specific, and under 500 lines.
- Generate `agents/openai.yaml` from every finalized skill with `skill-creator`.
- Never create, relocate, or duplicate `.specify/` or `specs/` files.
- Never silently overwrite existing project files.
- Treat `.autonomous/state.json` as current lifecycle truth; do not use it as an append-only log.
- Do not add a runtime, package dependency, deployment automation, or a skill per human role.

## File Structure

```text
skills/
├── autonomous-init/
│   ├── SKILL.md
│   ├── agents/openai.yaml
│   └── assets/project-state/{AGENTS,PROJECT,CONTEXT,GUARDRAILS,phase-manifest}.md
├── autonomous-spec/{SKILL.md,agents/openai.yaml}
├── autonomous-deliver/{SKILL.md,agents/openai.yaml}
└── autonomous-gate/{SKILL.md,agents/openai.yaml}
tests/fixtures/skills-consumer/README.md
README.md
```

Keep `AUTONOMOUS.md`, `framework/`, and `checklists/` as contributor references in this release.

### Task 1: Add `autonomous-init`

**Files:**

- Create: `skills/autonomous-init/SKILL.md`
- Create: `skills/autonomous-init/agents/openai.yaml`
- Create: `skills/autonomous-init/assets/project-state/AGENTS.md`
- Create: `skills/autonomous-init/assets/project-state/PROJECT.md`
- Create: `skills/autonomous-init/assets/project-state/CONTEXT.md`
- Create: `skills/autonomous-init/assets/project-state/GUARDRAILS.md`
- Create: `skills/autonomous-init/assets/project-state/phase-manifest.md`

**Produces:** `.autonomous/{PROJECT,CONTEXT,GUARDRAILS}.md`, `.autonomous/state.json`, `.autonomous/{phases,evidence,retrospectives}/`, and root `AGENTS.md` only when absent.

- [x] Initialize the skill with `skill-creator`, choosing `assets` as its only resource directory and creating UI metadata for “Autonomous Init.”
- [x] Write a body that reads existing instructions, maps existing project documents, inspects `AGENTS.md`, `.autonomous/`, `.specify/`, and `specs/`, preserves all existing files, asks before conflicts, creates Harness state and `state.json`, and reports whether Spec Kit is initialized without creating `.specify/`.
- [x] Initialize `state.json` with `schema_version: 1`, a `harness` object containing `initialized_at`, `skills_version`, and `spec_kit_initialized`, and a `project` object containing `status`, `active_milestone`, and `milestones`. Set `status` to `initialized`, `active_milestone` to `null`, and `milestones` to an empty array.
- [x] Create assets with these headings: `PROJECT.md`: Problem, Users, Outcomes, Out of Scope, Open Questions; `CONTEXT.md`: Source Documents, Domain Vocabulary, Architecture and Design References, Known Constraints; `GUARDRAILS.md`: data/secrets, destructive actions, approved scope, evidence, unknown facts; `phase-manifest.md`: Feature Directory, Status, Blockers, Evidence, Handoff.
- [x] Run `quick_validate.py` for `skills/autonomous-init`, check that `agents/openai.yaml` and `assets/project-state/GUARDRAILS.md` exist, and expect `Skill is valid!`.
- [x] Commit with `feat(skills): add project initializer`.

### Task 2: Add `autonomous-spec`

**Files:**

- Create: `skills/autonomous-spec/SKILL.md`
- Create: `skills/autonomous-spec/agents/openai.yaml`

**Consumes:** approved scope, Harness state, project references, and `.specify/`.

**Produces:** `specs/<number>-<slug>/{spec,plan,tasks}.md` through Spec Kit and `.autonomous/phases/phase-N.md` pointing to that directory.

- [x] Initialize the skill with `skill-creator` and UI metadata for “Autonomous Spec.”
- [x] Write a body that reads project state and approved scope, blocks if scope is unresolved or `.specify/` is missing, runs constitution → specify → clarify → plan → tasks → analyze through installed Codex Spec Kit skills, verifies `spec.md`, `plan.md`, and `tasks.md` in one `specs/` feature directory, writes its exact path and blockers into a phase manifest, then adds that path to the active milestone and sets the milestone status to `in_progress`.
- [x] State explicitly that it never writes a second spec, plan, or task format under `.autonomous/`.
- [x] Run `quick_validate.py`, verify `agents/openai.yaml`, expect `Skill is valid!`, and commit with `feat(skills): add Spec Kit bridge`.

### Task 3: Add `autonomous-deliver`

**Files:**

- Create: `skills/autonomous-deliver/SKILL.md`
- Create: `skills/autonomous-deliver/agents/openai.yaml`

**Consumes:** active `specs/<number>-<slug>/tasks.md`, project instructions, and blocking handoffs.

**Produces:** the requested vertical slice, raw evidence under `.autonomous/evidence/`, `handoff.json`, and an updated phase manifest.

- [x] Initialize the skill with `skill-creator` and UI metadata for “Autonomous Deliver.”
- [x] Write a body that requires the agent to read the active feature, guardrails, instructions, blockers, and current `state.json`; confirm one in-scope vertical slice; inspect existing patterns; plan seam-level tests; implement the smallest compliant change; retain raw logs, screenshots, or traces; update the phase manifest; and set the active milestone to `blocked` only when its handoff is blocked.
- [x] Specify this exact `handoff.json` contract: `ticket_id`, `status`, `changed_files`, `tests_run`, `tests_passed`, `decisions`, `blockers`, `needs_review`, and `evidence_paths`.
- [x] Run `quick_validate.py`, verify `agents/openai.yaml`, expect `Skill is valid!`, and commit with `feat(skills): add delivery workflow`.

### Task 4: Add `autonomous-gate`

**Files:**

- Create: `skills/autonomous-gate/SKILL.md`
- Create: `skills/autonomous-gate/agents/openai.yaml`

**Consumes:** a PRD, Spec Kit feature, implementation diff, or ship candidate plus the active phase manifest and evidence.

**Produces:** `gate-prd.md`, `gate-spec.md`, `gate-code.md`, or `gate-ship.md` beside the phase manifest, with a pass, fail, or blocked result.

- [x] Initialize the skill with `skill-creator` and UI metadata for “Autonomous Gate.”
- [x] Write a body that selects one of four modes, reads the relevant artifact and project constraints, derives specific checks from them, writes a dynamic checklist, links every pass to a source path or evidence file, treats missing evidence as fail, updates the phase manifest, and records `last_gate` in `state.json`. A passed ship gate sets the active milestone to `complete`; a passed earlier gate sets it to `ready_for_gate`.
- [x] State explicitly that this skill never deploys, merges, or waives a failed check.
- [x] Run `quick_validate.py`, verify `agents/openai.yaml`, expect `Skill is valid!`, and commit with `feat(skills): add evidence gate`.

### Task 5: Replace Clone-First Documentation

**Files:**

- Modify: `README.md`
- Modify: `docs/superpowers/specs/2026-07-14-skills-sh-migration-design.md`

- [x] Replace the quick start with the following commands:

```bash
cd existing-product
npx skills add Ntrakiyski/autonomous-harness -a codex
# Ask Codex: Use $autonomous-init to set up autonomous delivery in this project.
specify init --here --force --integration codex --integration-options="--skills"
```

- [x] Explain exact ownership: Harness installs in `.agents/skills/`; initialization creates `.autonomous/`; Spec Kit initialization creates `.specify/`; Spec Kit feature commands create `specs/<number>-<slug>/`; phase manifests only link to those artifacts.
- [x] Run this contradiction check before committing:

```bash
rg -n -i 'clone the harness|per-phase.*\.specify|\.autonomous/phases/.+spec\.md|fallback.*Spec Kit' README.md
```

Expected: no outdated clone-first or competing-artifact claim.

- [x] Commit with `docs(skills): document install-first workflow`.

### Task 6: Verify Package Discovery and Artifact Boundaries

**Files:**

- Create: `tests/fixtures/skills-consumer/README.md`

- [x] Create the fixture readme with this exact content: `# Skills Consumer Fixture`, followed by one paragraph explaining that it verifies project-local Autonomous Harness installation and its separation from Spec Kit artifacts.
- [x] From the repository root, run `quick_validate.py` for `autonomous-init`, `autonomous-spec`, `autonomous-deliver`, and `autonomous-gate`; expect four `Skill is valid!` lines.
- [x] From `tests/fixtures/skills-consumer`, run `npx skills add ../../.. --list`, then install the four named skills with `npx skills add ../../.. --skill autonomous-init --skill autonomous-spec --skill autonomous-deliver --skill autonomous-gate -a codex -y`, then run `find .agents/skills -maxdepth 2 -name SKILL.md | sort`.
- [x] Expect all four skills in the list and four project-local `SKILL.md` files after installation.
- [x] Run `specify init --here --force --integration codex --integration-options="--skills"`; verify `.specify/` and `.agents/skills/` exist, while `.autonomous/specs` and `.specify/autonomous` do not exist.
- [x] Use `autonomous-init` and `autonomous-spec` on a small approved feature; verify that the phase manifest is under `.autonomous/phases/`, feature documents are under `specs/`, and `state.json` records the active milestone and feature directory.
- [x] Commit with `test(skills): add installation fixture`.

## Plan Self-Review

- Spec coverage: Tasks 1–4 create the approved skills; Task 5 supplies the install-first flow; Task 6 proves discovery, Codex installation, and Spec Kit separation.
- Placeholder scan: every created file, command, artifact, and expected outcome is named.
- Interface consistency: `autonomous-init` creates Harness state; Spec Kit creates feature artifacts; later skills update phase manifests, evidence paths, and only their owned lifecycle fields in `state.json`.
