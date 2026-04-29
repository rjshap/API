# Limit Order

A limit order lets you specify the maximum price you are willing to pay (for buys) or the minimum price you are willing to accept (for sells). The order will only execute at the limit price or better, giving you full control over the execution price. The trade-off is that the order may not fill if the market never reaches your price.

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
| `Type` | string | Yes | Must be `Limit` for this order type. |
| `Price` | number | Yes | The limit price. Buy orders fill at this price or lower; sell orders fill at this price or higher. |
| `Quantity` | number | Yes | Number of shares to trade. |
| `TimeInForce` | string | Yes | `Day`, `GTC`, `IOC`, `FOK`, etc. |
| `Exchange` | string | No | Routing destination. `Auto` lets the system choose the best venue. |

## Important Notes

- A buy limit order will only execute at the `Price` or lower.
- A sell limit order will only execute at the `Price` or higher.
- There is no guarantee of execution. If the market price never reaches your limit, the order remains open until it expires or is canceled.
- Use `GTC` (Good Till Canceled) for `TimeInForce` if you want the order to persist across trading sessions.
- Limit orders are recommended for illiquid securities or large positions where price control matters.

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
    "Quantity": 100,
    "TimeInForce": "Day",
    "Exchange": "Auto"
  }'
```
