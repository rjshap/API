# Agreement Management

When building custom trading applications, you must implement agreement management. Traders are required to sign platform agreements before they can trade. This check must be performed **every time the trader logs in**.

## Workflow

1. **Check for outstanding agreements** via the API
2. **Display the signing page** if any agreements are unsigned
3. **Allow trading access** only after all agreements are signed or skipped

## Step 1: Check Outstanding Agreements

After authentication, call the agreements endpoint:

```
GET https://api.autoshares.dev/v1.0/users/@me/agreements
```

**Headers:**

| Header | Value |
|--------|-------|
| `Authorization` | `Bearer {token}` |
| `Et-App-Key` | Your application key |

**Response (agreements pending):**

```json
[
  {
    "Id": 1,
    "Type": "MarketDataAgreement",
    "Title": "NYSE Market Data Agreement",
    "CanSkip": false,
    "Version": "7.1"
  },
  {
    "Id": 2,
    "Type": "PlatformAgreement",
    "Title": "AutoShares Platform Agreement",
    "CanSkip": false,
    "Version": "3.2"
  }
]
```

**Response (no agreements pending):**

```json
[]
```

**Logic:**
- If the array has items → display the agreements page (Step 2)
- If the array is empty → allow trading access

## Step 2: Display Agreements Page

Embed the agreements signing UI in an iframe:

```html
<iframe
  id="agreements-frame"
  src="https://{agreements-url}"
  width="100%"
  height="600"
  frameborder="0">
</iframe>

<script src="https://{agreements-url}/assets/agreements.client.js"></script>
<script>
  const client = new AutoShares.AgreementsClient({
    iframe: document.getElementById('agreements-frame'),
    appKey: 'YOUR_ET_APP_KEY',
    token: 'BEARER_TOKEN',
    theme: 'Dark', // or 'Light'
  });

  client.on('action', function(event) {
    if (event.type === 'allAgreementsSignedOrSkipped') {
      // All agreements handled — redirect to trading
      window.location.href = '/trading';
    }
  });
</script>
```

## Step 3: Sign an Agreement via API

To sign an agreement programmatically (without the iframe):

```
POST https://api.autoshares.dev/v1.0/users/@me/agreements/{agreementId}/sign
```

**Headers:**

| Header | Value |
|--------|-------|
| `Authorization` | `Bearer {token}` |
| `Et-App-Key` | Your application key |
| `Content-Type` | `application/json` |

## Agreement Types

| Type | Description | Can Skip? |
|------|-------------|-----------|
| `PlatformAgreement` | AutoShares platform terms | No |
| `MarketDataAgreement` | NYSE/NASDAQ/OPRA data agreements | No |
| `MarginAgreement` | Margin account agreement | No |
| `OptionsAgreement` | Options trading agreement | No |
| `DayTradingAgreement` | Pattern day trader acknowledgment | Yes |

## Implementation Example (Python)

```python
import requests

def check_and_sign_agreements(base_url, token, app_key):
    headers = {
        "Authorization": f"Bearer {token}",
        "Et-App-Key": app_key,
        "Accept": "application/json",
    }

    # Check outstanding agreements
    r = requests.get(f"{base_url}/v1.0/users/@me/agreements", headers=headers)
    agreements = r.json()

    if not agreements:
        print("No outstanding agreements — trading access granted")
        return True

    print(f"{len(agreements)} agreements pending:")
    for a in agreements:
        print(f"  - {a['Title']} (v{a['Version']})")

        if not a.get('CanSkip', False):
            # Must sign
            sign_r = requests.post(
                f"{base_url}/v1.0/users/@me/agreements/{a['Id']}/sign",
                headers={**headers, "Content-Type": "application/json"},
            )
            print(f"    Signed: {sign_r.status_code}")

    return True
```

## Important Notes

- Agreement verification is **mandatory at every login** — do not cache the result
- Trading with unsigned agreements is **not permitted**
- Agreement versions may change — always check for the latest
- Contact **support@autoshares.com** for custom agreement branding
