# Stop-Limit Order

A stop-limit order combines the features of a stop order and a limit order. When the security reaches the specified stop price, the order becomes a limit order (rather than a market order) at the specified limit price. This gives you control over both the trigger point and the worst acceptable execution price.

## Endpoint

```
POST /v1.0/accounts/{accountId}/orders
```

## Request Body

```json
{
  "Symbol": "AAPL",
  "Side": "Sell",
  "Type": "StopLimit",
  "StopPrice": 160.00,
  "Price": 158.00,
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
| `Type` | string | Yes | Must be `StopLimit` for this order type. |
| `StopPrice` | number | Yes | The trigger price. When the market reaches this price, the order activates as a limit order. |
| `Price` | number | Yes | The limit price. Once triggered, the order will only fill at this price or better. |
| `Quantity` | number | Yes | Number of shares to trade. |
| `TimeInForce` | string | Yes | `Day`, `GTC`, `IOC`, `FOK`, etc. |
| `Exchange` | string | No | Routing destination. `Auto` lets the system choose the best venue. |

## Important Notes

- For a sell stop-limit: `StopPrice` is typically set above `Price`. When the market drops to `StopPrice`, a limit sell at `Price` is placed. If the market falls below `Price` before filling, the order will not execute.
- For a buy stop-limit: `StopPrice` is typically set below `Price`. When the market rises to `StopPrice`, a limit buy at `Price` is placed.
- The key risk is non-execution. If the market gaps past both your stop and limit prices, the order remains unfilled.
- Stop-limit orders are useful in volatile markets where you want downside protection but refuse to sell at any price.

## cURL Example

```bash
curl -X POST "https://api.autoshares.dev/v1.0/accounts/{accountId}/orders" \
  -H "Authorization: Bearer {accessToken}" \
  -H "Content-Type: application/json" \
  -d '{
    "Symbol": "AAPL",
    "Side": "Sell",
    "Type": "StopLimit",
    "StopPrice": 160.00,
    "Price": 158.00,
    "Quantity": 100,
    "TimeInForce": "Day",
    "Exchange": "Auto"
  }'
```
