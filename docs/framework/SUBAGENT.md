# SUBAGENT.md — Inter-Agent Contract

> **Every agent spawned by the pipeline MUST read this before starting work.**
> It defines the shared language, executable rules, pre-flight checklist,
> handoff protocol, and consistency requirements.

---

## Shared Vocabulary (Acronyms as Executable Rules)

| Acronym | Meaning | Rule |
|---|---|---|
| **KISS** | Keep It Simple, Stupid | Prefer the simplest solution. If your implementation needs a paragraph to explain, simplify it. |
| **DRY** | Don't Repeat Yourself | Scan the codebase for existing patterns before committing. Reuse or extend. One canonical representation per concept. |
| **YAGNI** | You Ain't Gonna Need It | Don't build for hypothetical futures. The ticket defines the boundary. |
| **SRP** | Single Responsibility | One reason to change. If the class name needs "and" or "or," split it. |
| **SPOT** | Single Point Of Truth | Config, constants, rules live in one place. Copying = extracting. |
| **DAMP** | Descriptive and Meaningful Phrases | Test names describe behaviour: `it('returns 404 for unknown users')`. |
| **RCA** | Root Cause Analysis | Trace bugs to root cause. Don't patch symptoms. Write root cause in commit. |
| **TDD** | Test-Driven Development | Red → Green → Refactor. Never write code without a failing test first. |
| **BDD** | Behavior-Driven Development | Tests from user perspective. Given/When/Then when appropriate. |
| **CI/CD** | Continuous Integration/Delivery | Every commit passes tests. Merge triggers staging deploy. |
| **CRUD** | Create Read Update Delete | RESTful: `POST /res`, `GET /res/:id`, `PUT /res/:id`, `DELETE /res/:id`. |
| **ACID** | Atomic, Consistent, Isolated, Durable | Transactions. Multi-table ops succeed or fail together. |
| **POC** | Proof of Concept | `/spike` for experiments. POC code never merges to delivery branches. |
| **MVP** | Minimum Viable Product | Smallest set that delivers value. Scope creep rejected by PRD Gate. |
| **LGTM** | Looks Good To Merge | Only Tester or Operator says this. Never self-approve. |
| **WPM** | WTFs Per Minute | Zero WTFs is the goal. Bad names, surprises, unclear logic all count. |
| **F.O.C.U.S** | Follow One Course Until Successful | One ticket at a time. Finish or block before starting another. |
| **DIE** | Duplication Is Evil | See it, kill it. Create it, own the cleanup. |
| **EDA** | Event-Driven Architecture | Emit events at module boundaries. Don't call across directly. |
| **SOLID** | (SRP included above) | OCP: extend, don't modify. LSP: subtypes substitutable. ISP: small interfaces. DIP: depend on abstractions. |

---

## Pre-Flight Checklist

Before writing any code, verify:

1. **Context loaded.** Ticket body, blocking tickets' handoffs, CONTEXT.md, DESIGN.md.
2. **Scope confirmed.** Within PRD boundaries. If unsure, block and ask.
3. **Architecture consulted.** Respects Architect's decisions. Don't redesign.
4. **Existing patterns scanned.** Search codebase for similar code. Follow exactly.
5. **Tests planned.** What seam? What acceptance criteria? Write test name first.
6. **Clean Code loaded.** Functions do one thing. Names reveal intent. Comments are failures.

---

## Handoff Protocol

Every completed or blocked task produces a structured handoff:

```json
{
  "ticket_id": "T-XXX",
  "status": "done | blocked",
  "summary": "one-line description of what was accomplished",
  "changed_files": ["path/to/file1.ts", "path/to/file2.ts"],
  "tests_run": 0,
  "tests_passed": 0,
  "decisions": [
    "Why key architectural/pattern choices were made"
  ],
  "blockers": [],
  "needs_review": false,
  "evidence_paths": ["evidence/phase-N/screenshot.png"]
}
```

Blocked tasks: `blockers` contains the question. Status = `blocked`.

---

## Consistency Rules (Non-Negotiable)

- **Naming.** Match existing conventions. `getUserById` ≠ `fetchUser` ≠ `retrieveUser`.
- **File structure.** Follow existing directory patterns.
- **Formatting.** Automated formatter on commit. No manual decisions.
- **Imports.** Same order and style as existing files.
- **Error handling.** Same pattern as codebase. `AppError` or `Result<T>` — match it.
- **Comments.** Explain WHY, not WHAT. No commented-out code.
- **Commits.** Atomic. `type(scope): description` format.

---

## Trust But Verify

- **Architecture check.** Implementation respects seams. Bypassing a seam = rejected.
- **Consistency check.** UI matches DESIGN.md and existing pages. Deviation = failed test.
- **Pattern check.** Novel patterns flagged. Codebase already does it → use that way.
- **Scope check.** Code outside ticket scope rejected. Improvements → separate ticket.
