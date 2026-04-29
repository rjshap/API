# Trailing Stop Order

A trailing stop order sets a dynamic stop price that follows the market in a favorable direction by a specified amount or percentage. As the market price moves in your favor, the stop price adjusts automatically. If the market reverses by the trailing amount, the stop triggers and the order executes as a market order. This allows you to lock in gains while letting profits run.

## Endpoint

```
POST /v1.0/accounts/{accountId}/orders
```

## Request Body (Absolute Amount)

```json
{
  "Symbol": "AAPL",
  "Side": "Sell",
  "Type": "TrailingStop",
  "TrailingStopAmountType": "Absolute",
  "TrailingStopAmount": 5.00,
  "Quantity": 100,
  "TimeInForce": "Day",
  "Exchange": "Auto"
}
```

## Request Body (Percentage)

```json
{
  "Symbol": "AAPL",
  "Side": "Sell",
  "Type": "TrailingStop",
  "TrailingStopAmountType": "Persentage",
  "TrailingStopAmount": 3.0,
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
| `Type` | string | Yes | Must be `TrailingStop` for this order type. |
| `TrailingStopAmountType` | string | Yes | `Absolute` (fixed dollar amount) or `Persentage` (percentage of market price). |
| `TrailingStopAmount` | number | Yes | The trailing distance. For `Absolute`, this is a dollar value (e.g., `5.00`). For `Persentage`, this is a percentage (e.g., `3.0` means 3%). |
| `Quantity` | number | Yes | Number of shares to trade. |
| `TimeInForce` | string | Yes | `Day`, `GTC`, etc. |
| `Exchange` | string | No | Routing destination. `Auto` lets the system choose the best venue. |

## Important Notes

- For a sell trailing stop: the stop price trails below the highest market price reached since the order was placed. If the stock drops by the trailing amount from its peak, the order triggers.
- For a buy trailing stop: the stop price trails above the lowest market price reached. If the stock rises by the trailing amount from its trough, the order triggers.
- Once triggered, the order becomes a market order and executes at the next available price.
- The `TrailingStopAmountType` value `Persentage` uses this exact spelling in the API.
- Trailing stops are popular for trend-following strategies where you want to stay in a position as long as the trend continues.

## cURL Example

```bash
curl -X POST "https://api.autoshares.dev/v1.0/accounts/{accountId}/orders" \
  -H "Authorization: Bearer {accessToken}" \
  -H "Content-Type: application/json" \
  -d '{
    "Symbol": "AAPL",
    "Side": "Sell",
    "Type": "TrailingStop",
    "TrailingStopAmountType": "Absolute",
    "TrailingStopAmount": 5.00,
    "Quantity": 100,
    "TimeInForce": "Day",
    "Exchange": "Auto"
  }'
```
