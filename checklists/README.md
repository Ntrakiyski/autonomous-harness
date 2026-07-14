# Checklist Methodology

> **Checklists are not static templates — they are dynamically generated
> per phase, per milestone, per project.** This file describes HOW to
> generate them, not what they contain.

## Why Dynamic Checklists

Hardcoded checklists rot. Every project has different requirements.
Every phase builds on previous learnings. A checklist that worked for
Phase 1 will be wrong for Phase 3 because:

- The architecture evolved (new modules, new seams)
- The team learned things (see LEARNINGS.md)
- The PRD added scope that the old checklist doesn't cover
- Bugs from previous phases revealed new failure modes

Dynamic checklists are generated FRESH for each gate by reading the
current state of the project and asking: "What must be true before we
can proceed?"

## Generation Principles

A good checklist item is:

- **Verifiable.** You can look at an artifact and answer yes/no. "The
  code is clean" is unverifiable. "Every function is under 20 lines" is
  verifiable.
- **Specific.** Names exact files, exact commands, exact values. "API
  docs exist" is vague. "ARCHITECTURE.md lists all endpoints with
  request/response bodies" is specific.
- **Grounded.** Derived from an actual source: the PRD, the architecture
  decisions, the DESIGN.md tokens, the LEARNINGS.md patterns. No
  checklist item should come from thin air.
- **Depth-aware.** Doesn't just check surface-level completeness ("is
  there a PRD?") but depth ("does the PRD define seams? edge cases?
  error states?").
- **Phase-scoped.** Only checks things relevant to the current phase.
  Don't check Phase 3 concerns during a Phase 1 gate.

## Generation Process

For each gate in the pipeline, follow this process:

### Step 1: Identify the Gate's Purpose

What is this gate protecting against? What happens if we proceed without
checking?

| Gate | Protects Against |
|---|---|
| PRD Gate | Building the wrong thing. Incomplete requirements. Unanswered questions. |
| Spec Review Gate | Building from broken specs. Missing dependencies. Scope drift from PRD. |
| Code Review Gate | Architecture violations. Pattern drift. Untested code. Scope creep. |
| Ship Gate | Bugs reaching production. Missing evidence. Skipped sign-offs. |

### Step 2: Read the Current State

Before generating a checklist, read:

1. **The PRD** (for PRD gate) or **the specs** (for spec review) or
   **the code** (for code review)
2. **ARCHITECTURE.md** — current module boundaries, seams, API contracts
3. **DESIGN.md** — current visual tokens and component specs
4. **LEARNINGS.md** — what went wrong in previous phases
5. **SUBAGENT.md** — consistency rules that must be enforced

### Step 3: Derive Checklist Items from Sources

For each source, ask: "What must be true about the artifact I'm reviewing?"

#### Deriving from the PRD
- Every scope item → one checklist item: "Is [scope item] addressed?"
- Every user story → one item: "Does the spec cover [user story]?"
- Every edge case → one item: "Is [edge case] tested?"

#### Deriving from ARCHITECTURE.md
- Every module boundary → "Does the implementation respect [boundary]?"
- Every API contract → "Are all endpoints implemented per [contract]?"
- Every seam → "Are tests written at [seam]?"

#### Deriving from DESIGN.md
- Every color token → "Is [color] used, not a raw hex?"
- Every component → "Does [component] match DESIGN.md spec?"
- Typography → "Are font sizes per DESIGN.md?"

#### Deriving from LEARNINGS.md
- Every "Never Again" → "Is [mistake] prevented?"
- Every recurring pattern → "Does the pattern mitigation hold?"

#### Deriving from SUBAGENT.md
- Every consistency rule → "Does the code follow [rule]?"
- Every acronym rule → "Is [DRY/KISS/SRP/etc.] satisfied?"

### Step 4: Add Phase-Specific Depth

Generic checks are shallow. Deepen them for this specific phase:

| Instead of... | Ask... |
|---|---|
| "Are there tests?" | "Do tests cover the new seams introduced in this phase's ARCHITECTURE.md?" |
| "Does the UI look right?" | "Does every new page use the `button-primary` component from DESIGN.md, not a custom button?" |
| "Is the code consistent?" | "Do new files follow the pattern established in `src/pages/home/` from Phase 1?" |
| "Are there edge cases?" | "Is the empty state from Phase 1's dashboard reused, not reinvented?" |

### Step 5: Add Gate-Specific Sign-Off

Every checklist ends with:
- [ ] Operator review of all items
- [ ] Date and signature recorded in the checklist file
- [ ] Link to the artifact being checked (PRD path, spec path, branch name)

## Checklist File Format

```
# [Gate Name] — Milestone [N], Phase [N]
Generated: [YYYY-MM-DD] | Operator: [name] | Artifact: [path/url]

## Source Documents Read
- PRD: prd/milestone-N-prd.md
- Architecture: framework/ARCHITECTURE.md
- Design: framework/DESIGN.md
- Learnings: framework/LEARNINGS.md

## Checks

### Derived from PRD
- [ ] [item]
- [ ] [item]

### Derived from Architecture
- [ ] [item]
- [ ] [item]

### Derived from Design
- [ ] [item]
- [ ] [item]

### Derived from Learnings
- [ ] [item]
- [ ] [item]

### Derived from Consistency Rules
- [ ] [item]
- [ ] [item]

### Phase-Specific Depth
- [ ] [item]
- [ ] [item]

## Sign-Off
- [ ] All checks passed or waived with documented reason
- [ ] Operator: ________ | Date: ________
```

## Example Checklist

See `prd-gate.md` for a concrete example generated using this methodology.
