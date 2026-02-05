# Messari Crypto Intel

**Real-time crypto market intelligence for AI agents — powered by Messari's MCP server.**

Drop Messari's entire crypto data stack into your AI workflow. This skill connects your agent to Messari's hosted MCP server, giving it access to AI-powered analysis, on-chain metrics, social sentiment, institutional research, news, and derivatives data across 34,000+ crypto assets.

No data pipelines. No custom integrations. Just plug in your API key and ask questions.

---

## What You Get

**Messari AI** — Chat completions trained on 30TB+ of structured and unstructured crypto data. Market data, fundraising rounds, network metrics, research reports, newsletters, podcasts, and curated news — all synthesized in real time.

**Signal** — Social sentiment scoring, token mindshare tracking, trending narrative detection, and influencer activity monitoring. Know what the market is talking about before it moves.

**Metrics** — Price, volume, market cap, and fundamental metrics for 34,000+ assets across 210+ exchanges. 175+ filterable metrics for screening opportunities.

**News** — Real-time crypto news aggregation. Breaking events, project updates, regulatory developments.

**Research** — Institutional-grade reports from Messari's analyst team. Sector deep dives, protocol diligence, quarterly reviews, governance analysis.

**Stablecoins** — On-chain metrics, historical timeseries, and per-chain breakdowns for 25+ stablecoins. Track supply shifts, flow patterns, and market share.

**Derivatives** — Funding rates, open interest, futures volumes, and liquidation data for 500+ assets.

---

## Requirements

- A **Messari API key** ([get one here](https://messari.io/api))
- **Messari AI credits** for AI completion endpoints
- Any MCP-compatible client (Claude Code, Cursor, Codex, OpenClaw, etc.)

---

## Quick Start

### 1. Install the skill

```bash
clawhub install messari/messari-crypto-intel
```

### 2. Configure the MCP server

Add to your MCP client config:

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

### 3. Start asking questions

- "What's the bull case for ETH right now?"
- "Which tokens are gaining mindshare this week?"
- "Show me SOL's 90-day price and volume trend"
- "How has USDC supply shifted across chains this quarter?"
- "What's the BTC perpetual funding rate?"
- "Summarize the latest research on restaking protocols"

---

## How It Works

This skill extends Messari's existing hosted MCP server (`@messari/sdk-ts-mcp`). It does **not** rebuild or duplicate any MCP tools. Instead, it provides:

1. **MCP server configuration** — ready-to-paste config for any MCP client
2. **Service routing guidance** — helps your agent pick the right Messari endpoint (AI vs. Metrics vs. Signal, etc.) based on the query type
3. **Reference documentation** — detailed breakdown of each API service so your agent knows what's available

The heavy lifting is done by Messari's MCP server. This skill is the onboarding layer that makes it work smoothly inside an AI agent.

---

## Filtering Endpoints

Don't need every service? Use resource filters for a lighter footprint:

```json
"args": ["-y", "@messari/sdk-ts-mcp@latest", "--resource", "'ai.*'"]
```

Available filters: `ai.*`, `signal.*`, `metrics.*`, `news.*`, `research.*`, `stablecoins.*`, `derivatives.*`

---

## Example Workflows

**Portfolio check**: "How are my top 5 holdings performing this week?" → Metrics pulls price/volume, Signal adds sentiment context, AI synthesizes

**Narrative tracking**: "What are the trending crypto narratives right now?" → Signal identifies trending topics, Research provides depth

**Due diligence**: "Give me a research summary on Eigen Layer" → Research pulls analyst reports, Metrics adds fundamental data, AI synthesizes

**Stablecoin monitoring**: "Show me USDT vs USDC market share over the last 6 months" → Stablecoins service pulls per-chain timeseries data

---

## Links

- [Messari API](https://messari.io/api)
- [MCP Server Docs](https://docs.messari.io/mcp-server/overview)
- [Messari](https://messari.io)

---

*Built by [Messari](https://messari.io). Open-source skill for the OpenClaw ecosystem.*
