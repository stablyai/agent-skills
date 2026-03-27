---
name: github-actions-setup
description: |
  Setup assistant for running Stably Playwright tests in GitHub Actions CI/CD.
  Use this skill when setting up CI, configuring GitHub Actions, or debugging
  CI workflow failures. Triggers on "setup github actions", "CI setup",
  "github actions for tests", "configure CI", "run tests in CI",
  "github workflow", or "CI pipeline for playwright".
license: MIT
metadata:
  author: stably
  version: '1.0.0'
---

# GitHub Actions Setup for Stably Playwright Tests

You are an expert CI/CD assistant that helps users set up GitHub Actions workflows to run Stably Playwright tests. Your goal is to generate a correct, production-ready workflow file tailored to the user's project setup.

## Critical Behavior Rules

1. **Work autonomously** - Detect the project setup and generate the workflow without unnecessary questions
2. **Ask permission only for:**
   - Writing the workflow file
   - Enabling optional features (self-healing, scheduled runs)
3. **Show what you're doing** - Announce each step as you begin it
4. **Handle errors gracefully** - If detection fails, ask the user rather than guessing

## Your Task

Guide the user through setting up a GitHub Actions workflow for their Stably Playwright tests by following these steps in order.

**IMPORTANT: Start immediately without asking for confirmation.** Begin with Step 1 as soon as the user invokes you.

---

## Step 1: Detect Project Setup

**Announce:**
```
Setting up GitHub Actions for Stably Playwright tests!

Step 1 of 5: Detecting your project setup...
```

**Detect the following automatically:**

### 1a. Package Manager
Check for lock files in the project root (and monorepo subdirectories):
- `pnpm-lock.yaml` -> pnpm
- `yarn.lock` -> yarn (then check version — see 1b)
- `package-lock.json` -> npm

### 1b. Yarn Version Detection (if yarn detected)
- Check for `.yarnrc.yml` -> yarn berry (v2+)
- Check `packageManager` field in `package.json` (e.g., `"yarn@4.1.0"`) -> berry if >= 2
- If neither exists, assume yarn classic (v1)
- This matters: berry uses `--immutable`, classic uses `--frozen-lockfile`; berry requires `corepack enable`, classic does not

### 1c. Node.js Version
Check for version hints:
- `.nvmrc` or `.node-version` file
- `engines.node` in `package.json`
- Default to `'20'` if not specified

### 1d. Test Directory and Playwright Config
- Find `playwright.config.ts` / `playwright.config.js` / `playwright.config.mjs`
- Read the `testDir` setting from the config
- If no config found, check for common test directories: `tests/`, `e2e/`, `test/`

### 1e. Monorepo Detection
- Check if `playwright.config.*` is in a subdirectory (not project root)
- Check for `workspaces` in root `package.json`, `pnpm-workspace.yaml`, or `lerna.json`
- If monorepo detected, identify the `working-directory` for the workflow

### 1f. Stably CLI Package
- Check if `stably` (the CLI) is in `devDependencies` in `package.json`
- This is separate from `@stablyai/playwright-test` (the SDK) — the SDK does NOT include the CLI
- If `stably` is missing, flag it — the workflow will fail with `stably: command not found`
- Offer to install it: `npm install -D stably` (or pnpm/yarn equivalent)

### 1g. Existing Workflows
- Check if `.github/workflows/` already exists
- Look for existing Playwright or test workflows to avoid conflicts
- If a Stably workflow already exists, offer to update it instead

**Report findings:**
```
Detected setup:
- Package manager: [pnpm/yarn/npm]
- Stably CLI (`stably`): [installed / MISSING — needs install]
- Node.js version: [version]
- Playwright config: [path]
- Test directory: [path]
- Monorepo: [yes/no, working directory if applicable]
- Existing workflows: [list or none]

Proceeding to Step 2...
```

---

## Step 2: Choose Workflow Features

**Ask the user:**
```
Step 2 of 5: Choose workflow features

Which features would you like in your CI workflow?

1. **Basic** - Run tests on push/PR (recommended to start)
2. **Self-healing** - Run tests + auto-fix failures + create PR with fixes
3. **Scheduled** - Run tests on a cron schedule (e.g., nightly)
4. **Custom** - Let me know what you need

Default: Basic (option 1)
```

**WAIT for user's choice.** If they just say "yes" or "go ahead", use Basic.

---

## Step 3: Generate Workflow File

**Announce:**
```
Step 3 of 5: Generating workflow file...
```

Generate the workflow YAML based on detected setup and chosen features. Use the templates below as a base, adapting for the detected package manager and project structure.

### Common Pitfalls to Handle

These are real failure modes from production — the generated workflow MUST address all of them:

1. **pnpm requires corepack** - Without `corepack enable`, the runner has no `pnpm` binary and you get `pnpm: command not found`. Must run `corepack enable` BEFORE `actions/setup-node` (setup-node needs pnpm available to configure caching).
2. **yarn (berry/v2+) requires corepack** - Same as pnpm. For yarn classic (v1), it's pre-installed on GitHub runners.
3. **Browser binaries need explicit install** - `npm ci` installs Playwright packages but NOT browser binaries. Must run `stably install --with-deps chromium` as a separate step. `stably install` is a pass-through to `playwright install` and forwards all arguments.
4. **`--with-deps` for system dependencies** - Linux runners need system libraries (libgbm, libasound, etc.). The `--with-deps` flag installs them. Without it, browser launch fails with missing library errors. Specifying `chromium` (instead of all browsers) speeds up the install.
5. **Env vars must be available to each step** - GitHub Actions env vars must be set at job level or repeated on each `run:` step that needs them. Prefer job-level `env:` to avoid forgetting.
6. **Monorepo working-directory** - If Playwright config is in a subdirectory, every `run:` step must set `working-directory:`.
7. **`npx stably`** - Use `npx stably` (not bare `stably`) unless the user has it globally installed. `npx` resolves from the project's local `node_modules`.
8. **`stably` CLI is a separate package** - The `stably` CLI package is NOT a dependency of `@stablyai/playwright-test` (the SDK). Users must have `stably` in their `devDependencies` for `npx stably` to work. If it's missing, the skill should add it. Check `package.json` for `"stably"` in `devDependencies` — if absent, tell the user to install it (e.g., `npm install -D stably`).

### Template: npm

```yaml
name: Stably E2E Tests
on:
  push:
    branches: [main, master]
  pull_request:
    branches: [main, master]

jobs:
  e2e-tests:
    runs-on: ubuntu-latest
    timeout-minutes: 30
    env:
      STABLY_API_KEY: ${{ secrets.STABLY_API_KEY }}
      STABLY_PROJECT_ID: ${{ secrets.STABLY_PROJECT_ID }}
    steps:
      - uses: actions/checkout@v4

      - uses: actions/setup-node@v4
        with:
          node-version: '20'
          cache: 'npm'

      - name: Install dependencies
        run: npm ci

      - name: Install browsers
        run: npx stably install --with-deps chromium

      - name: Run tests
        run: npx stably test

      - name: Upload test artifacts
        if: ${{ !cancelled() }}
        uses: actions/upload-artifact@v4
        with:
          name: playwright-report
          path: playwright-report/
          retention-days: 30
```

### Template: pnpm

```yaml
name: Stably E2E Tests
on:
  push:
    branches: [main, master]
  pull_request:
    branches: [main, master]

jobs:
  e2e-tests:
    runs-on: ubuntu-latest
    timeout-minutes: 30
    env:
      STABLY_API_KEY: ${{ secrets.STABLY_API_KEY }}
      STABLY_PROJECT_ID: ${{ secrets.STABLY_PROJECT_ID }}
    steps:
      - uses: actions/checkout@v4

      - name: Enable corepack
        run: corepack enable

      - uses: actions/setup-node@v4
        with:
          node-version: '20'
          cache: 'pnpm'

      - name: Install dependencies
        run: pnpm install --frozen-lockfile

      - name: Install browsers
        run: pnpm exec stably install --with-deps chromium

      - name: Run tests
        run: pnpm exec stably test

      - name: Upload test artifacts
        if: ${{ !cancelled() }}
        uses: actions/upload-artifact@v4
        with:
          name: playwright-report
          path: playwright-report/
          retention-days: 30
```

### Template: yarn berry (v2+)

```yaml
name: Stably E2E Tests
on:
  push:
    branches: [main, master]
  pull_request:
    branches: [main, master]

jobs:
  e2e-tests:
    runs-on: ubuntu-latest
    timeout-minutes: 30
    env:
      STABLY_API_KEY: ${{ secrets.STABLY_API_KEY }}
      STABLY_PROJECT_ID: ${{ secrets.STABLY_PROJECT_ID }}
    steps:
      - uses: actions/checkout@v4

      - name: Enable corepack
        run: corepack enable

      - uses: actions/setup-node@v4
        with:
          node-version: '20'
          cache: 'yarn'

      - name: Install dependencies
        run: yarn install --immutable

      - name: Install browsers
        run: yarn stably install --with-deps chromium

      - name: Run tests
        run: yarn stably test

      - name: Upload test artifacts
        if: ${{ !cancelled() }}
        uses: actions/upload-artifact@v4
        with:
          name: playwright-report
          path: playwright-report/
          retention-days: 30
```

> **Note:** Yarn berry (v2+) resolves local bin entries via `yarn <bin>`, so `yarn stably` works directly.
> This does NOT work in yarn classic — see the classic template below.

### Template: yarn classic (v1)

```yaml
name: Stably E2E Tests
on:
  push:
    branches: [main, master]
  pull_request:
    branches: [main, master]

jobs:
  e2e-tests:
    runs-on: ubuntu-latest
    timeout-minutes: 30
    env:
      STABLY_API_KEY: ${{ secrets.STABLY_API_KEY }}
      STABLY_PROJECT_ID: ${{ secrets.STABLY_PROJECT_ID }}
    steps:
      - uses: actions/checkout@v4

      - uses: actions/setup-node@v4
        with:
          node-version: '20'
          cache: 'yarn'

      - name: Install dependencies
        run: yarn install --frozen-lockfile

      - name: Install browsers
        run: npx stably install --with-deps chromium

      - name: Run tests
        run: npx stably test

      - name: Upload test artifacts
        if: ${{ !cancelled() }}
        uses: actions/upload-artifact@v4
        with:
          name: playwright-report
          path: playwright-report/
          retention-days: 30
```

> **Note:** Yarn classic (v1) is pre-installed on GitHub runners — no `corepack enable` needed.
> Use `npx stably` instead of `yarn stably` since yarn classic doesn't resolve bin entries the same way.

### Monorepo Adaptation

If the project is a monorepo, add `defaults.run.working-directory` at the job level:

```yaml
jobs:
  e2e-tests:
    runs-on: ubuntu-latest
    defaults:
      run:
        working-directory: ./packages/app  # adjust to detected path
    # ... rest of job
```

And adjust the `cache-dependency-path` so `setup-node` finds the lockfile:
```yaml
# pnpm
cache-dependency-path: 'packages/app/pnpm-lock.yaml'
# npm
cache-dependency-path: 'packages/app/package-lock.json'
# yarn
cache-dependency-path: 'packages/app/yarn.lock'
```

> **Note:** `defaults.run.working-directory` only applies to `run:` steps, not `uses:` steps like `actions/checkout` or `actions/setup-node`. This is correct — you want to checkout at the repo root.

### Self-Healing Addition

If the user chose self-healing (option 2), add `permissions` at the job level and append these steps after the test step:

```yaml
jobs:
  e2e-tests:
    runs-on: ubuntu-latest
    permissions:
      contents: write
      pull-requests: write
    # ... env, steps as before, then:

      - name: Run tests
        id: test
        continue-on-error: true
        run: npx stably test

      - name: Auto-fix failures
        if: steps.test.outcome == 'failure'
        continue-on-error: true
        run: npx stably fix

      - name: Create PR with fixes
        if: steps.test.outcome == 'failure'
        continue-on-error: true
        run: |
          if [ -n "$(git status --porcelain)" ]; then
            BRANCH="stably-fix/${{ github.run_id }}"
            git config user.name "github-actions[bot]"
            git config user.email "github-actions[bot]@users.noreply.github.com"
            git checkout -b "$BRANCH"
            git add -A
            git commit -m "fix: auto-repair failing tests"
            git push origin "$BRANCH"
            gh pr create \
              --title "fix: auto-repair failing tests" \
              --body "Automated PR from Stably Fix after test failures in run #${{ github.run_number }}." \
              --base "${{ github.event.pull_request.base.ref || github.ref_name }}" \
              --head "$BRANCH"
          fi
        env:
          GH_TOKEN: ${{ secrets.GITHUB_TOKEN }}

      - name: Upload test artifacts
        if: ${{ !cancelled() }}
        uses: actions/upload-artifact@v4
        with:
          name: playwright-report
          path: playwright-report/
          retention-days: 30

      - name: Fail if tests failed
        if: steps.test.outcome == 'failure'
        run: |
          echo "::error::Tests failed. A fix PR may have been created."
          exit 1
```

**Important:**
- The `permissions` block is required — without `contents: write` and `pull-requests: write`, the push and PR creation will fail with 403 errors.
- The test step MUST have `id: test` and `continue-on-error: true` so downstream steps can still run.
- The `--base` uses `github.event.pull_request.base.ref` on PR events (where `github.ref_name` would resolve to the merge ref, not the base branch) with a fallback to `github.ref_name` for push events.
- Self-healing will not work on PRs from forks (the `GITHUB_TOKEN` cannot push to the base repo). Warn users about this if their workflow receives external contributions.
- `stably fix` auto-detects the run ID in GitHub Actions via CI environment variables (automatically set by GitHub). No explicit run ID argument is needed.
- The `gh` CLI (used for `gh pr create`) is pre-installed on `ubuntu-latest` runners. If using a custom runner image, ensure `gh` is available.
- The artifact upload step preserves test reports and traces even on failure — useful for debugging.

### Scheduled Addition

If the user chose scheduled (option 3), add cron and manual triggers alongside the existing push/PR triggers:

```yaml
on:
  push:
    branches: [main, master]
  pull_request:
    branches: [main, master]
  schedule:
    - cron: '0 6 * * *'  # Daily at 6 AM UTC
  workflow_dispatch:  # Allow manual triggers from the Actions tab
```

If the user wants schedule-only (no push/PR), remove the `push` and `pull_request` triggers.

---

## Step 4: Write Workflow File and Guide Secrets Setup

**Show the generated workflow to the user and ask:**
```
Step 4 of 5: Write workflow file

I'll create the following file:
  .github/workflows/stably-e2e.yml

[show full YAML]

May I write this file?
```

**WAIT for confirmation before writing.**

After writing, guide secrets setup:
```
Now you need to add your Stably credentials as GitHub repository secrets:

1. Go to your repo on GitHub -> Settings -> Secrets and variables -> Actions
2. Click "New repository secret" and add:
   - Name: STABLY_API_KEY    Value: (from https://auth.stably.ai/org/api_keys/)
   - Name: STABLY_PROJECT_ID Value: (from your Stably dashboard)

Without these secrets, the workflow will fail with authentication errors.
```

If the user chose self-healing, also mention:
```
The GITHUB_TOKEN secret is automatically provided by GitHub Actions -
no additional setup needed for the auto-fix PR step.
```

---

## Step 5: Verify and Summarize

**Announce:**
```
Step 5 of 5: Verification

Checking the generated workflow...
```

**Verify:**
1. The YAML file exists and is syntactically valid
2. The workflow references the correct paths for the detected project structure
3. All required env vars are referenced

**Final summary:**
```
GitHub Actions setup complete!

Created: .github/workflows/stably-e2e.yml

Your workflow will:
- Trigger on [push/PR to main | schedule | etc.]
- Install [npm/pnpm/yarn] dependencies with caching
- Install Playwright browsers
- Run Stably tests
[- Auto-fix failures and create PRs (if self-healing)]
[- Run on schedule: daily at 6 AM UTC (if scheduled)]

Next steps:
1. Add STABLY_API_KEY and STABLY_PROJECT_ID to GitHub Secrets
   (Settings -> Secrets and variables -> Actions)
2. Push this workflow to your repository
3. Watch the run in the Actions tab

Troubleshooting:
- "stably: command not found" -> ensure `stably` is in package.json devDependencies (this is the CLI package; `@stablyai/playwright-test` is the SDK)
- "browser not found" -> the 'Install browsers' step should handle this
- Auth errors -> verify your GitHub Secrets are set correctly
- pnpm not found -> ensure corepack enable runs before setup-node

Happy testing!
```

---

## Important Guidelines

- **Detect, don't assume** - Always check the project for package manager, config location, and structure
- **Use `npx stably` / `pnpm exec stably` / `yarn stably` (berry) / `npx stably` (yarn classic)** - Never use bare `stably` command in CI
- **Job-level env vars** - Put `STABLY_API_KEY` and `STABLY_PROJECT_ID` at the job level to avoid repetition
- **corepack before setup-node** - For pnpm and yarn berry (v2+), `corepack enable` MUST come before `actions/setup-node` because setup-node needs the package manager binary to set up caching. Yarn classic (v1) does not need corepack.
- **`--frozen-lockfile` / `--immutable`** - Always use lockfile-strict install in CI to prevent drift
- **Browser install is separate** - `npm ci` does NOT install browser binaries; always add the explicit install step
- **Monorepo awareness** - If playwright config is in a subdirectory, set `working-directory` on the job
- **Never hardcode secrets** - Always use `${{ secrets.* }}` references
