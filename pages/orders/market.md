# Market Order

A market order is the simplest order type. It executes immediately at the best available price in the current market. Because it prioritizes speed of execution over price, the fill price may differ from the last quoted price, especially in fast-moving or illiquid markets.

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
| `Type` | string | Yes | Must be `Market` for this order type. |
| `Quantity` | number | Yes | Number of shares to trade (whole number). |
| `TimeInForce` | string | Yes | `Day`, `GTC` (Good Till Canceled), `IOC`, `FOK`, etc. |
| `Exchange` | string | No | Routing destination. `Auto` lets the system choose the best venue. |

## Important Notes

- Market orders have no price protection. The fill price depends entirely on current market conditions.
- During high volatility, the execution price may be significantly different from the last displayed quote.
- Market orders submitted outside of regular trading hours will queue until the market opens, unless extended hours trading is enabled.
- `TimeInForce` of `Day` means the order expires at the end of the current trading session if not filled.

## cURL Example

```bash
curl -X POST "https://api.autoshares.dev/v1.0/accounts/{accountId}/orders" \
  -H "Authorization: Bearer {accessToken}" \
  -H "Content-Type: application/json" \
  -d '{
    "Symbol": "AAPL",
    "Side": "Buy",
    "Type": "Market",
    "Quantity": 100,
    "TimeInForce": "Day",
    "Exchange": "Auto"
  }'
```
