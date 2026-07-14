# GUARDRAILS.md — Non-Negotiable Safety Rules

> **Every agent reads this. Every agent follows this. No exceptions.**
> These rules are not suggestions. They are hard boundaries that protect
> the project, the team, and the users. If you're about to do something
> that violates a guardrail, stop. Ask. Do not proceed.

---

## Git Safety

### G1: Never force-push to main/master
`git push --force` (or `--force-with-lease`) to the main branch is
blocked. Period. Use a delivery branch and open a PR. Only DevOps
merges to main.

**If you're about to:** `git push --force origin main`
**Stop.** Push to a branch. Open a PR. Let DevOps handle main.

### G2: Never commit secrets or credentials
Files matching `.env`, `*.pem`, `*.key`, `credentials.*`,
`secrets.*`, and any file containing API keys, tokens, or passwords
must never be committed. If a secret is accidentally committed,
rotate it immediately — don't just remove the file.

### G3: Never reset or delete shared history
`git reset --hard` on a branch that has been pushed is dangerous.
Rebase is fine on your own branch. On shared branches, prefer merge.

---

## File System Safety

### F1: Stay inside the project
Never write, delete, or modify files outside the project root.
No `rm -rf /`, no `cd ~ && ...`, no touching system files.
The project is your world. Stay in it.

### F2: Never delete without confirmation
`rm -rf` on directories, `git clean -fdx`, and similar destructive
commands require explicit user confirmation. If you're about to
delete something irreversible, ask first.

### F3: Never modify framework files marked "do not change"
`ROLE.md` and this file (`GUARDRAILS.md`) are the project's
constitution. Do not edit them unless explicitly instructed.

---

## Deployment Safety

### D1: Only DevOps deploys
Staging and production deployments are the DevOps role's
responsibility. Developers never deploy. Testers never deploy.
If you're not DevOps, you don't touch deploy commands.

### D2: Staging before production
Never deploy directly to production. Staging deployment comes first.
Tester sign-off comes before production. Ship Gate checklist comes
before every production deploy.

### D3: Never deploy with failing tests
A deploy with failing tests is a rejected deploy. All tests must pass
with evidence before staging. All tests must pass with evidence before
production. See `TESTING.md` — evidence, not verdicts.

---

## Data Safety

### DS1: Never drop or truncate production data
`DROP TABLE`, `TRUNCATE`, `DELETE FROM ...` without WHERE, and
similar destructive queries are blocked on production databases.
Migrations are reviewed by the Architect before running.

### DS2: Never expose user data
Logs, error messages, screenshots, and test evidence must never
contain real user data. Use mock data or anonymize. If user data
appears in a log, redact it before committing evidence.

### DS3: Never run untested migrations
Database migrations must be tested on a staging or local database
before running on production. A migration that hasn't been tested
is a migration that hasn't been written.

---

## Code Safety

### C1: Never bypass the review gate
Every change goes through review. Self-approving your own work is
not allowed. The reviewer is adversarial — assume it's wrong until
proven right.

### C2: Never merge with broken CI
If CI is red, fix it before merging. A red CI on main blocks
everyone. If a failure is unrelated to your change, report it —
don't merge past it.

### C3: Never leave dead code or commented-out code
Commented-out code creates confusion. Delete it. Git history keeps
the old version. If you're not sure whether to delete it, ask.

---

## Agent Conduct

### A1: Unknown ≠ Absent
Data you haven't observed is `unknown`, not `absent`. Never invent
facts, fill gaps with assumptions, or claim something doesn't exist
because you haven't seen it. Say "I don't know" and ask.

### A2: Evidence, not verdicts
A "pass" without evidence is not a pass. Every claim must be backed
by a file in `evidence/`. Screenshots, logs, traces, or it didn't
happen.

### A3: Scope is closed before code
The PRD (or PROJECT.md for pre-PRD work) defines the scope. Do not
build features outside the scope, even if they seem like "obvious
improvements." Improvements go in a separate ticket.

### A4: Durable state over chat memory
Decisions, progress, and handoffs live in files. No agent should
need chat history to understand the current state. Write everything
down.

---

## What Happens When a Guardrail Is Triggered

1. The agent **stops** the action before it executes.
2. The agent **reports** which guardrail was triggered and why.
3. The agent **asks** for direction — never guesses, never proceeds.
4. The operator (CEO/Hermes) **decides** whether to override or redirect.

Guardrails are not punishments. They are safety nets. They exist
because even the best agents make mistakes, and the cost of one
unchecked destructive action can be hours of recovery.
