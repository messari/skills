---
name: messari-api
description: >
  Queries Messari's REST API for crypto market data across 14 services: AI chat completions,
  asset metrics and prices, social sentiment and mindshare, news, institutional research,
  stablecoins, exchanges, L1/L2 networks, DeFi protocols, token unlocks, fundraising rounds,
  governance intel, trending topics, and X/Twitter user analytics. Covers 34,000+ assets and
  210+ exchanges. Use when answering questions about crypto markets, token analysis, sentiment,
  on-chain metrics, protocol data, fundraising, or any blockchain data question.
---

# Messari REST API

Crypto market intelligence across 14 data services via Messari's REST API.

## Authentication

Every request requires the `x-messari-api-key` header. Obtain a key at [messari.io/api](https://messari.io/api). If no key is available, ask the user to provide one before making requests.

**Base URL:** `https://api.messari.io`

```
x-messari-api-key: <API_KEY>
```

All endpoints accept and return JSON. Use `Content-Type: application/json` for POST requests.

## Service Routing

| User is asking about... | Route to | Base path |
|---|---|---|
| General crypto question, synthesis, "what do you think about X" | **AI** | `/ai/` |
| In-depth research report with citations (async, 5-10 min) | **Deep Research** | `/ai/v1/deep-research` |
| Price, volume, market cap, ROI, ATH, performance comparison | **Metrics** | `/metrics/v2/` |
| Trading pair data, market-specific price and volume | **Markets** | `/metrics/v1/markets` |
| Sentiment, mindshare, trending tokens, social buzz | **Signal** | `/signal/v1/` |
| Headlines, recent events, breaking news | **News** | `/news/v1/` |
| Analyst reports, deep dives, sector overviews | **Research** | `/research/v1/` |
| Stablecoin supply, flows, chain breakdowns | **Stablecoins** | `/stablecoins/v2/` |
| Exchange volumes, exchange comparisons | **Exchanges** | `/exchanges/v2/` |
| L1/L2 network activity, fees, active addresses | **Networks** | `/networks/v2/` |
| DeFi protocols, TVL, lending, DEX volume | **Protocols** | `/protocols/v2/` |
| Token unlocks, vesting schedules | **Token Unlocks** | `/token-unlocks/v1/` |
| Fundraising rounds, investors, VC activity, M&A | **Fundraising** | `/fundraising/v1/` |
| Governance events, protocol upgrades | **Intel** | `/intel/v1/` |
| Trending narratives, topic momentum | **Topics** | `/topics/v1/` |
| Crypto influencers, X/Twitter accounts | **X-Users** | `/signal/v1/x-users/` |
| Bulk data download, large historical datasets, CSV/JSONL export | **Bulk Data** | `/bulk/v1/` |
| Trending questions the community is asking | **Trending Questions** | `/ai/v1/questions/` |

For full endpoint details, parameters, and examples, see [references/endpoints.md](references/endpoints.md).

## Multi-Service Queries

Many questions benefit from combining services. Follow this workflow:

```
Multi-Service Query Checklist:
- [ ] Identify which services are relevant to the question
- [ ] Query each service for its piece of the answer
- [ ] If any request fails, retry once then note the gap
- [ ] Synthesize results into a unified answer
- [ ] Optionally route through AI service for final synthesis
```

**Examples:**

- **"Is SOL overvalued?"** → Metrics (price, fundamentals) + Signal (sentiment) + Token Unlocks (supply pressure) + AI (synthesize)
- **"Due diligence on Eigen Layer"** → Research (reports) + Metrics (fundamentals) + Fundraising (investors) + Intel (governance) + AI (synthesize)
- **"Trending narratives this week"** → Topics (trending classes) + Signal (mindshare gainers) + News (related headlines)
- **"Compare TVL across lending protocols"** → Protocols/lending (metrics) + Networks (chain context)

When in doubt, start with the **AI** service — it draws from all other sources and provides the broadest context.

## Common Parameters

Most GET endpoints share these query parameters:

- `limit`, `page` — pagination
- `start`, `end` — date range (ISO 8601)
- Asset filters: `assetSlugs` (comma-separated slugs like `bitcoin,ethereum`) or `assetIds`
- `metrics` — specific metrics to return
- `interval` — timeseries interval (`1d`, `1w`)

## Quick Examples

### AI Chat Completion

```bash
curl -X POST "https://api.messari.io/ai/v1/chat/completions" \
  -H "x-messari-api-key: $MESSARI_API_KEY" \
  -H "Content-Type: application/json" \
  -d '{"messages": [{"role": "user", "content": "What is the bull case for ETH right now?"}]}'
```

### Asset Metrics

```bash
curl "https://api.messari.io/metrics/v2/assets?assetSlugs=bitcoin,ethereum" \
  -H "x-messari-api-key: $MESSARI_API_KEY"
```

### Signal Mindshare Gainers

```bash
curl "https://api.messari.io/signal/v1/assets/gainers-losers?type=mindshare&limit=10" \
  -H "x-messari-api-key: $MESSARI_API_KEY"
```

### News Feed

```bash
curl "https://api.messari.io/news/v1/news/feed?limit=20" \
  -H "x-messari-api-key: $MESSARI_API_KEY"
```

## Notes

- The **AI** and **Deep Research** endpoints require Messari AI credits (a paid usage quota managed at [messari.io/account](https://messari.io/account)). All other endpoints work with just the API key.
- Deep Research jobs are asynchronous — submit a query, then poll for results (5-10 min). Each report costs 500 credits.
- The **Bulk Data** service requires an appropriate subscription tier for the requested dataset.
