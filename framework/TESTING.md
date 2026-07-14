# TESTING.md — Testing Strategy for Agent Swarms

> **Every agent tests. Every ticket ships with tests. Every test produces
> evidence.** This file defines HOW agents test, not just WHAT they test.

## Testing Philosophy

Tests are the primary feedback mechanism for agent swarms. A human can
look at a UI and say "that looks wrong." An agent cannot. Agents rely
on tests to verify their work. Therefore:

- **Tests are written BEFORE or WITH code, never after.**
- **Tests exist at the seam, not inside the implementation.**
- **Every test failure is a learning opportunity for LEARNINGS.md.**
- **Zero tolerance for flaky tests.** A flaky test is a bug — fix or delete.

### The Hard Rule: Evidence, Not Verdicts

**A "pass" without evidence is not a pass.** This is the most
important rule in this file. If you cannot produce a screenshot,
a log file, a trace, or a written artifact that proves a test
passed, then the test did not pass. Period.

- "Tests passed" in chat → not a pass.
- "I ran them and they're green" → not a pass.
- A terminal output pasted into a comment → not a pass.

A pass is: a file in `evidence/phase-N/` that anyone can open and
verify without running the tests themselves. Screenshots for visual
tests. Logs for backend tests. Playwright traces for E2E. Coverage
reports for unit tests. If it's not in `evidence/`, it didn't happen.

## Test Pyramid (Agent-Specific)

```
        ┌──────┐
        │ E2E  │  ← Critical paths only. Expensive. Run in CI.
        ├──────┤
        │Integr│  ← Module boundaries. Test that seams hold.
        ├──────┤
        │ Unit │  ← Functions, components, hooks. Fast. Run on every commit.
        └──────┘
```

| Layer | What it tests | Run frequency | Who runs it |
|---|---|---|---|
| **Unit** | Individual functions, components, hooks | Every commit (Husky) | Developer agent |
| **Integration** | Module boundaries, API contracts, DB queries | Every commit (Husky) | Developer agent |
| **E2E** | Critical user flows through the full stack | On PR, against staging | Tester agent (Playwright) |

## Testing Rules (Non-Negotiable)

### Rule 1: Every Component Handles ALL States

Every UI component test must cover:
- **Loading** — Skeleton, spinner, or Suspense fallback visible
- **Error** — Error message, retry button, or error boundary
- **Empty** — Zero-state message, CTA for first-time users
- **Data** — Normal render with valid data
- **Edge** — Null fields, boundary values, very long strings, special characters

```
✅ Good: 5 test cases covering all states
❌ Bad: 1 test case only testing the happy path
```

### Rule 2: Happy Paths AND Sad Paths

For every data source (API endpoint, hook, component prop):
- **Happy paths:** Every distinct valid state gets its own test
- **Sad paths:** Every distinct error type gets its own test

```
# Happy paths:
it('renders multiple wearable cards when API returns 2 devices')
it('renders single wearable card when API returns 1 device')

# Sad paths:
it('shows error state on 500 Internal Server Error')
it('shows error state on 403 Forbidden')
it('shows error state on network timeout')
it('handles null battery gracefully')
it('handles battery > 100 gracefully')
it('handles very long device names')
```

### Rule 3: Mock Data When Real Data Is Unavailable

When the API is not yet deployed, or the test environment has no fixtures:

```typescript
// Mock successful response
fetchMock.mockResolvedValue({
  json: () => ({ devices: [{ id: "w-1", name: "Watch", battery: 85, lastSync: new Date().toISOString() }] })
})

// Mock error response
fetchMock.mockRejectedValue(new Error('Internal Server Error'))

// Mock empty response
fetchMock.mockResolvedValue({ json: () => ({ devices: [] }) })
```

Mock data must match the API contract defined in ARCHITECTURE.md.

### Rule 4: Test at the Seam, Not the Implementation

Tests should verify behaviour at the module's interface. They should NOT
test internal implementation details. If you refactor the inside of a
module, tests should still pass.

```typescript
// ✅ Good: tests behaviour through the component's interface
it('shows device name and battery', () => {
  render(<WearableCard device={mockDevice} />)
  expect(screen.getByText('Apple Watch')).toBeVisible()
  expect(screen.getByText('85%')).toBeVisible()
})

// ❌ Bad: tests internal implementation
it('calls formatBattery internally', () => {
  const spy = jest.spyOn(utils, 'formatBattery')
  render(<WearableCard device={mockDevice} />)
  expect(spy).toHaveBeenCalled()
})
```

### Rule 5: Test Names Follow DAMP

**DAMP** = Descriptive and Meaningful Phrases.

```
✅ Good:
  it('shows empty state when user has no connected wearables')
  it('shows error state when API returns 500')
  it('renders step chart with 7 data points')

❌ Bad:
  it('test empty state')
  it('test error')
  it('renders correctly')
```

The test name should describe the behaviour from the USER's perspective.

## Evidence Collection

Every test run produces evidence. Evidence is NEVER summarized — raw
artifacts go in `evidence/phase-N/`.

### What to Capture

| Test Type | Evidence |
|---|---|
| **Unit tests** | Terminal output (pass/fail counts), coverage report |
| **Integration tests** | Terminal output, API response logs |
| **E2E tests** | Screenshots of every state, Playwright trace, video (on failure) |
| **Visual regression** | Before/after screenshots, WCAG contrast report |

### Evidence File Naming

```
evidence/phase-1/
├── e2e-dashboard-desktop.png       # Desktop viewport
├── e2e-dashboard-mobile.png        # Mobile viewport
├── e2e-dashboard-empty.png         # Empty state
├── e2e-dashboard-error.png         # Error state
├── e2e-results.xml                 # Playwright test results
├── unit-coverage.txt               # Coverage summary
├── wcag-contrast-report.txt        # DESIGN.md lint output
└── test-verdict.md                 # Human-readable summary
```

### Test Verdict Format

```markdown
# Test Verdict — Phase N, Ticket T-XXX

**Date:** YYYY-MM-DD
**Tester:** [agent identity]
**PR:** #NNN

## Summary
- Unit: 23 run, 23 passed, 0 skipped
- Integration: 8 run, 8 passed, 0 skipped
- E2E: 5 run, 5 passed, 0 skipped
- Coverage: 87.3% lines, 82.1% branches

## States Covered
- Loading: ✅
- Error: ✅ (500, 403, network timeout)
- Empty: ✅
- Data: ✅ (single, multiple)
- Edge: ✅ (null battery, long names, corrupted data)

## Evidence
- [e2e-dashboard-desktop.png](e2e-dashboard-desktop.png)
- [e2e-dashboard-mobile.png](e2e-dashboard-mobile.png)
- [wcag-contrast-report.txt](wcag-contrast-report.txt)

## Verdict
✅ All tests pass. All states covered. Ready for review.
```

## E2E Testing (Playwright)

### Configuration

```typescript
// playwright.config.ts
export default defineConfig({
  testDir: './e2e',
  retries: 2,             // Retry on CI flakiness
  workers: 4,             // Parallel execution
  reporter: [
    ['html', { outputFolder: 'evidence/playwright-report' }],
    ['junit', { outputFile: 'evidence/e2e-results.xml' }],
  ],
  use: {
    baseURL: process.env.STAGING_URL || 'http://localhost:3000',
    screenshot: 'on',       // Always capture — evidence
    video: 'retain-on-failure',
    trace: 'retain-on-failure',
  },
})
```

### Critical Paths to Test

- Login / Authentication flow
- Primary user journey (the "happy path" through the app)
- Data creation, editing, and deletion
- Error states (network failures, API errors, auth failures)
- Empty states (first-time user experience)
- Responsive breakpoints (mobile, tablet, desktop)
- WCAG accessibility (keyboard navigation, screen reader, contrast)

### Visual Consistency Check

```typescript
it('matches DESIGN.md tokens on dashboard page', async ({ page }) => {
  await page.goto('/dashboard')
  // Verify no raw hex colors
  const rawHexCount = await page.evaluate(() => {
    const all = document.querySelectorAll('*')
    return [...all].filter(el => {
      const style = getComputedStyle(el)
      return /#[0-9A-Fa-f]{6}/.test(style.color + style.backgroundColor)
    }).length
  })
  expect(rawHexCount).toBe(0)
})
```

## Testing in CI/CD

### Husky (Pre-Commit)

```bash
# .husky/pre-commit
npm test                    # All unit + integration tests
npm run test:coverage       # Fail if coverage drops
```

### GitHub Actions (PR)

```yaml
test:
  runs-on: ubuntu-latest
  steps:
    - run: npm test -- --coverage
    - run: npm run test:e2e  # Against staging
    - uses: actions/upload-artifact@v4
      with:
        name: test-evidence
        path: evidence/
```

## Test Failures — What Happens

1. **Husky catches it** → commit rejected. Agent fixes locally.
2. **CI catches it** → PR labeled `ci-failed`. Agent fixes and re-pushes.
3. **Tester agent catches it** → PR labeled `needs-changes`. Specific
   feedback posted. Agent fixes.
4. **Same test fails 3 times** → Agent blocks ticket. Operator reviews.

## Test Learnings → LEARNINGS.md

After every phase, test patterns are extracted:

```markdown
### 2026-07-14 — Flaky E2E: dashboard loading timeout

**What happened:** Dashboard E2E test timed out 2/5 runs because
wearable API response was > 5 seconds on cold start.

**Root cause:** Cold start on staging DB. No connection pooling.

**Action taken:** Increased Playwright timeout to 15s. Added connection
pooling to ARCHITECTURE.md database section.

**Prevention:** E2E tests should wait for API readiness before running.
Added health check to CI pipeline.
```

## Quick Reference

```bash
# Local testing (every commit):
npm test                    # Unit + integration
npm run test:coverage       # With coverage threshold
npm run lint                # Code quality
npm run typecheck           # Type safety

# E2E testing (PR review):
npm run test:e2e            # Against localhost
STAGING_URL=https://... npm run test:e2e  # Against staging

# Visual regression:
npx @google/design.md lint DESIGN.md       # WCAG contrast
npx playwright test --config=visual.config.ts  # Screenshot comparison

# Evidence collection:
mkdir -p evidence/phase-N
cp test-results/* evidence/phase-N/
```
