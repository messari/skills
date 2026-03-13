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

### Request Patterns

**API-key request pattern:**

```bash
curl "https://api.messari.io/metrics/v2/assets?assetSlugs=bitcoin,ethereum" \
  -H "x-messari-api-key: $MESSARI_API_KEY"
```

Use API-key mode for endpoints marked `api_key`-only.

**x402 request pattern:**

```python
import os
from dotenv import load_dotenv
from eth_account import Account
from x402 import x402ClientSync
from x402.http import x402HTTPClientSync
from x402.http.clients import x402_requests
from x402.mechanisms.evm import EthAccountSigner
from x402.mechanisms.evm.exact.register import register_exact_evm_client

load_dotenv()

requests_list = [
    {
        "method": "GET",
        "url": "https://api.messari.io/metrics/v2/assets/details?slugs=bitcoin",
    },
]

# Switch index to change request
r = requests_list[0]

account = Account.from_key(os.getenv("X402_PRIVATE_KEY"))
print(f"EVM address: {account.address}")

client = x402ClientSync()
register_exact_evm_client(client, EthAccountSigner(account))
http_client = x402HTTPClientSync(client)

print(f"Making request to: {r['url']}\n")

with x402_requests(client) as session:
    response = session.request(r["method"], r["url"], json=r.get("json"))
    print(f"Response status: {response.status_code}")
    print(f"Response body: {response.text}")
```

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

After deciding which service to use, read the detailed endpoint documentation from url in the "spec" column below before calling the endpoint. 

Many endpoints require an assetID, for these you should first call the unauthenticated lsit assets endpoint `/metrics/v2/assets?limit=200` with `curl` to get the assetID.

### AI Service
Chat completions trained on 30TB+ of structured crypto data. Handles synthesis, comparisons,
and open-ended research. x402 access does not require pre-purchased credits.

| Endpoint | Method | Spec |
|---|---|---|
| `/ai/v2/chat/completions` | POST | https://docs.messari.io/api-reference/endpoints/ai/post-v2-chat-completions.md |

**Body:** `{ "messages": [{"role": "user", "content": "..."}], "stream": false }`

---

### Metrics Service
Price, volume, market cap, fundamentals for 34,000+ assets. Use for quantitative questions,
performance comparisons, ROI, ATH, and historical timeseries. The NO AUTH REQUIRED endpoints are not authenticated so access them with `curl`

| Endpoint | Description | Spec |
|---|---|---|
| `/metrics/v2/assets` | [NO AUTH REQUIRED] List assets with market + fundamental metrics | https://docs.messari.io/api-reference/endpoints/metrics/v2/assets/get-v2-assets.md |
| `/metrics/v2/assets/details` | Rich point-in-time details for selected assets | https://docs.messari.io/api-reference/endpoints/metrics/v2/assets/get-v2-assets-details.md |
| `/metrics/v2/assets/ath` | All-time-high snapshots and drawdown context | https://docs.messari.io/api-reference/endpoints/metrics/v2/assets/get-v2-assets-ath.md |
| `/metrics/v2/assets/roi` | Multi-window ROI snapshots | https://docs.messari.io/api-reference/endpoints/metrics/v2/assets/get-v2-assets-roi.md |
| `/metrics/v2/assets/metrics` | [NO AUTH REQUIRED] List available dataset slugs + granularities | https://docs.messari.io/api-reference/endpoints/metrics/v2/assets/get-v2-assets-metrics.md |
| `/metrics/v2/assets/{assetID}/metrics/{datasetSlug}/time-series` | Historical timeseries | https://docs.messari.io/api-reference/endpoints/metrics/v2/assets/get-asset-timeseries.md |
| `/metrics/v2/assets/{assetID}/metrics/{datasetSlug}/time-series/{granularity}` | Timeseries at explicit granularity (`5m`, `15m`, `1h`, `1d`) | https://docs.messari.io/api-reference/endpoints/metrics/v2/assets/get-asset-timeseries.md |

**Key params:** `slugs`, `assetIds`, `metrics`, `start`, `end`, `interval`, `limit`, `page`

---

### Signal Service
Real-time social intelligence: sentiment scoring, mindshare tracking, trending.

| Endpoint | Description | Spec |
|---|---|---|
| `/signal/v1/assets` | Ranked asset signal feed (mindshare, sentiment, momentum) | https://docs.messari.io/api-reference/endpoints/signal/assets/get-v1-assets.md |
| `/signal/v1/assets/{assetID}` | Detailed social-signal profile for one asset | https://docs.messari.io/api-reference/endpoints/signal/assets/get-v1-assets-assetID.md |
| `/signal/v1/assets/mindshare-gainers-24h` | Biggest mindshare gains last 24h | https://docs.messari.io/api-reference/endpoints/signal/assets/get-v1-assets-mindshare-gainers-24h.md |
| `/signal/v1/assets/mindshare-gainers-7d` | Biggest mindshare gains last 7d | https://docs.messari.io/api-reference/endpoints/signal/assets/get-v1-assets-mindshare-gainers-7d.md |
| `/signal/v1/assets/mindshare-losers-24h` | Biggest mindshare losses last 24h | https://docs.messari.io/api-reference/endpoints/signal/assets/get-v1-assets-mindshare-losers-24h.md |
| `/signal/v1/assets/mindshare-losers-7d` | Biggest mindshare losses last 7d | https://docs.messari.io/api-reference/endpoints/signal/assets/get-v1-assets-mindshare-losers-7d.md |
| `/signal/v1/assets/time-series/1h` | Hourly social-signal timeseries | https://docs.messari.io/api-reference/endpoints/signal/assets/get-v1-assets-time-series-granularity.md |
| `/signal/v1/assets/time-series/1d` | Daily social-signal timeseries | https://docs.messari.io/api-reference/endpoints/signal/assets/get-v1-assets-time-series-granularity.md |

**Key params:** `assetIds`, `sort`, `sortDirection`, `limit`, `page`, `start`, `end`

---

### News Service

| Endpoint | Auth | Description | Spec |
|---|---|---|---|
| `/news/v1/news/feed` | api_key, x402 | Paginated cross-source crypto news feed | https://docs.messari.io/api-reference/endpoints/news/get-v1-news-feed.md |
| `/news/v1/news/sources` | api_key, x402 | List source metadata for filtering | https://docs.messari.io/api-reference/endpoints/news/get-v1-news-sources.md |

**Key params:** `assetSlugs`, `sourceIds`, `limit`, `page`

---

### Research Service *(api_key only)*
Institutional reports, sector deep dives, protocol diligence.

| Endpoint | Description | Spec |
|---|---|---|
| `/research/v1/reports` | List reports (filter by asset, tags) | https://docs.messari.io/api-reference/endpoints/research/get-v1-reports.md |
| `/research/v1/reports/{reportId}` | Retrieve a specific report | https://docs.messari.io/api-reference/endpoints/research/get-v1-reports-reportId.md |
| `/research/v1/reports/tags` | List available report tags | https://docs.messari.io/api-reference/endpoints/research/get-v1-reports-tags.md |

**Key params:** `tags`, `assetSlugs`, `limit`, `page`

---

### Stablecoins Service

| Endpoint | Description | Spec |
|---|---|---|
| `/metrics/v2/stablecoins` | [NO AUTH REQUIRED] List stablecoins with supply + market metrics | https://docs.messari.io/api-reference/endpoints/stablecoins/list-stablecoins.md |
| `/metrics/v2/stablecoins/metrics` | List dataset slugs + granularities | https://docs.messari.io/api-reference/endpoints/stablecoins/list-stablecoin-metrics.md |
| `/metrics/v2/stablecoins/{entityIdentifier}/metrics/{datasetSlug}/time-series` | Historical timeseries | https://docs.messari.io/api-reference/endpoints/stablecoins/stablecoin-timeseries-metric.md |

**Key params:** `metrics`, `chains`, `start`, `end`, `interval`, `limit`, `page`

---

### Exchanges Service *(api_key only)*

| Endpoint | Description | Spec |
|---|---|---|
| `/metrics/v1/exchanges` | [NO AUTH REQUIRED] List exchanges with spot/futures volume | https://docs.messari.io/api-reference/endpoints/exchanges/list-exchanges.md |
| `/metrics/v1/exchanges/{exchangeIdentifier}` | Single exchange with current metrics | https://docs.messari.io/api-reference/endpoints/exchanges/get-exchange.md |
| `/metrics/v1/exchanges/metrics` | Available metric datasets | https://docs.messari.io/api-reference/endpoints/exchanges/list-exchanges-metrics.md |
| `/metrics/v1/exchanges/{entityIdentifier}/metrics/{datasetSlug}/time-series/{granularity}` | Exchange timeseries | https://docs.messari.io/api-reference/endpoints/exchanges/get-exchange-timeseries-metric.md |

**Key params:** `type`, `granularity`, `start`, `end`, `limit`, `page`

---

### Networks Service
L1/L2 on-chain activity — fees, active addresses, usage metrics.

| Endpoint | Description | Spec |
|---|---|---|
| `/metrics/v2/networks` | List networks with activity + fee metrics | https://docs.messari.io/api-reference/endpoints/metrics/v2/networks/get-v2-networks.md |
| `/metrics/v2/networks/metrics` | List available dataset slugs + granularities | https://docs.messari.io/api-reference/endpoints/metrics/v2/networks/get-v2-networks-metrics.md |
| `/metrics/v2/networks/{entityIdentifier}/metrics/{datasetSlug}/time-series` | Historical timeseries | https://docs.messari.io/api-reference/endpoints/metrics/v2/networks/get-network-timeseries.md |
| `/metrics/v2/networks/{entityIdentifier}/metrics/{datasetSlug}/time-series/{granularity}` | Timeseries at `5m`, `15m`, `1h`, `1d` | https://docs.messari.io/api-reference/endpoints/metrics/v2/networks/get-time-series-timeseries-with-granularity.md |

**Key params:** `networkSlugs`, `metrics`, `start`, `end`, `interval`, `limit`, `page`

---

### Protocols Service *(api_key only)*
DeFi protocols: TVL, DEX, lending, liquid staking, bridges.

| Endpoint | Description | Spec |
|---|---|---|
| `/metrics/v2/protocols` | All protocols with aggregated core metrics | https://docs.messari.io/api-reference/endpoints/protocols/core/list-protocols.md |
| `/metrics/v2/protocols/dex` | DEX protocols | https://docs.messari.io/api-reference/endpoints/protocols/dex/list-dex-protocols.md |
| `/metrics/v2/protocols/lending` | Lending protocols | https://docs.messari.io/api-reference/endpoints/protocols/lending/list-lending-protocols.md |
| `/metrics/v2/protocols/liquid-staking` | Liquid staking protocols | https://docs.messari.io/api-reference/endpoints/protocols/liquid-staking/list-liquid-staking-protocols.md |
| `/metrics/v2/protocols/interop` | Bridge/interop protocols | https://docs.messari.io/api-reference/endpoints/protocols/interoperability/list-interop-protocols.md |
| `/metrics/v2/protocols/{protocolIdentifier}/metrics/core/time-series/{granularity}` | Core timeseries | https://docs.messari.io/api-reference/endpoints/protocols/core/protocol-timeseries-metric.md |
| `/metrics/v2/protocols/{protocolIdentifier}/metrics/dex/time-series/{granularity}` | DEX timeseries | https://docs.messari.io/api-reference/endpoints/protocols/dex/dex-timeseries-metric.md |
| `/metrics/v2/protocols/{protocolIdentifier}/metrics/lending/time-series/{granularity}` | Lending timeseries | https://docs.messari.io/api-reference/endpoints/protocols/lending/lending-timeseries-metric.md |
| `/metrics/v2/protocols/{protocolIdentifier}/metrics/liquid-staking/time-series/{granularity}` | Liquid staking timeseries | https://docs.messari.io/api-reference/endpoints/protocols/liquid-staking/liquid-staking-timeseries-metric.md |
| `/metrics/v2/protocols/{protocolIdentifier}/metrics/interop/time-series/{granularity}` | Interop timeseries | https://docs.messari.io/api-reference/endpoints/protocols/interoperability/interop-timeseries-metric.md |

**Key params:** `sort`, `order`, `limit`, `page`, `granularity`, `start`, `end`

---

### Token Unlocks Service

Before calling the endpoint, you should first curl the `/token-unlocks/v1/assets?limit=100` endpoint to get the assetID to filter by. This endpoint is not authenticated so access it with `curl`
| Endpoint | Description | Spec |
|---|---|---|
| `/token-unlocks/v1/assets` | [NO AUTH REQUIRED] Assets covered by unlock datasets | https://docs.messari.io/api-reference/endpoints/token-unlocks/get-v1-assets.md |
| `/token-unlocks/v1/allocations` | Allocation breakdown by category | https://docs.messari.io/api-reference/endpoints/token-unlocks/get-v1-allocations.md |
| `/token-unlocks/v1/assets/{assetId}/events` | Upcoming unlock events | https://docs.messari.io/api-reference/endpoints/token-unlocks/get-v1-assets-assetId-events.md |
| `/token-unlocks/v1/assets/{assetId}/unlocks` | Historical unlock timeseries | https://docs.messari.io/api-reference/endpoints/token-unlocks/get-v1-assets-assetId-unlocks.md |
| `/token-unlocks/v1/assets/{assetId}/vesting-schedule` | Forward-looking vesting schedule | https://docs.messari.io/api-reference/endpoints/token-unlocks/get-v1-assets-assetId-vesting-schedule.md |

**Key params:** `assetIDs`, `start`, `end`, `limit`, `page`

---

### Fundraising Service

| Endpoint | Description | Spec |
|---|---|---|
| `/funding/v1/rounds` | Funding rounds by stage, date, amount, participants | https://docs.messari.io/api-reference/endpoints/funding/fundraising-rounds/get-v1-rounds.md |
| `/funding/v1/rounds/investors` | Investors in matching rounds | https://docs.messari.io/api-reference/endpoints/funding/fundraising-rounds/get-v1-rounds-investors.md |
| `/funding/v1/mergers-and-acquisitions` | Crypto M&A deals | https://docs.messari.io/api-reference/endpoints/funding/mergers-acquisitions/get-v1-mergers-and-acquisitions.md |
| `/funding/v1/organizations` | Organizations in fundraising/investment | https://docs.messari.io/api-reference/endpoints/funding/entities/get-v1-organizations.md |
| `/funding/v1/projects` | Projects and fundraising attributes | https://docs.messari.io/api-reference/endpoints/funding/entities/get-v1-projects.md |
| `/funding/v1/funds` | Investment funds and fund metadata | https://docs.messari.io/api-reference/endpoints/funding/funds/get-v1-funds.md |
| `/funding/v1/funds/managers` | Fund managers | https://docs.messari.io/api-reference/endpoints/funding/funds/get-v1-funds-managers.md |

**Key params:** `assetSlugs`, `investorSlugs`, `roundTypes` (`seed`, `series-a`, …), `start`, `end`, `limit`, `page`

---

### Intel Service *(api_key only)*
Governance events, protocol upgrades, project milestones.

| Endpoint | Method | Description | Spec |
|---|---|---|---|
| `/intel/v1/events` | GET | List events (legacy; prefer POST) | https://docs.messari.io/api-reference/endpoints/intel/get-v1-events.md |
| `/intel/v1/events` | POST | Preferred — body-based filters | https://docs.messari.io/api-reference/endpoints/intel/post-v1-events.md |
| `/intel/v1/events/{eventId}` | GET | Single event with update history | https://docs.messari.io/api-reference/endpoints/intel/get-v1-events-eventId.md |
| `/intel/v1/assets` | GET | Assets covered by Intel dataset | https://docs.messari.io/api-reference/endpoints/intel/get-v1-assets.md |

**Key params:** `assetSlugs`, `eventTypes`, `start`, `end`, `limit`, `page`

---

### Topics Service *(api_key only)*

| Endpoint | Description | Spec |
|---|---|---|
| `/topics/v1/classes` | Available topic taxonomy labels | https://docs.messari.io/api-reference/endpoints/topics/get-v1-classes.md |
| `/topics/v1/current` | Currently trending topics with rank | https://docs.messari.io/api-reference/endpoints/topics/get-v1-current.md |
| `/topics/v1/daily` | Historical topic timeseries | https://docs.messari.io/api-reference/endpoints/topics/get-v1-daily.md |

**Key params:** `classes`, `assetIDs`, `start`, `end`, `granularity`, `sort`, `limit`, `page`

---

### X-Users Service
Crypto X/Twitter influencer metrics and rankings.

| Endpoint | Description | Spec |
|---|---|---|
| `/signal/v1/x-users` | Ranked crypto X-user feed (engagement, mindshare) | https://docs.messari.io/api-reference/endpoints/signal/x-users/get-v1-x-users.md |
| `/signal/v1/x-users/time-series/1d` | Daily X-user signal timeseries | https://docs.messari.io/api-reference/endpoints/signal/x-users/get-v1-x-users-time-series-granularity.md |
| `/signal/v1/x-users/{xUserID}` | Detailed profile for one X user | https://docs.messari.io/api-reference/endpoints/signal/x-users/get-v1-x-users-xUserID.md |

**Key params:** `sort`, `sortDirection`, `accountType`, `limit`, `page`, `start`, `end`
