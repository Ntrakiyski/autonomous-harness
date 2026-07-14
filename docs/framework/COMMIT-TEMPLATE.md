# COMMIT-TEMPLATE.md — Commit Message Protocol

> **Every agent uses this template. Every commit follows this format.
> No exceptions.** Referenced by ROLE.md at session startup.

## Template

```
type(scope): imperative-mood summary (max 72 characters)

Body: What was done, WHY this approach was chosen, what patterns
were followed, what decisions were made, what trade-offs were
considered. Each line max 72 characters.

Ticket: T-XXX
```

## Types

| Type | When to Use | Example |
|---|---|---|
| `feat` | New feature or functionality | `feat(dashboard): add health metrics widgets` |
| `fix` | Bug fix | `fix(wearables): handle null battery from Fitbit API` |
| `refactor` | Restructuring without behaviour change | `refactor(auth): extract token validation to middleware` |
| `test` | Adding or fixing tests | `test(dashboard): add edge case coverage for empty state` |
| `docs` | Documentation only | `docs(api): document wearable response schema` |
| `chore` | Dependencies, config, build, tooling | `chore(deps): upgrade react-query to v5` |

## Scope

Must be one of the modules defined in ARCHITECTURE.md. Common scopes:
`auth`, `api`, `dashboard`, `wearables`, `patients`, `ui`, `db`, `config`.

If a change spans multiple scopes, pick the primary one. If it truly
spans multiple equally, split into separate commits.

## Summary Rules

- **Imperative mood.** "add" not "added" or "adds". "fix" not "fixed".
- **Max 72 characters.** Truncate if needed. Details go in the body.
- **No period at the end.**
- **Descriptive, not generic.** "add rate limiter" not "update code".

## Body Rules

The body explains WHY, not WHAT. The diff already shows WHAT changed.

Include:
- What was built and for which ticket
- WHY this approach was chosen over alternatives
- What existing patterns were followed
- What trade-offs were considered
- For fixes: the root cause

Every line max 72 characters. Blank line between paragraphs.

## Examples

### Good — Feature

```
feat(dashboard): add user dashboard with health metrics

Implements T-007: dashboard page at /dashboard showing wearables,
step count, heart rate trend, and sleep score. Follows existing
/patients page pattern for layout, data fetching, and error states.
Reuses PatientCard component structure for WearableCard.
All components use DESIGN.md tokens (no raw hex values).
Empty state with CTA when no wearables connected.

Ticket: T-007
```

### Good — Bug Fix

```
fix(wearables): handle null battery from Fitbit API

Fitbit API returns null for battery when device is in power save
mode. Previously crashed the dashboard with TypeError. Now
shows "N/A" with a tooltip explaining the device status.

Root cause: Fitbit devices in power-save mode don't report
battery percentage through their v2 API. API contract in
ARCHITECTURE.md updated to document nullable battery.

Ticket: T-015
```

### Good — Refactor

```
refactor(auth): extract token validation to middleware

Moves JWT validation from 5 individual route handlers to a single
shared middleware. Reduces duplication — each route had identical
validation logic with subtle inconsistencies in error messages.

No behaviour change. All 23 auth tests pass. Middleware is placed
at the existing auth seam defined in ARCHITECTURE.md.

Ticket: T-012
```

### Bad — Rejected

```
❌ fix stuff
   → No scope, no description, no ticket reference

❌ updated code
   → Not imperative, no scope, says nothing

❌ WIP save point
   → Work-in-progress commits are squashed; use atomic commits

❌ feat: dashboard
   → Missing scope. Should be feat(dashboard): ...

❌ feat(dashboard): added a dashboard with some widgets and it works
   → "added" not "add". Summary too long (72 char max).
   → If this commit also fixes a bug, split into two commits.
```

## Learnings Extraction

After every commit, check if there is a learning to capture:

### For `fix` commits
1. What was the root cause?
2. Could it have been prevented by existing processes?
3. If yes → add to LEARNINGS.md as "Never Again" or update a checklist.
4. If it revealed a gap in documentation → update ARCHITECTURE.md,
   DATABASE.md, or DESIGN.md.

### For `refactor` commits
1. What was duplicated?
2. Why wasn't it caught earlier?
3. If pattern → add to LEARNINGS.md as "Recurring Pattern."
4. If architectural → propose ADR.

### For `feat` commits
1. Are there new patterns worth documenting?
2. Did the implementation reveal an ambiguity in the PRD?
3. If yes → note it. The retrospective will surface these.

### Learnings Template

```markdown
### [YYYY-MM-DD] [Phase N] — [One-line learning]

**What happened:** [Context]

**Root cause / Trigger:** [Why it happened]

**Action taken:** [What changed — file updated, rule added, skill patched]

**Prevention:** [How we stop this from recurring]

**Source commit:** [commit sha]
```

## Quick Reference

```bash
# Before committing, verify format:
COMMIT_MSG=$(cat)
echo "$COMMIT_MSG" | head -1 | grep -E '^(feat|fix|refactor|test|docs|chore)\([a-z-]+\): .{1,72}$'

# Good commit:
git commit -m "feat(dashboard): add health metrics widgets

Implements T-007: dashboard page at /dashboard. Follows /patients
page pattern for layout, data fetching, and error states.

Ticket: T-007"

# Bad commit (rejected by CI or self-inspection):
git commit -m "fix stuff"
```
