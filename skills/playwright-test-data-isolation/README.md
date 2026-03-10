# playwright-test-data-isolation

Playwright-first strategy for shared DB + shared test accounts. Achieve safe parallel E2E tests with deterministic cleanup — no data collisions, no flaky tests from cross-test interference.

## Installation

```bash
npx skills add stablyai/agent-skills --skill playwright-test-data-isolation
```

## When to Use

- Playwright tests share a database and keep colliding
- Running E2E safely with shared test accounts
- Isolating test data in CI with parallel workers
- Cleaning up test data without deleting real/baseline data
- Designing fixtures for safe create-and-cleanup patterns

## What It Does

This skill guides your agent to implement a test data isolation strategy that:

1. **Namespaces** all test-created data with unique prefixes (`e2e-<run>-<worker>-<uuid>`)
2. **Tracks cleanup** by exact IDs — never broad filters
3. **Treats shared accounts/workspaces as read-only** containers
4. **Serializes** tests that must mutate shared state
5. **Adds a stale-data janitor** as a backup safety net

## Core Contract

Every test must:

| Rule | Description |
|------|-------------|
| Create | Create the entities it needs (test-owned data) |
| Mutate own data only | Never modify entities created by other tests |
| Read-only shared state | Treat shared accounts, workspaces, and settings as read-only |
| ID-based cleanup | Clean up by exact IDs captured during the test |

## Key Patterns

- **Auth reuse** — Setup project signs in once, parallel tests reuse stored auth state
- **Namespace fixture** — Custom Playwright fixture generates unique namespace per test
- **Cleanup tracker** — LIFO cleanup stack ensures reverse-order teardown by exact IDs
- **Serial suites** — `test.describe.configure({ mode: "serial" })` for shared-state mutation tests
- **Janitor project** — Scheduled cleanup of stale `e2e-*` data older than a threshold

## Related

- [stably-cli](../stably-cli) - Full Stably CLI assistant for test management
- [stably-verify](../stably-verify) - Verify application behavior with AI-powered Playwright tests
