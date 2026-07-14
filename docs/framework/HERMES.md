# HERMES.md — Hermes Agent Operations

> **How Hermes orchestrates the agent swarm.** This file documents all
> Hermes-specific operations: kanban dispatcher, agent spawning, worktree
> management, cron jobs, and inter-agent coordination. It references other
> framework files — does not duplicate them.

## Role

Hermes is the **Operator** — it does not write code. It orchestrates the
pipeline: dispatches agents, manages the kanban board, runs review gates,
handles cron jobs, and ensures the pipeline contract (AUTONOMOUS.md) is
followed.

See `AUTONOMOUS.md` for the full role definitions and pipeline steps.

## Kanban Board

The kanban board is the central coordination layer. All tasks flow through it.

### Board Structure

```
Board: project-name (one per project)

Columns:
  todo       → created, waiting for blockers
  ready      → all blockers resolved, waiting for dispatcher
  running    → claimed by an agent, work in progress
  done       → completed by agent, waiting for review gate
  blocked    → agent needs input, waiting for operator/CEO
  archived   → finished, verified, closed
```

### Dispatcher Loop

Runs continuously (every ~5 seconds) inside the Hermes gateway process.
See `VERSION-CONTROL.md` "Worktrees" for the worktree creation logic.

```
while true:
  for each board:
    # 1. Find ready tasks (blockers resolved)
    tasks = SELECT * FROM kanban_tasks
            WHERE board = 'project-name'
            AND status = 'ready'
            ORDER BY priority DESC, created_at ASC

    for task in tasks:
      # 2. Verify all blockers are truly done
      blockers = SELECT * FROM task_dependencies
                 WHERE child_id = task.id
                 AND parent_id NOT IN
                   (SELECT id FROM kanban_tasks WHERE status = 'done')

      if blockers.length > 0:
        continue  # still blocked

      # 3. Atomic claim (prevents two dispatchers from claiming same task)
      claimed = UPDATE kanban_tasks
                SET status = 'running', claimed_at = now(), worker_id = uuid()
                WHERE id = task.id AND status = 'ready'

      if claimed.rows == 0:
        continue  # another dispatcher claimed it

      # 4. Determine worktree base
      base = resolve_base_branch(task)  # main or delivery/m3-phase-N
      workspace = f"/tmp/kanban-workspaces/{task.id}"

      # 5. Create worktree
      git worktree add {workspace} {base}
      cd {workspace}
      npm install

      # 6. Spawn agent
      profile = task.assignee  # "developer", "tester", etc.
      skills = load_skills_for_role(profile)  # from AUTONOMOUS.md Skills section

      spawn_agent(
        profile: profile,
        skills: skills,
        workdir: workspace,
        env: {
          HERMES_KANBAN_TASK: task.id,
          HERMES_KANBAN_WORKSPACE: workspace,
          HERMES_KANBAN_BOARD: 'project-name'
        },
        prompt: f"Implement ticket {task.id}. Read the ticket body,"
                f"blocking tickets' handoffs, and follow SUBAGENT.md."
      )

    sleep 5
```

### Kanban Tools Reference

Agents use these tools to interact with the board. Full documentation in
the `kanban-worker` and `kanban-orchestrator` skills.

| Tool | Purpose |
|---|---|
| `kanban_show(task_id)` | Read task details, comments, handoffs |
| `kanban_comment(task_id, body)` | Add structured handoff or question |
| `kanban_complete(summary, metadata, created_cards)` | Mark task done, pass handoff to next |
| `kanban_block(reason)` | Block task — need operator/CEO input |
| `kanban_create(title, assignee, body, parents)` | Create new task (orchestrator/decomposer only) |

See `SUBAGENT.md` for the handoff protocol format (JSON schema).

## Agent Spawning

### Profile → Role Mapping

From `AUTONOMOUS.md` Skills section:

| Profile | Role | Skills | Agent CLI |
|---|---|---|---|
| `planner` | Planner | domain-modeling, to-spec | Codex (analysis mode) |
| `architect` | Architect | codebase-design, domain-modeling | Codex |
| `designer` | Designer | frontend-design, design-md | Codex |
| `predoc` | PreDocumentation | to-tickets, Spec Kit CLI | Spec Kit + Codex |
| `developer` | Developer | implement, tdd, clean-code, codebase-design | Codex CLI |
| `tester` | Tester | code-review, grill-me, playwright-best-practices | Hermes + Playwright |
| `devops` | DevOps | github-pr-workflow, project deploy scripts | gh CLI + Vercel CLI |

### Agent Startup Sequence

Every agent reads files in this exact order (from `ROLE.md`):

```
1. ROLE.md
2. AGENTS.md
3. ARCHITECTURE.md
4. DATABASE.md
5. DESIGN.md
6. VERSION-CONTROL.md
7. COMMIT-TEMPLATE.md
8. SUBAGENT.md
9. LEARNINGS.md
10. AUTONOMOUS.md (optional)
```

### Spawn Command Template

```bash
# Developer agent:
hermes -p developer \
  --skills "implement,tdd,clean-code,codebase-design,diagnosing-bugs" \
  --workdir /tmp/kanban-workspaces/T-XXX \
  chat -q "You are ticket T-XXX: [title].
           Read the ticket body and comments on the kanban board.
           Read all blocking ticket handoffs.
           Follow SUBAGENT.md pre-flight checklist.
           Write code following ARCHITECTURE.md and DESIGN.md.
           Use git exactly as described in VERSION-CONTROL.md.
           Commit messages must follow COMMIT-TEMPLATE.md."

# Tester agent:
hermes -p tester \
  --skills "code-review,grill-me,design-md,clean-code" \
  chat -q "Review PR #NNN.
           Read PR description, git diff, ARCHITECTURE.md, DESIGN.md.
           Run `autonomous-gate` in code mode and link the evidence.
           Post verdict as PR comment."

# DevOps agent:
hermes -p devops \
  --skills "github-pr-workflow" \
  chat -q "Merge PR #NNN. Verify: lgtm label, green CI, no conflicts.
           If mid-phase: merge to delivery/m3-phase-N.
           If end-of-phase: run ship-gate checklist, merge to main."
```

## Worktree Management

### Directory Structure

```
/tmp/kanban-workspaces/
├── T-005/    → worktree of origin/main
├── T-007/    → worktree of origin/delivery/m3-phase-1
└── T-009/    → worktree of origin/main
```

### Creating a Worktree

```bash
# Determine base branch from ticket context
BASE=$(resolve_base_for_ticket "T-009")

git worktree add /tmp/kanban-workspaces/T-009 $BASE
cd /tmp/kanban-workspaces/T-009
git checkout -b ticket/T-009-slug
npm install
```

### Base Branch Resolution

See `VERSION-CONTROL.md` "Getting Updated Code" for the full logic.

```
if all blocking tickets are merged to main:
    base = "origin/main"
elif all blocking tickets are merged to delivery/m3-phase-N:
    base = "origin/delivery/m3-phase-N"
else:
    # Some blocker not merged → block this ticket, retry later
    return error
```

### Cleaning Up a Worktree

```bash
# After agent completes and PR is merged:
git worktree remove /tmp/kanban-workspaces/T-009 --force
```

### Listing Active Worktrees

```bash
git worktree list
# /path/to/project                abc1234 [main]
# /tmp/kanban-workspaces/T-007      def5678 [ticket/T-007-dashboard]
# /tmp/kanban-workspaces/T-009      ghi9012 [ticket/T-009-notifications]
```

## Cron Jobs

### Backlog Sweep (Weekly)

```bash
hermes cron create "0 9 * * 1" \
  --name "kanban-backlog-sweep" \
  --prompt "Scan kanban board 'project-name' for stale tasks.
            Close tasks archived > 30 days.
            Re-triage tasks blocked > 14 days.
            Report summary to operator."
```

### CI Health Check (Daily)

```bash
hermes cron create "0 8 * * *" \
  --name "ci-health-check" \
  --prompt "Check latest CI runs for the project repo.
            Report any recurring failures or flaky tests.
            Suggest fixes for patterns."
```

## Code Review Automation

### Tester Agent Trigger

Two triggers (either is sufficient):

**Trigger 1: GitHub Actions (PR labeled ready-for-review)**

```yaml
# .github/workflows/code-review.yml
on:
  pull_request:
    types: [labeled]

jobs:
  review:
    if: github.event.label.name == 'ready-for-review'
    steps:
      - run: |
          hermes -p tester \
            --skills "code-review,grill-me,design-md,clean-code" \
            chat -q "Review PR #${{ github.event.pull_request.number }}.
                     Run code review checklist. Post verdict as PR comment."
```

**Trigger 2: Kanban Dispatcher (review task created)**

When a Developer agent completes a task with `needs_review: true`, a
review task is auto-created on the kanban board. The dispatcher spawns
a Tester agent to handle it.

## Environments

Managed by Vercel GitHub integration. See `AGENTS.md` for project-specific
URLs.

```
Environment  Branch                  Deploy Trigger        Who
─────────────────────────────────────────────────────────────────
Local        ticket/T-XXX            manual (npm run dev)  Developer agent
Staging      delivery/m3-phase-N     auto on push          Vercel
Production   main                    auto on push (gated)  Vercel
Hotfix       hotfix/desc             manual                DevOps agent
```

## Monitoring & Recovery

### Stuck Agent Detection

```bash
# Find tasks running longer than expected:
SELECT * FROM kanban_tasks
WHERE status = 'running'
AND claimed_at < now() - interval '2 hours'

# Actions:
# 1. Check if agent is still responding (heartbeat)
# 2. If dead → reclaim task → dispatcher re-spawns
# 3. If alive but slow → check logs, consider expanding ticket scope
```

### Reclaiming a Stuck Task

```bash
hermes kanban reclaim T-007
# → Sets status back to 'ready'
# → Clears worker_id
# → Task will be picked up by next dispatcher cycle

hermes kanban reassign T-007 developer-v2 --reclaim
# → Changes assignee and reclaims in one operation
```

### Dashboard (hermes dashboard)

```bash
hermes dashboard
# Web UI showing:
# - Active workers and their status
# - Kanban board with drag-and-drop
# - Task details, comments, handoffs
# - Worker logs
# - Reclaim/reassign controls
```

## References

- **AUTONOMOUS.md** — Full pipeline contract (roles, principles, gates)
- **VERSION-CONTROL.md** — Git workflow, branches, commits, PRs, CI/CD, worktrees
- **SUBAGENT.md** — Inter-agent contract (vocabulary, checklist, handoff)
- **ROLE.md** — Agent bootstrap sequence
- **kanban-orchestrator skill** — Decomposition playbook
- **kanban-worker skill** — Worker lifecycle, pitfalls, handoff shapes
