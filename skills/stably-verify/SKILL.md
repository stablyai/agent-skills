---
name: stably-verify
description: |
  Run existing Playwright tests via Stably CLI to verify code changes. Use when
  an AI agent has made code changes and needs to validate they work correctly.
  This skill creates a tight feedback loop: make changes -> run tests -> fix
  failures -> repeat until green. Triggers on: "verify this works",
  "run tests to check", "stably verify", "validate my changes",
  "check if tests pass", "verify my changes", "run e2e to verify".
license: MIT
metadata:
  author: stably
  version: '1.0.0'
---

# Verify Code Changes with Stably

Run your existing Playwright test suite via Stably CLI to verify that code changes work correctly. This skill creates a tight feedback loop: make changes -> run tests -> fix failures -> repeat until green.

## Pre-flight

**Always run `stably --version` first.** If not found, install with `npm install -g stably` or use `npx stably`. Requires Node.js 20+, Playwright, and a [Stably account](https://app.stably.ai).

```bash
# Check if Stably CLI is available
stably --version

# Check for required env vars
echo "STABLY_API_KEY: ${STABLY_API_KEY:+set}"
echo "STABLY_PROJECT_ID: ${STABLY_PROJECT_ID:+set}"
```

## How It Works

1. **Detect Changes** - Identify which files were modified (via `git diff`)
2. **Select Tests** - Find relevant test files that cover the changed code
3. **Run Tests** - Execute with `stably test` (or scoped to specific files)
4. **Analyze Failures** - If tests fail, read error output and traces
5. **Fix Code** - Fix the **application code** to make tests pass (not the tests)
6. **Re-verify** - Run tests again until all pass

## Running Tests

```bash
# Run all tests
stably test

# Run specific test file
stably test tests/login.spec.ts

# Run tests matching a pattern
stably test --grep "checkout"

# Run with headed browser for debugging
stably test --headed

# Run with remote environment variables
stably --env staging test
```

All Playwright CLI options pass through: `--workers`, `--retries`, `--project`, `--reporter`, etc.

## Verification Loop

When tests fail after code changes:

1. **Read the failure** - Check test output for what broke
2. **Identify root cause** - Determine if the code change caused the failure
3. **Fix the application code** - Do NOT modify tests unless they have genuine bugs unrelated to your changes
4. **Re-run** - Execute `stably test` again
5. **Repeat** until all tests pass

**Critical rule:** Tests are the source of truth. When a test fails after your code change, fix the code — not the test. Tests define the expected behavior of the application.

## Scoping Tests to Changes

Don't run the full suite on every iteration — scope to what's relevant:

```bash
# Run only tests related to the feature you changed
stably test tests/checkout.spec.ts tests/cart.spec.ts

# Run tests matching a keyword
stably test --grep "payment"

# Run a specific project
stably test --project=chromium
```

To find relevant test files, check:
- Test files that import or reference the changed modules
- Test files whose names match the feature area (e.g., changed `auth.ts` -> run `auth.spec.ts`)
- `git log` for test files that were last modified alongside the changed code

## Chaining with stably fix

If failures are hard to diagnose from test output alone, chain with `stably fix` for deeper AI-powered analysis:

```bash
# Run tests, auto-fix on failure, re-run
stably test || stably fix && stably test
```

`stably fix` analyzes traces, screenshots, logs, and DOM state to provide better diagnostics than raw test output.

## Long-Running Commands (AI Agents)

`stably test` typically completes in under a minute, but large suites can take longer.

| Agent | Configuration |
|-------|--------------|
| **Claude Code** | `timeout: 300000` on Bash tool |
| **Cursor** | `block_until_ms: 300000` |

## Troubleshooting

| Problem | Solution |
|---------|----------|
| Tests pass locally but fail in agent | Check env vars are set: `STABLY_API_KEY`, `STABLY_PROJECT_ID` |
| "No tests found" | Verify test file path and `testDir` in `playwright.config.ts` |
| Flaky failures | Re-run with `--retries=1` to distinguish flaky from real failures |
| Can't see what failed | Use `--headed` mode or check traces in `.stably/` directory |

## Links

[Docs](https://docs.stably.ai) · [Verify Guide](https://docs.stably.ai/use-cases/verify-with-ai-agents) · [CLI Quickstart](https://docs.stably.ai/stably-cli/quickstart) · [Dashboard](https://app.stably.ai)
