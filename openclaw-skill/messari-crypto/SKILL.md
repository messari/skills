---
name: messari-crypto
description: Messari crypto market intelligence via REST API with one-of credentials for x402-enabled routes (MESSARI_API_KEY or X402_PRIVATE_KEY); use for AI, metrics, signal, news, research, stablecoins, exchanges, networks, protocols, token unlocks, fundraising, intel, topics, and X-users data.
homepage: https://github.com/messari/skills
metadata: {"openclaw":{"homepage":"https://github.com/messari/skills","primaryEnv":"MESSARI_API_KEY"}}
---

# Messari Crypto Intel

Real-time crypto market intelligence via Messari's REST API — AI-powered analysis,
on-chain metrics, sentiment, news, and institutional-grade research without building data pipelines.

## Prerequisites

- **Credential Mode A: API key** (`MESSARI_API_KEY`) — [sign up for an API key](https://messari.io/api) or [retrieve your existing key](https://messari.io/account/api). On credit-metered endpoints (for example, Messari AI), API-key access may require Messari AI credits (manage credits at [messari.io/account](https://messari.io/account)).
- **Credential Mode B: x402 pay-per-request** (`X402_PRIVATE_KEY`) — use x402 negotiation on x402-enabled routes; this flow does not require pre-purchased Messari AI credits.
- **Coverage note:** For full endpoint coverage, configure API key; x402-only credentials are limited to x402-enabled routes.

## REST API Overview

**Base URL:** `https://api.messari.io`

**Authentication modes:**
- **API key mode:** include API key header:

```
x-messari-api-key: <YOUR_API_KEY>
```

- **x402 mode:** send request normally, handle `402 Payment Required`, then retry with `Payment-Signature` (legacy: `X-PAYMENT`). See the sample code below for how to do this.

**Secrets guardrail:** Never commit secret values. Use env vars only and placeholders like `$MESSARI_API_KEY` and `$X402_PRIVATE_KEY` in docs/examples.

All endpoints accept and return JSON. Use `Content-Type: application/json` for POST requests.

## x402 Payments

Some Messari endpoints support pay-per-request access via x402.

- Discover payable resources dynamically with `GET https://api.messari.io/.well-known/x402`.
- Treat the runtime `402 Payment Required` challenge as the source of truth for payable route and price.
- Do not hardcode x402 prices or payable-route assumptions in this skill.
- Use the Service Routing Table below as the authoritative service-level view of currently supported authentication methods.
- On x402-enabled routes, x402 is an alternative to API-key.

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

**x402 request pattern (TypeScript payment client):**


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

## Service Routing Table

| User is asking about... | Service | Auth |
|---|---|---|
| General crypto question, synthesis, open-ended research | **AI** | api_key, x402 |
| Price, volume, market cap, ROI, ATH, performance comparison | **Metrics** | api_key, x402 |
| Sentiment, mindshare, trending tokens, social buzz | **Signal** | api_key, x402 |
| Headlines, recent events, breaking news | **News** | api_key, x402 |
| Analyst reports, deep dives, sector overviews | **Research** | api_key only |
| Stablecoin supply, flows, chain breakdowns | **Stablecoins** | api_key, x402 |
| Exchange volumes, comparisons | **Exchanges** | api_key only |
| L1/L2 network activity, fees, active addresses | **Networks** | api_key, x402 |
| DeFi protocols, TVL, lending, DEX volume | **Protocols** | api_key only |
| Token unlocks, vesting schedules | **Token Unlocks** | api_key, x402 |
| Fundraising rounds, investors, VC activity, M&A | **Fundraising** | api_key, x402 |
| Governance events, protocol upgrades | **Intel** | api_key only |
| Trending narratives, topic momentum | **Topics** | api_key only |
| Crypto influencers, X/Twitter accounts | **X-Users** | api_key, x402 |

**When in doubt, start with the AI service** — it synthesizes across all sources and handles
open-ended questions well.

For detailed endpoint documentation, see "API Service Directory" below or the full [Messari API docs](https://docs.messari.io/introduction).

## Example Request Routing
```
"Tell me about x402 and how it works."              → AI       /ai/v2/chat/completions
"What are upcoming token unlocks this month?"       → Unlocks  /token-unlocks/v1/assets
"10 most recent AI/compute fundraising rounds"      → Funding  /funding/v1/rounds
"Latest crypto regulation headlines"                → News     /news/v1/news/feed
"Recent DePIN sector developments"                  → Research /research/v1/reports
"Most active seed investor over last year"          → Funding  /funding/v1/rounds/investors
"Top AAVE events last quarter"                      → Intel    /intel/v1/events
"Solana ecosystem map"                              → Networks /metrics/v2/networks
"Recent a16z crypto investments"                    → Funding  /funding/v1/projects
"Compare BitTensor vs Render native assets"         → Metrics  /metrics/v2/assets/details
```

## API Service Directory

### AI Service
Chat completions trained on 30TB+ of structured crypto data. Handles synthesis, comparisons,
and open-ended research. x402 access does not require pre-purchased credits.

| Endpoint | Method |
|---|---|
| `/ai/v2/chat/completions` | POST |

**Body:** `{ "messages": [{"role": "user", "content": "..."}], "stream": false }`

---

### Metrics Service
Price, volume, market cap, fundamentals for 34,000+ assets. Use for quantitative questions,
performance comparisons, ROI, ATH, and historical timeseries.

| Endpoint | Description |
|---|---|
| `/metrics/v2/assets` | List assets with market + fundamental metrics |
| `/metrics/v2/assets/details` | Rich point-in-time details for selected assets |
| `/metrics/v2/assets/ath` | All-time-high snapshots and drawdown context |
| `/metrics/v2/assets/roi` | Multi-window ROI snapshots |
| `/metrics/v2/assets/metrics` | List available dataset slugs + granularities |
| `/metrics/v2/assets/{assetID}/metrics/{datasetSlug}/time-series` | Historical timeseries |
| `/metrics/v2/assets/{assetID}/metrics/{datasetSlug}/time-series/{granularity}` | Timeseries at explicit granularity (`5m`, `15m`, `1h`, `1d`) |

**Key params:** `slugs`, `assetIds`, `metrics`, `start`, `end`, `interval`, `limit`, `page`

---

### Signal Service
Real-time social intelligence: sentiment scoring, mindshare tracking, trending.

| Endpoint | Description |
|---|---|
| `/signal/v1/assets` | Ranked asset signal feed (mindshare, sentiment, momentum) |
| `/signal/v1/assets/{assetID}` | Detailed social-signal profile for one asset |
| `/signal/v1/assets/mindshare-gainers-24h` | Biggest mindshare gains last 24h |
| `/signal/v1/assets/mindshare-gainers-7d` | Biggest mindshare gains last 7d |
| `/signal/v1/assets/mindshare-losers-24h` | Biggest mindshare losses last 24h |
| `/signal/v1/assets/mindshare-losers-7d` | Biggest mindshare losses last 7d |
| `/signal/v1/assets/time-series/1h` | Hourly social-signal timeseries |
| `/signal/v1/assets/time-series/1d` | Daily social-signal timeseries |

**Key params:** `assetIds`, `sort`, `sortDirection`, `limit`, `page`, `start`, `end`

---

### News Service

| Endpoint | Auth | Description |
|---|---|---|
| `/news/v1/news/feed` | api_key, x402 | Paginated cross-source crypto news feed |
| `/news/v1/news/sources` | api_key, x402 | List source metadata for filtering |
| `/news/v1/news/assets` | api_key only | Assets currently tagged in news |

**Key params:** `assetSlugs`, `sourceIds`, `limit`, `page`

---

### Research Service *(api_key only)*
Institutional reports, sector deep dives, protocol diligence.

| Endpoint | Description |
|---|---|
| `/research/v1/reports` | List reports (filter by asset, tags) |
| `/research/v1/reports/{reportId}` | Retrieve a specific report |
| `/research/v1/reports/tags` | List available report tags |

**Key params:** `tags`, `assetSlugs`, `limit`, `page`

---

### Stablecoins Service

| Endpoint | Description |
|---|---|
| `/metrics/v2/stablecoins` | List stablecoins with supply + market metrics |
| `/metrics/v2/stablecoins/metrics` | List dataset slugs + granularities |
| `/metrics/v2/stablecoins/{entityIdentifier}/metrics/{datasetSlug}/time-series` | Historical timeseries |
| `/metrics/v2/stablecoins/{entityIdentifier}/metrics/{datasetSlug}/time-series/1d` | Daily timeseries |

**Key params:** `metrics`, `chains`, `start`, `end`, `interval`, `limit`, `page`

---

### Exchanges Service *(api_key only)*

| Endpoint | Description |
|---|---|
| `/metrics/v1/exchanges` | List exchanges with spot/futures volume |
| `/metrics/v1/exchanges/{exchangeIdentifier}` | Single exchange with current metrics |
| `/metrics/v1/exchanges/metrics` | Available metric datasets |
| `/metrics/v1/exchanges/{entityIdentifier}/metrics/{datasetSlug}/time-series/{granularity}` | Exchange timeseries |

**Key params:** `type`, `granularity`, `start`, `end`, `limit`, `page`

---

### Networks Service
L1/L2 on-chain activity — fees, active addresses, usage metrics.

| Endpoint | Description |
|---|---|
| `/metrics/v2/networks` | List networks with activity + fee metrics |
| `/metrics/v2/networks/metrics` | List available dataset slugs + granularities |
| `/metrics/v2/networks/{entityIdentifier}/metrics/{datasetSlug}/time-series` | Historical timeseries |
| `/metrics/v2/networks/{entityIdentifier}/metrics/{datasetSlug}/time-series/{granularity}` | Timeseries at `5m`, `15m`, `1h`, `1d` |

**Key params:** `networkSlugs`, `metrics`, `start`, `end`, `interval`, `limit`, `page`

---

### Protocols Service *(api_key only)*
DeFi protocols: TVL, DEX, lending, liquid staking, bridges.

| Endpoint | Description |
|---|---|
| `/metrics/v2/protocols` | All protocols with aggregated core metrics |
| `/metrics/v2/protocols/dex` | DEX protocols |
| `/metrics/v2/protocols/lending` | Lending protocols |
| `/metrics/v2/protocols/liquid-staking` | Liquid staking protocols |
| `/metrics/v2/protocols/interop` | Bridge/interop protocols |
| `/metrics/v2/protocols/{protocolIdentifier}/metrics/core/time-series/{granularity}` | Core timeseries |
| `/metrics/v2/protocols/{protocolIdentifier}/metrics/dex/time-series/{granularity}` | DEX timeseries |
| `/metrics/v2/protocols/{protocolIdentifier}/metrics/lending/time-series/{granularity}` | Lending timeseries |
| `/metrics/v2/protocols/{protocolIdentifier}/metrics/liquid-staking/time-series/{granularity}` | Liquid staking timeseries |
| `/metrics/v2/protocols/{protocolIdentifier}/metrics/interop/time-series/{granularity}` | Interop timeseries |

**Key params:** `sort`, `order`, `limit`, `page`, `granularity`, `start`, `end`

---

### Token Unlocks Service

| Endpoint | Description |
|---|---|
| `/token-unlocks/v1/assets` | Assets covered by unlock datasets |
| `/token-unlocks/v1/allocations` | Allocation breakdown by category |
| `/token-unlocks/v1/assets/{assetId}/events` | Upcoming unlock events |
| `/token-unlocks/v1/assets/{assetId}/unlocks` | Historical unlock timeseries |
| `/token-unlocks/v1/assets/{assetId}/vesting-schedule` | Forward-looking vesting schedule |

**Key params:** `assetSlugs`, `start`, `end`, `limit`, `page`

---

### Fundraising Service

| Endpoint | Description |
|---|---|
| `/funding/v1/rounds` | Funding rounds by stage, date, amount, participants |
| `/funding/v1/rounds/investors` | Investors in matching rounds |
| `/funding/v1/mergers-and-acquisitions` | Crypto M&A deals |
| `/funding/v1/organizations` | Organizations in fundraising/investment |
| `/funding/v1/projects` | Projects and fundraising attributes |
| `/funding/v1/funds` | Investment funds and fund metadata |
| `/funding/v1/funds/managers` | Fund managers |

**Key params:** `assetSlugs`, `investorSlugs`, `roundTypes` (`seed`, `series-a`, …), `start`, `end`, `limit`, `page`

---

### Intel Service *(api_key only)*
Governance events, protocol upgrades, project milestones.

| Endpoint | Method | Description |
|---|---|---|
| `/intel/v1/events` | GET | List events (legacy; prefer POST) |
| `/intel/v1/events` | POST | Preferred — body-based filters |
| `/intel/v1/events/{eventId}` | GET | Single event with update history |
| `/intel/v1/assets` | GET | Assets covered by Intel dataset |

**Key params:** `assetSlugs`, `eventTypes`, `start`, `end`, `limit`, `page`

---

### Topics Service *(api_key only)*

| Endpoint | Description |
|---|---|
| `/topics/v1/classes` | Available topic taxonomy labels |
| `/topics/v1/current` | Currently trending topics with rank |
| `/topics/v1/daily` | Historical topic timeseries |

**Key params:** `classes`, `assetIDs`, `start`, `end`, `granularity`, `sort`, `limit`, `page`

---

### X-Users Service
Crypto X/Twitter influencer metrics and rankings.

| Endpoint | Description |
|---|---|
| `/signal/v1/x-users` | Ranked crypto X-user feed (engagement, mindshare) |
| `/signal/v1/x-users/time-series/1d` | Daily X-user signal timeseries |
| `/signal/v1/x-users/{xUserID}` | Detailed profile for one X user |

**Key params:** `sort`, `sortDirection`, `accountType`, `limit`, `page`, `start`, `end`
