# Autonomous Delivery Flow

A role-based pipeline that turns a closed client scope into production
software, keeping the product consistent and the developer inside clear lines.

## Roles

- **Manager**: owns scope, price, time, and deliverables. Negotiates with the
  client until scope and expected deliverables are closed and signed off in a
  PDF.
  - *State*: client-facing negotiation; active until scope is signed.
  - *Deliverables*: signed closed-scope PDF with agreed price, time, and
    expected deliverables.
- **Planner**: takes the closed-scope PDF, strips client high-level language
  (without dropping any scope), and drives creation of an accurate PRD.
  - *State*: owns the PRD; coordinates Architect and Designer input.
  - *Deliverables*: one consolidated, phase-ready PRD (no client language,
    full scope preserved).
- **Architect**: defines what happens behind the scenes — data flow,
  APIs, databases, boundaries, and technical limitations.
  - *State*: technical authority; consulted by Developer throughout the build.
  - *Deliverables*: PRD technical section — data flow, API contracts, database
    models, boundaries, and limitations.
- **Designer**: defines what users see and can do, and keeps the product
  consistent so it feels like one app, not a mix of styles. Produces and
  maintains the DESIGN.md token spec as the single source of visual truth.
  - *State*: design authority; consulted by Developer throughout the build.
  - *Deliverables*: PRD UX/UI section — screens, user actions, consistency
    rules against the current app. DESIGN.md token file (colors, typography,
    spacing, components) with validated WCAG contrast. Tailwind/DTCG exports.
- **PreDocumentation**: splits the PRD into phases and runs the required Spec
  Kit commands to produce specs and tasks. This step must use Spec Kit.
  - *State*: owns phasing and Spec Kit artifacts between PRD and build.
  - *Deliverables*: phase breakdown plus Spec Kit specs with dependency-ordered
    tasks.
- **Developer**: implements from the specs/tasks. May ask the Designer,
  Architect, or other roles questions at any time. One developer is
  enough for now.
  - *State*: active during the build; signals done when tasks are complete.
  - *Deliverables*: implemented code on a delivery branch, ready for staging.
- **Tester**: runs E2E and backend tests, compares results against
  expectations, and sends work back to the Developer to fix if needed.
  - *State*: active after staging deploy; gates progression to production.
  - *Deliverables*: test results with pass/fail verdict against expectations
    and fix requests when needed. All test evidence (screenshots, logs,
    traces) is written to `evidence/` files so every verdict is auditable
    without replaying the tests.
- **DevOps**: deploys to staging on the Developer's request, then to production
  (main) once the Tester confirms everything is good.
  - *State*: on-call for deploys; the only role that ships.
  - *Deliverables*: staging deployment for testing and production (main)
    deployment after Tester sign-off.

## Pipeline

1. **Close scope** — Manager locks scope, price, time, and
   deliverables with the client into a signed PDF.
2. **Translate to PRD** — Planner + Architect + Designer turn the
   PDF into one high-level PRD covering: features, design, what users see and
   can do, what happens behind the scenes, data flow, APIs, and databases.
   It must carry the full requirements and limitations so the Developer stays
   inside the lines and consistent with the current app. Designer produces
   DESIGN.md alongside the PRD.
3. **PRD Gate** — Before the PRD proceeds to phasing, it must pass the
   checklist in `checklists/prd-gate.md` (dynamically generated per phase
   using `checklists/README.md`). Every item must be explicitly checked.
   The Operator or CEO signs off. No "TBD", "later", or unanswered critical
   questions remain. If the gate fails, the PRD returns to step 2.
4. **Phase + spec** — PreDocumentation splits the PRD into phases, then uses
   Spec Kit commands to produce specs with tasks.
5. **Review gate** — The operator reviews Spec Kit output before it reaches
   the Developer. This gate checks: are all requirements covered? Are
   dependency chains correct? Is scope preserved from the PRD? If the gate
   fails, PreDocumentation fixes and resubmits. The gate also applies
   between Developer → Tester (is the implementation ready for testing?)
   and Tester → DevOps (did all tests pass with evidence?).
6. **Build** — Developer implements from the tasks, asking other roles when
   unclear.
7. **Test on staging** — Developer signals done; DevOps pushes to staging;
   Tester runs E2E + backend tests and compares against expectations.
8. **Fix loop** — Tester returns issues to the Developer until all checks pass.
9. **Ship** — Once the Tester approves, DevOps deploys to production (main).
10. **Retrospective** — After the phase ships, the operator writes a
   `retrospectives/phase-N.md` covering: what worked, what didn't, which
   skills or tools need updating, and whether the pipeline itself needs
   adjustment before the next phase. Skills are patched immediately; pipeline
   changes are proposed for CEO approval.

## Guardrails

- Scope is fixed before the PRD; the PRD may clarify but must not skip scope.
- The PRD is the single source of requirements and limitations for the phase.
- Consistency with the current app state is a hard requirement, not a
  preference.
- Spec Kit is mandatory for turning phases into specs and tasks.
- Only DevOps deploys; staging comes before production, and production only
  after the Tester signs off.

## Checklists

Every pipeline step has a corresponding checklist in `checklists/`. These
are not hardcoded — they are dynamically generated per phase using the
methodology described in `checklists/README.md`. The operator runs the
checklist for each gate before allowing the pipeline to proceed.

- **PRD Gate:** `checklists/prd-gate.md` — validates the PRD before phasing
- **Review Gate (Specs):** `checklists/spec-review.md` — validates Spec Kit output
- **Review Gate (Code):** `checklists/code-review.md` — validates implementation
- **Ship Gate:** `checklists/ship-gate.md` — validates before production deploy

To generate a checklist: read `checklists/README.md` for the methodology,
then create the specific checklist file for the current phase and milestone.

## Principles

Design principles that hold the pipeline together. Every role and skill
speaks this language.

### Core

- **The deletion test.** Imagine removing a module. If complexity vanishes, it
  was a pass-through. If complexity reappears across N callers, it was earning
  its keep. Apply this to every architectural choice.
- **The interface is the test surface.** Callers and tests cross the same
  seam. If you need to test *past* the interface, the module is probably the
  wrong shape.
- **One adapter = hypothetical seam. Two = real.** Don't introduce a seam
  unless something actually varies across it. New seams are a cost, not a
  default.
- **Depth over surface area.** Prefer deep modules: a lot of behaviour behind
  a small interface. Shallow modules (interface nearly as complex as the
  implementation) are a smell.

### Delivery

- **Tracer bullets, not horizontal layers.** Every ticket cuts a narrow but
  complete path through every layer (schema → API → UI → tests). A completed
  slice is demoable on its own. Never batch by layer ("all APIs first, then
  all UI").
- **Consistency feels like one team.** The whole application must feel like
  one product, not a mix of mini-applications. If the homepage uses 3 files
  with a specific syntax pattern, the about page must follow the same pattern.
  Naming, formatting, file structure, and style are uniform across every agent
  and every ticket. No exceptions.
- **Blocking edges are explicit.** Every ticket declares which other tickets
  gate it. No ticket starts before its blockers are done. This eliminates
  coordination overhead between agents.
- **Durable state over chat memory.** Decisions, progress, test evidence, and
  handoffs live in files (PROGRESS.md, evidence/, retrospectives/). No agent
  should need chat history to understand the current state.
- **Scope is closed before code.** The PRD freezes requirements. Specs refine
  them but never expand them. Developer stays inside the lines.

### Quality

- **Evidence, not verdicts.** Every test result is backed by a written
  artifact (screenshot, log, trace) in `evidence/`. A "pass" without evidence
  is not a pass.
- **The reviewer is adversarial.** Review gates assume the work is wrong until
  proven otherwise. Grill the interface, the tests, and the assumptions — not
  just the diff.
- **Retrospective is mandatory.** After every phase, write what worked, what
  didn't, and what to change. Patch skills immediately. Pipeline changes are
  proposed to the CEO.

### Team Discipline (Agentic)

These are the same principles a real dev team follows — adapted for a team
of AI agents controlled by an operator.

- **Coding conventions.** Naming, formatting, file structure, and style are
  uniform across all agents. No agent invents its own patterns. If Agent A
  writes `getUserById`, Agent B never writes `fetch_user`. The codebase
  reads like one author wrote it.
- **Architecture principles.** Modules, seams, adapters, and boundaries follow
  the shared vocabulary (see Vocabulary section). The Architect's decisions
  are binding — Developer agents don't redesign the architecture mid-ticket.
- **Code quality.** Readable over clever. Simple over complex. Small changes
  over large ones. Every function does one thing. Every class has one
  responsibility. Every commit is atomic and reviewable.
- **Testing discipline.** Unit tests at every seam. Integration tests across
  boundaries. E2E tests for critical paths. Tests are written before or with
  the code (TDD), never after. Evidence goes in `evidence/`.
- **Code review standards.** Every change is reviewed before merge. The
  reviewer is adversarial: assume it's wrong until proven right. Check the
  interface, not just the diff. Check consistency with the existing codebase.
- **Documentation.** Enough context for the next agent to pick up without
  reading chat history. CONTEXT.md holds the domain glossary. ADRs hold
  irreversible decisions. PROGRESS.md holds verified status.
- **Version control workflow.** One delivery branch per phase. Atomic commits
  with meaningful messages. PRs are reviewed before merge. Only DevOps
  touches `main`. Full strategy in `framework/VERSION-CONTROL.md`.
- **Communication.** Agents communicate through the board (kanban comments,
  handoff metadata), not through chat. Blockers surface immediately.
  Expectations are explicit in ticket bodies.
- **Ownership.** Every ticket has one owner (the assigned role). Every bug
  belongs to the Developer who wrote the code. Every architectural decision
  belongs to the Architect. The Operator owns the pipeline.
- **Definition of done.** Code is written, tested, reviewed, and merged.
  Tests pass with evidence. The review gate is cleared. The retrospective
  is written. Nothing is "done" until the evidence exists.
- **Feedback loops.** Frequent demos (every tracer bullet). Retrospectives
  after every phase. Skills patched immediately when something breaks.
  Pipeline adjusted when patterns emerge across retrospectives.
- **Technical excellence.** Good design over fast delivery. Sustainable pace
  over crunch. Regular refactoring over accumulated debt. Deep modules over
  shallow ones.

## Vocabulary

A shared language used across all roles, skills, and artifacts. Consistent
terms eliminate translation overhead between agents.

| Term | Definition |
|---|---|
| **Module** | Anything with an interface and an implementation: a function, class, package, or cross-cutting slice. Scale-agnostic. |
| **Interface** | Everything a caller must know to use the module: type signatures, invariants, ordering constraints, error modes, performance characteristics. |
| **Implementation** | What's inside a module — its body of code. Distinct from adapter (which describes role, not substance). |
| **Depth** | Leverage at the interface: how much behaviour a caller can exercise per unit of interface they must learn. A deep module has a small interface and large implementation. |
| **Seam** | A place where behaviour can change without editing in that place — the location where a module's interface lives. |
| **Adapter** | A concrete thing that satisfies an interface at a seam. Describes role (what slot it fills), not what's inside. |
| **Leverage** | What callers gain from depth: more capability per unit of interface learned. One implementation pays back across N call sites and M tests. |
| **Locality** | What maintainers gain from depth: change, bugs, knowledge, and verification concentrate in one place instead of spreading. Fix once, fixed everywhere. |
| **Tracer bullet** | A vertical slice through every layer that is demoable on its own. The opposite of a horizontal layer ticket ("all APIs"). |
| **Blocking edge** | An explicit dependency: ticket B is blocked by ticket A → B cannot start until A is done. |
| **PRD** | Product Requirements Document. The single source of truth for scope, produced by Planner + Architect + Designer from the signed scope PDF. |
| **DESIGN.md** | Google's open token spec: a YAML + Markdown file that defines the entire visual identity (colors, typography, spacing, components) as machine-readable tokens. Agents read it to produce consistent UI. Lintable for WCAG contrast. |

## Skills

Every role loads a specific set of skills. Skills live in the hub
(https://skills.sh) and are installed per agent profile.

### Role → Skill Mapping

| Role | Skills | Source |
|---|---|---|
| **Planner** | `domain-modeling`, `to-spec` | mattpocock/skills |
| **Architect** | `codebase-design`, `domain-modeling` | mattpocock/skills |
| **Designer** | `codebase-design`, `frontend-design`, `design-md` | mattpocock + anthropics + Google |
| **PreDocumentation** | `to-tickets` + Spec Kit CLI | mattpocock + project-local |
| **Developer** | `implement`, `tdd`, `codebase-design`, `diagnosing-bugs` | mattpocock/skills |
| **Tester** | `code-review`, `grill-me`, `playwright-best-practices`, `webapp-testing` | mattpocock + Hermes builtin |
| **DevOps** | `github-pr-workflow`, project deploy scripts | Hermes builtin + project-local |
| **Operator (meta)** | `kanban-orchestrator`, `plan`, `spike`, `systematic-debugging`, `simplify-code` | Hermes builtin |

### Skill Categories

| Category | Skills | Purpose |
|---|---|---|
| **Design** | `codebase-design`, `domain-modeling` | Shared vocabulary, deep modules, seams |
| **Planning** | `to-spec`, `to-tickets`, `plan` | PRD → specs → tracer-bullet tickets |
| **Build** | `implement`, `tdd`, `prototype`, `research` | Red-green-refactor, spike before build |
| **Review** | `code-review`, `grill-me`, `grill-with-docs`, `requesting-code-review` | Adversarial review at every gate |
| **Debug** | `diagnosing-bugs`, `systematic-debugging` | Root cause before fix |
| **Quality** | `playwright-best-practices`, `webapp-testing`, `improve-codebase-architecture` | E2E tests, evidence, architecture health |
| **Orchestration** | `kanban-orchestrator`, `kanban-worker`, `simplify-code` | Multi-agent routing, parallel cleanup |

## Toolchain

Which tools each role uses, mapped to concrete CLI commands or agent
delegation targets.

| Role | Primary Tool | Fallback |
|---|---|---|
| **Planner** | Claude Code / Codex (analysis mode) | Direct agent reasoning |
| **Architect** | Codex + `codebase-design` skill | Claude Code |
| **Designer** | Codex + `frontend-design` skill | Claude Code |
| **PreDocumentation** | Spec Kit CLI (`specify`) | `to-tickets` skill |
| **Developer** | Codex CLI (`codex exec`) + Husky (pre-commit) | Claude Code CLI |
| **Tester** | Playwright (`npx playwright test`) + Husky | `webapp-testing` skill |
| **DevOps** | Vercel CLI / `gh` CLI / git | Manual deploy via CEO |
| **Operator** | Hermes Agent (kanban, cron, delegate) | Direct file ops |

## Artifacts

Every pipeline step produces concrete, auditable files. Nothing exists only
in chat memory.

```text
milestone3/
├── scope/                         # Manager output
│   └── signed-scope.pdf
├── prd/                           # Planner + Architect + Designer output
│   ├── milestone-N-prd.md
│   └── DESIGN.md                  # Visual token spec (Designer, per milestone)
├── specs/                         # PreDocumentation output (Spec Kit)
│   ├── phase-1/
│   │   ├── spec.md
│   │   └── tasks.md
│   └── phase-2/
├── evidence/                      # Tester output (per phase)
│   ├── phase-1/
│   │   ├── e2e-dashboard.png
│   │   ├── api-tests.log
│   │   └── test-verdict.md
│   └── phase-2/
├── retrospectives/                # Operator output (per phase)
│   ├── phase-1.md
│   └── phase-2.md
├── framework/
│   ├── ROLE.md                    # Agent bootstrap prompt (project-agnostic)
│   ├── AGENTS.md                  # Project-specific agent instructions
│   ├── ARCHITECTURE.md            # Module boundaries, data flow, APIs, seams
│   ├── DATABASE.md                # Models, relationships, migrations
│   ├── DESIGN.md                  # Visual token spec (Google format)
│   ├── SUBAGENT.md                # Inter-agent contract
│   ├── LEARNINGS.md               # Accumulated knowledge from retrospectives
│   └── README.md                  # Framework usage guide
├── checklists/                    # Dynamically generated per-phase checklists
│   ├── README.md                  # Methodology for generating checklists
│   ├── prd-gate.md                # PRD Gate checks
│   ├── spec-review.md             # Spec Review Gate checks
│   ├── code-review.md             # Code Review Gate checks
│   └── ship-gate.md               # Ship Gate checks
├── AUTONOMOUS.md                  # This file — the pipeline contract
```

**Artifact rules:**
- Every artifact is written before the next step starts.
- The review gate checks that artifacts exist and are complete.
- Test evidence is never summarized — raw screenshots and logs go in `evidence/`.
- Retrospectives are written within 24 hours of shipping the phase.

## Anti-Patterns

What NOT to do. If you catch yourself doing one of these, stop and return to
the pipeline.

| Anti-Pattern | Why it's wrong | The right way |
|---|---|---|
| **Skipping the review gate** | Unreviewed specs or code reach the next role, amplifying errors | Every handoff goes through the gate. No exceptions. |
| **Writing code before the PRD** | Developer invents scope that was never agreed | Wait for the PRD. If it's slow, fix the pipeline, not the discipline. |
| **Horizontal tickets** ("all APIs", "all UI") | Nothing is demoable until everything is done; integration hell at the end | Tracer bullets: every ticket is a complete vertical slice. |
| **Testing implementation, not interface** | Tests break on every refactor; you stop refactoring | Test through the seam. The interface is the test surface. |
| **Pass/fail verdicts without evidence** | "Tests passed" is not auditable; bugs ship to production | Every verdict links to `evidence/` artifacts. |
| **Chat memory as state** | Context windows compact; agents forget; handoffs break | Write everything to files: PROGRESS.md, evidence/, retrospectives/. |
| **Deploying without tester sign-off** | Bugs reach the client; trust erodes | DevOps only ships after Tester explicitly approves. |
| **Skipping retrospective** | The same mistakes repeat in the next phase | Write `retrospectives/phase-N.md`. Patch skills. Propose pipeline changes. |
| **The operator doing the work** | Operator becomes bottleneck; roles lose purpose | Route everything through the board. Operator only orchestrates. |
| **Silent failures** | A role fails and nobody knows; the pipeline stalls | Every role reports status. Blocked tasks surface on the board. The operator watches. |
| **PRD with unanswered questions** | Agents start building with incomplete context; wrong assumptions compound | The PRD Gate checklist (`checklists/prd-gate.md`) blocks progress until every item is checked. No "TBD" survives. |
| **UI inconsistency across agents** | Agent A uses #FF0000, Agent B uses #E53935; fonts shift between pages; the app looks like a patchwork | DESIGN.md is the single source of visual truth. Every Developer agent reads it before writing UI. Tester validates WCAG contrast. |

## Subagent Contract

Every agent spawned by the pipeline (kanban worker, Codex instance, delegated
subtask) MUST load this contract before starting work. It defines the shared
language, executable rules, and consistency requirements that make a team of
AI agents behave like a single engineering organization.

### Shared Vocabulary (Acronyms as Executable Rules)

These are not just definitions. Each acronym maps to a concrete behaviour
the agent must execute.

| Acronym | Meaning | Agent Rule |
|---|---|---|
| **KISS** | Keep It Simple, Stupid | Prefer the simplest solution that meets the ticket. If your implementation needs a paragraph to explain, simplify it. Rejected: over-engineering, premature abstraction, "future-proofing." |
| **DRY** | Don't Repeat Yourself | Before committing, scan the codebase for existing patterns. If you're writing something that already exists, reuse or extend it. Every piece of knowledge must have one canonical representation. |
| **YAGNI** | You Ain't Gonna Need It | Don't build features, hooks, or extension points "just in case." If the ticket doesn't require it, don't write it. The PRD is the boundary. |
| **SRP** | Single Responsibility Principle | Every function does one thing. Every class has one reason to change. If the class name needs "and" or "or," split it. |
| **SPOT** | Single Point Of Truth | Config, constants, business rules, and shared logic live in exactly one place. If you find yourself copying a value, extract it. |
| **DAMP** | Descriptive and Meaningful Phrases | Test names describe behaviour, not implementation. `it('returns 404 for unknown users')` not `it('testHandleUser404')`. |
| **RCA** | Root Cause Analysis | When a bug is found, trace it to the root cause before fixing. Don't patch the symptom. Write the root cause in the commit message. |
| **TDD** | Test-Driven Development | Red → Green → Refactor. Write the test first. Watch it fail. Write minimal code to pass. Refactor. Never write code without a failing test. |
| **BDD** | Behavior-Driven Development | User stories drive tests. Tests describe behaviour from the user's perspective. Gherkin when appropriate: Given/When/Then. |
| **CI/CD** | Continuous Integration / Delivery | Every commit must pass tests. Every merge to delivery branch triggers deploy to staging. Production only after Tester sign-off. |
| **CRUD** | Create Read Update Delete | API endpoints follow RESTful naming: `POST /resource`, `GET /resource/:id`, `PUT /resource/:id`, `DELETE /resource/:id`. Consistent across all resources. |
| **ACID** | Atomic, Consistent, Isolated, Durable | Database operations are transactional. If an operation spans multiple tables, they succeed or fail together. No partial state. |
| **POC** | Proof of Concept | Use `/spike` for throwaway experiments before building. POC code is never merged to delivery branches. |
| **MVP** | Minimum Viable Product | The smallest set of features that delivers value. If a feature can be cut without breaking the user flow, cut it. Scope creep is rejected by the PRD Gate. |
| **LGTM** | Looks Good To Merge | Only the Tester or Operator says this. Developers never self-approve. Review gate must pass first. |
| **WPM** | WTFs Per Minute | If reading your code makes the reviewer say "WTF," refactor. The goal is zero WTFs. Bad names, unclear logic, and surprises all count. |
| **F.O.C.U.S** | Follow One Course Until Successful | One ticket at a time. Finish it or block it before starting another. No multi-tasking across tickets. |
| **DIE** | Duplication Is Evil | Duplication is the root of all evil. If you see it, kill it. If you create it, you own the cleanup. |
| **EDA** | Event-Driven Architecture | When the architecture decision specifies events, emit events at module boundaries. Don't call across boundaries directly. |
| **SOLID** | (implicit in SRP above) | Open-Closed: extend, don't modify. Liskov: subtypes must be substitutable. Interface Segregation: small, focused interfaces. Dependency Inversion: depend on abstractions. |

### Pre-Flight Checklist (Every Agent, Every Task)

Before writing any code or executing any task, every agent must verify:

1. **Context loaded.** Read the ticket body, its blocking tickets' handoffs,
   CONTEXT.md (domain glossary), and DESIGN.md (visual tokens).
2. **Scope confirmed.** The task is within the PRD boundaries. If something
   feels out of scope, block and ask — don't guess.
3. **Architecture consulted.** The task respects the Architect's decisions
   (APIs, data flow, module boundaries). Don't redesign mid-ticket.
4. **Existing patterns scanned.** Search the codebase for similar code. If
   a pattern already exists, follow it exactly. Consistency over novelty.
5. **Tests planned.** What seam will you test through? What's the acceptance
   criteria? Write the test name before the implementation.
6. **Clean Code loaded.** Review the clean-code skill. Every function does
   one thing. Names are intention-revealing. Comments are failures.

### Handoff Protocol

When an agent completes or blocks a task, the handoff must include:

```json
{
  "ticket_id": "T-XXX",
  "status": "done | blocked",
  "summary": "one-line description of what was accomplished",
  "changed_files": ["path/to/file1.ts", "path/to/file2.ts"],
  "tests_run": 14,
  "tests_passed": 14,
  "decisions": [
    "Used token bucket pattern for rate limiter — matches existing pattern in auth module",
    "Primary key is user_id with IP fallback for anonymous requests"
  ],
  "blockers": [],
  "needs_review": false,
  "evidence_paths": ["evidence/phase-1/e2e-dashboard.png"]
}
```

If blocked: `blockers` contains the blocking question. Status is `blocked`.
The next agent reads the handoff of every blocking ticket before starting.

### Consistency Rules (Non-Negotiable)

These rules apply to every agent, every ticket, every commit:

- **Naming.** Use the project's existing naming conventions. If the codebase
  uses `getUserById`, never write `fetchUser` or `retrieveUser`. Match the
  existing verb, not the one you prefer.
- **File structure.** New files follow the existing directory pattern. If
  pages live in `src/pages/[name]/index.tsx`, new pages go there too.
- **Formatting.** Automated formatter (prettier, biome, black) runs on commit.
  No manual formatting decisions.
- **Imports.** Same order, same style as existing files. No relative imports
  where absolute paths are the convention.
- **Error handling.** Same pattern as the rest of the codebase. If errors are
  thrown as `AppError`, throw `AppError`. If they're returned as `Result<T>`,
  return `Result<T>`.
- **Comments.** Code should be self-documenting. Comments explain WHY, not
  WHAT. No commented-out code — delete it.
- **Commits.** Atomic, with meaningful messages. One logical change per commit.
  Commit message format: `type(scope): description` (e.g. `feat(auth): add
  rate limiter`).

### Trust But Verify

The pipeline trusts agents to follow the contract. But trust is verified:

- **Architecture check.** The review gate checks that the implementation
  respects architecture decisions. Code that bypasses a defined seam is
  rejected.
- **Consistency check.** Tester compares the new UI against DESIGN.md and
  existing pages. Any visual deviation is a failed test.
- **Pattern check.** Code review flags novel patterns. If the agent invented
  a new way to do something the codebase already does, it's sent back.
- **Scope check.** Any code outside the ticket's scope is rejected, even if
  it's "an improvement." Improvements go in a separate ticket.

## References

External sources referenced in this document. Paths are relative to the
Hermes skills directory (`~/.hermes/skills/`) unless absolute.

### Skills Hub

- **Matt Pocock skills:** 17 engineering skills installed from
  `mattpocock/skills` (169K stars). Installed locally at
  `~/.hermes/skills/.hub/quarantine/` (conflict with builtins).
  Key skills: `codebase-design`, `domain-modeling`, `to-spec`, `to-tickets`,
  `implement`, `tdd`, `code-review`, `grill-me`, `grill-with-docs`,
  `diagnosing-bugs`, `research`, `prototype`, `wayfinder`, `triage`,
  `resolving-merge-conflicts`, `ask-matt`.
  Source: https://github.com/mattpocock/skills
  Hub: https://skills.sh (search: "mattpocock")

- **DESIGN.md:** Google's open token spec for visual identity. Installed as
  `~/.hermes/skills/creative/design-md/SKILL.md`.
  CLI: `npx @google/design.md lint|diff|export`
  Source: https://github.com/google-labs-code/design.md

### Built-in Hermes Skills (loaded per role)

- `kanban-orchestrator` → `~/.hermes/skills/devops/kanban-orchestrator/SKILL.md`
- `kanban-worker` → `~/.hermes/skills/devops/kanban-worker/SKILL.md`
- `plan` → `~/.hermes/skills/software-development/plan/SKILL.md`
- `spike` → `~/.hermes/skills/software-development/spike/SKILL.md`
- `simplify-code` → `~/.hermes/skills/software-development/simplify-code/SKILL.md`
- `systematic-debugging` → `~/.hermes/skills/software-development/systematic-debugging/SKILL.md`
- `test-driven-development` → `~/.hermes/skills/software-development/test-driven-development/SKILL.md`
- `requesting-code-review` → `~/.hermes/skills/software-development/requesting-code-review/SKILL.md`
- `playwright-best-practices` → `~/.hermes/skills/playwright-best-practices/SKILL.md`
- `webapp-testing` → `~/.hermes/skills/webapp-testing/SKILL.md`
- `github-pr-workflow` → `~/.hermes/skills/github/github-pr-workflow/SKILL.md`
- `github-code-review` → `~/.hermes/skills/github/github-code-review/SKILL.md`
- `github-issues` → `~/.hermes/skills/github/github-issues/SKILL.md`
- `codebase-inspection` → `~/.hermes/skills/github/codebase-inspection/SKILL.md`

### External Tools

- **Husky:** Git hooks for pre-commit quality gates (lint, format, test, commit
  message validation). Configured in `.husky/pre-commit`, `.husky/commit-msg`,
  `.husky/pre-push`. See `framework/VERSION-CONTROL.md` "Pre-Commit Gates."
  Source: https://github.com/typicode/husky

- **Codex CLI:** OpenAI's autonomous coding agent. Installed at
  `/opt/homebrew/bin/codex` (v0.144.3). Used by Developer role.
  Source: https://github.com/openai/codex

- **Spec Kit:** Project-local spec generation tool. Located in
  `milestone3/.specify/`. Used by PreDocumentation role.

- **Feynman:** Research agent shell for paper ranking and synthesis.
  Installed at `~/.local/bin/feynman`. Used for academic validation of
  the pipeline approach.
  Source: https://github.com/companion-inc/feynman

- **cua-driver:** Background desktop automation. Installed at
  `/Applications/CuaDriver.app`. Used by Operator for computer-use tasks.
  Source: https://github.com/NousResearch/hermes-agent

### Books

- **Clean Code:** Robert C. Martin (2009). Extracted as a Hermes skill.
  Source: https://github.com/virgiliojr94/book-to-skill
