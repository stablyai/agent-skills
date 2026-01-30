# Stably Agent Skills

AI agent skills for the [Stably Playwright SDK](https://docs.stably.ai/getting-started/sdk-setup-guide).

## Available Skills

| Skill | Description | Install |
|-------|-------------|---------|
| [stably-sdk-setup](./skills/stably-sdk-setup) | Setup assistant for Stably Playwright SDK | `npx skills add stablyai/agent-skills --skill stably-sdk-setup` |
| [stably-sdk-rules](./skills/stably-sdk-rules) | AI rules for writing tests with Stably SDK | `npx skills add stablyai/agent-skills --skill stably-sdk-rules` |
| [stably-cli](./skills/stably-cli) | Expert assistant for Stably CLI commands | `npx skills add stablyai/agent-skills --skill stably-cli` |

## Installation

Install a specific skill:

```bash
# Setup skill - use when installing Stably SDK
npx skills add stablyai/agent-skills --skill stably-sdk-setup

# Rules skill - use when writing tests
npx skills add stablyai/agent-skills --skill stably-sdk-rules

# CLI skill - use when working with stably commands
npx skills add stablyai/agent-skills --skill stably-cli
```

Or install from URL:

```bash
npx skills add https://github.com/stablyai/agent-skills --skill stably-sdk-setup
```

## About Stably

Stably extends Playwright with AI-powered capabilities for more resilient testing:

- **AI Assertions** - `toMatchScreenshotPrompt()` for dynamic UI verification
- **AI Extraction** - `page.extract()` to pull data from the UI
- **AI Agent** - `agent.act()` for complex autonomous workflows

## Links

- [Stably Documentation](https://docs.stably.ai/getting-started/sdk-setup-guide)
- [Stably Dashboard](https://app.stably.ai)
- [Get API Key](https://auth.stably.ai/org/api_keys/)
- [skills.sh](https://skills.sh)

## License

MIT
