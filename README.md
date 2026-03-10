# Messari Agent Skills

This repository contains [Claude](https://claude.ai) and [OpenClaw](https://openclaw.ai) skill files that wrap the full [Messari REST API](https://docs.messari.io) — giving AI agents access to crypto market data, on-chain metrics, sentiment, news, research, and more across 34,000+ assets.

For AI-powered answers that synthesize across Messari's entire data warehouse, use the [Messari MCP Server](https://docs.messari.io/mcp-server/hosted).

## Getting Started

### 1. Get an API key

Sign up at [messari.io](https://messari.io) and get an [API key](https://messari.io/api).

### 2. Choose your integration

#### MCP Server (Recommended for AI-powered answers)

The Messari MCP Server connects any MCP-compatible client to Messari's data stack, including MessariAI for synthesized, citation-backed responses.

**Server URL:** `https://mcp.messari.io/mcp`

**Claude / Claude Code:**

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

**Codex:**

Add to `~/.codex/config.toml`:

```toml
[mcp_servers.messari]
url = "https://mcp.messari.io/mcp"
```

**Cursor:**

Add to `.cursor/mcp.json` or `~/.cursor/mcp.json`:

```json
{
  "mcpServers": {
    "messari": {
      "url": "https://mcp.messari.io/mcp"
    }
  }
}
```

Full MCP documentation: [docs.messari.io/mcp-server/hosted](https://docs.messari.io/mcp-server/hosted)

#### Skills (Direct REST API access)

These skills teach your agent to call Messari's REST API directly — useful when your client doesn't support MCP, or when you want full control over which endpoints are called.

**Claude / Claude Code:**

Point your agent at the skill file:

`https://raw.githubusercontent.com/messari/skills/main/claude/messari-api/SKILL.md`

Or see [`claude/messari-api/`](./claude/messari-api/) for the full skill directory.

**OpenClaw / ClawHub:**

```bash
clawhub install messari/messari-api
```

Or see [`openclaw/messari-api/`](./openclaw/messari-api/) for the full skill directory.

## API Coverage

Both skills provide routing, authentication, and endpoint reference for these Messari API services:

- **AI** — Chat completions and deep research reports across Messari's crypto data warehouse
- **Metrics** — Price, volume, market cap, and fundamentals for 34,000+ assets
- **Signal** — Sentiment, mindshare, and trending narrative tracking
- **News** — Real-time aggregated news from 500+ sources
- **Research** — Institutional-grade analyst reports and sector deep dives
- **Stablecoins** — On-chain metrics and per-chain breakdowns for 25+ stablecoins
- **Exchanges** — Volume and metrics across 210+ exchanges
- **Networks** — L1/L2 blockchain network activity, fees, active addresses
- **Protocols** — DeFi protocol metrics (DEXs, lending, liquid staking, bridges)
- **Token Unlocks** — Vesting schedules and upcoming unlock events
- **Fundraising** — Funding rounds, investors, funds, M&A activity
- **Intel** — Governance events, protocol upgrades, project milestones
- **Topics** — Trending topic classification and timeseries
- **X-Users** — Crypto X/Twitter user metrics and influence tracking
- **Bulk Data** — Large-scale CSV/JSONL dataset downloads

### 3. Start asking questions

```
"Which assets over $1B marketcap outperformed Bitcoin over the last 3 months?"
"What are some upcoming token unlock events this month?"
"Give me the 10 most recent fundraising rounds in the AI and compute sectors."
"What are the latest headlines related to crypto regulation?"
"Which investor has been the most active in seed rounds over the last year?"
"What were the top events for the AAVE protocol during the last quarter?"
"Compare TVL across top lending protocols."
"What are the trending crypto narratives right now?"
```

## Links

- [MCP Server Documentation](https://docs.messari.io/mcp-server/hosted)
- [Messari API Documentation](https://docs.messari.io)
- [Messari API Key](https://messari.io/api)
- [Messari](https://messari.io)
