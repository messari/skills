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

- **x402 mode:** send request normally, handle `402 Payment Required`, then retry with `Payment-Signature` (legacy: `X-PAYMENT`).

**Secrets guardrail:** Never commit secret values. Use env vars only and placeholders like `$MESSARI_API_KEY` and `$X402_PRIVATE_KEY` in docs/examples.

All endpoints accept and return JSON. Use `Content-Type: application/json` for POST requests.

## x402 Payments

Some Messari endpoints support pay-per-request access via x402.

- Discover payable resources dynamically with `GET https://api.messari.io/.well-known/x402`.
- Treat the runtime `402 Payment Required` challenge as the source of truth for payable route and price.
- Do not hardcode x402 prices or payable-route assumptions in this skill.
- Use the Service Routing Table below as the authoritative service-level view of currently supported authentication methods.
- On x402-enabled routes, x402 is an alternative to API-key credit billing and does not require pre-purchased Messari AI credits.

**Negotiation flow:**
1. Send the request normally.
2. If the response is `402 Payment Required`, parse the payment requirements from the response body and the `Payment-Required` header.
3. Create/sign the payment payload and retry with `Payment-Signature` (legacy compatibility: `X-PAYMENT`).
4. Continue once the retried request succeeds.

**Budget guardrail:** If there is no pre-approved budget or prior user consent, ask the user to confirm before executing paid x402 requests.

## Service Routing Table

| Service | Base Path | Authentication Method | Use When |
|---|---|---|---|
| **AI** | `/ai/` | `api_key`, `x402` | General crypto questions, synthesis across data sources |
| **Signal** | `/signal/v1/` | `api_key`, `x402` | Sentiment, mindshare, trending narratives |
| **Metrics** | `/metrics/v2/` | `api_key`, `x402` | Price, volume, market cap, fundamentals |
| **News** | `/news/v1/` | `api_key`, `x402` | Real-time crypto news, breaking events |
| **Research** | `/research/v1/` | `api_key` | Institutional reports, protocol deep dives |
| **Stablecoins** | `/metrics/v2/stablecoins/` | `api_key`, `x402` | Stablecoin supply, per-chain breakdowns |
| **Exchanges** | `/metrics/v1/exchanges/` | `api_key` | Exchange volume, metrics, timeseries |
| **Networks** | `/metrics/v2/networks/` | `api_key`, `x402` | L1/L2 network metrics, timeseries |
| **Protocols** | `/metrics/v2/protocols/` | `api_key` | DeFi protocol metrics (DEX, lending, staking) |
| **Token Unlocks** | `/token-unlocks/v1/` | `api_key`, `x402` | Vesting schedules, unlock events |
| **Fundraising** | `/funding/v1/` | `api_key`, `x402` | Funding rounds, investors, M&A |
| **Intel** | `/intel/v1/` | `api_key` | Governance events, protocol updates |
| **Topics** | `/topics/v1/` | `api_key` | Trending topic classes, daily timeseries |
| **X-Users** | `/signal/v1/x-users/` | `api_key`, `x402` | Crypto X/Twitter user metrics |

For detailed endpoint documentation, see [references/api_services.md](references/api_services.md) or the full [Messari API docs](https://docs.messari.io/introduction).

## Example Requests

### AI Chat Completion

```bash
curl -X POST "https://api.messari.io/ai/v1/chat/completions" \
  -H "x-messari-api-key: $MESSARI_API_KEY" \
  -H "Content-Type: application/json" \
  -d '{
    "messages": [
      {"role": "user", "content": "What is the bull case for ETH right now?"}
    ]
  }'
```

### Asset Metrics Lookup

```bash
curl "https://api.messari.io/metrics/v2/assets?assetSlugs=bitcoin,ethereum" \
  -H "x-messari-api-key: $MESSARI_API_KEY"
```

### Signal Mindshare Gainers

```bash
curl "https://api.messari.io/signal/v1/assets/mindshare-gainers-24h?limit=10" \
  -H "x-messari-api-key: $MESSARI_API_KEY"
```

### News Feed

```bash
curl "https://api.messari.io/news/v1/news/feed?limit=20" \
  -H "x-messari-api-key: $MESSARI_API_KEY"
```

## Routing Guidance

### General crypto questions
Route through **AI** first — broadest context, synthesizes across market data, research, news, social.

### Quantitative questions
Use **Metrics** for price/volume/fundamentals. **Exchanges** for exchange-level data. **Networks** for L1/L2 metrics. **Protocols** for DeFi-specific data.

### Sentiment and narratives
**Signal** for mindshare and sentiment. **Topics** for trending narrative classes. **X-Users** for influencer-level metrics.

### Specific asset classes
**Stablecoins** for stablecoin supply and flows. **Token Unlocks** for vesting schedules and upcoming unlocks.

### Research, news, and events
**Research** for deep dives and reports. **News** for real-time events. **Intel** for governance and protocol updates. **Fundraising** for funding rounds and M&A.

### Multi-service queries
Combine services for richer answers. Example — "Is SOL overvalued?":
1. **Metrics** → current price, volume, fundamentals
2. **Signal** → sentiment and mindshare trend
3. **Token Unlocks** → upcoming supply pressure
4. **AI** → synthesize a view from all data
