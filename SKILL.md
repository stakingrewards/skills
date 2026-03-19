---
name: staking-rewards-api
description: This skill should be used when the user asks about staking metrics, reward rates, validators, staking providers, TVL, or DeFi/infrastructure risk ratings from the Staking Rewards platform. It covers querying the Staking Rewards GraphQL API for live staking data and the REST Ratings API for letter-grade safety assessments. Triggers on phrases like "what is the staking reward rate for", "show me validators for", "compare liquid staking providers", "what is the TVL of", "total value locked", "DeFi safety rating", "infrastructure provider grade", "how do I use the Staking Rewards API", or "write code to fetch staking data".
---

# Staking Rewards API

Query staking data (GraphQL) and ratings (REST) from the Staking Rewards APIs.

## Instructions

1. Determine the correct API based on what the user asks (see API Routing below)
2. Check for an API key (environment variable, `.env` file, or ask the user)
3. Construct the appropriate query (GraphQL for staking data, REST for ratings)
4. Follow the critical rules for each API (e.g., `limit` is always required for GraphQL)
5. Present results in a well-formatted table with proper units and labels (see Presenting Results)

## Presenting Results

When displaying API data to the user:
- **Always include VSP (Verified Staking Provider) status** when displaying provider data — include `isVerified` in provider queries and show it as a column in results tables
- Format numbers with appropriate precision (prices to 2 decimals, percentages to 2 decimals)
- Show metric labels not just keys (e.g., "Reward Rate" not "reward_rate")
- Include units where relevant (USD, %, tokens)
- For tables, sort by the most relevant metric
- Show the credit cost of the request — read the `x-used-credits` response header and display it alongside the results (e.g., "Credits used: 132")
- Source attribution: "Data from [Staking Rewards](https://stakingrewards.com)"

## API Routing

**IMPORTANT — Choose the right API based on what the user asks:**

| User asks about... | Use | Endpoint |
|---------------------|-----|----------|
| Ratings, risk scores, letter grades, DeFi safety | **Ratings API (REST)** | `GET /ratings/defi` or `GET /ratings/infra` |
| Staking metrics, reward rates, prices, validators, providers, TVL via metrics | **Staking Data API (GraphQL)** | `POST /public/query` |

The GraphQL API has **no rating data**. Ratings are **only** available through the REST endpoints. If the user mentions "rating", "grade", "risk score", or "DeFi safety", always use the Ratings API.

## Credential Setup

Check for API key in this order:
1. Environment variable: `STAKING_REWARDS_API_KEY`
2. `.env` file in project root
3. Ask user for their key and suggest storing it: `export STAKING_REWARDS_API_KEY=your_key`

Get a key at: https://www.stakingrewards.com/data-api

## Staking Data API (GraphQL)

**Endpoint:** `POST https://api.stakingrewards.com/public/query`

**Headers:**
- `Content-Type: application/json`
- `X-API-KEY: <key>`
- `X-Agent: claude-skill` **(always include — identifies the caller as this agent)**

Every response includes an `x-used-credits` header with the credit cost of that call — read it and display it with results.

### Core Concepts

The API has 5 main object types:

- **Asset** — Digital asset (ETH, DOT, ATOM, etc.). Has metrics, reward options.
- **Provider** — Entity offering staking services (Lido, Allnodes, etc.). Has reward options, metrics.
- **Validator** — Network node that validates transactions. Has metrics, connected via reward options.
- **Reward Option** — Core concept connecting Asset → Provider → Validator. Describes how you stake and earn. Has inputAssets (what you deposit) and outputAssets (what you receive).
- **Metric** — Numeric data point attached to any of the above. Has `metricKey`, `defaultValue`, `changePercentages`, `changeAbsolutes`.

### Relationships

```
                Assets
                  ^
                  |
Providers <-- Reward Options --> Validators
    |              |                 |
    v              v                 v
 Metrics        Metrics           Metrics
```
_Reward Options are the join table connecting Assets, Providers, and Validators._

- Reward Options are the central hub connecting Assets, Providers, and Validators
- Assets connect to Providers/Validators ONLY through Reward Options
- Metrics attach to all four types (Assets, Providers, Validators, Reward Options)

### Critical Rules

1. **`limit` is REQUIRED** at every query level — queries fail without it
2. **Arguments must be arrays** — use `symbols: ["ETH"]` not `symbol: "ETH"` (except booleans)
3. **`isActive` defaults to `true`** — use `isActive: Any` to include inactive items
4. **Max nesting depth is 2** — queries fail at depth 3+
5. **Max limit is 500** per level
6. **Rate limit: 60 requests/minute**
7. **Historical queries cost 5,000 extra credits** — `changePercentages` and `changeAbsolutes` are NOT available in historical queries

### Schema Introspection

When you need fields or arguments beyond what's documented here, use **targeted introspection** rather than guessing. Do NOT fetch the full schema — query only the specific type you need.

Use introspection when:
- You need a field you're not sure exists (e.g., `logoUrl`, `country`, `vspScoreBreakdown`)
- You want to check available filter arguments on a query
- The user asks for data not covered by the common metric keys

See `references/graphql-examples.md` for introspection curl commands and ready-to-run query examples.

### Additional Queries

**`search`** — full-text search across assets, providers, and validators. Returns `entityType`, `entityID`, and `entityMeta` (JSON string with `name`, `slug`, `logo_url`, and type-specific fields). Useful for resolving a name to a slug before querying. Supports `providerIsVerified` and `providerIsClaimed` filters.

**`rewardOptionTypes`** — returns the live list of all valid reward option type keys as strings. Use this instead of the static list in `references/metric-keys.md` if you need to confirm available types.

### Metric Keys and Filters

See `references/metric-keys.md` for common metric keys by type, reward option type keys, sorting options, date filters, interval values, and advanced `metrics` query args.

## Ratings API (REST)

**Official documentation:** https://api-docs.stakingrewards.com/ratings-api/overview

> If any example below seems outdated or unclear, check the official docs first before proceeding.

**Base URL:** `https://api.stakingrewards.com`

**Auth:** `X-API-KEY` header (same key as GraphQL API)

**Required headers on every request:**
- `X-API-KEY: <key>`
- `X-Agent: claude-skill` **(always include — identifies the caller as this agent)**

Every response includes an `x-used-credits` header with the credit cost of that call — read it and display it with results.

### Endpoints

**DeFi Ratings:**

| Method | Path | Description |
|--------|------|-------------|
| GET | `/ratings/defi` | List all DeFi product ratings |
| GET | `/ratings/defi/{provider_slug}` | List DeFi ratings for a specific platform |
| GET | `/ratings/defi/{provider_slug}/{contract_address}` | Get single DeFi product rating |
| GET | `/ratings/defi/chains` | List all valid chain slugs for the `chain` filter |

**DeFi list query parameters:**
- `sort` — `rating`, `rated_at`, `rated_since`, `name`, `tvl`, `apy`, `users` (sort field)
- `order` — `asc` or `desc` (default: `asc`)
- `type` — `lending`, `vault`, `lst`, `liquid-staking`, `hosting`, `smart-contract`
- `chain` — chain slug from `/ratings/defi/chains` (e.g., `ethereum-2-0`, `base`, `solana`); note: the response `chain` field returns the display name, not the slug
- `version` — rating methodology version (e.g., `v2.0`)
- `tvl_gte` / `tvl_lte` — filter by TVL range
- `apy_gte` / `apy_lte` — filter by APY range
- `users_gte` / `users_lte` — filter by user count range
- `rated_at_gte` / `rated_at_lte` — filter by rating date range (YYYY-MM-DD)
- `rated_since_gte` / `rated_since_lte` — filter by rated-since date range
- `limit` / `offset` — pagination (default limit: 10)

**Infrastructure Ratings:**

| Method | Path | Description |
|--------|------|-------------|
| GET | `/ratings/infra` | List all infrastructure provider ratings |
| GET | `/ratings/infra/{slug}` | Get single infrastructure provider rating |

**Infrastructure list query parameters:**
- `sort` — `name`, `rating`, `rated_at`, `rated_since` (default: `rating`)
- `order` — `asc` or `desc` (default: `asc`)
- `rating` — filter by exact grade (e.g., `A`, `BBB`)
- `rating_gte` / `rating_lte` — filter by grade range (AAA to D)
- `version` — rating methodology version
- `rated_at_gte` / `rated_at_lte` — filter by rating date range
- `rated_since_gte` / `rated_since_lte` — filter by rated-since date range
- `limit` / `offset` — pagination

### Response Format

List endpoints return `{ "data": [...] }` wrapper. Single-item endpoints (`/ratings/defi/{platform}/{contract}`, `/ratings/infra/{slug}`) return a bare object.

See `references/response-schemas.md` for full field documentation for DeFi and Infrastructure rating objects.

### Rating Grades

Both DeFi and Infrastructure use letter grades: AAA (best) → AA → A → BBB → BB → B → CCC → CC → C → D (worst). Plus/minus modifiers (e.g., "A-", "AA+") may appear.

DeFi ratings may be "preview" (version suffix `-preview`) or "full" (complete team review).

See `references/ratings-api-examples.md` for ready-to-run example requests.

## Error Handling

| Code | Message | Fix |
|------|-----------------------------------------------------|-------------------------------|
| 401 | `you should be authenticated` | Check X-API-KEY header |
| 401 | `user not found` | Invalid API key |
| 400 | `limit field is required` | Add `limit` to query |
| 400 | `max limit is 500` | Reduce limit value |
| 400 | `query path is too long. Max depth is 2` | Reduce nesting |
| 429 | Rate limited | Wait, max 60 req/min |

## Official Docs

The official API documentation lives at **https://api-docs.stakingrewards.com/**. Key sections:
- Ratings API: https://api-docs.stakingrewards.com/ratings-api/overview
- GraphQL staking data API (schema & objects): https://api-docs.stakingrewards.com/staking-data-api/schema-and-objects — for exploring available types and fields; schema introspection (see `references/graphql-examples.md`) is often faster for targeted lookups
