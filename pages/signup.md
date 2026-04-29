# Get API Access

Complete this form to create your API sandbox account. You'll receive credentials to start integrating with the AutoShares Trading API immediately.

<div id="signup-form-container"></div>

## What Happens Next

After submitting the form:

1. **Your sandbox account is created** on demo.autoshares.com with the email and password you provided
2. **You'll receive your Et-App-Key** via email within minutes
3. **Start making API calls** immediately using your credentials

## Your First API Call

Once you have your credentials, authenticate and get a token:

```bash
curl -X POST \
  -H "Accept: application/json" \
  -H "Et-App-Key: YOUR_APP_KEY" \
  -H "Username: YOUR_EMAIL" \
  -H "Password: YOUR_PASSWORD" \
  "https://api.autoshares.dev/token"
```

## Sandbox Features

| Feature | Included |
|---------|----------|
| Trade Stocks, Options, ETFs, Mutual Funds | Simulated |
| All 103 API endpoints | Full access |
| WebSocket streaming | 15-min delayed |
| Order placement and management | Paper trading |
| Historical chart data | Full access |
| SDK support (Python, JS, C#, Go) | Full access |

## Need Production Access?

When you're ready to go live with real trading, contact **support@autoshares.com** to upgrade to production credentials.
