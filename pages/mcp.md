# MCP Server (AI Integration)

AutoShares provides a hosted MCP (Model Context Protocol) server that gives AI assistants **read-only** access to the AutoShares Trading API for documentation, lookups, and account inspection. Works with Claude Code, Claude Desktop, ChatGPT, and any MCP-compatible client.

> **Security model:** the hosted MCP is read-only by design. Authentication and order placement are intentionally **not** exposed — apps that need those must integrate with the AutoShares / ETNA API directly. See [Security](#security) below.

## Hosted MCP Server

Connect to our hosted MCP server — no installation required:

```
https://mcp.autoshares.dev/mcp
```

### 9 Read-Only Tools

| Tool | Description |
|------|-------------|
| `lookup_symbol` | Search for stocks, ETFs, options by ticker or name |
| `get_positions` | Get all current positions for an account |
| `get_orders` | Get all orders (working, filled, cancelled) |
| `get_account_info` | Account balances, buying power, margin details |
| `get_option_chain` | Full option chain for a symbol |
| `get_chart` | OHLCV chart data with configurable period/range |
| `get_watchlists` | Get all watchlists for a user |
| `get_news` | News articles for a security |
| `ask_api_docs` | Ask questions about the API (AI-powered) |

### Tools NOT Available on the Hosted Server

The following operations are intentionally **not** exposed on the hosted MCP for security reasons:

| Tool | Why It's Not Here | What To Do Instead |
|------|-------------------|--------------------|
| `authenticate` | Hosting a credential-validator endpoint creates a password-stuffing surface. | Authenticate against ETNA's `/token` endpoint directly from your app, then pass the resulting Bearer token to the MCP tools. |
| `place_order` | Write operations should not flow through a public docs/AI endpoint. | Call `POST /v1.0/accounts/{id}/orders` directly from your authenticated client. |
| `cancel_order` | Same reason — writes go direct. | Call `DELETE /v1.0/accounts/{id}/orders/{orderId}` directly. |

If you need these capabilities for an internal AI workflow, [run the self-hosted server](#self-hosted-option) where you control the trust boundary.

## Calling the Tools

Every authenticated tool requires you to pass:

- `baseUrl` — must be one of: `https://api.autoshares.dev`, `https://api.autoshares.com`, or your assigned ETNA sandbox host. Other hosts are rejected.
- `appKey` — your ETNA `Et-App-Key`
- `token` — a valid Bearer token (60-minute lifetime; refresh via direct ETNA call)

### Response Shape

All tool responses are wrapped in a flat envelope so internal upstream errors cannot leak through:

```json
{
  "ok": true,
  "status": 200,
  "body": { /* tool-specific payload */ }
}
```

Errors return:

```json
{
  "ok": false,
  "status": 401,
  "body": { "code": "UNAUTHORIZED", "message": "Token expired" }
}
```

## Connect with Claude Code

Add to your project's `.mcp.json`:

```json
{
  "mcpServers": {
    "autoshares": {
      "type": "url",
      "url": "https://mcp.autoshares.dev/mcp"
    }
  }
}
```

Then in Claude Code you can say:

- *"Look up AAPL and show me my current positions"* (you'll be asked for your token + account ID)
- *"Get the option chain for TSLA"*
- *"What's in my watchlist?"*
- *"Pull a 30-day chart for SPY"*

## Connect with Claude Desktop

In Claude Desktop settings, add a custom MCP server:

```json
{
  "mcpServers": {
    "autoshares": {
      "type": "url",
      "url": "https://mcp.autoshares.dev/mcp"
    }
  }
}
```

## Self-Hosted Option

For full control — including write operations like `place_order` and `cancel_order` — run the MCP server locally:

```bash
git clone https://github.com/AutoSharesAPI/mcp-server
cd mcp-server
npm install && npm run build
cp .env.example .env
# Edit .env with your credentials
node dist/index.js
```

The self-hosted version exposes all 30+ tools (including RQD clearing endpoints, order placement, and cancellation). Because you operate the host, you control the trust boundary and can decide which clients reach which tools.

## Example: Read-Only AI Workflow

```
You: "Look up AAPL, show me my positions, and pull a 30-day chart"

AI: 1. Calls lookup_symbol("AAPL") → confirms security ID
    2. Calls get_positions(accountId) → returns current holdings
    3. Calls get_chart(symbol="AAPL", days=30) → returns OHLCV data
    4. Summarizes the position size, cost basis, and recent price action
```

For order placement, the AI must hand off to your authenticated trading client — it cannot place orders through the hosted MCP.

## Security

- **No credentials at rest.** The MCP server does not store usernames, passwords, app keys, or tokens. Every call passes credentials in the request, used for that single upstream call, and discarded.
- **Read-only surface.** Order placement, cancellation, and credential validation are not exposed. Removing those eliminates the most damaging classes of abuse (write operations, password stuffing).
- **Host allowlist.** Tools may only call approved AutoShares/ETNA hosts. The server rejects calls to arbitrary `baseUrl` values to prevent SSRF / credential exfiltration.
- **CORS allowlist.** Browser calls are accepted only from `documentation.autoshares.dev`, `autoshares.com`, `autoshares.dev`, `autosharesapi.github.io`, and approved local development origins. Other origins are blocked at the browser layer.
- **Rate limiting.** Both an in-isolate throttle and a Cloudflare WAF rule cap traffic at 30 requests / minute / IP on `/mcp`. Excess traffic returns `429 Too many requests`.
- **Sanitized errors.** Upstream error bodies are stripped before being returned. Only safe fields (`code`, short `message`) are passed through.
- **Encrypted in transit.** All traffic runs over TLS on Cloudflare's edge network.
- **Use environment variables on the client.** Never hardcode credentials. Treat the Bearer token like a session cookie — short-lived, revocable, scoped to the user.

For production trading, use the self-hosted version with your own infrastructure and your own auth gating.
