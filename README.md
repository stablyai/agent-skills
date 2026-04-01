# Stably Agent Skills

AI agent skills for the [Stably Playwright SDK](https://docs.stably.ai/getting-started/sdk-setup-guide).

## Available Skills

| Skill | Description | Install |
|-------|-------------|---------|
| [playwright-test-data-isolation](./skills/playwright-test-data-isolation) | Safe Playwright strategy for shared DB/shared accounts with deterministic cleanup and parallel-safe data ownership | `npx skills add stablyai/agent-skills --skill playwright-test-data-isolation` |
| [stably-sdk-setup](./skills/stably-sdk-setup) | Setup assistant for Stably Playwright SDK | `npx skills add stablyai/agent-skills --skill stably-sdk-setup` |
| [stably-sdk-rules](./skills/stably-sdk-rules) | AI rules for writing tests with Stably SDK | `npx skills add stablyai/agent-skills --skill stably-sdk-rules` |
| [stably-cli](./skills/stably-cli) | Expert assistant for Stably CLI commands (including `plan`, `create`, `fix`, `verify`, and remote envs via `--env`) | `npx skills add stablyai/agent-skills --skill stably-cli` |
| [stably-verify](./skills/stably-verify) | Run existing tests to verify code changes work — ideal for AI agent iteration loops | `npx skills add stablyai/agent-skills --skill stably-verify` |
| [github-actions-setup](./skills/github-actions-setup) | Setup assistant for running Stably Playwright tests in GitHub Actions CI/CD | `npx skills add stablyai/agent-skills --skill github-actions-setup` |

## Installation

Install all skills at once:

```bash
npx skills add stablyai/agent-skills
```

Or install a specific skill:

```bash
# Playwright shared-environment safety/data isolation strategy
npx skills add stablyai/agent-skills --skill playwright-test-data-isolation

# Setup skill - use when installing Stably SDK
npx skills add stablyai/agent-skills --skill stably-sdk-setup

# Rules skill - use when writing tests
npx skills add stablyai/agent-skills --skill stably-sdk-rules

# CLI skill - use when working with stably commands
npx skills add stablyai/agent-skills --skill stably-cli

# Verify skill - use when validating code changes with existing tests
npx skills add stablyai/agent-skills --skill stably-verify

# GitHub Actions skill - use when setting up CI/CD
npx skills add stablyai/agent-skills --skill github-actions-setup
```

Or install from URL:

```bash
npx skills add https://github.com/stablyai/agent-skills --skill stably-sdk-setup
```

## About Stably

Stably extends Playwright with AI-powered capabilities for more resilient testing:

- **AI Assertions** - `aiAssert()` for dynamic UI verification
- **AI Extraction** - `page.extract()` to pull data from the UI
- **AI Agent** - `agent.act()` for complex autonomous workflows
- **AI Locator Finding** - `page.getLocatorsByAI()` to find elements using natural language

## Links

- [Stably Documentation](https://docs.stably.ai/getting-started/sdk-setup-guide)
- [Stably Dashboard](https://app.stably.ai)
- [Get API Key](https://auth.stably.ai/org/api_keys/)
- [skills.sh](https://skills.sh)

## License

MIT
