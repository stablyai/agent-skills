# stably-cli

Expert assistant for the Stably CLI tool.

## Installation

```bash
npx skills add stablyai/agent-skills --skill stably-cli
```

## When to Use

- Planning test coverage with `stably plan`
- Creating Playwright tests with AI using `stably create`
- Running tests with `stably test`
- Running tests with remote environments using `stably --env <name> test`
- Auto-fixing failing tests with `stably fix`
- Verifying app behavior with `stably verify`
- Checking test run history with `stably runs list`
- Viewing run details with `stably runs view`
- Analyzing flaky tests with `stably analytics flaky`
- Analyzing test failures with `stably analytics failures`
- Setting up Stably CLI in a new environment

## What It Does

Provides guidance for all Stably CLI commands:

- **Authentication**: `stably login`, `stably logout`, `stably whoami`
- **Setup**: `stably init`, `stably install`
- **Interactive Agent**: `stably` (no args) — **human only**, hangs AI agents
- **Coverage Planning**: `stably plan [prompt]` to discover gaps and generate `test.fixme()` skeletons
- **Test Creation**: `stably create <prompt>` for headless test generation
- **Test Execution**: `stably test` with Stably reporter
- **Remote Environments**: `stably env list`, `stably env inspect`, and `stably --env <name> test`
- **Test Repair**: `stably fix [runId]` for AI-powered fixes
- **Verification**: `stably verify <prompt>` for behavior checks without generating test files
- **Run History**: `stably runs list`, `stably runs view <runId>` for historical context across sessions
- **Analytics**: `stably analytics flaky`, `stably analytics failures` for test health insights

## Quick Reference

| Command | Description |
|---------|-------------|
| `stably` | Interactive chat — **do NOT use from AI agents** |
| `stably init` | Initialize project |
| `stably plan` | Discover coverage gaps, generate `test.fixme()` skeletons |
| `stably plan "focus on X"` | Scoped coverage plan |
| `stably create <prompt>` | Create tests with AI |
| `stably test` | Run tests |
| `stably --env <name> test` | Run tests with named remote environment variables |
| `stably env inspect <name>` | Inspect available variables in a remote env |
| `stably fix [runId]` | Auto-fix failures |
| `stably verify "description"` | Verify app behavior |
| `stably runs list` | List recent test runs |
| `stably runs view <runId>` | View run details |
| `stably analytics flaky` | Show most flaky tests by flaky rate |
| `stably analytics failures` | Show most failing tests by failure rate |

## Related

- [stably-plan](../stably-plan) - Coverage planning and `test.fixme()` generation
- [stably-sdk-setup](../stably-sdk-setup) - Full SDK setup assistant
- [stably-sdk-rules](../stably-sdk-rules) - AI rules for writing tests
