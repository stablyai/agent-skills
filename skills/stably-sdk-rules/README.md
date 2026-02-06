# stably-sdk-rules

AI rules for writing tests with Stably Playwright SDK.

## Installation

```bash
npx skills add stablyai/agent-skills --skill stably-sdk-rules
```

## When to Use

- Writing or modifying Playwright tests with Stably AI features
- Using `aiAssert`, `agent.act()`, `page.extract()`, or `page.getLocatorsByAI()`
- Deciding between Playwright vs Stably SDK methods

## Key Features

### AI Assertions
```typescript
await expect(page).aiAssert(
  "Shows revenue trend chart and spotlight card"
);
```

### AI Extraction
```typescript
const data = await page.extract("Get the order ID");
```

### AI Agent
```typescript
await agent.act("Complete the checkout process", { page });
```

### AI Locator Finding
```typescript
const { locator, count } = await page.getLocatorsByAI("the login button");
await locator.click();
```

## Related

- [stably-sdk-setup](../stably-sdk-setup) - Setup assistant for Stably SDK
