# DEBUGGING.md — Debugging & Error Logging for Agent Swarms

> **Every agent logs. Every error is traceable. Every session is debuggable.**
> When an agent fails, you should never have to guess what happened.

## Philosophy

Agents are black boxes by default — they think, write code, commit,
and either succeed or fail. When they fail, you need to know WHY
without re-running them. Debugging starts at failure time, not after.

- **Log everything relevant.** Disk is cheap. Lost context is expensive.
- **Structured logs over prose.** JSON lines, not paragraphs.
- **One log file per agent session.** No shared logs. No interleaving.
- **Logs survive agent termination.** Written to disk, not stdout.
- **Multi-agent RCA over single-agent debugging.** The agent that wrote the
  code is the least qualified to find its own root cause. A separate Tester
  or Reviewer agent performs RCA using historical patterns, semantic
  matching, and runtime verification — not guesswork.
- **Every session has a correlation_id.** Errors trace across agents
  through the ticket chain. A test failure in T-009 can be traced back
  to an API contract change in T-005.

## Academic Foundation

The multi-agent RCA approach is validated by:

- **MA-RCA (Fu et al., 2025):** 4-agent framework (RCA Agent, Retrieval
  Agent, Validation Agent, Report Agent) achieves 95.2% F1 on cloud-native
  platforms. Ablation study proves multi-agent decomposition prevents error
  propagation and hallucination — single-agent RCA performs significantly
  worse. Source: doi.org/10.1007/s40747-025-02096-0
- **GALR (2026):** Graph-based root cause localization with LLM-assisted
  recovery for microservice systems.
- **Distributed Tracing survey (2023):** Rule-induction systems for
  troubleshooting native cloud applications.

Key insight: "Single-agent architectures exacerbate error propagation and
context-switching failures during multi-step RCA reasoning." This directly
informs our design: the Developer agent logs errors with context, but the
Tester/Reviewer agent performs root cause analysis using the full historical
record. See `VERSION-CONTROL.md` Code Review phase for the review workflow.

## Correlation ID

Every agent session, every log entry, and every error carries a
correlation_id that links work across the full ticket chain:

```
correlation_id = "{milestone}-{phase}-{ticket_chain}"

Example: "m3-p1-T005-T007-T009"
  → T-005 built the API
  → T-007 consumed the API (Dashboard)
  → T-009 depends on T-007 (Notifications)

When T-009 has a test failure:
  1. grep correlation_id logs/ to see the full chain
  2. Find the earliest error in the chain
  3. Trace back: T-009 failure → T-007 changed API assumption → T-005 contract changed
```

### Correlation ID in Log Entries

```jsonl
{"ts":"...","level":"info","event":"session.start","correlation_id":"m3-p1-T005-T007-T009","ticket":"T-007"}
{"ts":"...","level":"error","event":"test.failure","correlation_id":"m3-p1-T005-T007-T009","test":"renders dashboard","rca_hint":"api_contract_drift","detail":"ARCHITECTURE.md says {devices} but T-005 API returns {data:{devices}}","fix_suggestion":"Update mock data in T-007 to match actual API response from T-005"}
```

## Log File Structure

```
logs/
├── T-005-2026-07-14-developer.jsonl    # Agent T-005, Developer role
├── T-005-2026-07-14-developer.err.log  # Stderr output
├── T-007-2026-07-14-developer.jsonl    # Agent T-007
├── T-007-2026-07-15-tester.jsonl       # Tester review of T-007
└── dispatcher-2026-07-14.jsonl         # Dispatcher events
```

## Structured Log Format (JSONL)

Every log entry is one JSON line. This makes logs grep-able, jq-able,
and machine-readable.

```jsonl
{"ts":"2026-07-14T09:15:00Z","level":"info","event":"session.start","profile":"developer","ticket":"T-007","workspace":"/tmp/kanban/T-007"}
{"ts":"2026-07-14T09:15:02Z","level":"info","event":"bootstrap.start","files":["AGENTS.md","ARCHITECTURE.md","DATABASE.md","DESIGN.md","TESTING.md"]}
{"ts":"2026-07-14T09:15:05Z","level":"info","event":"bootstrap.done","duration_ms":3000}
{"ts":"2026-07-14T09:15:10Z","level":"info","event":"ticket.read","id":"T-007","blockers":["T-005","T-006"]}
{"ts":"2026-07-14T09:15:15Z","level":"info","event":"patterns.scan","reference_file":"apps/web/src/pages/patients/index.tsx","patterns_found":4}
{"ts":"2026-07-14T09:20:00Z","level":"info","event":"file.created","path":"apps/web/src/hooks/useWearables.ts","pattern":"usePatients"}
{"ts":"2026-07-14T09:25:00Z","level":"info","event":"test.run","file":"dashboard.test.tsx","total":8,"passed":8,"failed":0}
{"ts":"2026-07-14T09:30:00Z","level":"info","event":"self_inspection.start","files_changed":9}
{"ts":"2026-07-14T09:30:15Z","level":"info","event":"self_inspection.done","violations":0}
{"ts":"2026-07-14T09:30:30Z","level":"info","event":"commit","hash":"a1b2c3d","type":"feat","scope":"dashboard"}
{"ts":"2026-07-14T09:31:00Z","level":"info","event":"push","branch":"ticket/T-007-user-dashboard"}
{"ts":"2026-07-14T09:32:00Z","level":"info","event":"pr.open","number":142,"url":"https://github.com/..."}
{"ts":"2026-07-14T09:32:30Z","level":"info","event":"kanban.complete","status":"done","tests_passed":8}
{"ts":"2026-07-14T09:32:30Z","level":"info","event":"session.end","duration_ms":1050000}
```

### Log Levels

| Level | When | Example |
|---|---|---|
| `info` | Normal flow — file created, test passed, commit done | Agent is progressing normally |
| `warn` | Recoverable issue — pattern mismatch found and fixed, test flaked once | Something went wrong but agent handled it |
| `error` | Non-recoverable — test fails 3 times, merge conflict can't resolve | Agent needs intervention |
| `debug` | Detailed state — exact diff, hook output, API response bodies | Deep investigation |

## Error Logging

### What to Log on Error

When an agent hits an error, log:

```jsonl
{"ts":"...","level":"error","event":"test.failure",
 "file":"dashboard.test.tsx",
 "test":"shows empty state when no wearables connected",
 "error":"Expected 'Connect your first wearable' but got 'No devices found'",
 "diff":"- Connect your first wearable\n+ No devices found",
 "stack":"at DashboardPage (index.tsx:42)\n    at render (test-utils.ts:15)"}
```

### Error Traceability

Every error must include enough context to reproduce WITHOUT re-running the agent:

1. **What the agent was doing** — event name (test.run, commit, push)
2. **What it expected** — the assertion or expected state
3. **What actually happened** — the error message, diff, or actual state
4. **Where** — file path, line number, test name
5. **Stack trace** — full call stack
6. **Agent state** — ticket ID, files changed, last successful step

### Common Errors and How to Debug Them

| Error | Log Entry | How to Debug |
|---|---|---|
| **Test failure** | `event:"test.failure"` + diff | Read the diff. Check if mock data matches API contract. Check DESIGN.md tokens. |
| **Pattern mismatch** | `event:"self_inspection.violation"` + file | Compare file against reference. grep for raw hex, different naming, different import order. |
| **Merge conflict** | `event:"rebase.conflict"` + files | Read both versions. Check which ticket created the conflict. Check ticket isolation. |
| **CI failure** | CI log (not agent log) | Read CI output. Most common: lint, typecheck, build. |
| **Husky rejection** | `event:"husky.rejected"` + hook | Read the hook output. Fix locally. |
| **API contract mismatch** | `event:"api.mismatch"` + expected vs actual | Compare against ARCHITECTURE.md API contract. |
| **Token violation** | `event:"design.violation"` + file + line | grep for raw hex. Replace with DESIGN.md token. |
| **Scope creep** | `event:"scope.violation"` + files | Remove files outside ticket scope. |

## Quick Debugging Commands

```bash
# Find all errors in today's logs:
grep '"level":"error"' logs/*-$(date +%Y-%m-%d)*.jsonl | jq .

# Find all test failures:
grep '"event":"test.failure"' logs/*.jsonl | jq '{test: .test, error: .error, file: .file}'

# Find what a specific agent did:
cat logs/T-007-*.jsonl | jq '{ts, event, file: .path, test: .test}'

# Find all pattern violations:
grep '"event":"self_inspection.violation"' logs/*.jsonl | jq .

# Timeline of one agent session:
cat logs/T-007-*.jsonl | jq -r '[.ts, .event, .path // .test // ""] | @tsv' | column -t

# Count errors by type:
grep '"level":"error"' logs/*.jsonl | jq -r .event | sort | uniq -c | sort -rn

# Find the last successful step before a failure:
grep -B1 '"level":"error"' logs/T-007-*.jsonl | tail -3

# Watch logs in real-time (during agent run):
tail -f logs/T-007-*.jsonl | jq .
```

## Integration with the Pipeline

### Agent Bootstrap (ROLE.md)

At session start, the agent creates its log file:

```
export SESSION_ID="T-007-$(date +%Y-%m-%d)-developer"
export LOG_FILE="logs/${SESSION_ID}.jsonl"
echo '{"ts":"'$(date -u +%FT%TZ)'","level":"info","event":"session.start"}' >> $LOG_FILE
```

### Key Events to Log

Every agent MUST log these events:

| Event | When | Data |
|---|---|---|
| `session.start` | Agent spawns | profile, ticket, workspace |
| `bootstrap.start/done` | Reading startup files | files list, duration |
| `ticket.read` | Reading ticket | ticket ID, blockers |
| `patterns.scan` | Scanning codebase | reference file, patterns found |
| `file.created/modified` | Writing files | path, pattern followed |
| `test.run` | Running tests | file, total, passed, failed |
| `test.failure` | Test fails | test name, error, diff, stack |
| `self_inspection.start/done` | Pre-commit check | files changed, violations |
| `self_inspection.violation` | Pattern/clean code issue | file, violation type, detail |
| `commit` | Git commit | hash, type, scope |
| `push` | Git push | branch, remote |
| `pr.open` | Opening PR | number, URL |
| `rebase.conflict` | Merge conflict | files, both branches |
| `kanban.complete/block` | Kanban handoff | status, summary |
| `session.end` | Agent finishes | duration, outcome |

### Error Recovery

When an agent fails, the operator can:

1. **Read the log:** `cat logs/T-007-*.jsonl | jq 'select(.level=="error")'`
2. **Find the root cause:** Last successful event before first error
3. **Fix the issue:** Update ARCHITECTURE.md, patch a skill, adjust a ticket
4. **Re-run or reclaim:** `hermes kanban reclaim T-007`

### Learning from Errors

Every error pattern that repeats becomes a learning:

```markdown
### 2026-07-14 — Recurring: test failures from API contract drift

**What happened:** 3 agents in Phase 1 had test failures because
ARCHITECTURE.md API contract said `{ devices: [...] }` but the actual
API returned `{ data: { devices: [...] } }`.

**Root cause:** ARCHITECTURE.md was updated after T-005 shipped but
the update didn't propagate to later tickets.

**Action:** Added API contract verification to pre-flight checklist.
ARCHITECTURE.md changes now auto-comment on all dependent tickets.

**Frequency:** 3/9 tickets in Phase 1. Mitigated in Phase 2.
```

## Stderr Capture

Raw agent output (stderr, tool output, CLI chatter) goes to a separate file:

```bash
# Agent spawn command captures stderr:
hermes -p developer chat -q "..." 2> logs/T-007-2026-07-14-developer.err.log
```

This captures:
- Tool call outputs (terminal, browser, file reads)
- Model responses (truncated)
- Python/node errors from scripts
- Git output (push, pull, rebase)

For structured debugging, read the JSONL log. For raw investigation,
read the stderr log.

## Multi-Agent RCA Workflow

When an error is detected (test failure, CI failure, pattern violation):

### Phase 1: Developer Agent Logs

The agent that encountered the error logs it with full context:

```jsonl
{"ts":"...","level":"error","event":"test.failure",
 "correlation_id":"m3-p1-T005-T007-T009",
 "ticket":"T-007","test":"renders dashboard",
 "error":"TypeError: Cannot read property 'battery' of null",
 "rca_hint":"null_field","detail":"battery is null in API response",
 "fix_suggestion":"Add null guard in WearableCard component",
 "stack":"at WearableCard (WearableCard.tsx:15)"}
```

### Phase 2: Retrieval (Context-Mode)

Before the Tester agent investigates, it searches historical patterns:

```bash
# Search for similar errors in the knowledge base:
context-mode search "null battery api response" --source "agent-logs" --limit 5
context-mode search "TypeError Cannot read property" --source "learnings" --limit 5

# If a matching pattern is found:
#   → Apply known fix from LEARNINGS.md
#   → No further investigation needed
# If no match:
#   → Proceed to Phase 3 (Validation)
```

### Phase 3: Tester Agent Validates

The Tester agent does NOT trust the Developer's rca_hint. It validates:

```bash
# 1. Read the failing test
# 2. Check the API contract in ARCHITECTURE.md
# 3. Check the actual API response (from T-005 handoff)
# 4. Compare: contract says {devices: [...], battery: number}
#             actual response has battery: null
# 5. Root cause: T-005 API contract is wrong (battery should be number | null)
# 6. Fix: Update ARCHITECTURE.md contract, add null guard in T-007
```

### Phase 4: Report Agent (Learning Extraction)

```jsonl
{"ts":"...","level":"info","event":"rca.resolved",
 "correlation_id":"m3-p1-T005-T007-T009",
 "root_cause":"API contract in ARCHITECTURE.md declared battery as number,
              but Fitbit API returns null for power-save devices",
 "fix_applied":"Updated ARCHITECTURE.md contract: battery: number | null.
               Added null guard in WearableCard.tsx. 8 tests re-pass.",
 "learning":"Null fields in API responses must be nullable in ARCHITECTURE.md
             contracts. Pre-flight checklist updated to verify this.",
 "prevention":"ARCHITECTURE.md changes auto-comment on dependent tickets"}
```

### Why Multi-Agent RCA Works

| Single-Agent RCA | Multi-Agent RCA (Our Approach) |
|---|---|
| Developer guesses root cause | Tester validates against ARCHITECTURE.md, DESIGN.md, LEARNINGS.md |
| No historical pattern matching | Retrieval step searches logs + learnings |
| Fix applied without verification | Validation step re-runs tests + checks contract |
| Error stays in one agent's log | Correlation ID links across the full chain |
| Same bug repeats in next phase | Learning extracted → LEARNINGS.md → prevents recurrence |

## Context-Mode Integration

Reduce context pressure during debugging sessions:

```bash
# Index framework docs (once per project setup):
context-mode index framework/ --ext ".md" --source "framework"

# Index logs for pattern search (after each phase):
context-mode index logs/ --source "agent-logs"

# Quick RCA: search instead of loading full logs
context-mode search "rca_hint: api_contract_drift" --source "agent-logs"
context-mode search "TypeError" --source "agent-logs" --limit 5

# Search framework without loading 12 files into context
context-mode search "self-inspection" --source "framework"
```

See the `context-mode` Hermes skill for full usage patterns.
