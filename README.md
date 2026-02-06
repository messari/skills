# Messari Agent Skills

Crypto intelligence for AI agents — MCP, REST API, and agent-native skill files for [Messari](https://messari.io).

## For AI Agents

Point your agent at [`SKILL.md`](./SKILL.md) for self-contained integration instructions — all endpoints, authentication, routing logic, and examples in one file.

**Raw URL:** `https://raw.githubusercontent.com/messari/skills/master/SKILL.md`

## What's Here

### [Messari Hosted MCP Server](https://docs.messari.io/mcp-server/hosted)

Messari's hosted MCP server. Gives any MCP-compatible AI client direct access to Messari's data stack:

- **MessariAI** — Integrate with MessariAI to unlock all of our AI capabilities
- **Asset & Market Data** — Market data such as price, trading volume and marketcap for over 40,000 assets
- **Signals** — Track social data and narratives including mindshare and sentiment across individual crypto projects, or at the sector/sub-sector level
- **News** — Aggregated news from 500+ sources including headlines, links and summaries, with assets tagged per article
- **Research** — Access to Messari's Protocol Reporting Research
- **Stablecoins** — Onchain data for 30+ stablecoins such as USDT, USDC and DAI, including stablecoin marketcap, active addresses, transfers, transaction volume and more
- **Networks** — Onchain data for 25+ Layer-1s and Layer-2s such as Ethereum and Solana, including transaction fees, active addresses, and more
- **Protocols** — Onchain data for 200+ protocols such as Uniswap and Aave, including Total Value Locked, active addresses, deposits/borrows, trading volume, and more
- **Exchanges** — Spot volumes, futures volume and futures open interest across cryptocurrency exchanges
- **Token Unlocks** — Monitor token vesting schedules, upcoming unlock events, and token allocation details for over 200 crypto projects
- **Fundraising** — Crypto's biggest fundraising database for funding rounds, M&A deals, and investors
- **Intel** — Real-time onchain and offchain event monitoring with human curated summaries
- **Topics** — Topic-based content aggregation and trending narrative tracking

Full documentation: **[docs.messari.io/mcp-server/hosted](https://docs.messari.io/mcp-server/hosted)**

### [`openclaw-skill/`](./openclaw-skill/)

An [OpenClaw](https://openclaw.ai) / [ClawHub](https://clawhub.ai) skill that wraps Messari's REST API directly. OpenClaw does not natively support MCP, so this skill provides endpoint routing, authentication, and API reference documentation for standard HTTP requests.

## Quick Start

### 1. Get a Messari account

Sign up at [messari.io](https://messari.io) and get an [API key](https://messari.io/api).

### 2. Set up your client

#### Claude / Claude Code

Run from your terminal:

```bash
claude mcp add --transport http --scope user messari https://mcp.messari.io/mcp
```

Or add to your project's `.mcp.json`:

```json
{
  "mcpServers": {
    "messari": {
      "url": "https://mcp.messari.io/mcp"
    }
  }
}
```

#### Codex

Add to `~/.codex/config.toml`:

```toml
[mcp_servers.messari]
url = "https://mcp.messari.io/mcp"
```

#### Cursor

Add to `.cursor/mcp.json` (project-level) or `~/.cursor/mcp.json` (global):

```json
{
  "mcpServers": {
    "messari": {
      "url": "https://mcp.messari.io/mcp"
    }
  }
}
```

#### OpenClaw

OpenClaw does not support MCP. Install the skill instead:

```bash
clawhub install messari/messari-crypto
```

The skill calls `https://api.messari.io` directly with the `x-messari-api-key` header. See [`openclaw-skill/`](./openclaw-skill/) for details.

### 3. Start asking questions

```
"Which assets over $1B marketcap outperformed Bitcoin over the last 3 months?"
"What are some upcoming token unlock events this month?"
"Give me the 10 most recent fundraising rounds in the AI and compute sectors."
"What is the percent change in price for Aave and Maker over the last month?"
"What are the latest headlines related to crypto regulation?"
"What are some recent developments in the DePin sector?"
"Which investor has been the most active in seed rounds over the last year?"
"What is decentralized compute and why it is important to AI?"
"What were the top events for the AAVE protocol during the last quarter?"
"Give me the latest ecosystem map of Solana."
"What is the average DexVolume and total fee revenue last month for Ethereum, Solana and Base?"
"Tell me about the recent investments from a16z crypto."
"What are some trends that Messari has written about recently?"
"Has the GMX protocol undergone any audits?"
"Compare and contrast the native asset functions of BitTensor vs Render."
```

## Links

- [MCP Server Documentation](https://docs.messari.io/mcp-server/hosted)
- [Messari API](https://messari.io/api)
- [Messari](https://messari.io)
- [OpenClaw Skill on ClawHub](https://clawhub.ai/messari/messari-crypto)
