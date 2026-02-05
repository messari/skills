# Messari API Services Reference

Detailed breakdown of each service available through the Messari MCP server.

## AI Service (`ai.*`)

Messari's AI completions endpoint, trained on 30TB+ of structured and unstructured crypto data.

**Data sources include:** market/price data, fundraising data, network metrics, Messari-exclusive
research reports, newsletters, podcasts, and curated third-party content (news, social posts).

**Use for:** open-ended crypto questions, synthesis across multiple data sources, market analysis,
protocol comparisons, narrative summaries.

**Requires:** Messari AI credits.

## Signal Service (`signal.*`)

Real-time social intelligence and narrative tracking.

**Capabilities:**
- Asset sentiment scoring
- Token mindshare tracking (share of social conversation)
- Trending topic and narrative detection
- Influencer activity monitoring

**Use for:** "what's the market talking about", sentiment shifts, narrative momentum,
social-driven alpha signals.

## Metrics Service (`metrics.*`)

Comprehensive quantitative data across the crypto market.

**Coverage:** 34,000+ assets, 210+ exchanges.

**Data types:**
- Price and volume (spot, historical timeseries)
- Market capitalization (circulating, fully diluted)
- Fundamental metrics (revenue, fees, active addresses, TVL)
- Asset profiles and metadata
- 175+ filterable metrics via screener

**Use for:** quantitative analysis, asset comparison, trend identification,
portfolio-level data pulls.

## News Service (`news.*`)

Real-time crypto news aggregation.

**Use for:** event-driven queries, breaking news, project updates, regulatory developments.

## Research Service (`research.*`)

Institutional-grade research from Messari's analyst team.

**Content types:** sector reports, protocol deep dives, quarterly reviews, diligence reports,
governance analysis.

**Use for:** fundamental research, due diligence, sector-level analysis, protocol evaluations.

## Stablecoins Service (`stablecoins.*`)

Dedicated stablecoin analytics.

**Coverage:** 25+ stablecoins with per-chain breakdowns.

**Data types:**
- Supply metrics (total, per-chain distribution)
- Historical timeseries
- On-chain flow data
- Market share tracking

**Use for:** stablecoin supply analysis, chain-level flow tracking, market share shifts.

## Derivatives Service (`derivatives.*`)

Crypto derivatives market data.

**Coverage:** 500+ assets.

**Data types:**
- Funding rates (perpetual futures)
- Open interest
- Futures volumes
- Liquidation data

**Use for:** derivatives market analysis, funding rate monitoring, positioning signals.

## Resource Filtering

Use `--resource` flags to limit which services the MCP server exposes:

- `'ai.*'` — AI completions only
- `'signal.*'` — Signal only
- `'metrics.*'` — Metrics only
- Multiple filters can be combined

Full docs: [docs.messari.io/mcp-server/hosted](https://docs.messari.io/mcp-server/hosted)
