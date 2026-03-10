# Messari REST API — Endpoint Reference

**Base URL:** `https://api.messari.io`
**Auth Header:** `x-messari-api-key: <API_KEY>`

## Contents

- [AI Service](#ai-service)
- [Metrics Service](#metrics-service)
- [Signal Service](#signal-service)
- [News Service](#news-service)
- [Research Service](#research-service)
- [Stablecoins Service](#stablecoins-service)
- [Exchanges Service](#exchanges-service)
- [Networks Service](#networks-service)
- [Protocols Service](#protocols-service)
- [Token Unlocks Service](#token-unlocks-service)
- [Fundraising Service](#fundraising-service)
- [Intel Service](#intel-service)
- [Topics Service](#topics-service)
- [X-Users Service](#x-users-service)

---

## AI Service

Chat completions and deep research reports across Messari's full crypto data warehouse. Route general or open-ended crypto questions here first.

Requires Messari AI credits.

### Chat Completions

| Endpoint | Method | Description |
|---|---|---|
| `/ai/v1/chat/completions` | POST | Chat completion against Messari's crypto data warehouse |
| `/ai/openai/chat/completions` | POST | OpenAI-compatible chat completion endpoint |
| `/ai/v2/chat/completions` | POST | v2 chat completion with inline citations, verbosity control, related questions |
| `/ai/v1/questions/trending` | GET | Suggested questions based on trending crypto topics |

**POST body:**
- `messages` — array of `{role, content}` message objects
- `stream` — boolean, enable streaming responses

**Additional v2 parameters:**
- `verbosity` — `succinct`, `balanced`, or `verbose`
- `inline_citations` — boolean, include citation references in metadata
- `generate_related_questions` — integer, number of follow-up suggestions
- `response_format` — `markdown` or `plaintext`

**Trending questions parameters (GET):**
- `limit` — number of trending questions to return (default: 5)

  ```bash
  curl -X POST "https://api.messari.io/ai/v1/chat/completions" \
  -H "x-messari-api-key: $MESSARI_API_KEY" \
  -H "Content-Type: application/json" \
  -d '{"messages": [{"role": "user", "content": "What is the bull case for ETH right now?"}]}'
```

### Deep Research

Asynchronous long-form research reports with citations. Each report costs 500 AI credits and takes 5-10 minutes.

| Endpoint | Method | Description |
|---|---|---|
| `/ai/v1/deep-research` | POST | Create a new deep research job |
| `/ai/v1/deep-research` | GET | List deep research jobs |
| `/ai/v1/deep-research/{id}` | GET | Get job status; includes report when completed |
| `/ai/v1/deep-research/{id}/cancel` | POST | Cancel a queued or in-progress job |

**POST body:**
- `query` — research topic or question (required)
- `instructions` — optional system instructions to guide research style
- `job_id` — optional, pass existing job ID for follow-up refinement

**GET list parameters:**
- `limit` — max results (default 20, max 100)
- `offset` — pagination offset
- `status` — filter: `queued`, `in_progress`, `completed`, `failed`, `cancelled`

**Job lifecycle:** `queued` → `in_progress` → `completed` | `failed` | `cancelled`

**Completed job response includes:**
- `output.report` — full report in markdown
- `output.sources` — list of cited sources

```bash
curl -X POST "https://api.messari.io/ai/v1/deep-research" \
  -H "x-messari-api-key: $MESSARI_API_KEY" \
  -H "Content-Type: application/json" \
  -d '{"query": "What is the current state of liquid staking on Ethereum?"}'
```

---

## Metrics Service

Price, volume, market cap, and fundamental metrics for 34,000+ assets across 210+ exchanges.

| Endpoint | Method | Description |
|---|---|---|
| `/metrics/v2/assets` | GET | List assets with market metrics |
| `/metrics/v2/assets/{assetId}` | GET | Detailed metrics for a specific asset |
| `/metrics/v2/assets/{assetId}/roi` | GET | ROI data for an asset |
| `/metrics/v2/assets/{assetId}/ath` | GET | All-time high data for an asset |
| `/metrics/v2/assets/{assetId}/timeseries` | GET | Historical metric timeseries |

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


## Signal Service

Real-time social intelligence — sentiment scoring, mindshare tracking, trending narratives.

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

## News Service

Real-time crypto news aggregation from 500+ sources.

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

## Research Service

Institutional-grade reports — sector deep dives, protocol diligence, quarterly reviews.

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

## Stablecoins Service

On-chain metrics and per-chain breakdowns for 25+ stablecoins.

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

## Exchanges Service

Exchange-level volume, metrics, and historical timeseries across 210+ exchanges.

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

## Networks Service

L1/L2 blockchain network metrics — chain-level activity, fees, active addresses.

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

## Protocols Service

DeFi protocol metrics across DEXs, lending, liquid staking, and bridges.

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

## Token Unlocks Service

Vesting schedules, upcoming unlock events, and supply pressure analysis.

| Endpoint | Method | Description |
|---|---|---|
| `/token-unlocks/v1/assets` | GET | List assets with unlock data |
| `/token-unlocks/v1/assets/{assetId}` | GET | Unlock details for an asset |
| `/token-unlocks/v1/assets/{assetId}/events` | GET | Upcoming unlock events |
| `/token-unlocks/v1/assets/{assetId}/vesting` | GET | Full vesting schedule |
| `/token-unlocks/v1/allocations` | GET | Token allocation data across assets |
| `/token-unlocks/v1/assets/{assetId}/unlocks` | GET | Interval-based unlock timeseries data |

**Query parameters:**
- `assetSlugs` — comma-separated asset slugs
- `start`, `end` — date range (ISO 8601)
- `limit`, `page` — pagination

---

## Fundraising Service

Funding rounds, investors, funds, organizations, projects, and M&A activity.

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

## Intel Service

Governance events, protocol upgrades, and key project milestones.

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

## Topics Service

Trending topic classification and daily timeseries.

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

## X-Users Service

Crypto X/Twitter user metrics and influence tracking.

| Endpoint | Method | Description |
|---|---|---|
| `/signal/v1/x-users` | GET | List crypto X users with metrics |
| `/signal/v1/x-users/{userId}` | GET | Metrics for a specific X user |
| `/signal/v1/x-users/{userId}/timeseries` | GET | Historical metrics for an X user |

**Query parameters:**
- `limit`, `page` — pagination
- `start`, `end` — date range (ISO 8601)

