---
name: stably-plan
description: |
  Discover test coverage gaps and generate test.fixme() plan files with `stably plan`.
  Use this skill when planning test coverage, finding untested flows, generating test
  skeletons, or creating a test automation roadmap. Triggers on: "stably plan",
  "plan tests", "coverage gaps", "what needs testing", "test plan", "test.fixme",
  "discover untested flows", "coverage map", "test roadmap", "what should we test".
license: MIT
metadata:
  author: stably
  version: '1.0.0'
---

# Plan Test Coverage with stably plan

`stably plan` autonomously explores your codebase, identifies critical test coverage gaps, and generates `test.fixme()` skeleton files directly in your repository. These skeletons serve as the test plan — branch-aware, reviewable in PRs, and upgradeable to real tests via `stably create`.

## Pre-flight

**Always run `stably --version` first.** If not found, install with `npm install -g stably` or use `npx stably`. Requires Node.js 20+, Playwright, and a [Stably account](https://app.stably.ai).

```bash
stably --version
echo "STABLY_API_KEY: ${STABLY_API_KEY:+set}"
echo "STABLY_PROJECT_ID: ${STABLY_PROJECT_ID:+set}"
```

## Usage

```bash
# Full autonomous discovery — analyze entire app
stably plan

# Scoped to specific areas
stably plan "focus on the checkout and team management flows"

# Express constraints and exclusions
stably plan "plan tests for the new billing page, ignore admin settings"

# In CI: plan tests for changed areas
stably plan "plan tests for the features changed in this PR"
```

### With and Without a Prompt

| | No prompt | With prompt |
|---|---|---|
| **Scope** | Full app — all reachable files and existing tests | Filtered to what the prompt describes |
| **Discovery** | Agent-driven codebase exploration | Same signals, prompt acts as relevance filter |
| **Priority** | System-generated across all flows | System-generated within prompt scope |
| **Output** | `test.fixme()` files for all discovered groups | Only groups matching the prompt scope |

The prompt is the highest-priority input — it overrides generic heuristics for scope and focus.

## Output: `test.fixme()` Skeletons

`stably plan` writes standard Playwright spec files with `test.fixme()` blocks as the sole plan artifact. No separate JSON or config files.

### Two-Level Structure

- **Flow** (`test.describe`): a business-level user journey (e.g., "Checkout"). Priority and business criticality live here.
- **Scenario** (`test.fixme`): a specific path through a flow — happy path, error, edge case, or authorization boundary.

### Example Output

```ts
import { test } from '@playwright/test';

// Rationale: Revenue path; critical business flow; no existing E2E coverage
// Roles: admin, guest
// Environment: staging
// Data Requirements: seeded products
test.describe('Checkout', { tag: '@checkout' }, () => {

  // Variant: happy_path
  // Role: guest
  // Preconditions: Seeded product; test payment card
  test.fixme('Guest user completes purchase with valid card', { tag: ['@p0'] }, async () => {});

  // Variant: error
  // Role: guest
  // Preconditions: Seeded product; declined test card
  test.fixme('Checkout fails gracefully with declined card', { tag: ['@p0'] }, async () => {});

  // Variant: authorization
  // Role: none
  // Preconditions: No session; items in cart via direct URL
  test.fixme('Unauthenticated user redirected to login at checkout', { tag: ['@p0'] }, async () => {});
});
```

### File Layout

Files are placed in the repository's existing test directory (discovered from `playwright.config.ts` `testDir` and existing test locations), one file per group:

```
<repo-test-dir>/
  checkout.spec.ts
  auth.spec.ts
  team-management.spec.ts
```

**Existing test files with real `test()` implementations are never modified.** The agent reports existing coverage in the CLI summary and only generates new spec files for uncovered groups.

### Grouping Strategy

The agent determines grouping by:

1. Following existing test organization conventions in the repo
2. Falling back to feature-area grouping (e.g., "checkout", "auth") if no convention exists
3. Presenting the chosen strategy and suggesting alternatives after generation

## Priority Model

Each scenario gets one priority tag: `@p0`, `@p1`, `@p2`, or `@p3`.

| Priority | Criteria |
|----------|----------|
| `@p0` | Revenue, login, signup, checkout, core workflows with no reliable E2E coverage |
| `@p1` | Common non-revenue flows used frequently by active users |
| `@p2` | Secondary workflows, settings, or role-specific flows |
| `@p3` | Rare paths, edge cases, or deferred flows |

Run specific priorities with standard Playwright grep: `npx playwright test --grep "@p0"`.

## Lifecycle: plan, create, fix

`test.fixme()` skeletons upgrade in place to real tests:

```
stably plan  -->  team reviews  -->  stably create  -->  stably test  -->  stably fix
(test.fixme)      (adjust tags)      (test.fixme -> test)   (run)         (heal)
```

1. `stably plan` writes `test.fixme()` skeletons
2. Team reviews in PR, adjusts priorities if needed
3. `stably create` implements chosen fixmes into full `test()` blocks in place
4. `git diff` shows the `test.fixme` to `test` transition — fully reviewable
5. Coverage status is derived by scanning: `test.fixme()` = planned, `test()` = implemented

### Using `stably create` with Plan Files

```bash
# Implement specific files
stably create "implement checkout.spec.ts and auth.spec.ts"

# Implement all P0 tests
stably create "implement all @p0 tests"

# Implement a single flow
stably create "implement the guest checkout flow"
```

## Idempotency

`stably plan` is safe to re-run:

- Scans existing `test()` and `test.fixme()` blocks before generating
- Uses semantic understanding to avoid duplicates (matches on intent, not just names)
- Does not modify files containing real `test()` implementations
- Best-effort preservation of human edits to existing `test.fixme()` files

## Scale

A single invocation generates at most **200 flows**. For larger codebases, use the prompt to scope discovery:

```bash
stably plan "focus on the payments module"
stably plan "cover the admin settings area"
```

## STABLY.md Updates

During exploration, the agent writes key project discoveries to `STABLY.md` — auth patterns, environment topology, framework conventions, and data seeding approaches. This gives future `stably plan`, `stably create`, and `stably fix` runs accumulated context.

## Framework Detection

The agent detects the app's framework from `package.json` and project structure:

| Framework | Discovery strategy |
|---|---|
| Next.js App Router | `app/**/page.tsx`, route groups |
| Next.js Pages Router | `pages/**/*.tsx` |
| React Router / Remix | `<Route>`, `createBrowserRouter` patterns |
| Vue / Nuxt | `pages/`, `router/` directories |
| SvelteKit | `src/routes/` |
| API routes | `api/` directories, Express/Fastify definitions |

## Validation

After writing plan files, the agent validates:

- No duplicate scenarios across files
- Every `@p0` flow has at least one `error` or `edge_case` variant
- All files are valid TypeScript with correct structure
- Every scenario has Variant, Role, and Preconditions comments
- Scope adherence if a user prompt was provided

## Long-Running Commands (AI Agents)

`stably plan` is AI-powered and can take **several minutes** depending on codebase size.

| Agent | Configuration |
|-------|--------------|
| **Claude Code** | `timeout: 600000` on Bash tool |
| **Cursor** | `block_until_ms: 900000` |

## Links

[Docs](https://docs.stably.ai) · [CLI Quickstart](https://docs.stably.ai/stably-cli/quickstart) · [Dashboard](https://app.stably.ai) · [API Keys](https://auth.stably.ai/org/api_keys/)
