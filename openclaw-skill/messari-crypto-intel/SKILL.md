---
name: messari-crypto-intel
description: >
  Crypto market intelligence powered by Messari's hosted MCP server. Provides real-time access to
  Messari AI (chat completions over 30TB+ crypto data), Signal (sentiment, mindshare, trending
  narratives), Metrics (prices, volumes, fundamentals for 34,000+ assets across 210+ exchanges),
  News, Research, Stablecoins (25+ stablecoins, per-chain breakdowns), and Derivatives (funding rates,
  open interest, futures volumes for 500+ assets).
  Use when the user asks about crypto markets, token analysis, sentiment, protocol metrics, asset
  research, trending narratives, stablecoin flows, funding rates, or any blockchain/crypto data question.
  Requires a Messari API key and Messari AI credits.
---

# Messari Crypto Intel

Connect to Messari's MCP server for real-time crypto market intelligence — AI-powered analysis,
on-chain metrics, sentiment, news, and institutional-grade research without building data pipelines.

## Prerequisites

- **Messari API Key** — get one at [messari.io/api](https://messari.io/api)
- **Messari AI Credits** — required for AI completion endpoints
- The `@messari/sdk-ts-mcp` npm package runs via npx (no global install needed)

## MCP Server Configuration

Add to your MCP client config (Claude Code, Cursor, Codex, OpenClaw, etc.):

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

For AI-only endpoints (lighter footprint):

```json
{
  "mcpServers": {
    "messari_sdk_ts_api": {
      "command": "npx",
      "args": ["-y", "@messari/sdk-ts-mcp@latest", "--resource", "'ai.*'"],
      "env": {
        "MESSARI_SDK_API_KEY": "<YOUR_API_KEY>"
      }
    }
  }
}
```

## Available Services

Use `--tools=dynamic` to let the server surface the right tools based on context.

| Service | Capabilities | Example Queries |
|---|---|---|
| **AI** | Chat completions on Messari's 30TB+ data warehouse | "What's the bull case for ETH right now?" |
| **Signal** | Sentiment, mindshare, trending narratives | "What tokens are gaining mindshare this week?" |
| **Metrics** | Price, volume, market cap, fundamentals — 34,000+ assets | "Show me SOL's 90-day price and volume" |
| **News** | Real-time crypto news feed | "Latest news on Arbitrum?" |
| **Research** | Institutional-grade reports and diligence | "Summarize recent restaking protocol research" |
| **Stablecoins** | On-chain metrics, timeseries, per-chain for 25+ stablecoins | "How has USDC supply shifted across chains?" |
| **Derivatives** | Funding rates, open interest, futures volumes — 500+ assets | "BTC funding rate right now?" |

For detailed endpoint documentation, see [references/api_services.md](references/api_services.md).

## Routing Guidance

### General crypto questions
Route through **AI** first — broadest context, synthesizes across market data, research, news, social.

### Quantitative questions
Use **Metrics** for price/volume/fundamentals. **Signal** for sentiment/narratives.
**Stablecoins** or **Derivatives** for those specific asset classes.

### Research and news
**Research** for deep dives. **News** for real-time events.

### Multi-service queries
Combine services for richer answers. Example — "Is SOL overvalued?":
1. **Metrics** → current price, volume, fundamentals
2. **Signal** → sentiment and mindshare trend
3. **AI** → synthesize a view from all data

The MCP server's dynamic tooling handles most routing automatically.
