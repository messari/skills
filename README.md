# Messari MCP

Model Context Protocol (MCP) integrations for [Messari](https://messari.io) — connecting AI agents to real-time crypto market intelligence.

## What's Here

### [Messari Hosted MCP Server](https://docs.messari.io/mcp-server/hosted)

Messari's hosted MCP server. Gives any MCP-compatible AI client direct access to Messari's data stack:

- **AI** — Chat completions trained on 30TB+ of structured and unstructured crypto data
- **Signal** — Social sentiment, token mindshare, trending narratives
- **Metrics** — Price, volume, market cap, and fundamentals for 34,000+ assets across 210+ exchanges
- **News** — Real-time crypto news aggregation
- **Research** — Institutional-grade reports, protocol deep dives, sector analysis
- **Stablecoins** — On-chain metrics, timeseries, and per-chain breakdowns for 25+ stablecoins
- **Derivatives** — Funding rates, open interest, futures volumes for 500+ assets

Full documentation: **[docs.messari.io/mcp-server/hosted](https://docs.messari.io/mcp-server/hosted)**

### [`openclaw-skill/`](./openclaw-skill/)

An [OpenClaw](https://openclaw.ai) / [ClawHub](https://clawhub.ai) skill that wraps Messari's REST API directly. OpenClaw does not natively support MCP, so this skill provides endpoint routing, authentication, and API reference documentation for standard HTTP requests.

## Quick Start

### 1. Get a Messari account

Sign up at [messari.io](https://messari.io). MCP clients authenticate via OAuth — you'll be guided through a login flow on first use. For the OpenClaw skill (REST API), you'll also need an [API key](https://messari.io/api).

### 2. Set up your client

#### Claude / Claude Code

Run from your terminal:

```bash
claude mcp add --transport http --scope user messari https://mcp.messari.io/mcp
```

Or add to your project's `.mcp.json`:

```json
{
  "mcpServers": {
    "messari": {
      "url": "https://mcp.messari.io/mcp"
    }
  }
}
```

#### Codex

Add to `~/.codex/config.toml`:

```toml
[mcp_servers.messari]
url = "https://mcp.messari.io/mcp"
```

#### Cursor

Add to `.cursor/mcp.json` (project-level) or `~/.cursor/mcp.json` (global):

```json
{
  "mcpServers": {
    "messari": {
      "url": "https://mcp.messari.io/mcp"
    }
  }
}
```

#### OpenClaw

OpenClaw does not support MCP. Install the skill instead:

```bash
clawhub install messari/messari-crypto
```

The skill calls `https://api.messari.io` directly with the `x-messari-api-key` header. See [`openclaw-skill/`](./openclaw-skill/) for details.

### 3. Start asking questions

```
"What's the bull case for ETH right now?"
"Which tokens are gaining mindshare this week?"
"Show me SOL's 90-day price and volume trend"
"What's the BTC perpetual funding rate?"
```

## Links

- [MCP Server Documentation](https://docs.messari.io/mcp-server/hosted)
- [Messari API](https://messari.io/api)
- [Messari](https://messari.io)
- [OpenClaw Skill on ClawHub](https://clawhub.ai/messari/messari-crypto)
