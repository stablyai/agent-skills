# stably-verify

Verify that your application works correctly using `stably verify` — describe expected behavior in plain English, get a PASS/FAIL/INCONCLUSIVE verdict from a real browser.

## Installation

```bash
npx skills add stablyai/agent-skills --skill stably-verify
```

## When to Use

- After making code changes that need validation
- When an AI agent needs to confirm a feature works in a real browser
- As a verification step in a build-then-verify iteration loop
- Quick feature checks without writing test files

## What It Does

`stably verify` launches an AI agent that:

1. **Navigates** your app in a real browser
2. **Interacts** with the UI (clicks, fills forms, scrolls)
3. **Takes screenshots** as evidence
4. **Reports** a structured PASS / FAIL / INCONCLUSIVE verdict

No test files are created — it verifies and reports.

## Quick Reference

| Command | Description |
|---------|-------------|
| `stably verify "description"` | Verify app behavior |
| `stably verify "..." --url <url>` | Verify with specific starting URL |
| `stably verify "..." --max-budget 10` | Set budget cap (default: $5) |
| `stably verify "..." --no-interactive` | Non-interactive mode for CI |
| `stably verify "..." --browser cloud` | Use cloud browser |

Exit codes: `0` = PASS, `1` = FAIL, `2` = INCONCLUSIVE

## Related

- [stably-cli](../stably-cli) - Full CLI assistant (create, run, fix, verify)
- [Verify with AI Agents guide](https://docs.stably.ai/use-cases/verify-with-ai-agents)
