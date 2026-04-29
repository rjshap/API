# API Overview

The AutoShares Trading API is organized by resource categories. Each section covers a different set of functionality — from authentication to order placement to streaming market data.

Each endpoint is documented with parameters, response schemas, code examples in 5 languages, and an interactive Try It panel.

<div id="api-overview-cards"></div>

## API Architecture

All API requests follow this pattern:

```
{METHOD} https://api.autoshares.dev/v1.0/{resource}
```

**Required headers on every request:**

| Header | Description |
|--------|-------------|
| `Et-App-Key` | Your application key (provided during onboarding) |
| `Authorization` | `Bearer {token}` from the `/token` endpoint |
| `Accept` | `application/json` |

## Rate Limits

| Category | Limit | Window |
|----------|-------|--------|
| Authentication | 10 requests | Per minute |
| Orders | 50 requests | Per minute |
| Queries (positions, orders, securities) | 120 requests | Per minute |
| Securities lookup | 200 requests | Per minute |

## Need Help?

- Use the **Ask AI** button in the header for instant answers
- Check the **Error Codes** guide for troubleshooting
- Contact **support@autoshares.com** for integration support
