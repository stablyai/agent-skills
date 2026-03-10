---
name: google-auth-account-setup
description: Step-by-step guide for setting up a Google account for automated authentication testing with Stably SDK. Covers Google 2FA/OTP configuration, environment variables, context.authWithGoogle() usage, and troubleshooting common Google auth issues (blocked sign-in, OTP failures, session problems).
license: MIT
metadata:
  author: stably
  version: '1.0.0'
---

# Google Auth Account Setup

Complete guide for configuring a Google account for automated authentication testing with the Stably Playwright SDK.

## Prerequisites

- A **dedicated Google Workspace test account** — never use a personal account
- You will need: email, password, and the OTP secret (obtained in the setup steps below)

## Step-by-Step Google Account Setup

### 1. Sign in to the test Google account

Go to [accounts.google.com](https://accounts.google.com) and sign in with the test account credentials.

### 2. Navigate to Security settings

From the left sidebar, click **Security**.

### 3. Find the Authenticator setting

Scroll to **"How you sign in to Google"** and click **"Authenticator"**.

### 4. Add or change the authenticator app

Click **"Add Authenticator"** (or **"Change Authenticator App"** if one is already set up).

> **Warning:** Clicking "Change Authenticator App" invalidates any previous authenticator setup.

### 5. Reveal the OTP secret

When the QR code appears, click **"Can't scan QR code"** to reveal the OTP secret — a 32-character base32 string (letters A–Z, digits 2–7).

### 6. Copy the OTP secret

Copy the secret string. This becomes your `otpSecret` parameter (and the `GOOGLE_TEST_OTP_SECRET` environment variable).

### 7. Complete Google's setup with a verification code

Paste the OTP secret into the **Stably OTP helper** at [https://app.stably.ai/utils/google-otp-secret](https://app.stably.ai/utils/google-otp-secret) and copy the generated 6-digit code.

### 8. Verify the code in Google

Return to Google's verification screen, paste the 6-digit code, and click **"Verify"**.

### 9. Ensure only Authenticator is enabled for 2FA

Back on the Google Account security page, confirm that:
- 2-Step Verification is **ON**
- Under "Second Steps", only **Authenticator** is enabled
- Disable phone prompts, backup codes, and any other second-step methods
- Sign out of Google on mobile devices if needed (phone prompts can interfere)

### 10. Store credentials as environment variables

```bash
GOOGLE_TEST_EMAIL=qa@example.com
GOOGLE_TEST_PASSWORD=...
GOOGLE_TEST_OTP_SECRET=...  # the 32-char base32 string from step 6
```

Add these to your `.env` file (must be gitignored) or your CI secret manager.

## Quick Code Reference

### Recommended: context method

```ts
await context.authWithGoogle({
  email: process.env.GOOGLE_TEST_EMAIL!,
  password: process.env.GOOGLE_TEST_PASSWORD!,
  otpSecret: process.env.GOOGLE_TEST_OTP_SECRET!,
});
```

### Alternative: standalone import

```ts
import { authWithGoogle } from "@stablyai/playwright-test";

await authWithGoogle({
  context,
  email: process.env.GOOGLE_TEST_EMAIL!,
  password: process.env.GOOGLE_TEST_PASSWORD!,
  otpSecret: process.env.GOOGLE_TEST_OTP_SECRET!,
});
```

### Signature

```ts
authWithGoogle(options: {
  context: BrowserContext;
  email: string;
  password: string;
  otpSecret: string;
  forceRefresh?: boolean;  // bypass cached session
}): Promise<void>
```

### Required environment variables

| Variable | Description |
|----------|-------------|
| `GOOGLE_TEST_EMAIL` | Test account email address |
| `GOOGLE_TEST_PASSWORD` | Test account password |
| `GOOGLE_TEST_OTP_SECRET` | 32-char base32 OTP secret from step 6 |

## Troubleshooting

| Problem | Solution |
|---------|----------|
| **Google blocks sign-in** ("Try again later", "unusual activity") | Use a dedicated test account, avoid rapid successive logins, try from a different IP |
| **OTP code expired** | Codes rotate every 30 seconds — use the [Stably OTP helper](https://app.stably.ai/utils/google-otp-secret) for a fresh code |
| **2FA prompt still appears despite automation** | Ensure only Authenticator is enabled under "Second Steps" (disable phone prompts, backup codes, etc.) |
| **Session cache stale** | Use `forceRefresh: true` in `authWithGoogle` options |
| **OTP secret looks wrong** | Must be a 32-character base32 string (letters A–Z, digits 2–7), no spaces |

## Best Practices

- **Dedicated test accounts only** — never use personal Google accounts
- **Store credentials securely** — `.env` files (gitignored) or CI secret managers
- **Use Stably Environments** — `stably --env staging test` for secure credential storage in CI
- **Reuse auth sessions** — create one auth smoke test, save `storageState`, reuse for other tests
- **Rotate credentials** — rotate test account passwords and OTP secrets on a schedule

## Full Documentation

- [Stably Google Auth Docs](https://docs.stably.ai/stably2/auth/auth-with-google)
