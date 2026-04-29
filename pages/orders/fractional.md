# Fractional / Partial Share Order

A fractional share order allows you to buy or sell a non-whole number of shares. Instead of purchasing 10 full shares, you can purchase 10.5 shares or any other decimal quantity. Fractional orders must be submitted as limit orders, and they enable investors to allocate exact dollar amounts to a position regardless of a stock's per-share price.

## Endpoint

```
POST /v1.0/accounts/{accountId}/orders
```

## Request Body

```json
{
  "Symbol": "AAPL",
  "Side": "Buy",
  "Type": "Limit",
  "Price": 175.50,
  "Quantity": 10.5,
  "TimeInForce": "Day",
  "Exchange": "Auto"
}
```

## Key Fields

| Field | Type | Required | Description |
|-------|------|----------|-------------|
| `Symbol` | string | Yes | Ticker symbol of the security. Must be a fractional-eligible security. |
| `Side` | string | Yes | `Buy` or `Sell`. |
| `Type` | string | Yes | Must be `Limit`. Fractional orders are not supported as market orders. |
| `Price` | number | Yes | The limit price per share. |
| `Quantity` | number | Yes | Number of shares, including a decimal portion (e.g., `10.5`, `0.25`, `3.75`). |
| `TimeInForce` | string | Yes | `Day`, `GTC`, etc. |
| `Exchange` | string | No | Routing destination. `Auto` lets the system choose the best venue. |

## Important Notes

- Fractional orders **must** use the `Limit` order type. Market orders with decimal quantities will be rejected.
- Not all securities support fractional shares. Check the security's eligibility before placing the order.
- The minimum fractional quantity may vary. Consult your account configuration for details.
- Fractional shares are typically only available during regular market hours.
- Fractional positions are displayed with decimal precision in portfolio and position endpoints.
- Selling fractional shares follows the same format: set `Side` to `Sell` and provide the decimal `Quantity` you wish to sell.

## cURL Example

```bash
curl -X POST "https://api.autoshares.dev/v1.0/accounts/{accountId}/orders" \
  -H "Authorization: Bearer {accessToken}" \
  -H "Content-Type: application/json" \
  -d '{
    "Symbol": "AAPL",
    "Side": "Buy",
    "Type": "Limit",
    "Price": 175.50,
    "Quantity": 10.5,
    "TimeInForce": "Day",
    "Exchange": "Auto"
  }'
```
