---
name: stably-sdk-rules
description: AI rules and complete reference for writing tests with Stably Playwright SDK. Covers when to use Playwright vs Stably methods, plus detailed patterns for aiAssert, extract, getLocatorsByAI, agent.act, Inbox, Google auth, and model selection.
license: MIT
metadata:
  author: stably
  version: '2.0.0'
---

# Stably SDK Rules & Reference

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

## Setup

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

**Note:** `toMatchScreenshotPrompt` is deprecated — use `aiAssert` instead. Do NOT use Playwright's `toHaveScreenshot()` — it is not supported. Use `aiAssert` for ALL visual assertions.

## AI Assertions (aiAssert)

Use `aiAssert` for intent-based visual verification of dynamic UIs:

```ts
// Page-level assertion
await expect(page).aiAssert(
  "Shows revenue trend chart and spotlight card",
  { timeout: 30_000 }
);

// Scoped to specific element (preferred for precision)
await expect(page.locator(".header"))
  .aiAssert("Nav with avatar and bell icon");
```

**Signature:** `expect(page|locator).aiAssert(prompt: string, options?: { timeout?: number, fullPage?: boolean, model?: AIModel })`

**Best Practices:**
- Use for **dynamic** UIs where deterministic assertions are insufficient
- Keep prompts specific with labels and units
- Scope with locators when possible (more precise, less noisy)
- **Consider `fullPage: true` carefully**: Only use when assertion requires content beyond the visible viewport. Viewport captures are faster and cheaper.

## AI Extraction (extract)

Extract data from visual content:

```ts
// Simple string extraction
const txt = await page.extract("List revenue, active users, and churn rate");

// Typed extraction with Zod schema
import { z } from "zod";
const Metrics = z.object({
  revenue: z.string(),
  activeUsers: z.number(),
  churnRate: z.number()
});
const m = await page.extract(
  "Return revenue (currency), active users, churn %",
  { schema: Metrics }
);
```

Before using `schema` with Zod, check the test project's `package.json` and install `zod` as a dev dependency if missing. If it already exists as a regular dependency, leave it as-is.

**Signatures:**
- `page.extract(prompt: string, options?: { model?: AIModel }): Promise<string>`
- `page.extract<T extends z.AnyZodObject>(prompt, { schema: T, model?: AIModel }): Promise<z.output<T>>`

## AI Locator Finding (getLocatorsByAI)

Find elements using natural language based on accessibility properties:

```ts
// Find a single element
const { locator: loginBtn, count } = await page.getLocatorsByAI("the login button");
expect(count).toBe(1);
await loginBtn.click();

// Find multiple elements
const { locator: productCards, count: cardCount } = await page.getLocatorsByAI(
  "all product cards in the grid"
);
await expect(productCards).toHaveCount(cardCount);
```

**Signature:** `page.getLocatorsByAI(prompt: string, options?: { model?: AIModel }): Promise<{ locator: Locator, count: number, reason: string }>`

**Properties:**
- Returns a Playwright `Locator` usable for interactions and assertions
- `count` indicates how many elements were found (0 if none)
- `reason` contains the AI's explanation of what it found
- Requires Playwright v1.54.1 or higher

**Best Practices:**
- Describe elements by accessible properties (labels, roles, text) rather than visual attributes (colors, positioning)
- Best for finding elements when CSS selectors or test IDs are unreliable

## AI Agent (agent.act)

Use the `agent` fixture for complex, autonomous workflows:

```ts
test("complex workflow", async ({ agent, page }) => {
  await page.goto("/orders");
  await agent.act("Find the first pending order and mark it as shipped", { page });
});

// Or create manually
const agent = context.newAgent();
await agent.act("Your task here", { page, maxCycles: 10 });
```

**Signature:** `agent.act(prompt: string, options: { page: Page, maxCycles?: number, model?: string }): Promise<{ success: boolean }>`

**Default maxCycles:** 30
**Supported models:** `anthropic/claude-sonnet-4-6` (default), `google/gemini-2.5-computer-use-preview-10-2025`

### Passing Variables to Prompts

Use template literals to pass variables:

```ts
const duration = 24 * 7 * 60;
await agent.act(`Enter the duration of ${duration} seconds`, { page });

const username = "john.doe@example.com";
await agent.act(`Login with username ${username}`, { page });
```

### Self-Contained Prompts (CRITICAL)

All prompts to Stably SDK AI methods must be self-contained with all necessary information:

1. **No implicit references to outside context:**
   - Bad: `agent.act("Verify the field you just filled in the form is 4", { page })`
   - Good: `agent.act("Verify the 'timeout' field in the form has value 4", { page })`
   - Bad: `agent.act("Pick something that's not in the previous step", { page })`
   - Good: `const selectedItem = "Option A"; await agent.act(\`Pick an option other than ${selectedItem}\`, { page })`

2. **Pass information between AI methods using explicit variables:**
   ```ts
   const orderId = await page.extract("Get the order ID from the first row");
   await agent.act(`Cancel order with ID ${orderId}`, { page });
   ```

3. **Include detailed instructions and domain knowledge:**
   - Bad: `agent.act("Fill in the form", { page })`
   - Good: `agent.act("Fill in the form with test data. On page 4 you might run into a popup asking for premium features - just click 'Skip' or 'Cancel' to ignore it", { page })`

### Offload Work to Playwright

The less actions/cycles agent.act() needs, the better it performs. Offload work to Playwright code:

1. **Repetition:** Use loops in code, not in prompts
   - Bad: "Click the button 5 times"
   - Good: "Click the button" (in a loop that runs 5 times)

2. **Calculations:** Calculate in code, pass result to prompt
   - Bad: "enter the duration of 24*7*60 seconds"
   - Good: `const sum = 24*7*60; agent.act(\`enter the duration of ${sum} seconds\`, { page })`

3. **Conditionals:** Use code for if/else when possible

### agent.act() Best Practices

1. **Split complex prompts into smaller tasks:**
   ```ts
   // Bad - too many steps
   await agent.act('Close popups, click menu, expand panel, find sliders', { page, maxCycles: 15 });

   // Good - single responsibility
   await agent.act('Close the tutorial popup if visible', { page, maxCycles: 5 });
   await agent.act('Click the color menu and expand Basic Color panel', { page, maxCycles: 8 });
   ```

2. **Be specific in prompts** - Include visual hints:
   ```ts
   // Bad
   await agent.act('Adjust the slider', { page });

   // Good
   await agent.act('Move the "Brightness" slider (the one with the sun icon) to approximately 75%', { page });
   ```

3. **Use appropriate maxCycles:**
   - Simple single action: 3-5 cycles
   - Multi-step interaction: 8-10 cycles
   - Complex workflow: 15-20 cycles
   - Never rely on the default 30 cycles

## Model Selection

`aiAssert`, `extract`, and `getLocatorsByAI` support an optional `model` parameter:

```ts
await expect(page).aiAssert("Shows the dashboard", { model: "google/gemini-3-flash-preview" });
const data = await page.extract("Get the price", { model: "openai/o4-mini" });
const { locator } = await page.getLocatorsByAI("the submit button", { model: "google/gemini-3-pro-preview" });
```

**Available models:**
- `"openai/o4-mini"` - OpenAI's efficient reasoning model
- `"google/gemini-3-pro-preview"` - Google's most capable model
- `"google/gemini-3-flash-preview"` - Google's fast, efficient model (good for simple tasks)

**Tips:**
- Use `gemini-3-flash-preview` for fast, simple operations
- Use `gemini-3-pro-preview` or `o4-mini` for complex reasoning

## Email Inbox Testing (@stablyai/email)

Use disposable email inboxes for testing email-dependent flows:

```ts
import { Inbox } from "@stablyai/email";

// Create a test-scoped inbox
const inbox = await Inbox.build({ suffix: `test-${Date.now()}` });
// inbox.address → "org+test-1706621234567@mail.stably.ai"

// Wait for an email
const email = await inbox.waitForEmail({
  from: "noreply@example.com",
  subject: "verification",
  timeoutMs: 60_000,
});

// AI-powered extraction
const { data: otp } = await inbox.extractFromEmail({
  id: email.id,
  prompt: "Extract the 6-digit OTP code",
});

// Structured extraction with Zod
import { z } from "zod";
const { data } = await inbox.extractFromEmail({
  id: email.id,
  prompt: "Extract verification URL and expiration",
  schema: z.object({ url: z.string().url(), expiresIn: z.string() }),
});
```

### Playwright Fixture Pattern

For test isolation and automatic cleanup:

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
```

### Key Options

**waitForEmail:** `from`, `subject`, `subjectMatch` (`'contains'` | `'exact'`), `timeoutMs` (default 120000), `pollIntervalMs` (default 3000)

**extractFromEmail:** `id` (required), `prompt` (required), `schema` (optional Zod). Returns `{ data, reason }`

**Best practices:**
- Always use unique suffixes for parallel test isolation
- Use the fixture pattern for automatic cleanup
- Prefer `waitForEmail` over polling with `listEmails`

## Google Auth (authWithGoogle)

Use the built-in helper instead of custom popup scripting:

```ts
// Recommended: context method (no need to pass context)
await context.authWithGoogle({
  email: process.env.GOOGLE_TEST_EMAIL!,
  password: process.env.GOOGLE_TEST_PASSWORD!,
  otpSecret: process.env.GOOGLE_TEST_OTP_SECRET!,
});

// Alternative: standalone import
import { authWithGoogle } from "@stablyai/playwright-test";
await authWithGoogle({ context, email, password, otpSecret });
```

**Signature:** `authWithGoogle(options: { context: BrowserContext, email: string, password: string, otpSecret: string, forceRefresh?: boolean, apiKey?: string }): Promise<void>`

**Required env vars:** `GOOGLE_TEST_EMAIL`, `GOOGLE_TEST_PASSWORD`, `GOOGLE_TEST_OTP_SECRET`

Use a dedicated test Google account only.

For step-by-step Google account setup (2FA, OTP secret, environment variables), invoke the `google-auth-account-setup` skill.

## When to Use SDK vs Playwright

**Use Playwright** when:
- Simple, concrete checks (element visible, text matches, URL correct)
- Faster and more reliable for deterministic scenarios

**Use Stably SDK** when:
1. Test accuracy and stability are paramount
2. Interactions are hard to express in Playwright or too brittle
3. Canvas-related operations or drag/click requiring coordinates → use `agent.act()`
4. Visual-heavy assertions → use `aiAssert`
5. Email verification flows → use `@stablyai/email`

## Minimal Template

```ts
import { test, expect } from "@stablyai/playwright-test";

test("AI-enhanced dashboard", async ({ page, agent }) => {
  await page.goto("/dashboard");

  // Use agent for complex workflows
  await agent.act("Navigate to settings and enable notifications", { page });

  // Use AI assertions for dynamic content
  await expect(page).aiAssert(
    "Dashboard shows revenue chart (>= 6 months) and account spotlight card"
  );
});
```

## Troubleshooting

- **Slow assertions** → Scope visuals with locators; reduce viewport
- **Agent stops early** → Increase `maxCycles` or break task into smaller steps
- **Email timeout** → Verify subject/from filter and use unique inbox suffixes

## Full References

- Stably docs: https://docs.stably.ai
- SDK setup skill: `skills/stably-sdk-setup/SKILL.md`
- Package docs: https://www.npmjs.com/package/@stablyai/playwright-test
- Email package docs: https://www.npmjs.com/package/@stablyai/email
