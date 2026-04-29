# Quick Start Guide

Get up and running with the AutoShares Trading API in 5 minutes.

## Prerequisites

- An AutoShares sandbox account ([sign up here](javascript:void(0)))
- Your **Et-App-Key** (provided after registration)
- A tool for making HTTP requests (cURL, Postman, or your preferred language)

## Step 1: Authenticate

Get a Bearer token by sending your credentials to the `/token` endpoint:

```bash
curl -X POST \
  -H "Accept: application/json" \
  -H "Et-App-Key: YOUR_APP_KEY" \
  -H "Username: YOUR_USERNAME" \
  -H "Password: YOUR_PASSWORD" \
  "https://api.autoshares.dev/token"
```

**Response:**

```json
{
  "State": "Succeeded",
  "Token": "VGhpcyBpcyBleGFtcGxlIHRva2Vu..."
}
```

Save the token — you'll use it in every subsequent request. Tokens are valid for **60 minutes**.

## Step 2: Look Up a Symbol

Search for a stock by ticker or name:

```bash
curl -X GET \
  -H "Authorization: Bearer YOUR_TOKEN" \
  -H "Et-App-Key: YOUR_APP_KEY" \
  "https://api.autoshares.dev/v1.0/equities/lookup?symbol=AAPL&count=5"
```

Note the **Id** field in the response — that's the security ID you'll need for orders.

## Step 3: Place an Order

Place a market order to buy 10 shares of AAPL:

```bash
curl -X POST \
  -H "Authorization: Bearer YOUR_TOKEN" \
  -H "Et-App-Key: YOUR_APP_KEY" \
  -H "Content-Type: application/json" \
  -d '{
    "Symbol": "AAPL",
    "Type": "Market",
    "Side": "Buy",
    "Quantity": 10
  }' \
  "https://api.autoshares.dev/v1.0/accounts/YOUR_ACCOUNT_ID/orders"
```

Or a limit order at a specific price:

```json
{
  "Symbol": "AAPL",
  "Type": "Limit",
  "Side": "Buy",
  "Quantity": 10,
  "Price": 180.00
}
```

## Step 4: Check Your Positions

See what you own:

```bash
curl -X GET \
  -H "Authorization: Bearer YOUR_TOKEN" \
  -H "Et-App-Key: YOUR_APP_KEY" \
  "https://api.autoshares.dev/v1.0/accounts/YOUR_ACCOUNT_ID/positions"
```

## Step 5: Check Order Status

Get the status of your orders:

```bash
curl -X GET \
  -H "Authorization: Bearer YOUR_TOKEN" \
  -H "Et-App-Key: YOUR_APP_KEY" \
  "https://api.autoshares.dev/v1.0/accounts/YOUR_ACCOUNT_ID/orders"
```

## Common Order Types

| Type | Required Fields | Example Use |
|------|----------------|-------------|
| **Market** | Symbol, Side, Quantity | Execute immediately at best price |
| **Limit** | + Price | Buy at $180 or better |
| **Stop** | + StopPrice | Sell if price drops to $170 |
| **Stop Limit** | + StopPrice, Price | Stop at $170, limit at $168 |
| **OTOCO Bracket** | 3 legs (entry + take-profit + stop-loss) | Buy at $180, sell at $190 or stop at $170 |

## What's Next?

- **Streaming Data** — Get real-time quotes via WebSocket instead of polling
- **Options Trading** — Look up option chains and place options orders
- **OTOCO Brackets** — Set up entry + take-profit + stop-loss in one order
- **ACH Transfers** — Link bank accounts and move funds
- **Price Alerts** — Get notified when a stock hits your target price

## Quick Reference

| Action | Method | Endpoint |
|--------|--------|----------|
| Authenticate | POST | `/token` |
| Search securities | GET | `/v1.0/equities/lookup?symbol=X` |
| Place order | POST | `/v1.0/accounts/{id}/orders` |
| Cancel order | DELETE | `/v1.0/accounts/{id}/orders/{orderId}` |
| Get positions | GET | `/v1.0/accounts/{id}/positions` |
| Get orders | GET | `/v1.0/accounts/{id}/orders` |
| Get account info | GET | `/v1.0/accounts/{id}/info` |
| Get option chain | GET | `/v1.0/options/optionChain/{symbol}` |
| Get chart data | PUT | `/v1.0/history/symbols` |
| Get watchlists | GET | `/v1.0/users/{id}/watchlists` |
| Get streamer info | GET | `/v1.0/streamers` |

## Need Help?

- Browse the **API Reference** in the sidebar for full endpoint details
- Use the **Ask AI** button to get instant answers
- Check **Error Codes** if something isn't working
- Contact us at **support@autoshares.com**
