# SDKs & Libraries

## Direct API Integration

The AutoShares Trading API is a standard REST API — you can call it from any language using HTTP requests. Below are production-ready client wrappers for the most common languages.

### Python

No package install needed — uses the built-in `requests` library.

```python
import requests
import os

class AutoSharesAPI:
    """AutoShares Trading API client for Python."""

    def __init__(self, base_url=None, app_key=None):
        self.base_url = base_url or os.environ["AUTOSHARES_BASE_URL"]
        self.app_key = app_key or os.environ["AUTOSHARES_APP_KEY"]
        self.token = None

    def _headers(self):
        h = {"Accept": "application/json", "Et-App-Key": self.app_key}
        if self.token:
            h["Authorization"] = f"Bearer {self.token}"
        return h

    def authenticate(self, username=None, password=None):
        """Get a Bearer token (valid 60 minutes)."""
        username = username or os.environ["AUTOSHARES_USER"]
        password = password or os.environ["AUTOSHARES_PASS"]
        r = requests.post(
            f"{self.base_url}/token",
            headers={
                "Accept": "application/json",
                "Et-App-Key": self.app_key,
                "Username": username,
                "Password": password,
            },
        )
        r.raise_for_status()
        data = r.json()
        if data.get("State") != "Succeeded":
            raise Exception(f"Auth failed: {data}")
        self.token = data["Token"]
        return self.token

    def lookup(self, symbol, count=5):
        """Search for a security by ticker or name."""
        r = requests.get(
            f"{self.base_url}/v1.0/equities/lookup",
            headers=self._headers(),
            params={"symbol": symbol, "count": count},
        )
        r.raise_for_status()
        return r.json()

    def get_positions(self, account_id):
        """Get all positions for an account."""
        r = requests.get(
            f"{self.base_url}/v1.0/accounts/{account_id}/positions",
            headers=self._headers(),
        )
        r.raise_for_status()
        return r.json()

    def get_orders(self, account_id):
        """Get all orders for an account."""
        r = requests.get(
            f"{self.base_url}/v1.0/accounts/{account_id}/orders",
            headers=self._headers(),
        )
        r.raise_for_status()
        return r.json()

    def place_order(self, account_id, order):
        """Place a new order. Returns order details."""
        r = requests.post(
            f"{self.base_url}/v1.0/accounts/{account_id}/orders",
            headers={**self._headers(), "Content-Type": "application/json"},
            json=order,
        )
        r.raise_for_status()
        return r.json()

    def cancel_order(self, account_id, order_id):
        """Cancel a working order."""
        r = requests.delete(
            f"{self.base_url}/v1.0/accounts/{account_id}/orders/{order_id}",
            headers=self._headers(),
        )
        r.raise_for_status()
        return r.json()

    def get_option_chain(self, symbol):
        """Get option chain for a symbol."""
        r = requests.get(
            f"{self.base_url}/v1.0/options/optionChain/{symbol}",
            headers=self._headers(),
        )
        r.raise_for_status()
        return r.json()

    def get_chart(self, symbol, period=1440, start=None, end=None):
        """Get OHLCV chart data."""
        import time
        end = end or int(time.time())
        start = start or end - (30 * 86400)
        r = requests.put(
            f"{self.base_url}/v1.0/history/symbols",
            headers={**self._headers(), "Content-Type": "application/json"},
            json={"Symbol": symbol, "Period": period,
                  "StartDate": start, "EndDate": end},
        )
        r.raise_for_status()
        return r.json()


# ── Usage ──
api = AutoSharesAPI()
api.authenticate()

# Search
results = api.lookup("AAPL")
print(f"Found: {results[0]['Symbol']} — {results[0]['Description']}")

# Place a limit order
order = api.place_order("YOUR_ACCOUNT_ID", {
    "Symbol": "AAPL",
    "Type": "Limit",
    "Side": "Buy",
    "Quantity": 10,
    "Price": 180.00,
})
print(f"Order #{order['Id']} — {order['Status']}")

# OTOCO bracket order
bracket = api.place_order("YOUR_ACCOUNT_ID", {
    "Type": "OneTriggerOneCancelOther",
    "Symbol": "",
    "Legs": [
        {"Symbol": "AAPL", "Type": "Limit", "Side": "Buy",
         "Price": 180, "Quantity": 100},
        {"Symbol": "AAPL", "Type": "Limit", "Side": "Sell",
         "Price": 195, "Quantity": 100},
        {"Symbol": "AAPL", "Type": "Stop", "Side": "Sell",
         "StopPrice": 170, "Quantity": 100},
    ],
})
```

### JavaScript / TypeScript

Works in Node.js and browsers (via proxy).

```javascript
class AutoSharesAPI {
  constructor(baseUrl, appKey) {
    this.baseUrl = baseUrl || process.env.AUTOSHARES_BASE_URL;
    this.appKey = appKey || process.env.AUTOSHARES_APP_KEY;
    this.token = null;
  }

  async authenticate(username, password) {
    const res = await fetch(`${this.baseUrl}/token`, {
      method: "POST",
      headers: {
        Accept: "application/json",
        "Et-App-Key": this.appKey,
        Username: username || process.env.AUTOSHARES_USER,
        Password: password || process.env.AUTOSHARES_PASS,
      },
    });
    const data = await res.json();
    if (data.State !== "Succeeded") throw new Error(`Auth failed: ${data.Reason}`);
    this.token = data.Token;
    return this.token;
  }

  async request(method, path, body) {
    const headers = {
      Accept: "application/json",
      "Et-App-Key": this.appKey,
      Authorization: `Bearer ${this.token}`,
    };
    if (body) headers["Content-Type"] = "application/json";
    const res = await fetch(`${this.baseUrl}${path}`, {
      method,
      headers,
      body: body ? JSON.stringify(body) : undefined,
    });
    if (!res.ok) throw new Error(`${res.status}: ${await res.text()}`);
    return res.json();
  }

  lookup(symbol) {
    return this.request("GET", `/v1.0/equities/lookup?symbol=${symbol}&count=5`);
  }
  getPositions(accountId) {
    return this.request("GET", `/v1.0/accounts/${accountId}/positions`);
  }
  getOrders(accountId) {
    return this.request("GET", `/v1.0/accounts/${accountId}/orders`);
  }
  placeOrder(accountId, order) {
    return this.request("POST", `/v1.0/accounts/${accountId}/orders`, order);
  }
  cancelOrder(accountId, orderId) {
    return this.request("DELETE", `/v1.0/accounts/${accountId}/orders/${orderId}`);
  }
  getOptionChain(symbol) {
    return this.request("GET", `/v1.0/options/optionChain/${symbol}`);
  }
}

// ── Usage ──
const api = new AutoSharesAPI();
await api.authenticate();

const positions = await api.getPositions("YOUR_ACCOUNT_ID");
console.log(`${positions.length} positions`);

const order = await api.placeOrder("YOUR_ACCOUNT_ID", {
  Symbol: "AAPL", Type: "Market", Side: "Buy", Quantity: 10,
});
console.log(`Order #${order.Id} — ${order.Status}`);
```

### C#

```csharp
using System.Net.Http;
using System.Text;
using System.Text.Json;

public class AutoSharesAPI
{
    private readonly HttpClient _client = new();
    private readonly string _baseUrl;
    private readonly string _appKey;
    private string _token;

    public AutoSharesAPI(string baseUrl, string appKey)
    {
        _baseUrl = baseUrl;
        _appKey = appKey;
    }

    public async Task Authenticate(string username, string password)
    {
        var req = new HttpRequestMessage(HttpMethod.Post, $"{_baseUrl}/token");
        req.Headers.Add("Accept", "application/json");
        req.Headers.Add("Et-App-Key", _appKey);
        req.Headers.Add("Username", username);
        req.Headers.Add("Password", password);

        var res = await _client.SendAsync(req);
        var json = JsonDocument.Parse(await res.Content.ReadAsStringAsync());
        _token = json.RootElement.GetProperty("Token").GetString();
    }

    private HttpRequestMessage CreateRequest(HttpMethod method, string path)
    {
        var req = new HttpRequestMessage(method, $"{_baseUrl}{path}");
        req.Headers.Add("Accept", "application/json");
        req.Headers.Add("Et-App-Key", _appKey);
        req.Headers.Add("Authorization", $"Bearer {_token}");
        return req;
    }

    public async Task<string> GetPositions(string accountId)
    {
        var req = CreateRequest(HttpMethod.Get, $"/v1.0/accounts/{accountId}/positions");
        var res = await _client.SendAsync(req);
        return await res.Content.ReadAsStringAsync();
    }

    public async Task<string> PlaceOrder(string accountId, object order)
    {
        var req = CreateRequest(HttpMethod.Post, $"/v1.0/accounts/{accountId}/orders");
        req.Content = new StringContent(
            JsonSerializer.Serialize(order), Encoding.UTF8, "application/json");
        var res = await _client.SendAsync(req);
        return await res.Content.ReadAsStringAsync();
    }
}
```

### Go

```go
package autoshares

import (
	"bytes"
	"encoding/json"
	"fmt"
	"io"
	"net/http"
)

type Client struct {
	BaseURL string
	AppKey  string
	Token   string
}

func (c *Client) Authenticate(username, password string) error {
	req, _ := http.NewRequest("POST", c.BaseURL+"/token", nil)
	req.Header.Set("Accept", "application/json")
	req.Header.Set("Et-App-Key", c.AppKey)
	req.Header.Set("Username", username)
	req.Header.Set("Password", password)

	resp, err := http.DefaultClient.Do(req)
	if err != nil {
		return err
	}
	defer resp.Body.Close()

	var result map[string]interface{}
	json.NewDecoder(resp.Body).Decode(&result)
	c.Token = result["Token"].(string)
	return nil
}

func (c *Client) Request(method, path string, body interface{}) ([]byte, error) {
	var bodyReader io.Reader
	if body != nil {
		b, _ := json.Marshal(body)
		bodyReader = bytes.NewReader(b)
	}
	req, _ := http.NewRequest(method, c.BaseURL+path, bodyReader)
	req.Header.Set("Accept", "application/json")
	req.Header.Set("Et-App-Key", c.AppKey)
	req.Header.Set("Authorization", "Bearer "+c.Token)
	if body != nil {
		req.Header.Set("Content-Type", "application/json")
	}
	resp, err := http.DefaultClient.Do(req)
	if err != nil {
		return nil, err
	}
	defer resp.Body.Close()
	return io.ReadAll(resp.Body)
}

func (c *Client) GetPositions(accountID string) ([]byte, error) {
	return c.Request("GET", "/v1.0/accounts/"+accountID+"/positions", nil)
}

func (c *Client) PlaceOrder(accountID string, order map[string]interface{}) ([]byte, error) {
	return c.Request("POST", "/v1.0/accounts/"+accountID+"/orders", order)
}
```

### cURL

No library needed — works from any terminal.

```bash
# Set your credentials
export BASE_URL="https://api.autoshares.dev"
export APP_KEY="your-et-app-key"
export USERNAME="your-username"
export PASSWORD="your-password"

# Authenticate
TOKEN=$(curl -s -X POST "$BASE_URL/token" \
  -H "Accept: application/json" \
  -H "Et-App-Key: $APP_KEY" \
  -H "Username: $USERNAME" \
  -H "Password: $PASSWORD" | jq -r '.Token')

# Get positions
curl -s "$BASE_URL/v1.0/accounts/YOUR_ACCOUNT_ID/positions" \
  -H "Authorization: Bearer $TOKEN" \
  -H "Et-App-Key: $APP_KEY" | jq

# Place an order
curl -s -X POST "$BASE_URL/v1.0/accounts/YOUR_ACCOUNT_ID/orders" \
  -H "Authorization: Bearer $TOKEN" \
  -H "Et-App-Key: $APP_KEY" \
  -H "Content-Type: application/json" \
  -d '{"Symbol":"AAPL","Type":"Limit","Side":"Buy","Quantity":10,"Price":180}' | jq
```

## MCP Server (AI Integration)

AutoShares provides an MCP (Model Context Protocol) server that gives AI assistants direct access to the trading API. Works with Claude, ChatGPT, and any MCP-compatible client.

```bash
# Clone and build
git clone https://github.com/AutoSharesAPI/mcp-server
cd mcp-server
npm install && npm run build

# Configure credentials (never hardcode)
cp .env.example .env
# Edit .env with your credentials

# Add to Claude Code
# In your project's .mcp.json:
{
  "mcpServers": {
    "autoshares": {
      "command": "node",
      "args": ["path/to/mcp-server/dist/index.js"]
    }
  }
}
```

30+ tools available: `lookup_symbol`, `place_order`, `get_positions`, `get_orders`, `get_chart`, `get_option_chain`, and more.

## OpenAPI Specification

Download the spec to generate your own client or import into Postman:

| Format | URL |
|--------|-----|
| **JSON** | `{baseURL}/reference/schema/v1` |
| **YAML** | [Download from GitHub](https://github.com/AutoSharesAPI/API/blob/main/openapi.yaml) |

### Import into Postman

1. Open Postman → **Import** → **Link**
2. Enter: `https://api.autoshares.dev/reference/schema/v1`
3. Postman generates a collection with all 103 endpoints
4. Set `Et-App-Key` and `Authorization` as environment variables

### Generate Your Own SDK

Use any OpenAPI code generator:

```bash
# OpenAPI Generator (Java, Ruby, PHP, Swift, Kotlin, etc.)
npx @openapitools/openapi-generator-cli generate \
  -i https://api.autoshares.dev/reference/schema/v1 \
  -g python \
  -o ./my-autoshares-sdk

# Swagger Codegen
swagger-codegen generate \
  -i https://api.autoshares.dev/reference/schema/v1 \
  -l javascript \
  -o ./my-autoshares-sdk
```

## Token Management

AutoShares API tokens expire after **60 minutes** of inactivity. For a seamless user experience, implement automatic token refresh so users are never interrupted during a trading session.

### Auto-Refresh (Python)

```python
import time
import requests

class AutoSharesSession:
    """Manages token lifecycle — auto-refreshes before expiry."""

    def __init__(self, base_url, app_key, username, password):
        self.base_url = base_url
        self.app_key = app_key
        self.username = username
        self.password = password
        self.token = None
        self.token_time = 0

    def _ensure_token(self):
        """Re-authenticate if token is older than 50 minutes."""
        if self.token and (time.time() - self.token_time) < 3000:  # 50 min
            return
        r = requests.post(f"{self.base_url}/token", headers={
            "Accept": "application/json",
            "Et-App-Key": self.app_key,
            "Username": self.username,
            "Password": self.password,
        })
        r.raise_for_status()
        data = r.json()
        if data.get("State") != "Succeeded":
            raise Exception(f"Auth failed: {data}")
        self.token = data["Token"]
        self.token_time = time.time()

    def request(self, method, path, **kwargs):
        """Make an API request with auto-refresh."""
        self._ensure_token()
        r = requests.request(method, f"{self.base_url}{path}",
            headers={
                "Authorization": f"Bearer {self.token}",
                "Et-App-Key": self.app_key,
                "Accept": "application/json",
                **kwargs.pop("headers", {}),
            }, **kwargs)
        # If 401, token expired — refresh and retry once
        if r.status_code == 401:
            self.token = None
            self._ensure_token()
            r = requests.request(method, f"{self.base_url}{path}",
                headers={
                    "Authorization": f"Bearer {self.token}",
                    "Et-App-Key": self.app_key,
                    "Accept": "application/json",
                }, **kwargs)
        return r

# Usage — token refreshes automatically, never expires during session
session = AutoSharesSession(
    base_url="https://api.autoshares.dev",
    app_key=os.environ["APP_KEY"],
    username=os.environ["USERNAME"],
    password=os.environ["PASSWORD"],
)

# These calls auto-refresh the token if needed
positions = session.request("GET", "/v1.0/accounts/123/positions").json()
order = session.request("POST", "/v1.0/accounts/123/orders",
    json={"Symbol": "AAPL", "Type": "Market", "Side": "Buy", "Quantity": 10}).json()
```

### Auto-Refresh (JavaScript)

```javascript
class AutoSharesSession {
  constructor(baseUrl, appKey, username, password) {
    this.baseUrl = baseUrl;
    this.appKey = appKey;
    this.username = username;
    this.password = password;
    this.token = null;
    this.tokenTime = 0;
  }

  async ensureToken() {
    if (this.token && (Date.now() - this.tokenTime) < 3000000) return; // 50 min
    const res = await fetch(`${this.baseUrl}/token`, {
      method: "POST",
      headers: {
        Accept: "application/json",
        "Et-App-Key": this.appKey,
        Username: this.username,
        Password: this.password,
      },
    });
    const data = await res.json();
    if (data.State !== "Succeeded") throw new Error(`Auth failed: ${data.Reason}`);
    this.token = data.Token;
    this.tokenTime = Date.now();
  }

  async request(method, path, body) {
    await this.ensureToken();
    const headers = {
      Authorization: `Bearer ${this.token}`,
      "Et-App-Key": this.appKey,
      Accept: "application/json",
    };
    if (body) headers["Content-Type"] = "application/json";
    let res = await fetch(`${this.baseUrl}${path}`, {
      method, headers,
      body: body ? JSON.stringify(body) : undefined,
    });
    // Auto-retry on 401
    if (res.status === 401) {
      this.token = null;
      await this.ensureToken();
      headers.Authorization = `Bearer ${this.token}`;
      res = await fetch(`${this.baseUrl}${path}`, {
        method, headers,
        body: body ? JSON.stringify(body) : undefined,
      });
    }
    return res.json();
  }
}

// Usage — never worry about token expiry
const session = new AutoSharesSession(
  "https://api.autoshares.dev", APP_KEY, USERNAME, PASSWORD
);
const positions = await session.request("GET", "/v1.0/accounts/123/positions");
```

### Token Lifecycle

| Event | What Happens |
|-------|-------------|
| **Login** | Token issued, valid 60 minutes |
| **API call at 50 min** | Auto-refresh: new token issued silently |
| **401 response** | Retry: re-authenticate + replay the request |
| **Logout** | Token invalidated via `POST /logout` |
| **Browser close** | Token in sessionStorage is cleared |
| **Password change** | All tokens invalidated — re-auth required |

### Best Practice: Session-Based UX

Even though tokens expire in 60 minutes, users should **never see an expiry**. Your app should:

1. **Proactive refresh** — re-authenticate at 50 minutes (before expiry)
2. **Reactive retry** — catch 401, refresh, replay the failed request
3. **Seamless to user** — no login prompts during active sessions
4. **Explicit logout only** — user clicks "Sign Out" to end the session

This gives a **session-based experience** (like Interactive Brokers) on top of token-based auth.

## Security Best Practices

- **Never hardcode credentials** — use environment variables or a secrets manager
- **Use sessionStorage** — tokens cleared when tab closes, never persisted to disk
- **Use HTTPS only** — all API endpoints require TLS
- **Proxy in production** — all browser requests go through `api.autoshares.dev`
- **Rotate credentials** — change passwords periodically and rotate app keys if compromised
- **Force re-auth** on sensitive actions (first withdrawal, password change)
- **Invalidate all tokens** on password change
