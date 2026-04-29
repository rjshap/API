# Base URL & Environments

## Sandbox Endpoints

All sandbox API calls go through `api.autoshares.dev`:

| Service | Sandbox URL | Description |
|---------|------------|-------------|
| **Authentication** | `https://api.autoshares.dev/token` | Get Bearer token |
| **Logout** | `https://api.autoshares.dev/logout` | End session |
| **Orders** | `https://api.autoshares.dev/v1.0/accounts/{id}/orders` | Place, cancel, list orders |
| **Positions** | `https://api.autoshares.dev/v1.0/accounts/{id}/positions` | Current positions |
| **Securities** | `https://api.autoshares.dev/v1.0/equities/lookup?symbol=X` | Search symbols |
| **Option Chain** | `https://api.autoshares.dev/v1.0/options/optionChain/{symbol}` | Options data |
| **Chart Data** | `https://api.autoshares.dev/v1.0/history/symbols` | OHLCV candles |
| **Watchlists** | `https://api.autoshares.dev/v1.0/users/{id}/watchlists` | Manage watchlists |
| **Price Alerts** | `https://api.autoshares.dev/v1.0/users/{id}/pricealerts` | Alert triggers |
| **News** | `https://api.autoshares.dev/v1.0/news?symbol=X` | Company news |
| **Streamers** | `https://api.autoshares.dev/v1.0/streamers` | Get WebSocket URLs |
| **Agreements** | `https://api.autoshares.dev/v1.0/users/@me/agreements` | Market data agreements |
| **Health** | `https://api.autoshares.dev/health` | API gateway status |

### Back Office (Clearing) — Sandbox

| Service | Sandbox URL | Description |
|---------|------------|-------------|
| **Accounts** | `https://api.autoshares.dev/clearing/GetAccounts` | List accounts |
| **Add Account** | `https://api.autoshares.dev/clearing/AddAccount` | Open new account |
| **Balances** | `https://api.autoshares.dev/clearing/GetBalances` | Account balances |
| **Positions** | `https://api.autoshares.dev/clearing/GetPositions` | Settled positions |
| **Orders** | `https://api.autoshares.dev/clearing/GetOrdersV2` | Order history |
| **Add Order** | `https://api.autoshares.dev/clearing/AddOrderV2` | Submit order |
| **ACH Profiles** | `https://api.autoshares.dev/clearing/GetACHProfiles` | Bank accounts |
| **ACH Request** | `https://api.autoshares.dev/clearing/AddACHRequest` | Deposit/withdraw |
| **Wire Profiles** | `https://api.autoshares.dev/clearing/GetWireProfiles` | Wire accounts |
| **ACATS** | `https://api.autoshares.dev/clearing/GetACATS` | Account transfers |
| **Add ACATS** | `https://api.autoshares.dev/clearing/AddACATS` | Initiate transfer |
| **Reports** | `https://api.autoshares.dev/clearing/GetReports` | Statements/confirms |
| **Margin** | `https://api.autoshares.dev/clearing/GetMarginCalls` | Margin calls |
| **Transactions** | `https://api.autoshares.dev/clearing/GetTransactions` | Transaction ledger |
| **System Status** | `https://api.autoshares.dev/clearing/GetSystemStatus` | Clearing status |

### WebSocket Streaming (Direct Connection)

WebSocket connections cannot be proxied — connect directly to the streamer hosts returned by `/v1.0/streamers`:

| Streamer | URL | Data |
|----------|-----|------|
| **Quote** | `wss://{quote-host}:443` | Real-time bid/ask/last/volume |
| **Trade** | `wss://{trade-host}:443` | Order updates, position changes |

## Production Endpoints

Production URLs will be provided during onboarding. The API interface is identical to sandbox — only the base URL changes:

| Environment | Trading API | Clearing API |
|-------------|------------|-------------|
| **Sandbox** | `https://api.autoshares.dev/*` | `https://api.autoshares.dev/clearing/*` |
| **Production** | `https://api.autoshares.com/*` | `https://api.autoshares.com/clearing/*` |

## Authentication

All API requests (except `/token`) require two headers:

| Header | Description |
|--------|-------------|
| `Et-App-Key` | Your application key (provided during onboarding) |
| `Authorization` | `Bearer {token}` from the `/token` endpoint |

For clearing endpoints, use:

| Header | Description |
|--------|-------------|
| `Ocp-Apim-Subscription-Key` | Your clearing API key |

All clearing endpoints also require `?Corr={corr}&Office={office}` query parameters.

## Getting Your Credentials

To access the API, you need:

1. **Et-App-Key** — your application identifier
2. **Username & Password** — trading account credentials
3. **Clearing API Key** — for back office endpoints (separate access)

Contact **support@autoshares.com** or use the [Get API Access](/pages/signup.md) form.

## Token Lifecycle

| Event | Behavior |
|-------|----------|
| Login | Token issued, valid 24 hours |
| API call | Timer does NOT reset (absolute expiry) |
| 401 response | Token expired — re-authenticate |
| Logout | Token invalidated immediately |

Use the auto-refresh pattern from the [SDKs page](/pages/sdks.md) for seamless session management.

## Sandbox vs Production

| Feature | Sandbox | Production |
|---------|---------|------------|
| Real orders | No (paper trading) | Yes |
| Market data | 15-min delayed | Real-time (with agreements) |
| Account balances | Simulated | Real |
| CORS | Handled by proxy | Handled by proxy |
| Rate limits | Same as production | Enforced |
| WebSocket streaming | Available (delayed) | Available (real-time) |
| Clearing/Settlement | Simulated | Real (T+1) |
