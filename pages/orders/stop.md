# Stop Order

A stop order (also called a stop-loss order) becomes a market order once the security reaches a specified stop price. It is commonly used to limit losses or protect profits on an existing position. Once triggered, the order executes at the next available market price, which may differ from the stop price.

## Endpoint

```
POST /v1.0/accounts/{accountId}/orders
```

## Request Body

```json
{
  "Symbol": "AAPL",
  "Side": "Sell",
  "Type": "Stop",
  "StopPrice": 160.00,
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
| `Type` | string | Yes | Must be `Stop` for this order type. |
| `StopPrice` | number | Yes | The trigger price. When the market reaches this price, the stop order converts to a market order. |
| `Quantity` | number | Yes | Number of shares to trade. |
| `TimeInForce` | string | Yes | `Day`, `GTC`, `IOC`, `FOK`, etc. |
| `Exchange` | string | No | Routing destination. `Auto` lets the system choose the best venue. |

## Important Notes

- A sell stop order triggers when the market price drops to or below the `StopPrice`.
- A buy stop order triggers when the market price rises to or above the `StopPrice`.
- Once triggered, the order becomes a market order and is subject to the same execution risks as any market order (slippage in volatile conditions).
- Stop orders do not guarantee a specific execution price. If the market gaps past your stop price, the fill may be significantly worse than expected.
- For price-protected exits, consider a stop-limit order instead.

## cURL Example

```bash
curl -X POST "https://api.autoshares.dev/v1.0/accounts/{accountId}/orders" \
  -H "Authorization: Bearer {accessToken}" \
  -H "Content-Type: application/json" \
  -d '{
    "Symbol": "AAPL",
    "Side": "Sell",
    "Type": "Stop",
    "StopPrice": 160.00,
    "Quantity": 100,
    "TimeInForce": "Day",
    "Exchange": "Auto"
  }'
```
