---
name: messari
description: "Use this skill whenever the user asks any question about crypto assets, blockchain networks, DeFi protocols, token unlocks, fundraising rounds, market data, social sentiment, news, or governance — and live data from the Messari API would improve the answer. This includes questions like \"what's the price of X\", \"who invested in Y\", \"what are the top DeFi protocols by TVL\", \"what's trending in crypto\", \"show me upcoming token unlocks\", \"what's the latest news on Ethereum\", \"compare L1 network activity\", \"which exchanges have the most volume\", and any other crypto query. Always use this skill proactively when the user's query could benefit from real-time or historical crypto data — don't wait for the user to explicitly ask to \"use Messari\". If the user asks a general crypto question that requires synthesis or research, route to the Messari AI service first."
---

# Messari — Leading Crypto Data Platform

Answer crypto questions using the Messari API: 34,000+ assets, 210+ exchanges, 14 data services
covering market metrics, social sentiment, research, on-chain analytics, fundraising, governance,
and more.

**Base URL for all endpoints:** `https://api.messari.io`

---

## Step 1: Confirm Authentication

Follow this decision tree **before every session**. Do not skip to Step 2 until auth is confirmed.

```
1. Is `payments-mcp` available as a tool?
   ├── YES → Run `payments-mcp:get_wallet_balance`
   │          ├── Success → x402 is ready. Proceed to Step 2.
   │          └── Error   → Run `payments-mcp:show_wallet_app`, then prompt user (see Setup below)
   └── NO  → Is $MESSARI_API_KEY set or has the user provided an API key?
              ├── YES → API key mode. Include `x-messari-api-key: <key>` on all requests. Proceed to Step 2.
              └── NO  → Neither auth is configured. Prompt user (see Setup below).
```

### x402 Setup (Recommended — wallet-based, no API key needed)

If x402 is not yet configured, tell the user:

> **To use Messari, you'll need to set up a payment wallet first. Here's how:**
>
> 1. Run this command to install the payments connector (replace `claude` with your client if different):
>    ```
>    npx @coinbase/payments-mcp --client claude --auto-config
>    ```
>    Supported clients: `claude` | `claude-code` | `codex` | `gemini`
>
> 2. Restart your AI client to load the connector.
>
> 3. Come back and I'll open the wallet app for you to sign in.
>
> Once signed in, deposit some Base USDC to cover API requests (costs are fractions of a cent per call).

After the user has installed and restarted, run `payments-mcp:show_wallet_app` to open the wallet and prompt them to sign in and deposit USDC.

### API Key (Alternative)

If the user has a Messari API key, include this header on every request:
```
x-messari-api-key: <MESSARI_API_KEY>
```
Note: some endpoints are **only** available via API key (marked `api_key only` below).

---

## Step 2: Route to the Right Service

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

### Example query routing
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

---

## Step 3: Read detailed endpoint documentation and call the endpoint

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
