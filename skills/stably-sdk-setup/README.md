# stably-sdk-setup

Setup assistant for Stably Playwright SDK.

## Installation

```bash
npx skills add stablyai/agent-skills --skill stably-sdk-setup
```

## When to Use

- Setting up Stably SDK in a new project
- Migrating from `@playwright/test` to `@stablyai/playwright-test`
- Configuring Stably reporter for CI/CD

## What It Does

Guides through a 9-step installation process:

1. Check for existing Playwright setup
2. Verify Playwright installation status
3. Install/Update Stably SDK
4. Replace Playwright imports
5. Setup AI rules & commands
6. Configure Playwright config
7. Setup API credentials
8. Install Playwright MCP (optional)
9. Run verification test

## Related

- [stably-sdk-rules](../stably-sdk-rules) - AI rules for writing tests
