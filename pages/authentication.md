# Authentication

The AutoShares Trading API supports two authentication models depending on how your application accesses AutoShares accounts:

| Model | When to use | Auth header |
|-------|-------------|-------------|
| **OAuth 2.0** *(third-party apps)* | Your app is integrating against AutoShares users' accounts — they sign in with their own credentials and explicitly authorize your app | `Authorization: Bearer asat_…` |
| **Direct AutoShares token** *(legacy)* | Your app is the AutoShares browser/mobile itself | `Authorization: Bearer …` + `Ocp-Apim-Subscription-Key: …` |

**For all new integrations, use OAuth 2.0.** The direct AutoShares token flow is documented for completeness but is reserved for AutoShares' own applications.

---

## OAuth 2.0 — for third-party developers

AutoShares implements the standard OAuth 2.0 authorization-code grant (RFC 6749). The flow is:

1. **You** register your app in the [Developer Portal](javascript:loadPortal\(\)) (click "Developer Portal" in the top nav) and receive a `client_id` and `client_secret`.
2. **Your app** redirects the user's browser to AutoShares' `/oauth/authorize` endpoint with your `client_id`, a `redirect_uri` you've registered, and the scopes you need.
3. **The user** sees an AutoShares consent screen listing what your app is asking for, then clicks **Allow**.
4. AutoShares redirects back to your `redirect_uri` with a one-time `code`.
5. **Your app** (server-side) exchanges the code at `/oauth/token` for an access token.
6. **Your app** calls `/api/v1/*` endpoints with `Authorization: Bearer asat_…`.

```
Your app                                                AutoShares
  │                                                       │
  │ 1. Redirect user to                                   │
  │    api.autoshares.dev/oauth/authorize?                │
  │       client_id=as_xxx                                │
  │      &redirect_uri=https://yourapp.com/cb             │
  │      &scope=read+orders                               │
  │      &state=anti-csrf-string                          │
  │                                                       │
  │ ────────────────────────────────────────────────────► │
  │                                          (consent UI) │
  │                                                       │
  │ 2. User clicks Allow → AutoShares 302s back to        │
  │    https://yourapp.com/cb?code=XYZ&state=…            │
  │ ◄──────────────────────────────────────────────────── │
  │                                                       │
  │ 3. Your SERVER POSTs to /oauth/token:                 │
  │    { grant_type, code, client_id, client_secret }     │
  │ ────────────────────────────────────────────────────► │
  │                                                       │
  │ 4. Receive: { access_token, expires_in, scope }       │
  │ ◄──────────────────────────────────────────────────── │
  │                                                       │
  │ 5. Call any /api/v1/* endpoint with                   │
  │    Authorization: Bearer asat_…                       │
  │ ────────────────────────────────────────────────────► │
  │                                                       │
```

### What the access token actually does

Under the hood, AutoShares uses **Microsoft Entra On-Behalf-Of** to translate your `asat_…` token into a fresh AutoShares-scoped token bound to the specific user. Your token is **proof of consent**; the upstream credentials are minted per request and never exposed to your application.

This means:

- You never see or store the user's AutoShares credentials
- Token revocation flows through Entra — disabling a user's AutoShares account immediately invalidates every third-party token tied to them
- Every action your app takes is attributed to the real user at the AutoShares level (clean audit trail for compliance)

### Scopes

| Scope | Grants |
|-------|--------|
| `read` | Read account balances, positions, order history |
| `orders` | Place, modify, and cancel orders on the user's behalf |
| `clearing` | Read clearing records — transfers, transactions, statements |
| `account` | Read profile, KYC status, and account documents |
| `streaming` | Subscribe to WebSocket quote feed for the user's watchlist |

Request the **minimum scopes** your app needs. Users see every scope on the consent screen with a write-action warning (`!` icon) for the dangerous ones. Apps asking for unnecessary scopes will see drop-off at consent.

### Two-step scope check

Every API call passes through two gates:

1. **Token scope** — the scopes the user authorized when they granted consent
2. **User scope ceiling** — scopes the user has enabled at the platform level (in their AutoShares account preferences)

A call requires `read`? Both gates must include `read`. If a user disables `orders` from their account preferences, every outstanding access token loses its `orders` capability immediately — no token revocation needed.

### Token lifetime

| Token | TTL | Notes |
|-------|-----|-------|
| Authorization code | 10 minutes | One-time use |
| Access token (`asat_…`) | 90 minutes | Bearer for all `/api/v1/*` calls |
| Refresh token | *Coming soon* | For now, send the user through `/oauth/authorize` again after expiration |

### Endpoints

| Endpoint | Method | Purpose |
|----------|--------|---------|
| `/oauth/authorize` | GET | Initiate the consent flow (browser redirect) |
| `/oauth/token` | POST | Exchange `code` for `access_token` |
| `/oauth/revoke` | POST | Revoke a token before its TTL |

Hostnames:

- **Sandbox:** `api.autoshares.dev`
- **Production:** `api.autoshares.com` *(activates after FINRA approval)*

### See also

- **[OAuth Quickstart](oauth-quickstart.md)** — copy-paste curl examples for the full flow
- **[Get API access](signup.md)** — how to register your app

---

## Direct AutoShares token — for AutoShares' own applications

> This authentication path is reserved for AutoShares' first-party web and mobile applications. Third-party integrations should use OAuth 2.0 (above).

The browser/native app authenticates against AutoShares' Microsoft Entra External ID tenant. The Entra OIDC `sub` claim is the user's stable identity and is recognized natively by AutoShares via a federated trust relationship.

### Headers required on every call

| Header | Value |
|--------|-------|
| `Authorization` | `Bearer <Entra access token>` |
| `Ocp-Apim-Subscription-Key` | Master tenant subscription key |

The subscription key is injected server-side by the AutoShares API proxy at `api.autoshares.dev` — first-party app code should never include this key in client-side bundles.

### Legacy direct-token endpoint

For service-to-service flows that AutoShares operates internally, a direct token can be obtained via AutoShares's `/token` endpoint. This is an internal interface and is NOT exposed to third-party developers — use OAuth 2.0 instead.

```
POST /token
Et-App-Key: <internal app key>
Username: <user>
Password: <pass>
```

Returns a standard OAuth bearer + refresh token usable against the AutoShares REST API directly.
