# Conditional Orders

Conditional orders let you link multiple orders together with logical relationships. AutoShares supports three conditional order types: OTO (One-Triggers-Other), OCO (One-Cancels-Other), and OTOCO (One-Triggers-One-Cancels-Other). These are essential for building automated trading workflows such as bracket orders, entry-exit pairs, and profit/stop-loss combinations.

## Endpoint

```
POST /v1.0/accounts/{accountId}/orders
```

All conditional orders use the same endpoint. The `Legs` array and order structure define the conditional behavior.

---

## OTO (One-Triggers-Other)

An OTO order consists of two legs. When the first leg executes, the second leg is automatically submitted. This is useful for setting up an entry order with a predetermined exit.

### Request Body

```json
{
  "Type": "OneTriggerOther",
  "Legs": [
    {
      "Symbol": "AAPL",
      "Side": "Buy",
      "Type": "Limit",
      "Price": 170.00,
      "Quantity": 100,
      "TimeInForce": "GTC",
      "Exchange": "Auto"
    },
    {
      "Symbol": "AAPL",
      "Side": "Sell",
      "Type": "Stop",
      "StopPrice": 160.00,
      "Quantity": 100,
      "TimeInForce": "GTC",
      "Exchange": "Auto"
    }
  ]
}
```

### Key Fields

| Field | Type | Required | Description |
|-------|------|----------|-------------|
| `Type` | string | Yes | Must be `OneTriggerOther`. |
| `Legs` | array | Yes | Exactly 2 order legs. The first leg triggers the second on fill. |

### OTO Rules

- The first leg **cannot** be a Market order, because it would execute immediately and the conditional logic would be meaningless.
- The second leg is held inactive until the first leg fills.
- If the first leg is canceled, the second leg is also canceled.

---

## OCO (One-Cancels-Other)

An OCO order consists of two legs that are both active simultaneously. When either leg executes, the other is automatically canceled. This is the standard profit-target / stop-loss combination.

### Request Body

```json
{
  "Type": "OneCancelOther",
  "Legs": [
    {
      "Symbol": "AAPL",
      "Side": "Sell",
      "Type": "Limit",
      "Price": 190.00,
      "Quantity": 100,
      "TimeInForce": "GTC",
      "Exchange": "Auto"
    },
    {
      "Symbol": "AAPL",
      "Side": "Sell",
      "Type": "Stop",
      "StopPrice": 160.00,
      "Quantity": 100,
      "TimeInForce": "GTC",
      "Exchange": "Auto"
    }
  ]
}
```

### Key Fields

| Field | Type | Required | Description |
|-------|------|----------|-------------|
| `Type` | string | Yes | Must be `OneCancelOther`. |
| `Legs` | array | Yes | Exactly 2 order legs. Both are active; filling one cancels the other. |

### OCO Rules

- **Both** legs cannot be Market orders. At least one must be a Limit, Stop, or StopLimit order.
- Both legs are submitted and active at the same time.
- When one leg fills, the other is automatically canceled.
- OCO is commonly used to bracket an existing position with a profit target (limit sell) and a stop-loss (stop sell).

---

## OTOCO (One-Triggers-One-Cancels-Other)

An OTOCO order is a 3-leg structure. The first leg is an entry order. When it fills, the remaining two legs are activated as an OCO pair. This creates a complete bracket: an entry, a profit target, and a stop-loss, all in a single order submission.

### Request Body

```json
{
  "Type": "OneTriggerOneCancelOther",
  "Legs": [
    {
      "Symbol": "AAPL",
      "Side": "Buy",
      "Type": "Limit",
      "Price": 170.00,
      "Quantity": 100,
      "TimeInForce": "GTC",
      "Exchange": "Auto"
    },
    {
      "Symbol": "AAPL",
      "Side": "Sell",
      "Type": "Limit",
      "Price": 190.00,
      "Quantity": 100,
      "TimeInForce": "GTC",
      "Exchange": "Auto"
    },
    {
      "Symbol": "AAPL",
      "Side": "Sell",
      "Type": "Stop",
      "StopPrice": 160.00,
      "Quantity": 100,
      "TimeInForce": "GTC",
      "Exchange": "Auto"
    }
  ]
}
```

### Key Fields

| Field | Type | Required | Description |
|-------|------|----------|-------------|
| `Type` | string | Yes | Must be `OneTriggerOneCancelOther`. |
| `Legs` | array | Yes | Exactly 3 order legs. Leg 1 is the trigger. Legs 2 and 3 form the OCO pair. |

### OTOCO Rules

- The first leg **cannot** be a Market order.
- Legs 2 and 3 are held inactive until the first leg fills.
- Once the first leg fills, legs 2 and 3 activate as an OCO pair: when one fills, the other is canceled.
- This is the standard bracket order pattern: buy entry + sell limit (take profit) + sell stop (stop loss).

---

## Important Notes (All Conditional Types)

- Each leg in the `Legs` array follows the same field structure as a standalone order (`Symbol`, `Side`, `Type`, `Quantity`, etc.).
- Use `GTC` for `TimeInForce` on conditional orders to avoid legs expiring before the conditional logic plays out.
- All legs should reference the same `Symbol` and `Quantity` for predictable behavior.
- If any leg is rejected (e.g., insufficient buying power), the entire conditional order may be rejected.
- Conditional orders can be canceled as a group. Canceling the parent cancels all child legs.

## cURL Example (OTOCO Bracket Order)

```bash
curl -X POST "https://{your-environment}.etnasoft.us/api/v1.0/accounts/{accountId}/orders" \
  -H "Authorization: Bearer {accessToken}" \
  -H "Content-Type: application/json" \
  -d '{
    "Type": "OneTriggerOneCancelOther",
    "Legs": [
      {
        "Symbol": "AAPL",
        "Side": "Buy",
        "Type": "Limit",
        "Price": 170.00,
        "Quantity": 100,
        "TimeInForce": "GTC",
        "Exchange": "Auto"
      },
      {
        "Symbol": "AAPL",
        "Side": "Sell",
        "Type": "Limit",
        "Price": 190.00,
        "Quantity": 100,
        "TimeInForce": "GTC",
        "Exchange": "Auto"
      },
      {
        "Symbol": "AAPL",
        "Side": "Sell",
        "Type": "Stop",
        "StopPrice": 160.00,
        "Quantity": 100,
        "TimeInForce": "GTC",
        "Exchange": "Auto"
      }
    ]
  }'
```
