# Messari MCP

Model Context Protocol (MCP) integrations for [Messari](https://messari.io) — connecting AI agents to real-time crypto market intelligence.

## What's Here

### [`@messari/sdk-ts-mcp`](https://docs.messari.io/mcp-server/hosted)

Messari's hosted MCP server. Gives any MCP-compatible AI client (Claude Code, Cursor, Codex, etc.) direct access to Messari's data stack:

- **AI** — Chat completions trained on 30TB+ of structured and unstructured crypto data
- **Signal** — Social sentiment, token mindshare, trending narratives
- **Metrics** — Price, volume, market cap, and fundamentals for 34,000+ assets across 210+ exchanges
- **News** — Real-time crypto news aggregation
- **Research** — Institutional-grade reports, protocol deep dives, sector analysis
- **Stablecoins** — On-chain metrics, timeseries, and per-chain breakdowns for 25+ stablecoins
- **Derivatives** — Funding rates, open interest, futures volumes for 500+ assets

Full documentation: **[docs.messari.io/mcp-server/hosted](https://docs.messari.io/mcp-server/hosted)**

### [`openclaw-skill/`](./openclaw-skill/)

An [OpenClaw](https://openclaw.ai) / [ClawHub](https://clawhub.ai) skill that wraps Messari's REST API directly. Since OpenClaw does not natively support MCP, this skill provides endpoint routing, authentication guidance, and full API reference documentation so AI agents can call Messari's REST API via standard HTTP requests.

For clients that natively support MCP (Claude Code, Cursor, Codex, etc.), use the MCP server above instead.

Install via ClawHub:

```bash
clawhub install messari/messari-crypto
```

## Quick Start

### 1. Get a Messari API key

Sign up at [messari.io/api](https://messari.io/api). You'll also need Messari AI credits for the AI completion endpoints.

### 2. Choose your integration method

**For MCP-compatible clients** (Claude Code, Cursor, Codex), add this to your MCP client config:

```json
{
  "mcpServers": {
    "messari_sdk_ts_api": {
      "command": "npx",
      "args": ["-y", "@messari/sdk-ts-mcp", "--client=claude", "--tools=dynamic"],
      "env": {
        "MESSARI_SDK_API_KEY": "<YOUR_API_KEY>"
      }
    }
  }
}
```

**For OpenClaw**, install the skill and set your API key:

```bash
clawhub install messari/messari-crypto
```

The skill routes requests to `https://api.messari.io` with the `x-messari-api-key` header.

### 3. Start asking questions

```
"What's the bull case for ETH right now?"
"Which tokens are gaining mindshare this week?"
"Show me SOL's 90-day price and volume trend"
"What's the BTC perpetual funding rate?"
```

## Filtering Endpoints (MCP only)

Don't need every service? Use `--resource` flags:

```json
"args": ["-y", "@messari/sdk-ts-mcp@latest", "--resource", "'ai.*'"]
```

Available filters: `ai.*`, `signal.*`, `metrics.*`, `news.*`, `research.*`, `stablecoins.*`, `derivatives.*`

## Links

- [MCP Server Documentation](https://docs.messari.io/mcp-server/hosted)
- [Messari API](https://messari.io/api)
- [Messari](https://messari.io)
- [OpenClaw Skill on ClawHub](https://clawhub.ai/messari/messari-crypto)
