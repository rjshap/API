# Trailing Stop-Limit Order

A trailing stop-limit order combines a trailing stop with a limit order. Like a trailing stop, the trigger price moves with the market in a favorable direction. However, once triggered, it places a limit order instead of a market order. This gives you both dynamic stop protection and control over the execution price, at the cost of possible non-execution if the market moves past your limit.

## Endpoint

```
POST /v1.0/accounts/{accountId}/orders
```

## Request Body

```json
{
  "Symbol": "AAPL",
  "Side": "Sell",
  "Type": "TrailingStopLimit",
  "TrailingStopAmountType": "Absolute",
  "TrailingStopAmount": 5.00,
  "TrailingLimitAmountType": "Absolute",
  "TrailingLimitAmount": 1.00,
  "Quantity": 100,
  "TimeInForce": "Day",
  "Exchange": "Auto"
}
```

## Key Fields

| Field | Type | Required | Description |
|-------|------|----------|-------------|
| `Symbol` | string | Yes | Ticker symbol of the security. |
| `Side` | string | Yes | `Buy` or `Sell` (also `SellShort`, `BuyToCover`). |
| `Type` | string | Yes | Must be `TrailingStopLimit` for this order type. |
| `TrailingStopAmountType` | string | Yes | `Absolute` or `Persentage`. Determines how the trailing stop distance is measured. |
| `TrailingStopAmount` | number | Yes | The trailing stop distance from the peak/trough price. |
| `TrailingLimitAmountType` | string | Yes | `Absolute` or `Persentage`. Determines how the limit offset is measured relative to the triggered stop price. |
| `TrailingLimitAmount` | number | Yes | The offset from the stop price that defines the limit. For a sell, the limit is set below the triggered stop price by this amount. |
| `Quantity` | number | Yes | Number of shares to trade. |
| `TimeInForce` | string | Yes | `Day`, `GTC`, etc. |
| `Exchange` | string | No | Routing destination. `Auto` lets the system choose the best venue. |

## How It Works (Sell Example)

1. You own AAPL at $180. You set a trailing stop of $5.00 (absolute) and a trailing limit offset of $1.00 (absolute).
2. AAPL rises to $190. The trailing stop price adjusts to $185.00 ($190 - $5).
3. AAPL drops to $185.00, triggering the stop. A limit sell order is placed at $184.00 ($185 - $1).
4. The order fills only at $184.00 or higher.

## Important Notes

- The `TrailingLimitAmount` defines how far from the triggered stop price the limit will be set.
- Both amount types (`TrailingStopAmountType` and `TrailingLimitAmountType`) can be set independently. You can trail the stop by a percentage and offset the limit by an absolute dollar amount, or any combination.
- The spelling `Persentage` is the correct API value (not "Percentage").
- If the market gaps past the limit price, the order will not fill, leaving you without protection.

## cURL Example

```bash
curl -X POST "https://api.autoshares.dev/v1.0/accounts/{accountId}/orders" \
  -H "Authorization: Bearer {accessToken}" \
  -H "Content-Type: application/json" \
  -d '{
    "Symbol": "AAPL",
    "Side": "Sell",
    "Type": "TrailingStopLimit",
    "TrailingStopAmountType": "Absolute",
    "TrailingStopAmount": 5.00,
    "TrailingLimitAmountType": "Absolute",
    "TrailingLimitAmount": 1.00,
    "Quantity": 100,
    "TimeInForce": "Day",
    "Exchange": "Auto"
  }'
```
