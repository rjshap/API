# Extended Hours Trading

Extended hours orders allow trading outside of regular market hours (9:30 AM - 4:00 PM ET). You can participate in pre-market and post-market sessions by setting the `ExtendedHours` field. Extended hours trading provides the ability to react to earnings announcements, news events, and other catalysts that occur outside regular hours.

## Endpoint

```
POST /v1.0/accounts/{accountId}/orders
```

## Extended Hours Sessions

| Value | Session | Typical Hours (ET) |
|-------|---------|---------------------|
| `PRE` | Pre-market only | 4:00 AM - 9:30 AM |
| `POST` | Post-market only | 4:00 PM - 8:00 PM |
| `REGPRE` | Regular + Pre-market | 4:00 AM - 4:00 PM |
| `REGPOST` | Regular + Post-market | 9:30 AM - 8:00 PM |

## Request Body (Pre-Market)

```json
{
  "Symbol": "AAPL",
  "Side": "Buy",
  "Type": "Limit",
  "Price": 175.50,
  "Quantity": 100,
  "TimeInForce": "Day",
  "ExtendedHours": "PRE",
  "Exchange": "Auto"
}
```

## Request Body (Post-Market)

```json
{
  "Symbol": "AAPL",
  "Side": "Sell",
  "Type": "Limit",
  "Price": 180.00,
  "Quantity": 50,
  "TimeInForce": "Day",
  "ExtendedHours": "POST",
  "Exchange": "Auto"
}
```

## Request Body (Regular + Post-Market)

```json
{
  "Symbol": "AAPL",
  "Side": "Buy",
  "Type": "Limit",
  "Price": 175.50,
  "Quantity": 100,
  "TimeInForce": "Day",
  "ExtendedHours": "REGPOST",
  "Exchange": "Auto"
}
```

## Key Fields

| Field | Type | Required | Description |
|-------|------|----------|-------------|
| `Symbol` | string | Yes | Ticker symbol of the security. |
| `Side` | string | Yes | `Buy` or `Sell` (also `SellShort`, `BuyToCover`). |
| `Type` | string | Yes | Must be `Limit` for extended hours orders. Market orders are generally not supported outside regular hours. |
| `Price` | number | Yes | The limit price. |
| `Quantity` | number | Yes | Number of shares to trade. |
| `TimeInForce` | string | Yes | `Day`, `GTC`, etc. |
| `ExtendedHours` | string | Yes | `PRE`, `POST`, `REGPRE`, or `REGPOST`. Controls which sessions the order is active in. |
| `Exchange` | string | No | Routing destination. `Auto` lets the system choose the best venue. |

## Important Notes

- Extended hours orders **must** be limit orders. Market orders are not accepted during pre-market or post-market sessions.
- Liquidity is typically much lower during extended hours. Wider spreads and partial fills are common.
- Not all securities trade during extended hours. ETFs and large-cap stocks generally have the best extended hours liquidity.
- `PRE` and `POST` restrict the order to only that session. If unfilled, the order does not carry into regular hours.
- `REGPRE` and `REGPOST` allow the order to be active during both the extended session and regular market hours, increasing the chance of a fill.
- Extended hours availability depends on your account configuration and permissions.

## cURL Example

```bash
curl -X POST "https://{your-environment}.etnasoft.us/api/v1.0/accounts/{accountId}/orders" \
  -H "Authorization: Bearer {accessToken}" \
  -H "Content-Type: application/json" \
  -d '{
    "Symbol": "AAPL",
    "Side": "Buy",
    "Type": "Limit",
    "Price": 175.50,
    "Quantity": 100,
    "TimeInForce": "Day",
    "ExtendedHours": "REGPOST",
    "Exchange": "Auto"
  }'
```
