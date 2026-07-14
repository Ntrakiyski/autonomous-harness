# PRD Gate — Milestone [N], Phase [N]

Generated: [YYYY-MM-DD] | Operator: [name] | Artifact: prd/milestone-N-prd.md

## Source Documents Read
- Signed scope: scope/signed-scope.pdf
- Architecture: framework/ARCHITECTURE.md
- Design: framework/DESIGN.md
- Learnings: framework/LEARNINGS.md
- Consistency rules: framework/SUBAGENT.md

## Checks

### Derived from Signed Scope PDF

- [ ] Every scope item from the signed PDF is addressed in the PRD
- [ ] No new scope was added beyond the PDF (clarification ≠ expansion)
- [ ] Every user story has an actor, action, and benefit
- [ ] All user flows have a defined start, middle, and end state

### Derived from Architecture (ARCHITECTURE.md)

- [ ] All module boundaries for this milestone are defined
- [ ] All new API contracts are specified (route, method, request, response)
- [ ] Database models for new features are defined with relationships
- [ ] Data flow is described for every user action in scope
- [ ] Authentication and authorization boundaries are clear for all new endpoints
- [ ] Third-party integrations are specified (what, why, how, fallback)

### Derived from Design (DESIGN.md)

- [ ] DESIGN.md exists and is valid (run `npx @google/design.md lint DESIGN.md`)
- [ ] All new UI components are defined in DESIGN.md's components section
- [ ] Every screen/component references DESIGN.md token values, not raw hex
- [ ] Consistency rules against the existing app are written explicitly
- [ ] Responsive breakpoints are defined (mobile, tablet, desktop)

### Derived from Learnings (LEARNINGS.md)

- [ ] Every "Never Again" pattern from previous phases is prevented by design
- [ ] Recurring issues from prior phases have explicit mitigations in the PRD
- [ ] [PHASE-SPECIFIC: Add items from this project's LEARNINGS.md]

### Derived from Consistency Rules (SUBAGENT.md)

- [ ] The PRD uses the project's domain glossary terms consistently
- [ ] Naming conventions from the existing codebase are preserved
- [ ] The PRD references existing seams — doesn't invent new ones unnecessarily

### Phase-Specific Depth

- [ ] Edge cases are enumerated: empty states, error states, loading states,
      permission-denied states, rate-limited states
- [ ] Every "TBD", "later", "to be decided", or "out of scope (but maybe)"
      has been resolved or explicitly deferred with CEO approval
- [ ] The Planner has challenged every vague client term with a precise
      alternative (e.g., "fast" → "under 200ms p95")
- [ ] Conflicting requirements have been surfaced and resolved in the PRD
- [ ] The PRD defines what "done" means for this milestone
- [ ] Rollback and failure modes are described for critical paths

## Sign-Off

- [ ] All checks passed or waived with documented reason
- [ ] Operator: ________ | Date: ________
- [ ] CEO (if required): ________ | Date: ________
