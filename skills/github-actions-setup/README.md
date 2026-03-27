# github-actions-setup

Setup assistant for running Stably Playwright tests in GitHub Actions. Detects your project setup and generates a production-ready workflow file.

## Installation

```bash
npx skills add stablyai/agent-skills --skill github-actions-setup
```

## When to Use

- Setting up CI/CD for Stably Playwright tests for the first time
- Debugging a failing GitHub Actions workflow for Playwright tests
- Adding self-healing (auto-fix + PR) to your test pipeline
- Configuring scheduled test runs (nightly, etc.)

## What It Does

The skill detects your project setup and generates a `.github/workflows/stably-e2e.yml` file that:

1. **Detects** your package manager (npm/pnpm/yarn), Node.js version, test directory, and monorepo structure
2. **Generates** a workflow with correct setup steps (corepack, caching, browser install)
3. **Handles common pitfalls** like missing `corepack enable` for pnpm, missing browser binaries, and monorepo working directories
4. **Guides secrets setup** for `STABLY_API_KEY` and `STABLY_PROJECT_ID`

## Workflow Features

| Feature | Description |
|---------|-------------|
| **Basic** | Run tests on push/PR to main |
| **Self-healing** | Auto-fix failures with `stably fix` and create a PR |
| **Scheduled** | Run tests on a cron schedule (e.g., nightly) |

## Common Pitfalls It Prevents

| Pitfall | What goes wrong | How the skill handles it |
|---------|----------------|------------------------|
| Missing corepack | `pnpm: command not found` | Adds `corepack enable` before `setup-node` |
| Missing browsers | `browserType.launch: Executable doesn't exist` | Adds `stably install --with-deps chromium` step |
| Bare `stably` command | `stably: command not found` | Uses `npx stably` / `pnpm exec stably`; ensures `stably` package is in devDependencies |
| Missing env vars | Auth failures in CI | Sets job-level env from GitHub Secrets |
| Monorepo path | Tests run in wrong directory | Sets `working-directory` on the job |

## Related

- [stably-cli](../stably-cli) - Full CLI assistant (includes CI/CD templates)
- [stably-sdk-setup](../stably-sdk-setup) - SDK installation and configuration
