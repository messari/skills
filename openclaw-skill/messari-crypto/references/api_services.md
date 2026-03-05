# Messari REST API Services Reference

Detailed breakdown of each service available through the Messari REST API.

**Base URL:** `https://api.messari.io`
**Auth Header:** `x-messari-api-key: <YOUR_API_KEY>`

## x402 Payments

Some Messari endpoints support pay-per-request access via x402.

- Discover payable resources dynamically with `GET https://api.messari.io/.well-known/x402`.
- Treat the runtime `402 Payment Required` challenge as the source of truth for payable route and price.
- Do not hardcode x402 prices or payable-route assumptions in this reference.
- Use the endpoint tables in this reference as the authoritative view of currently supported authentication methods per endpoint.

**Negotiation flow:**
1. Send the request normally.
2. If the response is `402 Payment Required`, parse the payment requirements from the response body and the `Payment-Required` header.
3. Create/sign the payment payload and retry with `Payment-Signature` (legacy compatibility: `X-PAYMENT`).
4. Continue once the retried request succeeds.

**Budget guardrail:** If there is no pre-approved budget or prior user consent, ask the user to confirm before executing paid x402 requests.

---

## AI Service

Messari's AI completions endpoint, trained on 30TB+ of structured and unstructured crypto data.

**Data sources include:** market/price data, fundraising data, network metrics, Messari-exclusive
research reports, newsletters, podcasts, and curated third-party content (news, social posts).

**Use for:** open-ended crypto questions, synthesis across multiple data sources, market analysis,
protocol comparisons, narrative summaries.

**Requires:** Paid access (for example, Messari AI credits and/or x402 negotiation depending on account and route policy).

| Endpoint | Method | Authentication Method | Description |
|---|---|---|---|
| `/ai/v2/chat/completions` | POST | `api_key`, `x402` | Messari-native chat completions with structured response objects (x402-capable route) |
| `/ai/v1/chat/completions` | POST | `api_key` | Legacy Messari chat completion endpoint over Messari's crypto data corpus |
| `/ai/openai/chat/completions` | POST | `api_key` | OpenAI-compatible chat completion response format for drop-in client support |

**Key parameters (POST body):**
- `messages` — array of `{role, content}` message objects
- `stream` — boolean, enable streaming responses

---

## Signal Service

Real-time social intelligence and narrative tracking.

**Use for:** sentiment shifts, mindshare tracking, trending narratives, social-driven signals.

| Endpoint | Method | Authentication Method | Description |
|---|---|---|---|
| `/signal/v1/assets` | GET | `api_key`, `x402` | Ranked asset signal feed (mindshare, sentiment, momentum) with sorting/filtering |
| `/signal/v1/assets/{assetID}` | GET | `api_key`, `x402` | Detailed social-signal profile for one asset |
| `/signal/v1/assets/mindshare-gainers-24h` | GET | `api_key`, `x402` | Assets with the largest mindshare gains over the last 24 hours |
| `/signal/v1/assets/mindshare-gainers-7d` | GET | `api_key`, `x402` | Assets with the largest mindshare gains over the last 7 days |
| `/signal/v1/assets/mindshare-losers-24h` | GET | `api_key`, `x402` | Assets with the largest mindshare losses over the last 24 hours |
| `/signal/v1/assets/mindshare-losers-7d` | GET | `api_key`, `x402` | Assets with the largest mindshare losses over the last 7 days |
| `/signal/v1/assets/time-series/1h` | GET | `api_key`, `x402` | Hourly asset social-signal timeseries (x402-capable route) |
| `/signal/v1/assets/time-series/1d` | GET | `api_key`, `x402` | Daily asset social-signal timeseries (x402-capable route) |

**Key query parameters:**
- `assetIds` — comma-separated asset IDs/slugs to filter list responses
- `sort`, `sortDirection` — ranking field and direction for list responses
- `limit`, `page` — pagination controls
- `start`, `end` — date range for time-series calls (RFC3339 or unix timestamp)

---

## Metrics Service

Comprehensive quantitative data across the crypto market.

**Coverage:** 34,000+ assets, 210+ exchanges, 175+ filterable metrics.

**Use for:** quantitative analysis, asset comparison, trend identification, portfolio-level data.

| Endpoint | Method | Authentication Method | Description |
|---|---|---|---|
| `/metrics/v2/assets` | GET | `api_key`, `x402` | List assets with market and fundamental metrics |
| `/metrics/v2/assets/metrics` | GET | `api_key`, `x402` | List dataset slugs and supported granularities for asset timeseries |
| `/metrics/v2/assets/ath` | GET | `api_key`, `x402` | Return all-time-high snapshots and drawdown context for selected assets |
| `/metrics/v2/assets/details` | GET | `api_key`, `x402` | Return rich point-in-time details for selected assets |
| `/metrics/v2/assets/roi` | GET | `api_key`, `x402` | Return multi-window ROI snapshots for selected assets |
| `/metrics/v2/assets/{assetID}/metrics/{datasetSlug}/time-series` | GET | `api_key`, `x402` | Return historical timeseries for an asset metric dataset |
| `/metrics/v2/assets/{assetID}/metrics/{datasetSlug}/time-series/{granularity}` | GET | `api_key`, `x402` | Return asset metric timeseries at explicit granularity (`5m`, `15m`, `1h`, `1d`) |

**Key query parameters:**
- `assetSlugs` — comma-separated asset slugs (e.g., `bitcoin,ethereum`)
- `assetIds` — comma-separated asset IDs
- `metrics` — specific metrics to return
- `start`, `end` — date range for timeseries (ISO 8601)
- `interval` — timeseries interval (e.g., `1d`, `1w`)
- `limit`, `page` — pagination

---

## News Service

Real-time crypto news aggregation.

**Use for:** event-driven queries, breaking news, project updates, regulatory developments.

| Endpoint | Method | Authentication Method | Description |
|---|---|---|---|
| `/news/v1/news/assets` | GET | `api_key` | List assets currently tagged in news coverage |
| `/news/v1/news/feed` | GET | `api_key`, `x402` | Paginated cross-source crypto news feed |
| `/news/v1/news/sources` | GET | `api_key`, `x402` | List source metadata and IDs for feed filtering |

**Key query parameters:**
- `assetSlugs` — filter news by asset
- `limit`, `page` — pagination
- `sourceIds` — filter by specific news sources

---

## Research Service

Institutional-grade research from Messari's analyst team.

**Use for:** fundamental research, due diligence, sector analysis, protocol evaluations.

| Endpoint | Method | Authentication Method | Description |
|---|---|---|---|
| `/research/v1/reports` | GET | `api_key` | List research reports with filters like asset IDs and tags |
| `/research/v1/reports/{reportId}` | GET | `api_key` | Retrieve a specific research report by report ID |
| `/research/v1/reports/tags` | GET | `api_key` | List research tags available for report filtering |

**Key query parameters:**
- `tags` — filter by report tags
- `assetSlugs` — filter by related assets
- `limit`, `page` — pagination

---

## Stablecoins Service

Dedicated stablecoin analytics.

**Coverage:** 25+ stablecoins with per-chain breakdowns.

**Use for:** stablecoin supply analysis, chain-level flow tracking, market share shifts.

| Endpoint | Method | Authentication Method | Description |
|---|---|---|---|
| `/metrics/v2/stablecoins` | GET | `api_key`, `x402` | List stablecoins with supply and market metrics |
| `/metrics/v2/stablecoins/metrics` | GET | `api_key`, `x402` | List dataset slugs and supported granularities for stablecoin timeseries |
| `/metrics/v2/stablecoins/{entityIdentifier}/metrics/{datasetSlug}/time-series` | GET | `api_key`, `x402` | Return historical timeseries for a stablecoin metric dataset |
| `/metrics/v2/stablecoins/{entityIdentifier}/metrics/{datasetSlug}/time-series/1d` | GET | `api_key`, `x402` | Return daily stablecoin metric timeseries via explicit `1d` route |

**Key query parameters:**
- `metrics` — specific stablecoin metrics
- `chains` — filter by blockchain
- `start`, `end` — date range for timeseries (ISO 8601)
- `interval` — timeseries interval
- `limit`, `page` — pagination

---

## Exchanges Service

Crypto exchange market data.

**Use for:** exchange volume analysis, exchange-level metrics, cross-exchange comparisons.

| Endpoint | Method | Authentication Method | Description |
|---|---|---|---|
| `/metrics/v1/exchanges` | GET | `api_key` | List exchanges with spot/futures volume and open-interest context |
| `/metrics/v1/exchanges/metrics` | GET | `api_key` | List available metric datasets for exchange timeseries |
| `/metrics/v1/exchanges/{exchangeIdentifier}` | GET | `api_key` | Retrieve one exchange by slug or ID with current metrics |
| `/metrics/v1/exchanges/{entityIdentifier}/metrics/{datasetSlug}/time-series/{granularity}` | GET | `api_key` | Exchange metric timeseries at selected granularity |

**Key query parameters:**
- `type` — filter exchanges by type
- `limit`, `page` — pagination controls
- `start`, `end` — date range for timeseries calls
- `granularity` — timeseries interval for metric calls

---

## Networks Service

L1/L2 blockchain network metrics.

**Use for:** network activity analysis, chain comparisons, on-chain metrics.

| Endpoint | Method | Authentication Method | Description |
|---|---|---|---|
| `/metrics/v2/networks` | GET | `api_key`, `x402` | List blockchain networks with activity, fee, and usage metrics |
| `/metrics/v2/networks/metrics` | GET | `api_key`, `x402` | List dataset slugs and supported granularities for network timeseries |
| `/metrics/v2/networks/{entityIdentifier}/metrics/{datasetSlug}/time-series` | GET | `api_key`, `x402` | Return historical timeseries for a network metric dataset |
| `/metrics/v2/networks/{entityIdentifier}/metrics/{datasetSlug}/time-series/{granularity}` | GET | `api_key`, `x402` | Return network metric timeseries at explicit granularity (`5m`, `15m`, `1h`, `1d`) |

**Key query parameters:**
- `networkSlugs` — comma-separated network slugs
- `metrics` — specific network metrics
- `start`, `end` — date range for timeseries (ISO 8601)
- `interval` — timeseries interval
- `limit`, `page` — pagination

---

## Protocols Service

DeFi protocol metrics across multiple categories.

**Use for:** DeFi analysis, protocol comparisons, TVL tracking, yield data.

| Endpoint | Method | Authentication Method | Description |
|---|---|---|---|
| `/metrics/v2/protocols` | GET | `api_key` | List all protocols with aggregated core metrics across deployments |
| `/metrics/v2/protocols/metrics` | GET | `api_key` | List available core protocol timeseries metric datasets |
| `/metrics/v2/protocols/{protocolIdentifier}/metrics/core/time-series/{granularity}` | GET | `api_key` | Core protocol metric timeseries for a specific protocol |
| `/metrics/v2/protocols/dex` | GET | `api_key` | List DEX protocols with aggregated category metrics |
| `/metrics/v2/protocols/dex/metrics` | GET | `api_key` | List available DEX protocol timeseries metric datasets |
| `/metrics/v2/protocols/{protocolIdentifier}/metrics/dex/time-series/{granularity}` | GET | `api_key` | DEX protocol metric timeseries for a specific protocol |
| `/metrics/v2/protocols/lending` | GET | `api_key` | List lending protocols with aggregated category metrics |
| `/metrics/v2/protocols/lending/metrics` | GET | `api_key` | List available lending protocol timeseries metric datasets |
| `/metrics/v2/protocols/{protocolIdentifier}/metrics/lending/time-series/{granularity}` | GET | `api_key` | Lending protocol metric timeseries for a specific protocol |
| `/metrics/v2/protocols/interop` | GET | `api_key` | List interoperability/bridge protocols with aggregated category metrics |
| `/metrics/v2/protocols/interop/metrics` | GET | `api_key` | List available interoperability protocol timeseries metric datasets |
| `/metrics/v2/protocols/{protocolIdentifier}/metrics/interop/time-series/{granularity}` | GET | `api_key` | Interoperability protocol metric timeseries for a specific protocol |
| `/metrics/v2/protocols/liquid-staking` | GET | `api_key` | List liquid-staking protocols with aggregated category metrics |
| `/metrics/v2/protocols/liquid-staking/metrics` | GET | `api_key` | List available liquid-staking protocol timeseries metric datasets |
| `/metrics/v2/protocols/{protocolIdentifier}/metrics/liquid-staking/time-series/{granularity}` | GET | `api_key` | Liquid-staking protocol metric timeseries for a specific protocol |

**Key query parameters:**
- `sort`, `order` — ranking column and sort direction for list endpoints
- `limit`, `page` — pagination controls
- `protocolIdentifier` — protocol slug or ID for timeseries endpoints
- `granularity`, `start`, `end` — timeseries interval and date range

---

## Token Unlocks Service

Token vesting schedules and unlock events.

**Use for:** tracking upcoming token unlocks, supply pressure analysis, vesting schedule lookup.

| Endpoint | Method | Authentication Method | Description |
|---|---|---|---|
| `/token-unlocks/v1/assets` | GET | `api_key`, `x402` | List assets covered by unlock and allocation datasets |
| `/token-unlocks/v1/allocations` | GET | `api_key`, `x402` | Allocation breakdown by bucket/category for selected assets |
| `/token-unlocks/v1/assets/{assetId}/events` | GET | `api_key`, `x402` | Upcoming token unlock events for a specific asset |
| `/token-unlocks/v1/assets/{assetId}/unlocks` | GET | `api_key`, `x402` | Historical unlock timeseries for a specific asset |
| `/token-unlocks/v1/assets/{assetId}/vesting-schedule` | GET | `api_key`, `x402` | Forward-looking vesting schedule timeseries for a specific asset |

**Key query parameters:**
- `assetSlugs` — comma-separated asset slugs
- `start`, `end` — date range filter (ISO 8601)
- `limit`, `page` — pagination

---

## Fundraising Service

Crypto fundraising data including rounds, investors, and M&A activity.

**Use for:** tracking funding rounds, investor activity, project fundraising history, M&A deals.

| Endpoint | Method | Authentication Method | Description |
|---|---|---|---|
| `/funding/v1/rounds` | GET | `api_key`, `x402` | Query funding rounds by stage, date, amount, and participants |
| `/funding/v1/rounds/investors` | GET | `api_key`, `x402` | Return investors participating in rounds matching applied filters |
| `/funding/v1/mergers-and-acquisitions` | GET | `api_key`, `x402` | Query crypto M&A deals with acquirer/target metadata |
| `/funding/v1/organizations` | GET | `api_key`, `x402` | Query organizations active in fundraising and investment activity |
| `/funding/v1/projects` | GET | `api_key`, `x402` | Query projects and their fundraising attributes |
| `/funding/v1/funds` | GET | `api_key`, `x402` | Query investment funds and fund-level metadata |
| `/funding/v1/funds/managers` | GET | `api_key`, `x402` | Return organizations/people managing funds matching filters |

**Key query parameters:**
- `assetSlugs` — filter by related asset
- `investorSlugs` — filter by investor
- `roundTypes` — filter by round type (e.g., `seed`, `series-a`)
- `start`, `end` — date range (ISO 8601)
- `limit`, `page` — pagination

---

## Intel Service

Governance events, protocol updates, and project milestones.

**Use for:** tracking governance proposals, protocol upgrades, key project events.

| Endpoint | Method | Authentication Method | Description |
|---|---|---|---|
| `/intel/v1/events` | GET | `api_key` | List intel events (legacy GET route; prefer POST for advanced filtering) |
| `/intel/v1/events` | POST | `api_key` | Preferred event query route with body-based filters |
| `/intel/v1/events/{eventId}` | GET | `api_key` | Retrieve one event with its update history |
| `/intel/v1/assets` | GET | `api_key` | List assets covered by the Intel event dataset |

**Key query parameters:**
- `assetSlugs` — filter by asset
- `eventTypes` — filter by event type
- `start`, `end` — date range (ISO 8601)
- `limit`, `page` — pagination

---

## Topics Service

Trending topic classification and timeseries.

**Use for:** identifying trending narratives, tracking topic momentum over time.

| Endpoint | Method | Authentication Method | Description |
|---|---|---|---|
| `/topics/v1/classes` | GET | `api_key` | List available topic classes (taxonomy labels) |
| `/topics/v1/current` | GET | `api_key` | Snapshot of currently trending topics with rank and metadata |
| `/topics/v1/daily` | GET | `api_key` | Historical topic timeseries over a specified date range |

**Key query parameters:**
- `classes` — filter current topics by class names
- `assetIDs` — filter topics by associated assets (UUID, slug, or symbol)
- `start`, `end`, `granularity` — time window controls for `/topics/v1/daily`
- `sort`, `limit`, `page` — sorting and pagination for `/topics/v1/current`

---

## X-Users Service

Crypto X/Twitter user metrics and influence tracking.

**Use for:** tracking influential crypto accounts, social activity metrics, influence timeseries.

| Endpoint | Method | Authentication Method | Description |
|---|---|---|---|
| `/signal/v1/x-users` | GET | `api_key`, `x402` | Ranked crypto X-user signal feed with engagement and mindshare metrics |
| `/signal/v1/x-users/time-series/1d` | GET | `api_key`, `x402` | Daily X-user social-signal timeseries (x402-capable route) |
| `/signal/v1/x-users/{xUserID}` | GET | `api_key`, `x402` | Detailed social-signal profile for one X user |

**Key query parameters:**
- `limit`, `page` — pagination controls
- `sort`, `sortDirection`, `accountType` — ranking/filter controls for list endpoint
- `start`, `end` — date range for timeseries calls
