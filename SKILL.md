# Messari — Crypto Market Intelligence for AI Agents

You are integrating with Messari, the leading crypto data platform. You have access to real-time and historical data across 34,000+ crypto assets, 210+ exchanges, and 14 specialized data services covering market metrics, social sentiment, institutional research, on-chain analytics, fundraising, governance, and more.

## Integration Paths

### Option A: MCP Server (Preferred)

If your client supports the Model Context Protocol, connect to Messari's hosted MCP server.

**Server URL:** `https://mcp.messari.io/mcp`

Requires a Messari API key. Get one at [messari.io/api](https://messari.io/api).

### Option B: REST API (Direct HTTP)

If your client does not support MCP, call Messari's REST API directly.

**Base URL:** `https://api.messari.io`

**Authentication:** Include the API key in every request:

```
x-messari-api-key: <API_KEY>
```

All endpoints accept and return JSON. Use `Content-Type: application/json` for POST requests.

**API key:** The user needs a Messari API key from [messari.io/api](https://messari.io/api). If no key is available, ask the user to provide one before making requests.

---

## Services and Endpoints

### AI Service

Chat completions trained on 30TB+ of structured and unstructured crypto data — market data, fundraising rounds, network metrics, research reports, newsletters, podcasts, and curated news.

Route general or open-ended crypto questions here first. This service synthesizes across all other data sources.

Requires Messari AI credits.

| Endpoint | Method | Description |
|---|---|---|
| `/ai/v1/chat/completions` | POST | Chat completion against Messari's crypto data warehouse |
| `/ai/openai/chat/completions` | POST | OpenAI-compatible chat completion endpoint |
| `/ai/v2/chat/completions` | POST | v2 chat completion with inline citations, related questions, and verbosity control |
| `/ai/v1/questions/trending` | GET | Suggested questions based on trending crypto topics |

**POST body:**
- `messages` — array of `{role, content}` message objects
- `stream` — boolean, enable streaming responses
- `verbosity` — `succinct`, `balanced`, or `verbose` (v2 only)
- `inline_citations` — boolean, include citation references in metadata (v2 only)
- `generate_related_questions` — integer, number of follow-up suggestions (v2 only)
- `response_format` — `markdown` or `plaintext` (v2 only)

```bash
curl -X POST "https://api.messari.io/ai/v1/chat/completions" \
  -H "x-messari-api-key: $MESSARI_API_KEY" \
  -H "Content-Type: application/json" \
  -d '{"messages": [{"role": "user", "content": "What is the bull case for ETH right now?"}]}'
```

**Deep Research** — Generate comprehensive long-form research reports asynchronously (5-10 minutes). Reports include markdown content with cited sources. Costs 500 credits per report; Enterprise teams receive 10 free/month.

| Endpoint | Method | Description |
|---|---|---|
| `/ai/v1/deep-research` | POST | Create a new deep research job |
| `/ai/v1/deep-research` | GET | List your deep research jobs |
| `/ai/v1/deep-research/{id}` | GET | Get job status; includes report when completed |
| `/ai/v1/deep-research/{id}/cancel` | POST | Cancel a queued or in-progress job |

**POST body (Create):**
- `query` — research topic or question (required)
- `instructions` — optional system instructions to guide research style
- `job_id` — optional, pass existing job ID for follow-up refinement

**Query parameters (List):**
- `limit` — max results (default 20, max 100)
- `offset` — pagination offset
- `status` — filter by: `queued`, `in_progress`, `completed`, `failed`, `cancelled`

```bash
curl -X POST "https://api.messari.io/ai/v1/deep-research" \
  -H "x-messari-api-key: $MESSARI_API_KEY" \
  -H "Content-Type: application/json" \
  -d '{"query": "What is the current state of liquid staking on Ethereum?"}'
```

---

### Metrics Service

Price, volume, market cap, and fundamental metrics for 34,000+ assets across 210+ exchanges. 175+ filterable metrics.

Route quantitative and comparative questions here — price lookups, performance comparisons, ROI, all-time highs, historical timeseries.

| Endpoint | Method | Description |
|---|---|---|
| `/metrics/v2/assets` | GET | List assets with market metrics |
| `/metrics/v2/assets/{assetId}` | GET | Detailed metrics for a specific asset |
| `/metrics/v2/assets/{assetId}/roi` | GET | ROI data for an asset |
| `/metrics/v2/assets/{assetId}/ath` | GET | All-time high data for an asset |
| `/metrics/v2/assets/{assetId}/timeseries` | GET | Historical metric timeseries |
| `/metrics/v1/markets` | GET | List trading pairs/markets across exchanges |
| `/metrics/v1/markets/{marketIdentifier}` | GET | Get a specific trading pair/market |
| `/metrics/v1/markets/metrics` | GET | List available market timeseries metrics |

**Query parameters:**
- `assetSlugs` — comma-separated slugs (e.g., `bitcoin,ethereum`)
- `assetIds` — comma-separated asset IDs
- `metrics` — specific metrics to return
- `start`, `end` — date range (ISO 8601)
- `interval` — timeseries interval (`1d`, `1w`)
- `limit`, `page` — pagination

```bash
curl "https://api.messari.io/metrics/v2/assets?assetSlugs=bitcoin,ethereum" \
  -H "x-messari-api-key: $MESSARI_API_KEY"
```

---

### Signal Service

Real-time social intelligence — sentiment scoring, mindshare tracking, trending narratives.

Route questions about market sentiment, social buzz, what's trending, mindshare gainers/losers here.

| Endpoint | Method | Description |
|---|---|---|
| `/signal/v1/assets` | GET | List assets with signal metrics |
| `/signal/v1/assets/{assetId}` | GET | Signal metrics for a specific asset |
| `/signal/v1/assets/{assetId}/timeseries` | GET | Historical signal timeseries |
| `/signal/v1/assets/gainers-losers` | GET | Top mindshare gainers and losers |
| `/signal/v1/assets/mindshare-gainers-24h` | GET | Assets with biggest mindshare gains (24h) |
| `/signal/v1/assets/mindshare-gainers-7d` | GET | Assets with biggest mindshare gains (7d) |
| `/signal/v1/assets/mindshare-losers-24h` | GET | Assets with biggest mindshare losses (24h) |
| `/signal/v1/assets/mindshare-losers-7d` | GET | Assets with biggest mindshare losses (7d) |

**Query parameters:**
- `type` — signal type (e.g., `mindshare`)
- `limit` — number of results
- `start`, `end` — date range (ISO 8601)

```bash
curl "https://api.messari.io/signal/v1/assets/gainers-losers?type=mindshare&limit=10" \
  -H "x-messari-api-key: $MESSARI_API_KEY"
```

---

### News Service

Real-time crypto news aggregation — breaking events, project updates, regulatory developments.

Route questions about recent headlines, current events, or "what happened with X" here.

| Endpoint | Method | Description |
|---|---|---|
| `/news/v1/news/feed` | GET | Aggregated crypto news feed |
| `/news/v1/news/sources` | GET | List available news sources |

**Query parameters:**
- `assetSlugs` — filter news by asset
- `sourceIds` — filter by news source
- `limit`, `page` — pagination

```bash
curl "https://api.messari.io/news/v1/news/feed?limit=20" \
  -H "x-messari-api-key: $MESSARI_API_KEY"
```

---

### Research Service

Institutional-grade reports — sector deep dives, protocol diligence, quarterly reviews, governance analysis.

Route questions about fundamental research, due diligence, analyst opinions, and sector analysis here.

| Endpoint | Method | Description |
|---|---|---|
| `/research/v1/reports` | GET | List research reports |
| `/research/v1/reports/{reportId}` | GET | Get a specific report |
| `/research/v1/reports/tags` | GET | List available report tags |

**Query parameters:**
- `tags` — filter by report tags
- `assetSlugs` — filter by related assets
- `limit`, `page` — pagination

---

### Stablecoins Service

On-chain metrics, historical timeseries, and per-chain breakdowns for 25+ stablecoins.

Route stablecoin-specific questions here — supply, flows, chain-level breakdowns, market share.

| Endpoint | Method | Description |
|---|---|---|
| `/stablecoins/v2/stablecoins` | GET | List stablecoins with metrics |
| `/stablecoins/v2/stablecoins/{stablecoinId}` | GET | Detailed metrics for a stablecoin |
| `/stablecoins/v2/stablecoins/{stablecoinId}/timeseries` | GET | Historical timeseries |

**Query parameters:**
- `metrics` — specific stablecoin metrics
- `chains` — filter by blockchain
- `start`, `end` — date range (ISO 8601)
- `interval` — timeseries interval
- `limit`, `page` — pagination

---

### Exchanges Service

Exchange-level volume, metrics, and historical timeseries across 210+ exchanges.

Route questions about exchange volumes, comparisons, or exchange-specific data here.

| Endpoint | Method | Description |
|---|---|---|
| `/exchanges/v2/exchanges` | GET | List exchanges with metrics |
| `/exchanges/v2/exchanges/{exchangeId}` | GET | Detailed metrics for an exchange |
| `/exchanges/v2/exchanges/{exchangeId}/timeseries` | GET | Historical timeseries |

**Query parameters:**
- `exchangeSlugs` — comma-separated exchange slugs
- `metrics` — specific exchange metrics
- `start`, `end` — date range (ISO 8601)
- `interval` — timeseries interval
- `limit`, `page` — pagination

---

### Networks Service

L1/L2 blockchain network metrics — chain-level activity, fees, active addresses.

Route questions about blockchain networks, chain comparisons, or on-chain metrics here.

| Endpoint | Method | Description |
|---|---|---|
| `/networks/v2/networks` | GET | List networks with metrics |
| `/networks/v2/networks/{networkId}` | GET | Detailed metrics for a network |
| `/networks/v2/networks/{networkId}/timeseries` | GET | Historical timeseries |

**Query parameters:**
- `networkSlugs` — comma-separated network slugs
- `metrics` — specific network metrics
- `start`, `end` — date range (ISO 8601)
- `interval` — timeseries interval
- `limit`, `page` — pagination

---

### Protocols Service

DeFi protocol metrics across DEXs, lending, liquid staking, and bridges.

Route DeFi-specific questions here — TVL, protocol comparisons, category-specific data.

| Endpoint | Method | Description |
|---|---|---|
| `/protocols/v2/protocols` | GET | List protocols with metrics |
| `/protocols/v2/protocols/{protocolId}` | GET | Detailed metrics for a protocol |
| `/protocols/v2/protocols/dex` | GET | DEX-specific metrics |
| `/protocols/v2/protocols/lending` | GET | Lending protocol metrics |
| `/protocols/v2/protocols/interop` | GET | Bridge/interoperability metrics |
| `/protocols/v2/protocols/liquid-staking` | GET | Liquid staking metrics |

**Query parameters:**
- `protocolSlugs` — comma-separated protocol slugs
- `metrics` — specific protocol metrics
- `limit`, `page` — pagination

---

### Token Unlocks Service

Vesting schedules, upcoming unlock events, and supply pressure analysis.

Route questions about token unlocks, vesting, or upcoming supply events here.

| Endpoint | Method | Description |
|---|---|---|
| `/token-unlocks/v1/assets` | GET | List assets with unlock data |
| `/token-unlocks/v1/assets/{assetId}` | GET | Unlock details for an asset |
| `/token-unlocks/v1/assets/{assetId}/events` | GET | Upcoming unlock events |
| `/token-unlocks/v1/assets/{assetId}/vesting` | GET | Full vesting schedule |
| `/token-unlocks/v1/allocations` | GET | Get token allocation data across assets |
| `/token-unlocks/v1/assets/{assetId}/unlocks` | GET | Interval-based unlock timeseries data |

**Query parameters:**
- `assetSlugs` — comma-separated asset slugs
- `start`, `end` — date range (ISO 8601)
- `limit`, `page` — pagination

---

### Fundraising Service

Funding rounds, investors, funds, organizations, projects, and M&A activity.

Route questions about who invested in what, fundraising rounds, investor activity, or M&A here.

| Endpoint | Method | Description |
|---|---|---|
| `/fundraising/v1/rounds` | GET | List fundraising rounds |
| `/fundraising/v1/organizations` | GET | List organizations |
| `/fundraising/v1/projects` | GET | List projects that raised funding |
| `/fundraising/v1/investors` | GET | List investors and activity |
| `/fundraising/v1/funds` | GET | List investment funds |
| `/fundraising/v1/mergers-acquisitions` | GET | List M&A transactions |
| `/fundraising/v1/rounds/investors` | GET | Investors that participated in filtered rounds |
| `/fundraising/v1/funds/managers` | GET | Managers of filtered funds |

**Query parameters:**
- `assetSlugs` — filter by related asset
- `investorSlugs` — filter by investor
- `roundTypes` — filter by round type (`seed`, `series-a`)
- `start`, `end` — date range (ISO 8601)
- `limit`, `page` — pagination

---

### Intel Service

Governance events, protocol upgrades, and key project milestones.

Route questions about governance proposals, protocol upgrades, or project events here.

| Endpoint | Method | Description |
|---|---|---|
| `/intel/v1/events` | GET | List intel events |
| `/intel/v1/events/{eventId}` | GET | Details for a specific event |
| `/intel/v1/assets` | GET | List assets with intel data |

**Query parameters:**
- `assetSlugs` — filter by asset
- `eventTypes` — filter by event type
- `start`, `end` — date range (ISO 8601)
- `limit`, `page` — pagination

---

### Topics Service

Trending topic classification and daily timeseries.

Route questions about trending narratives or topic momentum here.

| Endpoint | Method | Description |
|---|---|---|
| `/topics/v1/classes` | GET | List topic classes/categories |
| `/topics/v1/current` | GET | Currently trending topics |
| `/topics/v1/timeseries` | GET | Daily topic trend timeseries |

**Query parameters:**
- `classIds` — filter by topic class
- `start`, `end` — date range (ISO 8601)
- `limit` — number of results

---

### X-Users Service

Crypto X/Twitter user metrics and influence tracking.

Route questions about crypto influencers, social account metrics, or X/Twitter activity here.

| Endpoint | Method | Description |
|---|---|---|
| `/signal/v1/x-users` | GET | List crypto X users with metrics |
| `/signal/v1/x-users/{userId}` | GET | Metrics for a specific X user |
| `/signal/v1/x-users/{userId}/timeseries` | GET | Historical metrics for an X user |

**Query parameters:**
- `limit`, `page` — pagination
- `start`, `end` — date range (ISO 8601)

### Bulk Data Service

High-performance bulk data download in CSV or JSONL format. Designed for data scientists, analysts, and researchers needing large historical datasets.

Route bulk download, historical data export, CSV/JSONL dataset requests here.

| Endpoint | Method | Description |
|---|---|---|
| `/bulk/v1/datasets` | GET | List available bulk datasets for your subscription tier |
| `/bulk/v1/datasets/{datasetSlug}/{granularity}/data` | GET | Download dataset in CSV or JSONL format |

**Query parameters:**
- `format` — `csv` or `jsonl`
- `start`, `end` — date range (ISO 8601)
- Granularities: `5m`, `15m`, `30m`, `1h`, `1d`

---

## Routing Guide

Use this logic to pick the right service for a query:

| User is asking about... | Route to |
|---|---|
| General crypto question, synthesis, "what do you think about X" | **AI** |
| Price, volume, market cap, ROI, ATH, performance comparison | **Metrics** |
| Sentiment, mindshare, trending tokens, social buzz | **Signal** |
| Headlines, recent events, breaking news | **News** |
| Analyst reports, deep dives, sector overviews | **Research** |
| Stablecoin supply, flows, chain breakdowns | **Stablecoins** |
| Exchange volumes, exchange comparisons | **Exchanges** |
| L1/L2 network activity, fees, active addresses | **Networks** |
| DeFi protocols, TVL, lending, DEX volume | **Protocols** |
| Token unlocks, vesting schedules | **Token Unlocks** |
| Fundraising rounds, investors, VC activity, M&A | **Fundraising** |
| Governance events, protocol upgrades | **Intel** |
| Trending narratives, topic momentum | **Topics** |
| Crypto influencers, X/Twitter accounts | **X-Users** |
| In-depth research report, comprehensive analysis on a crypto topic | **Deep Research** (under AI) |
| Bulk data download, large historical datasets, CSV/JSONL export | **Bulk Data** |
| Trading pair data, market-specific price and volume | **Markets** (under Metrics) |

### Multi-service queries

Many questions benefit from combining services. Examples:

- **"Is SOL overvalued?"** → Metrics (price, fundamentals) + Signal (sentiment) + Token Unlocks (supply pressure) + AI (synthesize)
- **"Due diligence on Eigen Layer"** → Research (reports) + Metrics (fundamentals) + Fundraising (investors) + Intel (governance) + AI (synthesize)
- **"What are the trending narratives this week?"** → Topics (trending classes) + Signal (mindshare gainers) + News (related headlines)
- **"Compare TVL across lending protocols"** → Protocols/lending (metrics) + Networks (chain context)

When in doubt, start with the **AI** service — it draws from all other sources and provides the broadest context.

---

## Example Questions

```
"Which assets over $1B marketcap outperformed Bitcoin over the last 3 months?"
"What are some upcoming token unlock events this month?"
"Give me the 10 most recent fundraising rounds in the AI and compute sectors."
"What are the latest headlines related to crypto regulation?"
"What are some recent developments in the DePin sector?"
"Which investor has been the most active in seed rounds over the last year?"
"What were the top events for the AAVE protocol during the last quarter?"
"Give me the latest ecosystem map of Solana."
"Tell me about the recent investments from a16z crypto."
"Compare and contrast the native asset functions of BitTensor vs Render."
"Generate a deep research report on the current state of liquid staking."
"What are the trending questions in crypto right now?"
"Download bulk historical price data for Ethereum over the last year."
```
