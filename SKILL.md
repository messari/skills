# Messari — Crypto Market Intelligence for AI Agents

You are integrating with Messari, the leading crypto data platform. You have access to real-time and historical data across 34,000+ crypto assets, 210+ exchanges, and 14 specialized data services covering market metrics, social sentiment, institutional research, on-chain analytics, fundraising, governance, and more.

## Integration Paths

### Option A: MCP Server (Preferred)

If your client supports the Model Context Protocol, connect to Messari's hosted MCP server.

**Server URL:** `https://mcp.messari.io/mcp`

Hosted MCP access uses API-key authentication. Get a key at [messari.io/api](https://messari.io/api).

### Option B: REST API (Direct HTTP)

If your client does not support MCP, call Messari's REST API directly.

**Base URL:** `https://api.messari.io`

**Authentication modes:**
- **API key mode:** include API key header:

```
x-messari-api-key: <API_KEY>
```

- **x402 mode:** send request normally, handle `402 Payment Required`, then retry with `Payment-Signature` (legacy: `X-PAYMENT`).

All endpoints accept and return JSON. Use `Content-Type: application/json` for POST requests.

**Credentials:** For x402-enabled routes, use either `MESSARI_API_KEY` or `X402_PRIVATE_KEY`. Endpoints marked `api_key`-only require `MESSARI_API_KEY`. If required credentials are missing, ask the user to provide them before making requests.

### Credential Modes

- **API-key mode (`MESSARI_API_KEY`)**: Works for all `api_key` endpoints. Credit-metered endpoints (for example, Messari AI) may require Messari AI credits.
- **x402 mode (`X402_PRIVATE_KEY`)**: Works on x402-enabled endpoints via runtime payment negotiation and does not require pre-purchased Messari AI credits.
- **Coverage caveat:** x402-only credentials cannot call `api_key`-only endpoints.
- **Secrets guardrail:** Never commit secret values. Use env vars only and placeholders like `$MESSARI_API_KEY` and `$X402_PRIVATE_KEY` in docs/examples.

## x402 Payments

Some Messari endpoints support pay-per-request access via x402.

- Discover payable resources dynamically with `GET https://api.messari.io/.well-known/x402`.
- Treat the runtime `402 Payment Required` challenge as the source of truth for payable route and price.
- Do not hardcode x402 prices or payable-route assumptions in this skill.
- Use the endpoint tables under `## Services and Endpoints` as the authoritative documentation for currently supported authentication methods per endpoint.

**Negotiation flow:**
1. Send the request normally.
2. If the response is `402 Payment Required`, parse the payment requirements from the response body and the `Payment-Required` header.
3. Create/sign the payment payload and retry with `Payment-Signature` (legacy compatibility: `X-PAYMENT`).
4. Continue once the retried request succeeds.

**Budget guardrail:** If there is no pre-approved budget or prior user consent, ask the user to confirm before executing paid x402 requests.

### Request Patterns (curl + TypeScript)

**API-key request pattern (baseline):**

```bash
curl "https://api.messari.io/metrics/v2/assets?assetSlugs=bitcoin,ethereum" \
  -H "x-messari-api-key: $MESSARI_API_KEY"
```

Use API-key mode for endpoints marked `api_key`-only.

**x402 request pattern (discovery + TypeScript payment client):**

1. Discover payable routes:

```bash
curl "https://api.messari.io/.well-known/x402"
```

2. Use an x402-enabled client for paid requests:

```bash
npm install @x402/fetch @x402/evm viem
```

```typescript
import { wrapFetchWithPaymentFromConfig } from '@x402/fetch';
import { ExactEvmScheme } from '@x402/evm';
import { privateKeyToAccount } from 'viem/accounts';

const privateKey = process.env.X402_PRIVATE_KEY as `0x${string}` | undefined;
if (!privateKey) {
  throw new Error('Set X402_PRIVATE_KEY');
}

const account = privateKeyToAccount(privateKey);
const fetchWithPayment = wrapFetchWithPaymentFromConfig(fetch, {
  schemes: [{ network: 'eip155:8453', client: new ExactEvmScheme(account) }],
});

const response = await fetchWithPayment('https://api.messari.io/ai/v2/chat/completions', {
  method: 'POST',
  headers: { 'Content-Type': 'application/json' },
  body: JSON.stringify({
    messages: [{ role: 'user', content: 'Summarize ETH market sentiment today.' }],
    stream: false,
  }),
});

if (!response.ok) {
  throw new Error(`Request failed: ${response.status}`);
}

console.log(await response.json());
```

`@x402/fetch` handles runtime `402 Payment Required` negotiation, parses `Payment-Required`, and retries with `Payment-Signature` (legacy compatibility: `X-PAYMENT`) after signing through `@x402/evm`.

**Secrets note:** Never commit credentials or signatures. Use placeholders only (`$MESSARI_API_KEY`, `$X402_PRIVATE_KEY`).

---

## Services and Endpoints

### AI Service

Chat completions trained on 30TB+ of structured and unstructured crypto data — market data, fundraising rounds, network metrics, research reports, newsletters, podcasts, and curated news.

Route general or open-ended crypto questions here first. This service synthesizes across all other data sources.

AI usage is paid. For credit-metered AI routes, API-key access may require Messari AI credits; x402 access uses runtime payment negotiation and does not require pre-purchased credits.

| Endpoint | Method | Authentication Method | Description |
|---|---|---|---|
| `/ai/v2/chat/completions` | POST | `api_key`, `x402` | Messari-native chat completions with structured response objects (x402-capable route) |
| `/ai/v1/chat/completions` | POST | `api_key` | Legacy Messari chat completion endpoint over Messari's crypto data corpus |
| `/ai/openai/chat/completions` | POST | `api_key` | OpenAI-compatible chat completion response format for drop-in client support |

**POST body:**
- `messages` — array of `{role, content}` message objects
- `stream` — boolean, enable streaming responses

---

### Metrics Service

Price, volume, market cap, and fundamental metrics for 34,000+ assets across 210+ exchanges. 175+ filterable metrics.

Route quantitative and comparative questions here — price lookups, performance comparisons, ROI, all-time highs, historical timeseries.

| Endpoint | Method | Authentication Method | Description |
|---|---|---|---|
| `/metrics/v2/assets` | GET | `api_key`, `x402` | List assets with market and fundamental metrics |
| `/metrics/v2/assets/metrics` | GET | `api_key`, `x402` | List dataset slugs and supported granularities for asset timeseries |
| `/metrics/v2/assets/ath` | GET | `api_key`, `x402` | Return all-time-high snapshots and drawdown context for selected assets |
| `/metrics/v2/assets/details` | GET | `api_key`, `x402` | Return rich point-in-time details for selected assets |
| `/metrics/v2/assets/roi` | GET | `api_key`, `x402` | Return multi-window ROI snapshots for selected assets |
| `/metrics/v2/assets/{assetID}/metrics/{datasetSlug}/time-series` | GET | `api_key`, `x402` | Return historical timeseries for an asset metric dataset |
| `/metrics/v2/assets/{assetID}/metrics/{datasetSlug}/time-series/{granularity}` | GET | `api_key`, `x402` | Return asset metric timeseries at explicit granularity (`5m`, `15m`, `1h`, `1d`) |

**Query parameters:**
- `assetSlugs` — comma-separated slugs (e.g., `bitcoin,ethereum`)
- `assetIds` — comma-separated asset IDs
- `metrics` — specific metrics to return
- `start`, `end` — date range (ISO 8601)
- `interval` — timeseries interval (`1d`, `1w`)
- `limit`, `page` — pagination

---

### Signal Service

Real-time social intelligence — sentiment scoring, mindshare tracking, trending narratives.

Route questions about market sentiment, social buzz, what's trending, mindshare gainers/losers here.

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

**Query parameters:**
- `assetIds` — comma-separated asset IDs/slugs to filter list responses
- `sort`, `sortDirection` — ranking field and direction for list responses
- `limit`, `page` — pagination controls
- `start`, `end` — date range for time-series calls (RFC3339 or unix timestamp)

---

### News Service

Real-time crypto news aggregation — breaking events, project updates, regulatory developments.

Route questions about recent headlines, current events, or "what happened with X" here.

| Endpoint | Method | Authentication Method | Description |
|---|---|---|---|
| `/news/v1/news/assets` | GET | `api_key` | List assets currently tagged in news coverage |
| `/news/v1/news/feed` | GET | `api_key`, `x402` | Paginated cross-source crypto news feed |
| `/news/v1/news/sources` | GET | `api_key`, `x402` | List source metadata and IDs for feed filtering |

**Query parameters:**
- `assetSlugs` — filter news by asset
- `sourceIds` — filter by news source
- `limit`, `page` — pagination

---

### Research Service

Institutional-grade reports — sector deep dives, protocol diligence, quarterly reviews, governance analysis.

Route questions about fundamental research, due diligence, analyst opinions, and sector analysis here.

| Endpoint | Method | Authentication Method | Description |
|---|---|---|---|
| `/research/v1/reports` | GET | `api_key` | List research reports with filters like asset IDs and tags |
| `/research/v1/reports/{reportId}` | GET | `api_key` | Retrieve a specific research report by report ID |
| `/research/v1/reports/tags` | GET | `api_key` | List research tags available for report filtering |

**Query parameters:**
- `tags` — filter by report tags
- `assetSlugs` — filter by related assets
- `limit`, `page` — pagination

---

### Stablecoins Service

On-chain metrics, historical timeseries, and per-chain breakdowns for 25+ stablecoins.

Route stablecoin-specific questions here — supply, flows, chain-level breakdowns, market share.

| Endpoint | Method | Authentication Method | Description |
|---|---|---|---|
| `/metrics/v2/stablecoins` | GET | `api_key`, `x402` | List stablecoins with supply and market metrics |
| `/metrics/v2/stablecoins/metrics` | GET | `api_key`, `x402` | List dataset slugs and supported granularities for stablecoin timeseries |
| `/metrics/v2/stablecoins/{entityIdentifier}/metrics/{datasetSlug}/time-series` | GET | `api_key`, `x402` | Return historical timeseries for a stablecoin metric dataset |
| `/metrics/v2/stablecoins/{entityIdentifier}/metrics/{datasetSlug}/time-series/1d` | GET | `api_key`, `x402` | Return daily stablecoin metric timeseries via explicit `1d` route |

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

| Endpoint | Method | Authentication Method | Description |
|---|---|---|---|
| `/metrics/v1/exchanges` | GET | `api_key` | List exchanges with spot/futures volume and open-interest context |
| `/metrics/v1/exchanges/metrics` | GET | `api_key` | List available metric datasets for exchange timeseries |
| `/metrics/v1/exchanges/{exchangeIdentifier}` | GET | `api_key` | Retrieve one exchange by slug or ID with current metrics |
| `/metrics/v1/exchanges/{entityIdentifier}/metrics/{datasetSlug}/time-series/{granularity}` | GET | `api_key` | Exchange metric timeseries at selected granularity |

**Query parameters:**
- `type` — filter exchanges by type
- `limit`, `page` — pagination controls
- `start`, `end` — date range for timeseries calls
- `granularity` — timeseries interval for metric calls

---

### Networks Service

L1/L2 blockchain network metrics — chain-level activity, fees, active addresses.

Route questions about blockchain networks, chain comparisons, or on-chain metrics here.

| Endpoint | Method | Authentication Method | Description |
|---|---|---|---|
| `/metrics/v2/networks` | GET | `api_key`, `x402` | List blockchain networks with activity, fee, and usage metrics |
| `/metrics/v2/networks/metrics` | GET | `api_key`, `x402` | List dataset slugs and supported granularities for network timeseries |
| `/metrics/v2/networks/{entityIdentifier}/metrics/{datasetSlug}/time-series` | GET | `api_key`, `x402` | Return historical timeseries for a network metric dataset |
| `/metrics/v2/networks/{entityIdentifier}/metrics/{datasetSlug}/time-series/{granularity}` | GET | `api_key`, `x402` | Return network metric timeseries at explicit granularity (`5m`, `15m`, `1h`, `1d`) |

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

**Query parameters:**
- `sort`, `order` — ranking column and sort direction for list endpoints
- `limit`, `page` — pagination controls
- `protocolIdentifier` — protocol slug or ID for timeseries endpoints
- `granularity`, `start`, `end` — timeseries interval and date range

---

### Token Unlocks Service

Vesting schedules, upcoming unlock events, and supply pressure analysis.

Route questions about token unlocks, vesting, or upcoming supply events here.

| Endpoint | Method | Authentication Method | Description |
|---|---|---|---|
| `/token-unlocks/v1/assets` | GET | `api_key`, `x402` | List assets covered by unlock and allocation datasets |
| `/token-unlocks/v1/allocations` | GET | `api_key`, `x402` | Allocation breakdown by bucket/category for selected assets |
| `/token-unlocks/v1/assets/{assetId}/events` | GET | `api_key`, `x402` | Upcoming token unlock events for a specific asset |
| `/token-unlocks/v1/assets/{assetId}/unlocks` | GET | `api_key`, `x402` | Historical unlock timeseries for a specific asset |
| `/token-unlocks/v1/assets/{assetId}/vesting-schedule` | GET | `api_key`, `x402` | Forward-looking vesting schedule timeseries for a specific asset |

**Query parameters:**
- `assetSlugs` — comma-separated asset slugs
- `start`, `end` — date range (ISO 8601)
- `limit`, `page` — pagination

---

### Fundraising Service

Funding rounds, investors, funds, organizations, projects, and M&A activity.

Route questions about who invested in what, fundraising rounds, investor activity, or M&A here.

| Endpoint | Method | Authentication Method | Description |
|---|---|---|---|
| `/funding/v1/rounds` | GET | `api_key`, `x402` | Query funding rounds by stage, date, amount, and participants |
| `/funding/v1/rounds/investors` | GET | `api_key`, `x402` | Return investors participating in rounds matching applied filters |
| `/funding/v1/mergers-and-acquisitions` | GET | `api_key`, `x402` | Query crypto M&A deals with acquirer/target metadata |
| `/funding/v1/organizations` | GET | `api_key`, `x402` | Query organizations active in fundraising and investment activity |
| `/funding/v1/projects` | GET | `api_key`, `x402` | Query projects and their fundraising attributes |
| `/funding/v1/funds` | GET | `api_key`, `x402` | Query investment funds and fund-level metadata |
| `/funding/v1/funds/managers` | GET | `api_key`, `x402` | Return organizations/people managing funds matching filters |

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

| Endpoint | Method | Authentication Method | Description |
|---|---|---|---|
| `/intel/v1/events` | GET | `api_key` | List intel events (legacy GET route; prefer POST for advanced filtering) |
| `/intel/v1/events` | POST | `api_key` | Preferred event query route with body-based filters |
| `/intel/v1/events/{eventId}` | GET | `api_key` | Retrieve one event with its update history |
| `/intel/v1/assets` | GET | `api_key` | List assets covered by the Intel event dataset |

**Query parameters:**
- `assetSlugs` — filter by asset
- `eventTypes` — filter by event type
- `start`, `end` — date range (ISO 8601)
- `limit`, `page` — pagination

---

### Topics Service

Trending topic classification and daily timeseries.

Route questions about trending narratives or topic momentum here.

| Endpoint | Method | Authentication Method | Description |
|---|---|---|---|
| `/topics/v1/classes` | GET | `api_key` | List available topic classes (taxonomy labels) |
| `/topics/v1/current` | GET | `api_key` | Snapshot of currently trending topics with rank and metadata |
| `/topics/v1/daily` | GET | `api_key` | Historical topic timeseries over a specified date range |

**Query parameters:**
- `classes` — filter current topics by class names
- `assetIDs` — filter topics by associated assets (UUID, slug, or symbol)
- `start`, `end`, `granularity` — time window controls for `/topics/v1/daily`
- `sort`, `limit`, `page` — sorting and pagination for `/topics/v1/current`

---

### X-Users Service

Crypto X/Twitter user metrics and influence tracking.

Route questions about crypto influencers, social account metrics, or X/Twitter activity here.

| Endpoint | Method | Authentication Method | Description |
|---|---|---|---|
| `/signal/v1/x-users` | GET | `api_key`, `x402` | Ranked crypto X-user signal feed with engagement and mindshare metrics |
| `/signal/v1/x-users/time-series/1d` | GET | `api_key`, `x402` | Daily X-user social-signal timeseries (x402-capable route) |
| `/signal/v1/x-users/{xUserID}` | GET | `api_key`, `x402` | Detailed social-signal profile for one X user |

**Query parameters:**
- `limit`, `page` — pagination controls
- `sort`, `sortDirection`, `accountType` — ranking/filter controls for list endpoint
- `start`, `end` — date range for timeseries calls

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
```
