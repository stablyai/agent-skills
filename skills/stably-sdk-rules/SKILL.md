---
name: stably-sdk-rules
description: |
  AI rules for writing tests with Stably Playwright SDK.
  Use this skill when writing or modifying Playwright tests with Stably AI
  features. Covers when to use Playwright vs Stably methods, plus minimal
  patterns for aiAssert, extract, getLocatorsByAI, agent.act, Inbox, and
  Google auth.
license: MIT
metadata:
  author: stably
  version: '1.1.0'
---

# Stably SDK Rules

## Quick Rules

1. Prefer raw Playwright for deterministic actions/assertions (faster + cheaper).
2. Prioritize reliability over cost when Playwright becomes brittle.
3. Use `agent.act()` for canvas, coordinate-based drag/click, or unstable multi-step flows.
4. Use `expect(...).aiAssert()` for dynamic visual assertions; keep prompts specific.
5. Use `page.extract()` / `locator.extract()` when you need visual-to-data extraction.
6. Use `page.getLocatorsByAI()` when semantic selectors are hard with standard locators.
7. Use `Inbox` for OTP/magic-link/verification email flows.
8. All prompts must be self-contained; never rely on implicit previous context.
9. Keep `agent.act()` tasks small; do loops/calculations/conditionals in code.
10. Use `fullPage: true` only if content outside viewport matters.
11. Always add `.describe("...")` to locators for trace readability.
12. For email isolation, use unique `Inbox.build({ suffix })` per test and clean up.

## Setup (Single Block)

```bash
npm install -D @playwright/test @stablyai/playwright-test @stablyai/email
export STABLY_API_KEY=YOUR_KEY
export STABLY_PROJECT_ID=YOUR_PROJECT_ID
```

```ts
import { test, expect } from "@stablyai/playwright-test";
import { Inbox } from "@stablyai/email";
```

Optional: set API key programmatically.

```ts
import { setApiKey } from "@stablyai/playwright-test";
setApiKey("YOUR_KEY");
```

## Core Rules

- Locator rule: every locator interaction should use `.describe("...")`.
- Assertion choice:
  - Use Playwright assertions first.
  - Use `aiAssert` for dynamic/visual-heavy checks.
- Interaction choice:
  - Use Playwright for deterministic steps.
  - Use `agent.act` for brittle or semantic tasks (especially canvas/coordinates).
- Prompt quality:
  - Include explicit target, intent, and constraints.
  - Pass cross-step data through variables, not vague references.

## Minimal Usage Patterns

### `aiAssert`

```ts
await expect(page).aiAssert("Shows revenue trend chart and spotlight card");
await expect(page.locator(".header").describe("Header")).aiAssert("Has nav, avatar, and bell icon");
```

Use `fullPage: true` only when assertion needs off-screen content.

### `extract`

```ts
const orderId = await page.extract("Extract the order ID from the first row");
```

With schema:

```ts
import { z } from "zod";
const Schema = z.object({ revenue: z.string(), users: z.number() });
const metrics = await page.extract("Get revenue and active users", { schema: Schema });
```

### `getLocatorsByAI`

Requires Playwright `>= 1.54.1`.

```ts
const { locator, count } = await page.getLocatorsByAI("the login button");
expect(count).toBe(1);
await locator.describe("Login button located by AI").click();
```

### `agent.act`

```ts
await agent.act("Find the first pending order and mark it as shipped", { page });
```

Good pattern: compute values in code, then pass concrete values into the prompt.

### `Inbox` (Email Isolation)

Install: `npm install -D @stablyai/email`. Requires `STABLY_API_KEY` and `STABLY_PROJECT_ID` env vars (or pass to `Inbox.build()`).

```ts
const inbox = await Inbox.build({ suffix: `test-${Date.now()}` });
// inbox.address → "my-org+test-1706621234567@mail.stably.ai"

await page.getByLabel("Email").describe("Email input").fill(inbox.address);

const email = await inbox.waitForEmail({ subject: "verification", timeoutMs: 60_000 });
const { data: otp } = await inbox.extractFromEmail({
  id: email.id,
  prompt: "Extract the 6-digit OTP code",
});

await inbox.deleteAllEmails();
```

#### Inbox.build Options

| Option | Type | Description |
|--------|------|-------------|
| `suffix` | string | Suffix for test isolation (e.g., `"test-123"` → `"org+test-123@mail.stably.ai"`) |
| `apiKey` | string | Defaults to `STABLY_API_KEY` env var |
| `projectId` | string | Defaults to `STABLY_PROJECT_ID` env var |

Always use a unique `suffix` per test for parallel isolation. The inbox automatically filters out emails received before it was created.

#### waitForEmail

```ts
const email = await inbox.waitForEmail({
  from: "noreply@example.com",    // filter by sender
  subject: "verification",         // contains match by default
  subjectMatch: "exact",           // or "contains" (default)
  timeoutMs: 60_000,               // default: 120000 (2 min)
  pollIntervalMs: 5000,            // default: 3000 (3 sec)
});
```

Throws `EmailTimeoutError` if no match arrives within the timeout.

#### extractFromEmail

Returns `{ data, reason }`. Throws `EmailExtractionError` on failure.

```ts
// String extraction
const { data: otp } = await inbox.extractFromEmail({
  id: email.id,
  prompt: "Extract the 6-digit OTP code",
});

// Structured extraction with Zod schema
import { z } from "zod";
const { data } = await inbox.extractFromEmail({
  id: email.id,
  prompt: "Extract the verification URL and expiration time",
  schema: z.object({ url: z.string().url(), expiresIn: z.string() }),
});
```

#### Inbox Properties

| Property | Type | Description |
|----------|------|-------------|
| `address` | string | Full email address (with suffix if provided) |
| `suffix` | string \| undefined | The suffix passed to `Inbox.build()` |
| `createdAt` | Date | Inbox creation time; emails before this are auto-filtered |

#### listEmails

```ts
const { emails, nextCursor } = await inbox.listEmails(options?);
```

| Option | Type | Default | Description |
|--------|------|---------|-------------|
| `from` | string | — | Filter by sender address |
| `subject` | string | — | Filter by subject |
| `subjectMatch` | `'contains'` \| `'exact'` | `'contains'` | Subject matching mode |
| `limit` | number | 20 | Max results (max: 100) |
| `cursor` | string | — | Pagination cursor from previous `nextCursor` |
| `since` | Date | — | Override the default creation-time filter |
| `includeOlder` | boolean | false | Include emails received before inbox creation |

#### Other Methods

```ts
const email = await inbox.getEmail(id);                          // get by ID
await inbox.deleteEmail(email.id);                               // delete single
await inbox.deleteAllEmails();                                   // delete all (this inbox only)
```

#### Email Object Properties

| Property | Type | Description |
|----------|------|-------------|
| `id` | string | Unique identifier |
| `mailbox` | string | Container (e.g., `"INBOX"`) |
| `from` | `{ address: string, name?: string }` | Sender |
| `to` | `{ address: string, name?: string }[]` | Recipients |
| `subject` | string | Subject line |
| `receivedAt` | Date | Arrival timestamp |
| `text` | string? | Plain text body |
| `html` | string[]? | HTML body parts |

#### Playwright Fixture Pattern

```ts
import { test as base } from "@stablyai/playwright-test";
import { Inbox } from "@stablyai/email";

const test = base.extend<{ inbox: Inbox }>({
  inbox: async ({}, use, testInfo) => {
    const inbox = await Inbox.build({ suffix: `test-${testInfo.testId}` });
    await use(inbox);
    await inbox.deleteAllEmails();
  },
});

test("signup flow", async ({ page, inbox }) => {
  await page.fill("#email", inbox.address);
  await page.click("#signup");
  const email = await inbox.waitForEmail({ subject: "Welcome" });
  // ...
});
```

### Finding Your Organization's Email Address

Your email address is visible in the Stably dashboard:
- **Settings > Email Inbox**: Displays the full address with a copy button
- **In code**: `inbox.address` after calling `Inbox.build()` returns your full address

The pattern is `{org-name}@mail.stably.ai`. If the user needs to allowlist, they should add `mail.stably.ai` to their email provider's allowlist.

Direct users to the dashboard Settings > Email Inbox to find their specific address.

## Auth Flows (Google)

Use the helper instead of custom popup scripting:

```ts
import { authWithGoogle } from "@stablyai/playwright-test/auth";

await authWithGoogle({
  context,
  email: process.env.GOOGLE_AUTH_EMAIL!,
  password: process.env.GOOGLE_AUTH_PASSWORD!,
  otpSecret: process.env.GOOGLE_AUTH_OTP_SECRET!,
});
```

Required env vars:
- `GOOGLE_AUTH_EMAIL`
- `GOOGLE_AUTH_PASSWORD`
- `GOOGLE_AUTH_OTP_SECRET`

Use a dedicated test Google account only.

## Troubleshooting (Short)

- `aiAssert` is slow/flaky: scope to a locator, tighten prompt, avoid unnecessary `fullPage: true`.
- `agent.act` fails: split into smaller tasks, pass explicit constraints, raise `maxCycles` only when needed.
- Email timeout: verify subject/from filter and use unique inbox suffixes.

## Full References

- Stably docs: https://docs.stably.ai
- SDK setup skill: `skills/stably-sdk-setup/SKILL.md`
- Package docs: https://www.npmjs.com/package/@stablyai/playwright-test
- Email package docs: https://www.npmjs.com/package/@stablyai/email
- Local overview: `skills/stably-sdk-rules/README.md`
