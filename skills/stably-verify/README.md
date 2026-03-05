# stably-verify

Run existing Playwright tests via Stably CLI to verify code changes work correctly.

## Installation

```bash
npx skills add stablyai/agent-skills --skill stably-verify
```

## When to Use

- After making code changes that need validation
- When an AI agent needs to confirm its changes don't break anything
- As a verification step in an iterate-and-verify development loop
- When refactoring code that has existing test coverage

## What It Does

Creates a tight feedback loop for AI agents:

1. **Run tests** - Execute `stably test` (scoped to relevant files)
2. **Analyze failures** - Read error output and traces
3. **Fix application code** - Fix the code, not the tests
4. **Re-verify** - Run tests again until all pass

## Quick Reference

| Command | Description |
|---------|-------------|
| `stably test` | Run all tests |
| `stably test tests/file.spec.ts` | Run specific test file |
| `stably test --grep "pattern"` | Run tests matching a pattern |
| `stably test --headed` | Run with visible browser |
| `stably test \|\| stably fix` | Run tests, auto-fix on failure |

## Related

- [stably-cli](../stably-cli) - Full CLI assistant (create, run, fix)
- [stably-sdk-rules](../stably-sdk-rules) - AI rules for writing tests
- [Verify with AI Agents guide](https://docs.stably.ai/use-cases/verify-with-ai-agents)
