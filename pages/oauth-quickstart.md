# OAuth 2.0 Quickstart

This guide walks you through a complete OAuth integration in 15 minutes using curl. By the end you'll have:

- A registered AutoShares developer app with a `client_id` and `client_secret`
- An access token authorized by an AutoShares user
- A successful call to `/api/v1/me` returning the user's identity

> **Sandbox:** All examples use `api.autoshares.dev` (sandbox). Production goes live at `api.autoshares.com` after FINRA approval.

---

## Prerequisites

- An AutoShares account (sandbox accounts available free at [autoshares.dev/signup](https://www.autoshares.dev/signup))
- `curl` installed
- A way to receive a browser redirect — `http://localhost:3000/cb` works for development

---

## Step 1 — Register your app

Click **Developer Portal** in the top navigation of [documentation.autoshares.dev](https://documentation.autoshares.dev). Sign in with your AutoShares account credentials.

Click **+ New app** and fill in:

- **App name** — what your users will see on the consent screen
- **Redirect URI** — must match exactly when you initiate the OAuth flow. For development use `http://localhost:3000/cb`
- **Scopes** — start with `read` only; you can add more later

Click **Create app**. You'll see:

- **client_id** — `as_xxxxxxxxxxxxxxxx` — public, safe to put in app bundles
- **client_secret** — `xxxxxxxx…` — **shown once** — store this in your server-side env vars

```bash
# Save these for later
export AS_CLIENT_ID="as_xxxxxxxxxxxxxxxx"
export AS_CLIENT_SECRET="xxxxxxxx..."
export AS_REDIRECT="http://localhost:3000/cb"
```

---

## Step 2 — Send your user to the consent screen

In a real app, you'd open this URL in the user's browser when they click "Connect AutoShares". For the quickstart, paste it into your own browser:

```
https://api.autoshares.dev/oauth/authorize?client_id=as_xxxxxxxxxxxxxxxx&redirect_uri=http%3A%2F%2Flocalhost%3A3000%2Fcb&scope=read&state=anti-csrf-secret
```

| Parameter | Required | Notes |
|-----------|----------|-------|
| `client_id` | yes | Your app's client ID from Step 1 |
| `redirect_uri` | yes | URL-encoded. Must match what you registered |
| `scope` | yes | Space-separated list of requested scopes |
| `state` | yes | Random string your app generates; AutoShares echoes it back to defend against CSRF |
| `response_type` | optional | Defaults to `code` (the only value we currently support) |

The user sees:

> **Test App** wants permission to act on your AutoShares account.
>
> It will be able to:
> - ✓ Read your account balances, positions, and order history
>
> After approval you'll return to `http://localhost:3000/cb`
>
> [Cancel] [Allow access]

They click **Allow access**. Their browser is redirected to:

```
http://localhost:3000/cb?code=abcd1234ef…&state=anti-csrf-secret
```

The `state` value matches what you sent — verify it on your end to defeat CSRF.

```bash
# Copy the code from the URL
export AS_CODE="abcd1234ef..."
```

> If the user clicks **Cancel**, you'll receive `?error=access_denied&state=…` instead.

---

## Step 3 — Exchange the code for an access token

This step is **server-side only** — the `client_secret` must never appear in browser code.

```bash
curl -X POST https://api.autoshares.dev/oauth/token \
  -H "Content-Type: application/json" \
  -d "{
    \"grant_type\": \"authorization_code\",
    \"code\": \"${AS_CODE}\",
    \"client_id\": \"${AS_CLIENT_ID}\",
    \"client_secret\": \"${AS_CLIENT_SECRET}\"
  }"
```

Response:

```json
{
  "access_token": "asat_xxxxxxxxxxxxxxxxxxxxxxxx",
  "token_type": "Bearer",
  "expires_in": 5400,
  "scope": "read"
}
```

`expires_in` is in seconds (5400 = 90 minutes). The token is single-use across requests — you can call any `/api/v1/*` endpoint until it expires.

```bash
export AS_TOKEN="asat_xxxxxxxxxxxxxxxxxxxxxxxx"
```

---

## Step 4 — Call the API

### Who am I?

```bash
curl https://api.autoshares.dev/api/v1/me \
  -H "Authorization: Bearer ${AS_TOKEN}"
```

```json
{
  "userId": "auth0|abc123…",
  "appName": "Test App",
  "clientId": "as_xxxxxxxxxxxxxxxx",
  "grantedScopes": ["read"],
  "enabledScopes": ["read", "orders", "clearing", "account", "streaming"],
  "accountCount": 2
}
```

### List the user's accounts

```bash
curl https://api.autoshares.dev/api/v1/accounts \
  -H "Authorization: Bearer ${AS_TOKEN}"
```

```json
{
  "ok": true,
  "userId": "auth0|abc123…",
  "accounts": [
    { "acct_no": "12345678", "acct_type": "C", "corr": "TPRO", "office": "001", "kind": "individual" }
  ]
}
```

### Get account balances

```bash
curl https://api.autoshares.dev/api/v1/accounts/12345678/balances \
  -H "Authorization: Bearer ${AS_TOKEN}"
```

```json
{
  "ok": true,
  "account": "12345678",
  "balances": {
    "currency": "USD",
    "equity": 125000.00,
    "cashBalance": 12500.00,
    "buyingPower": 25000.00,
    "dayTradeBuyingPower": 100000.00
  }
}
```

### Try an account the user doesn't own

```bash
curl https://api.autoshares.dev/api/v1/accounts/99999999/balances \
  -H "Authorization: Bearer ${AS_TOKEN}"
```

```json
{ "error": "account does not belong to authenticated user" }
```

HTTP 403. AutoShares verifies account ownership on every call — a token granted by user A can never access user B's accounts, regardless of what's in the URL.

### Try a write call with read-only scope

```bash
curl -X POST https://api.autoshares.dev/api/v1/accounts/12345678/orders \
  -H "Authorization: Bearer ${AS_TOKEN}" \
  -H "Content-Type: application/json" \
  -d '{ "symbol": "AAPL", "side": "buy", "qty": 10 }'
```

```json
{ "error": "token missing required scope: orders" }
```

HTTP 403. The user authorized `read` only — the token can't place orders. To add order capability, re-register your app with `orders` scope checked and send the user back through `/oauth/authorize?scope=read+orders`.

---

## Step 5 — Revoking a token

If your user signs out of your app, you should proactively revoke their token (don't wait for the 90-minute TTL):

```bash
curl -X POST https://api.autoshares.dev/oauth/revoke \
  -H "Content-Type: application/json" \
  -d "{ \"token\": \"${AS_TOKEN}\" }"
```

```json
{ "ok": true }
```

The endpoint returns `200` even for unknown tokens (RFC 7009).

---

## What's next

| Topic | Where |
|-------|-------|
| Full list of `/api/v1/*` endpoints | [API Overview](api-overview.md) |
| How to place orders | [Placing Orders](place-order.md) |
| Real-time data via WebSocket | [Streaming Data](streaming-data.md) |
| Rate limits | [Rate Limiting](rate-limits.md) |
| Error codes | [Error Codes](error-codes.md) |

## Common questions

### Can I use OAuth from a single-page app (no server)?

You can use the **PKCE extension** to OAuth, which lets you skip the `client_secret` exchange. Send `code_challenge` + `code_challenge_method=S256` on `/oauth/authorize`, then `code_verifier` on `/oauth/token`. Standard PKCE flow — works the same way as Auth0 / Cognito / Okta. (Native mobile apps should also use PKCE.)

> **Note:** PKCE support is queued and will be the default for browser-side integrations. Until it ships, all OAuth flows require server-side code exchange.

### How do I handle token expiration mid-session?

Two options:

1. **Repeat the OAuth flow** — send the user back through `/oauth/authorize`. If they're still logged into AutoShares and they previously authorized your app, this completes in a single round-trip with no consent prompt.
2. **Use refresh tokens** *(coming soon)* — request the `offline_access` scope; receive a refresh token alongside the access token; exchange it for a fresh access token at `/oauth/token` with `grant_type=refresh_token`.

### What if my redirect URI changes?

You can register multiple redirect URIs per app from the Developer Portal — useful for separating staging from production. Each must match exactly when the OAuth flow is initiated.

### Can a single AutoShares user authorize multiple apps?

Yes, with separate tokens for each. Each authorization grant is independent — revoking one app's access doesn't affect others.

### Does the user see what my app does in real time?

Yes. AutoShares users can see every active third-party app from their account preferences and revoke any of them with one click. Audit logs persist at the ETNA level too — every order, transfer, or account change is attributed to your specific `client_id` for compliance review.

---

## Need help?

- **Email:** [developers@autoshares.com](mailto:developers@autoshares.com)
- **Slack:** Coming soon — sign up for an invite at [autoshares.dev/developers](https://www.autoshares.dev/developers)
