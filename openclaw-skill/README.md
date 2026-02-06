# Messari Crypto Intel

**Real-time crypto market intelligence for AI agents — powered by Messari's REST API.**

Drop Messari's entire crypto data stack into your AI workflow. This skill gives your agent direct access to Messari's REST API for AI-powered analysis, on-chain metrics, social sentiment, institutional research, news, DeFi protocol data, token unlock schedules, fundraising intelligence, and more across 34,000+ crypto assets.

No data pipelines. No custom integrations. Just plug in your API key and ask questions.

---

## What You Get

**Messari AI** — Chat completions trained on 30TB+ of structured and unstructured crypto data. Market data, fundraising rounds, network metrics, research reports, newsletters, podcasts, and curated news — all synthesized in real time.

**Signal** — Social sentiment scoring, token mindshare tracking, trending narrative detection, and influencer activity monitoring. Know what the market is talking about before it moves.

**Metrics** — Price, volume, market cap, and fundamental metrics for 34,000+ assets across 210+ exchanges. 175+ filterable metrics for screening opportunities.

**News** — Real-time crypto news aggregation. Breaking events, project updates, regulatory developments.

**Research** — Institutional-grade reports from Messari's analyst team. Sector deep dives, protocol diligence, quarterly reviews, governance analysis.

**Stablecoins** — On-chain metrics, historical timeseries, and per-chain breakdowns for 25+ stablecoins. Track supply shifts, flow patterns, and market share.

**Exchanges** — Exchange-level volume, metrics, and historical timeseries across 210+ exchanges.

**Networks** — L1/L2 blockchain network metrics and timeseries. Chain-level activity, fees, active addresses.

**Protocols** — DeFi protocol metrics across DEXs, lending, liquid staking, and interoperability bridges.

**Token Unlocks** — Vesting schedules, upcoming unlock events, and supply pressure analysis.

**Fundraising** — Funding rounds, investors, funds, organizations, projects, and M&A activity.

**Intel** — Governance events, protocol upgrades, and key project milestones.

**Topics** — Trending narrative classes and daily topic timeseries.

**X-Users** — Crypto X/Twitter user metrics and influence tracking.

---

## Requirements

- A **Messari API key** ([get one here](https://messari.io/api))
- **Messari AI credits** for AI completion endpoints

---

## Quick Start

### 1. Install the skill

```bash
clawhub install messari/messari-crypto
```

### 2. Set your API key

Set the `MESSARI_API_KEY` environment variable, or configure it through your OpenClaw client. The skill passes this key as the `x-messari-api-key` header on every request.

### 3. Start asking questions

- "What's the bull case for ETH right now?"
- "Which tokens are gaining mindshare this week?"
- "Show me SOL's 90-day price and volume trend"
- "How has USDC supply shifted across chains this quarter?"
- "What token unlocks are coming up in the next 30 days?"
- "Summarize the latest research on restaking protocols"
- "Who are the most active crypto VCs this quarter?"

---

## How It Works

This skill wraps Messari's REST API directly. It provides:

1. **REST API configuration** — base URL, authentication, and endpoint routing so your agent can call Messari's API via HTTP
2. **Service routing guidance** — helps your agent pick the right Messari endpoint (AI vs. Metrics vs. Signal, etc.) based on the query type
3. **Reference documentation** — detailed breakdown of all 14 API services with endpoint paths, methods, and parameters

Your agent makes standard HTTP requests to `https://api.messari.io` with the `x-messari-api-key` header. No MCP server, no npm packages, no local processes — just REST calls.

---

## Authentication

Every request requires the `x-messari-api-key` header:

```bash
curl "https://api.messari.io/metrics/v2/assets?assetSlugs=bitcoin" \
  -H "x-messari-api-key: $MESSARI_API_KEY"
```

---

## Example Workflows

**Portfolio check**: "How are my top 5 holdings performing this week?" → Metrics pulls price/volume, Signal adds sentiment context, AI synthesizes

**Narrative tracking**: "What are the trending crypto narratives right now?" → Topics identifies trending classes, Signal provides mindshare data, Research provides depth

**Due diligence**: "Give me a research summary on Eigen Layer" → Research pulls analyst reports, Metrics adds fundamental data, Fundraising shows investor backing, AI synthesizes

**Stablecoin monitoring**: "Show me USDT vs USDC market share over the last 6 months" → Stablecoins service pulls per-chain timeseries data

**Supply analysis**: "What major token unlocks are happening this month?" → Token Unlocks pulls upcoming events, Metrics adds current price context

**DeFi research**: "Compare TVL across top lending protocols" → Protocols pulls lending-specific metrics, Networks adds chain-level context

---

## Links

- [Messari API](https://messari.io/api)
- [Messari API Docs](https://docs.messari.io)
- [Messari](https://messari.io)

---

*Built by [Messari](https://messari.io). Open-source skill for the OpenClaw ecosystem.*
