# stably-cli

Expert assistant for the Stably CLI tool.

## Installation

```bash
npx skills add stablyai/agent-skills --skill stably-cli
```

## When to Use

- Creating Playwright tests with AI using `stably create`
- Running tests with `stably test`
- Running tests with remote environments using `stably --env <name> test`
- Auto-fixing failing tests with `stably fix`
- Using the interactive Stably agent
- Setting up Stably CLI in a new environment

## What It Does

Provides guidance for all Stably CLI commands:

- **Authentication**: `stably login`, `stably logout`, `stably whoami`
- **Setup**: `stably init`, `stably install`
- **Interactive Agent**: `stably` for conversational test work
- **Test Creation**: `stably create <prompt>` for headless test generation
- **Test Execution**: `stably test` with Stably reporter
- **Remote Environments**: `stably env list`, `stably env inspect`, and `stably --env <name> test`
- **Test Repair**: `stably fix [runId]` for AI-powered fixes

## Quick Reference

| Command | Description |
|---------|-------------|
| `stably` | Launch interactive agent |
| `stably init` | Initialize project |
| `stably create <prompt>` | Create tests with AI |
| `stably test` | Run tests |
| `stably --env <name> test` | Run tests with named remote environment variables |
| `stably env inspect <name>` | Inspect available variables in a remote env |
| `stably fix [runId]` | Auto-fix failures |

## Related

- [stably-sdk-setup](../stably-sdk-setup) - Full SDK setup assistant
- [stably-sdk-rules](../stably-sdk-rules) - AI rules for writing tests
