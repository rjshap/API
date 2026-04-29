# Notional / Dollar Amount Order

A notional order (also called a dollar amount order) lets you specify a dollar amount to invest rather than a number of shares. The system calculates the appropriate quantity (including fractional shares) based on the current price. This is useful for portfolio allocation, recurring investments, or when you want to invest an exact dollar amount regardless of the per-share price.

## Endpoint

```
POST /v1.0/accounts/{accountId}/orders
```

## Request Body

```json
{
  "Symbol": "AAPL",
  "Side": "Buy",
  "Type": "Market",
  "Quantity": 500.00,
  "Price": 500.00,
  "ExecInst": "CashAmountOrder",
  "TimeInForce": "Day",
  "Exchange": "Auto"
}
```

## Key Fields

| Field | Type | Required | Description |
|-------|------|----------|-------------|
| `Symbol` | string | Yes | Ticker symbol of the security. |
| `Side` | string | Yes | `Buy` or `Sell`. |
| `Type` | string | Yes | `Market` or `Limit`. |
| `ExecInst` | string | Yes | Must be `CashAmountOrder` to indicate this is a notional/dollar-based order. |
| `Quantity` | number | Yes | The dollar amount to invest (e.g., `500.00` means $500). |
| `Price` | number | Yes | Set to the same value as `Quantity` (the dollar amount). |
| `TimeInForce` | string | Yes | `Day`, `GTC`, etc. |
| `Exchange` | string | No | Routing destination. `Auto` lets the system choose the best venue. |

## Important Notes

- Both `Quantity` and `Price` should be set to the dollar amount you want to invest. The system interprets these as the notional value, not share count or per-share price.
- The `ExecInst` field set to `CashAmountOrder` is what distinguishes this from a regular order. Without it, the system treats `Quantity` as a share count.
- The resulting fill will likely include fractional shares. For example, investing $500 in a stock trading at $175.50 would yield approximately 2.849 shares.
- Notional orders are ideal for dollar-cost averaging strategies and robo-advisor-style portfolio rebalancing.
- Not all securities support notional ordering. The security must be eligible for fractional shares.

## cURL Example

```bash
curl -X POST "https://{your-environment}.etnasoft.us/api/v1.0/accounts/{accountId}/orders" \
  -H "Authorization: Bearer {accessToken}" \
  -H "Content-Type: application/json" \
  -d '{
    "Symbol": "AAPL",
    "Side": "Buy",
    "Type": "Market",
    "Quantity": 500.00,
    "Price": 500.00,
    "ExecInst": "CashAmountOrder",
    "TimeInForce": "Day",
    "Exchange": "Auto"
  }'
```
