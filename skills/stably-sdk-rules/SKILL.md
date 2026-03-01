---
name: stably-sdk-rules
description: |
  AI rules and SDK reference for writing tests with Stably Playwright SDK.
  Use this skill when writing or modifying Playwright tests with Stably AI
  features. Triggers on tasks involving aiAssert, agent.act(), page.extract(),
  page.getLocatorsByAI(), Inbox (email testing), or when deciding between
  Playwright vs Stably SDK methods. Includes best practices for AI assertions,
  extraction, locator finding, autonomous agents, and email inbox testing.
license: MIT
metadata:
  author: stably
  version: '1.0.0'
---

# Stably SDK — AI Rules (Playwright‑compatible)

**Assumption:** Full Playwright parity unless noted below.

## Import

```ts
import { test, expect } from "@stablyai/playwright-test";

// Additional exports available:
import {
  defineConfig,       // Enhanced defineConfig with stably project support
  stablyReporter,     // Reporter for CI/cloud integration
  setApiKey,          // Programmatic API key configuration
  getDirname,         // ESM __dirname equivalent
  getFilename,        // ESM __filename equivalent
} from "@stablyai/playwright-test";

// Type imports for model selection
import type { AIModel } from "@stablyai/playwright-test";

// Email inbox testing
import { Inbox } from "@stablyai/email";
```

## Install & Setup

```bash
# npm
npm install -D @playwright/test @stablyai/playwright-test

# pnpm
pnpm add -D @playwright/test @stablyai/playwright-test

# yarn
yarn add -D @playwright/test @stablyai/playwright-test

# bun
bun add -d @playwright/test @stablyai/playwright-test

export STABLY_API_KEY=YOUR_KEY
```

```ts
import { setApiKey } from "@stablyai/playwright-test";
setApiKey("YOUR_KEY");
```

## Environment Variables

| Variable | Description | Default |
|----------|-------------|---------|
| `STABLY_API_KEY` | API key for authentication | (required) |
| `STABLY_PROJECT_ID` | Project ID for reporter | (required for reporter) |
| `STABLY_API_URL` | Custom API endpoint | `https://api.stably.ai` |
| `STABLY_WS_URL` | Custom WebSocket endpoint | `wss://api.stably.ai/reporter` |

Both `STABLY_API_KEY` and `STABLY_PROJECT_ID` are shared across `@stablyai/playwright-test` and `@stablyai/email`.

## Model Selection

All AI methods support an optional `model` parameter to specify which AI model to use:

```ts
import type { AIModel } from "@stablyai/playwright-test";

// Available models:
// - "openai/o4-mini" - OpenAI's efficient reasoning model
// - "google/gemini-3-pro-preview" - Google's most capable model
// - "google/gemini-3-flash-preview" - Google's fast, efficient model
// - Custom model strings are also supported

// Example usage with different methods:
await expect(page).aiAssert("Shows dashboard", { model: "openai/o4-mini" });
const data = await page.extract("Get heading", { model: "google/gemini-3-flash-preview" });
const { locator } = await page.getLocatorsByAI("the login button", { model: "google/gemini-3-pro-preview" });
```

If no model is specified, the backend default is used.

## When to Use Stably SDK vs Playwright

**Prioritization:**
1. **Test accuracy and stability are the most important factors** - prioritize reliability over cost/speed.
2. **Otherwise, use Playwright whenever possible** since it's cheaper and faster.
3. **For interactions:** If the interaction will be hard to express as Playwright or will be too brittle that way (e.g., the scroll amount changes every time), then use `agent.act()`. **Any canvas-related operations, or any drag/click operations that require coordinates, must use `agent.act()`** (more semantic meaning, and less flaky).
4. **For assertions:** Use Playwright if it fulfills the purpose. But if the assertion is very visual-heavy, use Stably's `aiAssert`.
5. **For email verification flows:** Use `@stablyai/email` `Inbox` to receive and extract data from emails (OTP codes, magic links, confirmation emails).
6. **Use Stably SDK methods if it helps your tests pass** - when Playwright methods are insufficient or unreliable.

## AI Assertions (intent‑based visuals)

> **Note:** `toMatchScreenshotPrompt` is deprecated. Use `aiAssert` instead.

```ts
await expect(page).aiAssert(
  "Shows revenue trend chart and spotlight card",
  { timeout: 30_000 }
);
await expect(page.locator(".header"))
  .aiAssert("Nav with avatar and bell icon");
```

**Signature:** `expect(page|locator).aiAssert(prompt: string, options?: ScreenshotOptions & { model?: AIModel })`

* Use for **dynamic** UIs; keep prompts specific; scope with elements (using locators) when possible.
* **Consider whether you need `fullPage: true`**: Ask yourself if the assertion requires content beyond the visible viewport (e.g., long scrollable lists, full page layout checks). If only viewport content matters, omit `fullPage: true` — it's faster and cheaper. Use it only when you genuinely need to capture content outside the browser window's visible area.

## AI Extraction (visual → data)

```ts
// Extract from entire page
const txt = await page.extract("List revenue, active users, and churn rate");

// Extract from specific element (more focused, better results)
const headerText = await page.locator(".header").extract("Get the username displayed");
```

Typed with Zod:

```ts
import { z } from "zod";
const Metrics = z.object({ revenue: z.string(), activeUsers: z.number(), churnRate: z.number() });
const m = await page.extract("Return revenue (currency), active users, churn %", { schema: Metrics });

// Also works on locators
const UserSchema = z.object({ name: z.string(), role: z.string() });
const userData = await page.locator(".user-panel").extract("Get user info", { schema: UserSchema });
```

**Signatures:**

* `page.extract(prompt: string, options?: { model?: AIModel }): Promise<string>`
* `page.extract<T extends z.AnyZodObject>(prompt, { schema: T, model?: AIModel }): Promise<z.output<T>>`
* `locator.extract(prompt: string, options?: { model?: AIModel }): Promise<string>`
* `locator.extract<T extends z.AnyZodObject>(prompt, { schema: T, model?: AIModel }): Promise<z.output<T>>`

## AI Locator Finding (accessibility-based)

Use `getLocatorsByAI` to find elements using natural language based on the page's accessibility tree. Requires Playwright v1.54.1+.

```ts
// Find a single element
const { locator: loginBtn, count } = await page.getLocatorsByAI("the login button");
expect(count).toBe(1);
await loginBtn.click();

// Find multiple elements
const { locator: cards, count: cardCount } = await page.getLocatorsByAI("all product cards in the grid");
console.log(`Found ${cardCount} product cards`);
await expect(cards.first()).toBeVisible();

// With model selection
const { locator } = await page.getLocatorsByAI("the submit button", {
  model: "google/gemini-3-flash-preview"
});
```

**Signature:** `page.getLocatorsByAI(prompt: string, options?: { model?: AIModel }): Promise<{ locator: Locator, count: number, reason: string }>`

**Returns:**
* `locator` - Playwright Locator for the found elements (matches nothing if count is 0)
* `count` - Number of elements found
* `reason` - AI's explanation of what it found and why

**Best Practices:**
* Describe elements by their accessible properties (labels, roles, text) rather than visual attributes
* Use for elements that are hard to locate with traditional selectors
* Check the `count` to verify expected number of matches before interacting

## AI Agent (autonomous workflows)

### When Test Generation Uses `agent.act()` vs Raw Playwright

Stably's test generation agent intelligently chooses between raw Playwright code and `agent.act()`:

- **For repeatable, stable actions**: The test generation agent will try to turn these into raw Playwright code when first generating tests. Raw Playwright is faster and more cost-effective.
- **For dynamic or unstable pages**: If the agent finds that the page changes frequently and it can't find stable Playwright selectors/code for an action, it will use `agent.act()` instead.

This means you may see a mix of both in generated tests—this is intentional and optimizes for reliability while keeping costs down where possible.

### Using the Agent Fixture

Use the `agent` fixture to execute complex, human-like workflows:

```ts
test("complex workflow", async ({ agent, page }) => {
  await page.goto("/orders");
  await agent.act("Find the first pending order and mark it as shipped", { page });
});

// Or create manually
const agent = context.newAgent();
await agent.act("Your task here", { page, maxCycles: 10 }); // split into smaller steps if possible
```

**Signature:** `agent.act(prompt: string, options: { page: Page, maxCycles?: number, model?: Model }): Promise<void>`

* Throws `Error` if the agent fails to complete the task (check `error.message` for the AI's failure reason)
* Default maxCycles: 30
* Supported models: `"anthropic/claude-sonnet-4-5-20250929"` (default), `"google/gemini-2.5-computer-use-preview-10-2025"`, or any custom model string

### Passing Variables to Prompts

You can use template literals to pass variables into your prompts:

```ts
const duration = 24 * 7 * 60;
await agent.act(`Enter the duration of ${duration} seconds`, { page });

const username = "john.doe@example.com";
await agent.act(`Login with username ${username}`, { page });
```

### Self-Contained Prompts

All prompts to Stably SDK AI methods (agent.act, aiAssert, extract) must be self-contained with all necessary information:

1. **No implicit references to outside context** - prompts cannot reference previous actions or state that the AI method doesn't have access to:
   - ❌ Bad: `agent.act("Verify the field you just filled in the form is 4", { page })`
   - ✅ Good: `agent.act("Verify the 'timeout' field in the form has value 4", { page })`
   - ❌ Bad: `agent.act("Pick something that's not in the previous step", { page })`
   - ✅ Good: `const selectedItem = "Option A"; await agent.act(\`Pick an option other than ${selectedItem}\`, { page })`

2. **Pass information between AI methods using explicit variables:**
   ```ts
   // Extract data, then use it in next action
   const orderId = await page.extract("Get the order ID from the first row");
   await agent.act(`Cancel order with ID ${orderId}`, { page });
   ```

3. **Include detailed instructions and domain knowledge** to help the AI perform the task successfully:
   - ❌ Bad: `agent.act("Fill in the form", { page })`
   - ✅ Good: `agent.act("Fill in the form with test data. On page 4 you might run into a popup asking for premium features - just click 'Skip' or 'Cancel' to ignore it", { page })`

### Optimizing Agent Performance

**IMPORTANT:** The fewer actions/cycles agent.act() needs to do, the better it performs. Offload work to Playwright code when possible:

1. If your prompt has work that could be done by Playwright code, use Playwright for that work, and only use agent.act() for actions that are hard for Playwright (canvas operations, dynamic decision making, etc.)
2. If your prompt has repetition (e.g., do it 5 times), calculations (e.g., type 24*7*60 seconds), or other code-suitable tasks, use code for those, and only have agent.act() perform the agent-suitable part.
3. If your prompt has an if/else condition that can be expressed in code, use code for the condition, and only have agent.act() perform the agent-suitable part.

**Examples:**
- ❌ Bad: `"Click the button 5 times"`
- ✅ Good: `"Click the button"` (and include this in a loop that runs 5 times)
- ❌ Bad: `"enter the duration of 24*7*60 seconds"`
- ✅ Good: Calculate in code (`const sum = 24*7*60`), then use `\`enter the duration of ${sum} seconds\``

## Email Inbox (`@stablyai/email`)

The `@stablyai/email` package provides disposable email inboxes for testing email-dependent flows (OTP codes, verification links, order confirmations, etc.).

### Install

```bash
# npm
npm install -D @stablyai/email

# pnpm
pnpm add -D @stablyai/email

# yarn
yarn add -D @stablyai/email

# bun
bun add -d @stablyai/email
```

Requires `STABLY_API_KEY` and `STABLY_PROJECT_ID` environment variables (same as `@stablyai/playwright-test`).

### Creating an Inbox

```ts
import { Inbox } from "@stablyai/email";

const inbox = await Inbox.build({ suffix: `test-${Date.now()}` });
// Generates address like: "org+test-1706621234567@mail.stably.ai"

console.log(inbox.address);   // Full email address
console.log(inbox.suffix);    // Applied suffix
console.log(inbox.createdAt); // Inbox creation timestamp
```

**`Inbox.build()` Parameters:**

| Parameter | Type | Description |
|-----------|------|-------------|
| `suffix` | string (optional) | Test isolation identifier appended to address |
| `apiKey` | string (optional) | Overrides `STABLY_API_KEY` env var |
| `projectId` | string (optional) | Overrides `STABLY_PROJECT_ID` env var |

### Waiting for Emails

```ts
const email = await inbox.waitForEmail({
  from: "noreply@example.com",
  subject: "verification",
  timeoutMs: 60_000,
});
```

**`inbox.waitForEmail()` Parameters:**

| Parameter | Type | Default | Description |
|-----------|------|---------|-------------|
| `from` | string | — | Sender address filter |
| `subject` | string | — | Subject line filter |
| `subjectMatch` | `'contains'` \| `'exact'` | `'contains'` | Subject matching mode |
| `timeoutMs` | number | `120000` | Max wait duration (ms) |
| `pollIntervalMs` | number | `3000` | Poll frequency (ms) |

Returns an `Email` object or throws `EmailTimeoutError`. Only checks emails received **after** the inbox was created.

### Email Object Properties

| Property | Type | Description |
|----------|------|-------------|
| `id` | string | Unique identifier |
| `mailbox` | string | Container (e.g., `"INBOX"`) |
| `from` | `{ address, name }` | Sender info |
| `to` | array | Recipients |
| `subject` | string | Subject line |
| `receivedAt` | Date | Delivery timestamp |
| `text` | string? | Plain text body |
| `html` | array? | HTML body parts |

### AI Extraction from Emails

Use `inbox.extractFromEmail()` to extract structured data from emails using AI:

**String extraction:**
```ts
const { data: otp, reason } = await inbox.extractFromEmail({
  id: email.id,
  prompt: "Extract the 6-digit OTP code",
});
```

**Structured extraction with Zod:**
```ts
import { z } from "zod";

const { data } = await inbox.extractFromEmail({
  id: email.id,
  prompt: "Extract verification URL and expiration",
  schema: z.object({
    url: z.string().url(),
    expiresIn: z.string(),
  }),
});
```

**`inbox.extractFromEmail()` Parameters:**
- `id` (required): Email identifier
- `prompt` (required): Extraction instruction
- `schema` (optional): Zod schema for structured output

Returns `{ data, reason }` or throws `EmailExtractionError`.

### Listing & Retrieving Emails

```ts
// List emails with optional filters
const { emails, nextCursor } = await inbox.listEmails({
  from: "notifications@example.com",
  limit: 10,
});

// Get a specific email by ID
const email = await inbox.getEmail(emailId);
```

**`inbox.listEmails()` Parameters:**

| Parameter | Type | Default | Description |
|-----------|------|---------|-------------|
| `from` | string | — | Sender filter |
| `subject` | string | — | Subject filter |
| `subjectMatch` | `'contains'` \| `'exact'` | `'contains'` | Matching mode |
| `limit` | number | `20` (max 100) | Result count |
| `cursor` | string | — | Pagination cursor |
| `since` | Date | — | Override default time filter |
| `includeOlder` | boolean | `false` | Include pre-creation emails |

### Cleanup

```ts
await inbox.deleteEmail(email.id);  // Delete single email
await inbox.deleteAllEmails();       // Delete all emails in this inbox
```

Only deletes inbox-scoped emails, not the entire org mailbox.

### Playwright Fixture Pattern

Create a reusable `inbox` fixture for test isolation and automatic cleanup:

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

test("OTP login flow", async ({ page, inbox }) => {
  await page.goto("/login");
  await page.getByLabel("Email").describe("Email input").fill(inbox.address);
  await page.getByRole("button", { name: "Send OTP" }).describe("Send OTP button").click();

  const email = await inbox.waitForEmail({
    subject: "verification code",
    timeoutMs: 60_000,
  });

  const { data: otp } = await inbox.extractFromEmail({
    id: email.id,
    prompt: "Extract the 6-digit OTP code",
  });

  await page.getByLabel("OTP").describe("OTP input").fill(otp);
  await page.getByRole("button", { name: "Verify" }).describe("Verify button").click();
  await expect(page).toHaveURL("/dashboard");
});
```

### Email Testing Best Practices

* **Always use unique suffixes** (e.g., `testInfo.testId` or `Date.now()`) for parallel test isolation.
* **Use the fixture pattern** for automatic cleanup after each test.
* **Keep `timeoutMs` reasonable** — default is 120s, but 60s is usually sufficient.
* **Prefer `waitForEmail`** over polling with `listEmails` — it handles retry logic automatically.
* **Use structured extraction with Zod schemas** when you need typed data (URLs, codes, dates).
* **Common extraction prompts:**
  - `"Extract the OTP or verification code"`
  - `"Extract the sign-in or verification URL"`
  - `"Extract the order number and delivery date"`

## CI Reporter / Cloud

```bash
# npm
npm install -D @stablyai/playwright-test

# pnpm
pnpm add -D @stablyai/playwright-test

# yarn
yarn add -D @stablyai/playwright-test

# bun
bun add -d @stablyai/playwright-test
```

```ts
// playwright.config.ts
import { defineConfig, stablyReporter } from "@stablyai/playwright-test";

export default defineConfig({
  reporter: [
    ["list"],
    stablyReporter({
      apiKey: process.env.STABLY_API_KEY,
      projectId: process.env.STABLY_PROJECT_ID,
      // Optional: Scrub sensitive values from traces before upload
      sensitiveValues: ["secret-password", process.env.API_SECRET].filter(Boolean),
    }),
  ],
  use: {
    trace: "on", // Required for trace uploads
  },
});
```

**Reporter Options:**
- `apiKey` (required): Your Stably API key
- `projectId` (required): Your Stably project ID
- `sensitiveValues` (optional): Array of strings to scrub from trace files
- `notificationConfigs` (optional): Per-project notification settings for Slack/email

## Notifications (Email/Slack)

Configure notifications per project via `stably` property in defineConfig:

```ts
import { defineConfig, stablyReporter } from "@stablyai/playwright-test";

export default defineConfig({
  reporter: [stablyReporter({ apiKey: "...", projectId: "..." })],
  projects: [
    {
      name: "smoke",
      stably: {
        notifications: {
          slack: {
            channelName: "#test-alerts",
            notifyOnStart: true,
            notifyOnResult: "failures-only", // "all" | "failures-only"
          },
          email: {
            to: ["team@example.com"],
            notifyOnResult: "all",
          },
        },
      },
    },
  ],
});
```

## Commands

```bash
# Recommended for Stably reporter + auto-heal
# npm
npx stably test
# pnpm
pnpm dlx stably test
# yarn berry
yarn dlx stably test
# bun
bunx stably test

# Still supported (requires your reporter/config to be set up)
# npm
npm exec playwright test
# pnpm
pnpm exec playwright test
# yarn
yarn playwright test
# bun
bunx playwright test
# All Playwright CLI flags still work (headed, ui, project, file filters…)

# When running tests for debugging/getting stacktraces:
npm exec playwright test --reporter=list  # or pm-equivalent; disable HTML reporter for direct terminal output
```

## ESM Utilities

For ESM projects needing `__dirname` or `__filename` equivalents:

```ts
import { getDirname, getFilename } from "@stablyai/playwright-test";

const __dirname = getDirname(import.meta.url);
const __filename = getFilename(import.meta.url);

// Use in tests for file paths
import path from "path";
await page.setInputFiles("input", path.join(__dirname, "fixtures", "file.pdf"));
```

## Best Practices

* **CRITICAL: All locators must use the `.describe()` method** for readability in trace views and test reports. Example: `page.getByRole('button', { name: 'Submit' }).describe('Submit button')` or `page.locator('table tbody tr').first().describe('First table row')`
* Scope visual checks with locators; keep prompts specific with labels/units.
* Use `toHaveScreenshot` for stable pixel‑perfect UIs; `aiAssert` for dynamic UIs.
* **Be deliberate with `fullPage: true`**: Default to viewport-only screenshots. Only use `fullPage: true` when your assertion genuinely requires content beyond the visible viewport (e.g., verifying footer content on a long page, checking full scrollable lists). Viewport captures are faster and more cost-effective.

## Troubleshooting

* **Slow assertions** → scope visuals; reduce viewport.
* **Agent stops early** → increase `maxCycles` or break task into smaller steps.

## Minimal Template

```ts
import { test, expect } from "@stablyai/playwright-test";

test("AI‑enhanced dashboard", async ({ page, agent }) => {
  await page.goto("/dashboard");

  // Use agent for complex workflows
  await agent.act("Navigate to settings and enable notifications", { page });

  // Use AI assertions for dynamic content
  await expect(page).aiAssert(
    "Dashboard shows revenue chart (>= 6 months) and account spotlight card"
  );
});
```

---

## Creating E2E Tests with Stably SDK

When creating end-to-end tests, follow these guidelines:

### 1. Understand Requirements
- Ask clarifying questions if the test scenario is unclear
- Identify the user flow, expected outcomes, and edge cases
- Determine which pages/components need testing

### 2. Choose the Right Tools

**Use Playwright when:**
- Simple, deterministic interactions (clicks, fills, selects)
- Static content that doesn't change
- Cost and speed are priorities

**Use Stably SDK when:**
- Visual assertions on dynamic UIs → `aiAssert()`
- Complex multi-step workflows → `agent.act()`
- Canvas interactions or coordinate-based operations → `agent.act()`
- Data extraction from UI → `page.extract()`
- Elements are hard to locate reliably → `page.getLocatorsByAI()`
- Email verification flows (OTP, magic links, confirmations) → `Inbox` from `@stablyai/email`

### 3. Structure the Test

```ts
import { test, expect } from "@stablyai/playwright-test";

test("descriptive test name", async ({ page, agent }) => {
  // 1. Setup & Navigation
  await page.goto("/your-page");

  // 2. Interactions (prefer Playwright, use agent for complex workflows)
  await page.getByRole('button', { name: 'Login' }).describe('Login button').click();

  // For complex workflows:
  await agent.act("Complete the multi-step checkout process", { page });

  // 3. Assertions (prefer Playwright, use AI assertions for dynamic content)
  await expect(page.getByText('Welcome')).toBeVisible();

  // For visual/dynamic assertions:
  await expect(page).aiAssert(
    "Dashboard shows revenue chart and user profile card"
  );
});
```

### 4. Best Practices

**CRITICAL:**
- **Always use `.describe()` on locators** for better traceability
  ```ts
  page.getByRole('button', { name: 'Submit' }).describe('Submit form button')
  ```

**General:**
- Write clear, descriptive test names
- Add comments explaining complex logic
- Use semantic locators (getByRole, getByLabel) over CSS selectors
- Keep prompts specific for AI assertions
- Scope visual checks with locators when possible
- Default to viewport screenshots (only use `fullPage: true` when needed)
- Handle async operations with proper waits
- Consider error states and edge cases

### 5. Example Patterns

**Login Flow:**
```ts
test("user can login successfully", async ({ page }) => {
  await page.goto("/login");
  await page.getByLabel('Email').describe('Email input').fill('user@example.com');
  await page.getByLabel('Password').describe('Password input').fill('password123');
  await page.getByRole('button', { name: 'Sign In' }).describe('Sign in button').click();

  await expect(page).toHaveURL('/dashboard');
  await expect(page.getByText('Welcome back')).toBeVisible();
});
```

**Complex Workflow with AI:**
```ts
test("complete checkout process", async ({ page, agent }) => {
  await page.goto("/products");

  // Use agent for complex multi-step process
  await agent.act(
    "Add 2 items to cart, proceed to checkout, and fill shipping details",
    { page }
  );

  // Verify outcome
  await expect(page).aiAssert(
    "Order confirmation page with order number and thank you message"
  );
});
```

**Email Verification Flow:**
```ts
import { Inbox } from "@stablyai/email";

test("signup with email verification", async ({ page }) => {
  const inbox = await Inbox.build({ suffix: `signup-${Date.now()}` });

  await page.goto("/signup");
  await page.getByLabel("Email").describe("Email input").fill(inbox.address);
  await page.getByLabel("Password").describe("Password input").fill("SecurePass123!");
  await page.getByRole("button", { name: "Sign Up" }).describe("Sign up button").click();

  // Wait for verification email
  const email = await inbox.waitForEmail({
    subject: "verify your email",
    timeoutMs: 60_000,
  });

  // Extract the verification link using AI
  const { data: verifyUrl } = await inbox.extractFromEmail({
    id: email.id,
    prompt: "Extract the email verification URL",
  });

  await page.goto(verifyUrl);
  await expect(page).toHaveURL("/welcome");

  // Cleanup
  await inbox.deleteAllEmails();
});
```

**Visual Regression:**
```ts
test("homepage matches design", async ({ page }) => {
  await page.goto("/");

  // AI-powered visual assertion for dynamic content
  await expect(page).aiAssert(
    "Hero section with call-to-action button, feature cards below, and navigation bar at top"
  );
});
```
