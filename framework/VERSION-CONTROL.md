# VERSION-CONTROL.md — Git, Branches, PRs, CI/CD, Code Review for Agent Swarms

> **How a swarm of AI agents collaborates through git.** Human teams have
> GitHub Flow and Git Flow. Agent swarms need something different — this
> file defines it.

## Research Foundation

Academic literature confirms CI/CD and code review as foundational practices
(Feynman PaperRank, 2026). However, zero papers address AI agent git
workflows — this field is being pioneered here. The strategy below combines
empirically-validated human practices (trunk-based development, CI gates,
structured code review) with agent-specific adaptations (atomic ticket
commits, sequential merging, auto-generated PRs, reviewer-as-agent).

## Core Strategy: Trunk-Based with Ticket Branches

```
main ──────────────────────────────────────────────────────
       \               \               \
        ticket/T-001    ticket/T-002    ticket/T-003
        (2-4 commits)   (1 commit)      (3 commits)
        → merge         → merge         → merge
```

### Why This, Not That

| Pattern | Human Teams | Agent Swarms | Our Choice |
|---|---|---|---|
| Git Flow (develop, release, hotfix) | ✅ Mature | ❌ Too complex — agents lose track of which branch is authoritative | **No** |
| GitHub Flow (feature branches → main) | ✅ Common | ⚠️ Works but needs discipline | **Modified** |
| Trunk-Based (short-lived branches) | ✅ Best for CI/CD | ✅ Best for agents — one ticket, one branch, merge fast | **Yes** |
| Long-lived feature branches | ⚠️ Merge hell | ❌ Agents can't coordinate over weeks — conflicts cascade | **No** |
| Direct commits to main | ❌ No review | ❌ No review, no evidence, no gate | **No** |

## Branching Rules

### Branch Naming

```
ticket/<ticket-id>-<short-slug>
```

Examples:
- `ticket/T-001-add-rate-limiter`
- `ticket/T-002-user-dashboard`
- `ticket/T-003-fix-auth-bug`

### Branch Lifecycle

1. **Create** — Agent checks out from latest `main`. Branch name from ticket ID.
2. **Work** — Agent commits atomically within the branch. One logical change
   per commit.
3. **Sync** — Before opening PR, agent rebases onto latest `main` to resolve
   conflicts locally.
4. **PR** — Agent opens PR with structured Handoff Protocol as description.
5. **CI** — Automated tests, lint, build run on PR. Failures block merge.
6. **Review** — Reviewer agent (Tester role) checks PR against review
   checklist. Approve or request changes.
7. **Merge** — Squash merge to `main`. Branch deleted after merge.

### Branch Ownership

- **One agent = one branch.** Never share a branch between agents.
- **One ticket = one branch.** Multiple tickets don't share a branch.
- **Branch is temporary.** Live for hours, not days. Merge and delete.

## Commit Rules

### Commit Format

```
type(scope): imperative-mood description

Detailed body explaining WHY, not WHAT. Reference ticket ID.
Include root cause for fixes.

Ticket: T-XXX
```

Types: `feat`, `fix`, `refactor`, `test`, `docs`, `chore`

### Commit Atomicity

- **One logical change per commit.** Don't mix a bug fix with a feature.
- **Every commit passes tests.** CI runs on every commit to the branch.
- **Commits are signed.** Author is the agent profile that wrote the code.

### What NEVER Goes in a Commit

- Secrets, API keys, tokens (use env vars)
- Large binary files (use git-lfs or external storage)
- node_modules, .env, build artifacts (use .gitignore)
- Commented-out code (delete it — git remembers)
- Debug logs, console.log, print statements (remove before commit)
- "WIP" or "save" commits (squash before PR or use atomic commits from start)

### Self-Inspection Protocol (Before Commit)

After writing code but BEFORE committing, every agent must run a self-inspection.
This is not optional — a commit that hasn't passed self-inspection is rejected.

#### Step 1: Compare Initial State with Current State

```bash
git diff --stat main...HEAD
```

This shows every file that was added, modified, or deleted. The agent
must verify that EVERY changed file is intentional and within the
ticket's scope.

#### Step 2: Pattern Compliance Check

For every new or modified file, compare against the reference file
identified during the codebase pattern scan. Ask:

- **Structure:** Same import order? Same component structure (props -> state -> effects -> render)? Same conditional flow (isLoading -> error -> empty -> data)?
- **Naming:** Same verb patterns? `use[Noun]` for hooks? `[Noun]Card` for cards? `get[Noun]ById` for data accessors?
- **Tokens:** Zero raw hex colors? All DESIGN.md tokens used through Tailwind classes that map to token values?
- **Error handling:** Same pattern as reference? `ErrorState` component? `throw` vs `return` consistent?
- **File layout:** Same directory structure? `__tests__/` in the same directory?

If ANY pattern mismatch is found -> fix before commit.

#### Step 3: Clean Code Compliance Check

For every file:
- [ ] Every function does exactly one thing
- [ ] Every name reveals intention
- [ ] Zero comments explaining WHAT the code does (code is self-documenting)
- [ ] Zero commented-out code
- [ ] Zero `console.log`, `print`, debug statements
- [ ] Zero hardcoded magic values (use named constants or DESIGN.md tokens)

#### Step 4: State Coverage Check

Every UI component must handle ALL of these states:
- [ ] Loading (isLoading, Suspense, skeleton)
- [ ] Error (API failure, network error, auth error)
- [ ] Empty (no data, zero results, first-time user)
- [ ] Data (normal render with valid data)
- [ ] Edge (null fields, boundary values, very long strings)

If any state is missing -> add it before commit.

#### Step 5: Scope Boundary Check

- [ ] Every changed file is within the ticket's scope
- [ ] No "extra improvement" slipped in (those go in a separate ticket)
- [ ] No commented-out feature preview
- [ ] No dependency added without justification in the PR description

#### Step 6: Self-Inspection Verdict

Only proceed to commit if: ALL pattern checks pass, ALL clean code checks
pass, ALL states are covered, and ALL files are within scope.

If self-inspection fails:
1. Fix the violations
2. Re-run all local tests
3. Re-run self-inspection until clean

## Pull Request Rules

### PR Creation

The Developer agent creates the PR using the Handoff Protocol from
SUBAGENT.md. The PR description MUST include:

```markdown
## Ticket: T-XXX — [Title]

### Summary
[One-line description of what was accomplished]

### Changed Files
- `path/to/file1.ts` — [what changed]
- `path/to/file2.ts` — [what changed]

### Decisions
- [Why key architectural/pattern choices were made]
- [Any deviations from the spec and why]

### Tests
- Tests run: 14
- Tests passed: 14
- Evidence: [link to evidence/ files]

### Pre-Merge Checklist
- [ ] CI passes (tests, lint, build)
- [ ] No conflicts with main
- [ ] DESIGN.md tokens used, not raw values
- [ ] Existing patterns followed, not reinvented
- [ ] Handoff metadata attached
```

### PR Rules

- **One PR per ticket.** One ticket should not span multiple PRs.
- **Small PRs.** Under 400 lines changed. If larger, split the ticket.
- **CI must pass before review.** Reviewer agent only looks at green CI.
- **Reviewer responds within one iteration.** Approve or request changes
  with specific, actionable feedback.
- **No self-merge.** Developer opens PR. Tester or Operator merges.
- **Squash merge.** Clean history on main. One commit per ticket after merge.

## CI/CD Pipeline

### CI Gates (Run on Every PR and Commit)

1. **Install** — `[package manager] install`
2. **Lint** — `[lint command]` (zero warnings)
3. **Format check** — `[format check command]` (zero diffs)
4. **Type check** — `[type check command]` (zero errors)
5. **Unit tests** — `[test command]` (100% pass)
6. **Build** — `[build command]` (zero errors)

All gates must pass. Any failure blocks the PR. CI configuration lives in
the repo (`.github/workflows/` or equivalent).

### Pre-Commit Gates (Husky)

Before code reaches CI, git hooks enforce quality checks LOCALLY on the
agent's machine. This catches issues in milliseconds instead of waiting
for a 5-minute CI cycle.

**Why:** CI is the safety net. Husky is the guard rail. If Husky passes,
CI almost always passes on the first try. Zero wasted CI minutes.

**Setup:**

```bash
npm install --save-dev husky
npx husky init
```

**Three hooks:**

```bash
# 1. pre-commit — Self-Inspection Protocol (Steps 1-5)
.husky/pre-commit:
  npm run lint           # 0 errors, 0 warnings
  npm run format:check   # 0 unformatted files
  npm run typecheck      # 0 type errors
  npm test               # all tests pass

# 2. commit-msg — Commit Message Format Validation
.husky/commit-msg:
  # Validates against COMMIT-TEMPLATE.md format
  # type(scope): description
  # Blocks commits that don't match

# 3. pre-push — Final Check Before Remote
.husky/pre-push:
  git diff --stat main...HEAD  # verify scope
  echo "Ready to push"
```

Any hook failure cancels the action — commit is rejected, push is blocked.
The agent fixes locally and retries. No CI cycles wasted.

### Worktrees — Agent Isolation

Every agent works in its own isolated git worktree. No shared state.
No cross-contamination. No Docker overhead.

```
/tmp/kanban-workspaces/
├── T-005/  → git worktree of main
│   ├── .git (linked to main repo)
│   ├── node_modules/
│   └── apps/web/src/... (T-005's branch: ticket/T-005-api)
├── T-007/  → git worktree of main
│   └── apps/web/src/... (T-007's branch: ticket/T-007-dashboard)
└── T-009/  → git worktree of delivery/m3-phase-1
    └── apps/web/src/... (T-009's branch: ticket/T-009-notifications)
```

**Worktree lifecycle:**

```bash
# Create (dispatcher or operator):
git worktree add /tmp/kanban-workspaces/T-009 main
cd /tmp/kanban-workspaces/T-009
git checkout -b ticket/T-009-notifications
npm install  # separate node_modules per worktree

# Agent works here — isolated from all other agents

# Cleanup after ticket done:
git worktree remove /tmp/kanban-workspaces/T-009
```

**Isolation guarantees:**
- Each worktree has its own checked-out files
- Each worktree can have its own branch
- Changes in one worktree are invisible to others
- `npm install` in T-007 doesn't affect T-009
- Agent A's `git reset --hard` can't destroy Agent B's work

### Getting Updated Code: Three Scenarios

When an agent starts work, it needs the code from its blocking tickets.
There are three scenarios:

#### Scenario A: Blocker merged to main

```bash
# T-009 blocked by T-005 (done, merged to main)

# 1. Verify T-005 commit is in main
kanban_show("T-005")
→ comment: "Merged in PR #140. Commit: a1b2c3d on main"

git fetch origin main
git log origin/main --oneline | grep a1b2c3d
# ✅ Commit is in main — proceed

# 2. Get latest main and branch from it
git checkout main
git pull origin main
git checkout -b ticket/T-009-notifications
```

#### Scenario B: Blocker merged to delivery branch (not yet main)

```bash
# T-009 blocked by T-007 (merged to delivery/m3-phase-1, not in main)

# 1. Verify T-007 is in delivery branch
kanban_show("T-007")
→ comment: "Merged to delivery/m3-phase-1. PR #142."

git fetch origin delivery/m3-phase-1
git log origin/delivery/m3-phase-1 --oneline | grep T-007
# ✅ Commit is in delivery branch — proceed

# 2. Get delivery branch and branch from it
git checkout delivery/m3-phase-1
git pull origin delivery/m3-phase-1
git checkout -b ticket/T-009-notifications
```

#### Scenario C: Blocker done but NOT merged anywhere

```bash
# T-009 blocked by T-007 (done, PR open, NOT merged)

# CRITICAL RULE: Agent NEVER bases work on unmerged code.

# Agent blocks itself:
kanban_block(
  reason="T-007 (PR #142) must be merged to delivery/m3-phase-1
          before T-009 can start. Waiting for code review completion."
)

# When T-007 gets merged:
#   → Dispatcher detects T-007.status = 'done' AND merged
#   → Dispatcher unblocks T-009
#   → T-009 respawns → Scenario B

# NEVER do this:
# ❌ git merge unmerged-branch  (work will be thrown away if PR changes)
# ❌ git cherry-pick from PR    (commit hash will change on squash)
# ❌ Copy-paste code manually   (no git history, untraceable)
```

#### Worktree Resolution Strategy

```bash
# When agent spawns, dispatcher picks correct base:

BLOCKERS_DONE=$(check all blockers' handoffs for merge status)

if all_blockers_merged_to_main:
    BASE="origin/main"
elif all_blockers_merged_to_delivery:
    BASE="origin/delivery/m3-phase-N"
else:
    # Some blocker not merged — block this ticket
    exit 1  # dispatcher will retry when blockers are merged

git worktree add $WORKSPACE $BASE
cd $WORKSPACE
git checkout -b ticket/T-XXX-slug
```

1. CI passes on main
2. DevOps deploys to staging URL
3. E2E tests run against staging
4. Evidence collected in `evidence/phase-N/`
5. Tester reviews evidence

### Production Deploy (after Ship Gate)

1. Ship Gate checklist (`checklists/ship-gate.md`) passes
2. DevOps deploys to production
3. Deployment recorded in ship-gate.md

## Code Review (Agent-to-Agent)

### Reviewer Identity

The **Tester** role is the reviewer. Not a human. Not the Developer
self-reviewing. A separate agent instance with the `code-review` and
`grill-me` skills loaded.

### Review Checklist (What the Reviewer Checks)

The reviewer uses `checklists/code-review.md` (dynamically generated per
phase). Core checks include:

- **Architecture compliance.** Does the code respect the seams and API
  contracts in ARCHITECTURE.md?
- **Design compliance.** Does the UI use DESIGN.md tokens? Are fonts,
  colors, spacing correct?
- **Pattern compliance.** Does new code follow existing patterns?
  Or did the agent invent a novel approach?
- **Scope compliance.** Is anything built outside the ticket scope?
- **Clean Code compliance.** Functions do one thing? Names reveal intent?
  Comments explain WHY, not WHAT?
- **Test quality.** Tests at the right seams? Evidence in evidence/?
- **Consistency.** Naming, formatting, imports, error handling match codebase.

### Review Outcome

- **Approve** → LGTM comment. Operator or DevOps merges.
- **Request changes** → Specific, actionable feedback. Ticket goes back to
  Developer. Fix loop starts.

### Review Rules

- **Adversarial stance.** Assume it's wrong until proven right.
- **Specific feedback.** "Rename `processData` to `transformOrder`" not
  "improve naming."
- **Single iteration.** Reviewer doesn't go back-and-forth. One review
  cycle per PR.
- **Evidence required.** Reviewer can request screenshots, logs, or
  test runs as evidence.

## Conflict Resolution

Agent swarms create merge conflicts when two agents touch the same files.
Our strategy:

### Prevention
- **Ticket isolation.** Tickets are designed (in Spec Kit phase) to
  minimize overlapping files. The PreDocumentation role ensures no two
  tickets touch the same modules simultaneously.
- **Sequential PR queue.** If tickets T-001 and T-002 touch overlapping
  files, T-002 is blocked until T-001 merges.

### When Conflicts Happen
- **Agent resolves locally.** Before opening PR, agent rebases onto
  latest main. Resolves conflicts in its own branch.
- **Test both sides.** After resolving a conflict, the agent runs the
  FULL test suite — not just its own tests. The other branch's tests
  must also pass. If the conflicting commit introduced new tests, those
  tests must pass against the merged code. If they fail, the merge is
  wrong — re-resolve.
- **Verify both behaviours.** The conflict was between two agents who
  both had valid reasons for their code. After merge, BOTH behaviours
  must work. Agent verifies: does my feature still work? Does the
  other agent's feature still work? If either fails, re-resolve.
- **If can't resolve.** Agent blocks the ticket with a specific question
  for the Operator. Includes: the conflicting file, both versions,
  what each version does, and a recommendation.

## Agent Identity in Git

Every agent commits with its own identity so blame/git-log is traceable:

```bash
git config user.name "[Role] — [Agent Profile]"
git config user.email "agent+[role]@autonomous.local"
```

Example:
- `Developer — codex-worker <agent+developer@autonomous.local>`
- `Tester — playwright-worker <agent+tester@autonomous.local>`
- `DevOps — deploy-worker <agent+devops@autonomous.local>`

This makes `git log` and `git blame` show which role made which change —
critical for debugging and retrospectives.

## Delivery Branches

For milestone-level work that spans multiple tickets:

```
delivery/m3-phase-1    ← accumulates all Phase 1 tickets
delivery/m3-phase-2    ← accumulates all Phase 2 tickets
```

- Ticket branches merge into the delivery branch (not main directly).
- Delivery branch merges to main after Ship Gate passes.
- Delivery branches are named `delivery/m<number>-phase-<number>`.

## Emergency: Hotfix Protocol

If a bug reaches production:

1. **Branch:** `hotfix/<description>` from `main`
2. **Fix:** Minimal change. One commit.
3. **PR:** Expedited review. Tester approves.
4. **Merge:** To `main` directly (skip delivery branch).
5. **Deploy:** DevOps deploys immediately.
6. **Retrospective:** Write why it happened in LEARNINGS.md.
7. **Backport:** Merge hotfix into active delivery branch.

No new features in hotfixes. No refactoring. Just the fix.

## Automation Layer

Automated tooling that reduces manual operator intervention in the
review → fix → merge → cleanup cycle.

### PR Auto-Review (ClawPatch-Style)

After the Developer agent opens a PR:

1. CI runs automatically (lint, typecheck, unit tests, build).
2. If CI fails, PR is auto-labeled `ci-failed`. Developer fixes and
   re-pushes.
3. If CI passes, an automated reviewer (Tester agent with `code-review`
   and `grill-me` skills) is triggered.
4. The reviewer checks architecture compliance, design tokens,
   pattern consistency, scope boundaries, and clean code rules.
5. If the reviewer approves → PR is labeled `lgtm` and ready for merge.
6. If the reviewer requests changes → specific feedback is posted as
   PR comments. PR labeled `needs-changes`. Goes back to Developer.

Future: integrate ClawPatch or an equivalent automated reviewer that
runs on every PR without manual operator triggering.

### PR Auto-Merge

When a PR has `lgtm` label AND CI is green:

1. Operator or DevOps reviews the automated reviewer's verdict.
2. If the Ship Gate checklist is not yet due (mid-phase), merge to
   the delivery branch (`delivery/m3-phase-N`).
3. If the Ship Gate checklist passes (end of phase), merge to `main`
   and trigger production deploy.
4. Squash-merge. Branch deleted. One clean commit on the target branch.

Auto-merge conditions (all must be true):
- CI: green across all gates
- Review: `lgtm` from automated reviewer
- No merge conflicts with target branch
- No `do-not-merge` label
- Evidence files linked in PR description

### Backlog Cleanup (ClawSweeper-Style)

Run at the end of every phase (part of the retrospective step):

1. Scan all open issues and PRs in the repository.
2. For each item, check: last activity date, whether the linked ticket
   is done/archived, whether the PR branch still exists.
3. Auto-suggest closure for:
   - PRs on deleted branches
   - Issues linked to completed tickets with no remaining action items
   - PRs with no activity in 14+ days and `needs-changes` label
   - Duplicate issues (same title/body pattern)
4. Operator reviews suggestions and bulk-closes.
5. Remaining open items are re-triaged with updated labels.

Future: run ClawSweeper or equivalent as a weekly cron job via Hermes.

## End-to-End Lifecycle: From Ticket to Production

This is the complete path a unit of work takes through the system.
Every step is observable, auditable, and reversible.

### Phase 1: Ticket Creation (PreDocumentation)

```
Spec Kit generates tasks.md
  → Operator runs checklists/spec-review.md
  → Gate passes
  → Tickets created on kanban board (or GitHub Issues)
  → Each ticket: title, body, acceptance criteria, blocking edges
  → Tickets labeled ready-for-agent
```

### Phase 2: Agent Claims Ticket (Operator/Dispatcher)

```
Operator assigns ticket to Developer profile
  OR kanban dispatcher auto-claims ready ticket
  → Agent spawned with constrained workspace:
    - HERMES_KANBAN_TASK=ticket-id
    - HERMES_KANBAN_WORKSPACE=/path/to/worktree
  → Agent reads startup sequence:
    1. AGENTS.md (role, permissions)
    2. ARCHITECTURE.md (seams, API contracts)
    3. DATABASE.md (models)
    4. DESIGN.md (visual tokens)
    5. VERSION-CONTROL.md (git rules)
    6. SUBAGENT.md (pre-flight checklist, consistency rules)
```

### Phase 3: Agent Works (Developer)

```
1. PRE-FLIGHT CHECKLIST (SUBAGENT.md):
   - Reads ticket body and blocking tickets' handoffs
   - Searches codebase for existing patterns
   - Confirms scope within PRD boundaries
   - Plans tests at the right seams

2. BRANCH:
   git checkout -b ticket/T-XXX-add-feature main

3. IMPLEMENT:
   - Writes code following ARCHITECTURE.md seams
   - Uses DESIGN.md tokens, not raw values
   - Follows existing codebase patterns
   - Runs tests locally: [test command]

4. COMMIT:
   git add path/to/files
   git commit -m "feat(scope): add feature description

   Detailed body. Why this approach.
   Root cause if this is a fix.

   Ticket: T-XXX"

5. PUSH:
   git push -u origin ticket/T-XXX-add-feature

6. OPEN PR:
   Creates PR on GitHub (or via gh CLI)

   PR DESCRIPTION (Handoff Protocol):
   ## Ticket: T-XXX
   ### Summary
   [one line]
   ### Changed Files
   - path/to/file.ts — [what]
   ### Decisions
   - [why]
   ### Tests
   - Run: 14 | Passed: 14
   - Evidence: [links]

7. HANDOFF:
   kanban_comment(body=handoff_json)
   kanban_complete(summary, metadata, created_cards)
```

### Phase 4: CI Runs (Automated)

```
GitHub Actions triggered on PR open and every push:

1. INSTALL:
   npm ci (or equivalent)

2. LINT:
   npm run lint → 0 errors, 0 warnings

3. FORMAT CHECK:
   npm run format:check → no diffs

4. TYPE CHECK:
   npm run typecheck → 0 errors

5. UNIT TESTS:
   npm test → all pass

6. BUILD:
   npm run build → success

Result: ✅ All checks passed
        OR
        ❌ CI failed → PR labeled ci-failed → back to Developer
```

### Phase 5: Code Review (Automated Reviewer / Tester)

```
Triggered after green CI:

1. REVIEWER LOADS CONTEXT:
   - Reads PR description (handoff protocol)
   - Reads changed files (git diff)
   - Reads ARCHITECTURE.md (are seams respected?)
   - Reads DESIGN.md (are tokens used?)
   - Reads checklists/code-review.md

2. CHECKLIST EXECUTION:
   - [ ] Architecture compliance
   - [ ] Design token compliance
   - [ ] Pattern consistency
   - [ ] Scope boundaries
   - [ ] Clean Code rules
   - [ ] Test quality and evidence

3. OUTCOME:
   APPROVE:
   → PR labeled lgtm
   → Comment: "LGTM. Verified: architecture ✓, design ✓, tests ✓"

   REQUEST CHANGES:
   → PR labeled needs-changes
   → Specific, actionable comments on the PR
   → Back to Developer (Fix Loop, Pipeline step 8)
```

### Phase 6: Merge (DevOps / Operator)

```
Preconditions: lgtm label + green CI + no conflicts

1. REBASE CHECK:
   git fetch origin main
   git rebase origin/main
   → If conflicts: agent resolves, force-pushes, CI re-runs

2. MERGE:
   Mid-phase:
   git checkout delivery/m3-phase-1
   git merge --squash ticket/T-XXX-add-feature
   git push origin delivery/m3-phase-1

   End-of-phase (Ship Gate passed):
   git checkout main
   git merge --squash delivery/m3-phase-1
   git push origin main
   git branch -d ticket/T-XXX-add-feature

3. TAG:
   git tag -a v1.2.0 -m "Milestone 3, Phase 1 complete"
   git push --tags
```

### Phase 7: Deploy (DevOps)

```
1. STAGING DEPLOY (every merge to delivery branch):
   vercel deploy
   → Deploys to staging URL

2. E2E TESTS (Tester):
   npx playwright test
   → Evidence captured in evidence/phase-N/
   → Screenshots compared against DESIGN.md tokens
   → WCAG contrast re-validated

3. SHIP GATE (end of phase only):
   checklists/ship-gate.md executed
   → All prior gates signed
   → Evidence audited
   → Tester approval recorded
   → Operator / CEO sign-off

4. PRODUCTION DEPLOY (Ship Gate passed):
   git push origin main
   → Production auto-deploys
   → Deployment URL recorded in ship-gate.md

5. DEPLOYMENT RECORD:
   Recorded in checklists/ship-gate.md:
   - Deployed by: [agent identity]
   - Deployed at: [timestamp]
   - Commit: [sha]
   - URL: [production URL]
```

### Phase 8: Cleanup & Retrospective

```
1. BRANCH CLEANUP:
   git branch -d ticket/T-XXX-add-feature (local)
   git push origin --delete ticket/T-XXX-add-feature (remote)

2. BACKLOG SWEEP:
   ClawSweeper-style scan of open issues/PRs
   → Close stale items
   → Re-triage remaining

3. RETROSPECTIVE:
   Write retrospectives/phase-N.md
   → What worked, what broke, what to change
   → Skills patched immediately
   → LEARNINGS.md updated

4. KANBAN ARCHIVE:
   Archive completed tickets
   → Board reflects current state
```

## Quick Reference: Commands the Agent Runs

```bash
# Start work
git checkout -b ticket/T-XXX-slug main

# During work
git add <files>
git commit -m "type(scope): description

Body.

Ticket: T-XXX"

# Before PR
git fetch origin main
git rebase origin/main
npm test && npm run lint && npm run typecheck

# Open PR
git push -u origin ticket/T-XXX-slug
gh pr create --title "feat(scope): description" --body-file pr-body.md

# After merge
git checkout main
git pull origin main
git branch -d ticket/T-XXX-slug
```
