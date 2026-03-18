---
name: staking-rewards-api
description: Query staking data and ratings via Staking Rewards APIs. Use when user asks about staking metrics, reward rates, validators, providers, or DeFi/infrastructure ratings. Triggers on "staking data", "reward rate", "staking rewards API", "validator metrics", "provider data", "DeFi ratings", "infrastructure ratings".
---

# Staking Rewards API

Query staking data (GraphQL) and ratings (REST) from the Staking Rewards APIs.

## When to use

Use this skill when the user asks about:
- Staking metrics, reward rates, prices, validators, providers, or TVL
- DeFi or infrastructure ratings, risk scores, or letter grades
- Historical staking data or ecosystem-level metrics
- Writing integration code against the Staking Rewards API

## Instructions

1. Determine the correct API based on what the user asks (see API Routing below)
2. Check for an API key (environment variable, `.env` file, or ask the user)
3. Construct the appropriate query (GraphQL for staking data, REST for ratings)
4. Follow the critical rules for each API (e.g., `limit` is always required for GraphQL)
5. Present results in a well-formatted table with proper units and labels

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

**Discover fields on a type:**
```bash
curl -X POST https://api.stakingrewards.com/public/query \
  -H "Content-Type: application/json" \
  -H "X-API-KEY: $STAKING_REWARDS_API_KEY" \
  -d '{"query": "{ __type(name: \"Provider\") { name fields { name type { name kind ofType { name } } } } }"}'
```

**Discover arguments on a query:**
```bash
curl -X POST https://api.stakingrewards.com/public/query \
  -H "Content-Type: application/json" \
  -H "X-API-KEY: $STAKING_REWARDS_API_KEY" \
  -d '{"query": "{ __type(name: \"Query\") { fields { name args { name type { name kind ofType { name } } } } } }"}'
```

**Available type names:** `Asset`, `Provider`, `Validator`, `RewardOption`, `Metric`, `Query`

Use introspection when:
- You need a field you're not sure exists (e.g., `logoUrl`, `country`, `vspScoreBreakdown`)
- You want to check available filter arguments on a query
- The user asks for data not covered by the common metric keys below

### Common Queries

**Get asset data with metrics:**
```bash
curl -X POST https://api.stakingrewards.com/public/query \
  -H "Content-Type: application/json" \
  -H "X-API-KEY: $STAKING_REWARDS_API_KEY" \
  -d '{"query": "{ assets(where: { symbols: [\"ETH\"] }, limit: 1) { name symbol slug metrics(where: { metricKeys: [\"reward_rate\", \"price\", \"staked_tokens\"] }, limit: 3) { metricKey defaultValue } } }"}'
```

**Get providers with AUM:**
```bash
curl -X POST https://api.stakingrewards.com/public/query \
  -H "Content-Type: application/json" \
  -H "X-API-KEY: $STAKING_REWARDS_API_KEY" \
  -d '{"query": "{ providers(where: { isVerified: true }, order: { metricKey_desc: \"assets_under_management\" }, limit: 10) { name isVerified metrics(where: { metricKeys: [\"assets_under_management\"] }, limit: 1) { defaultValue } } }"}'
```

**Get reward options for an asset:**
```bash
curl -X POST https://api.stakingrewards.com/public/query \
  -H "Content-Type: application/json" \
  -H "X-API-KEY: $STAKING_REWARDS_API_KEY" \
  -d '{"query": "{ rewardOptions(where: { inputAsset: { symbols: [\"ETH\"] }, typeKeys: [\"liquid-staking\"] }, limit: 10) { providers(limit: 1) { name isVerified } metrics(where: { metricKeys: [\"reward_rate\", \"staked_tokens\", \"commission\"] }, limit: 3) { metricKey defaultValue } } }"}'
```

**Get validators for an asset via reward options:**
```bash
curl -X POST https://api.stakingrewards.com/public/query \
  -H "Content-Type: application/json" \
  -H "X-API-KEY: $STAKING_REWARDS_API_KEY" \
  -d '{"query": "{ rewardOptions(where: { inputAsset: { slugs: [\"cosmos\"] }, typeKeys: [\"pos\"] }, limit: 10) { providers(limit: 1) { slug } validators(limit: 5) { address metrics(where: { metricKeys: [\"staked_tokens\", \"commission\"] }, limit: 2) { metricKey defaultValue } } } }"}'
```

**Get historical data:**
```bash
curl -X POST https://api.stakingrewards.com/public/query \
  -H "Content-Type: application/json" \
  -H "X-API-KEY: $STAKING_REWARDS_API_KEY" \
  -d '{"query": "{ assets(where: { slugs: [\"ethereum-2-0\"] }, limit: 1) { metrics(where: { metricKeys: [\"reward_rate\"], createdAt_gt: \"2024-01-01\" }, interval: day, limit: 100, order: { createdAt: asc }) { defaultValue createdAt } } }"}'
```

**Get global ecosystem metrics:**
```bash
curl -X POST https://api.stakingrewards.com/public/query \
  -H "Content-Type: application/json" \
  -H "X-API-KEY: $STAKING_REWARDS_API_KEY" \
  -d '{"query": "{ metrics(where: { asset: null, provider: null, rewardOption: null, validator: null, metricKeys: [\"marketcap\", \"staking_marketcap\", \"benchmark_reward_rate\"] }, limit: 3) { metricKey defaultValue changePercentages } }"}'
```

**Check billing status:**
```bash
curl -H "X-API-KEY: $STAKING_REWARDS_API_KEY" \
  https://api.stakingrewards.com/public/billing/status
```

**Check credits used per request:**

Every API response includes an `x-used-credits` header with the exact credit cost of that call:
```bash
curl -sD - -H "X-API-KEY: $STAKING_REWARDS_API_KEY" \
  "https://api.stakingrewards.com/ratings/defi?limit=3" -o /dev/null \
  | grep x-used-credits
# x-used-credits: 132
```

### Common Metric Keys

**Asset metrics:** `price`, `reward_rate`, `real_reward_rate`, `staked_tokens`, `staking_marketcap`, `staking_ratio`, `marketcap`, `inflation_rate`, `active_validators`, `total_validators`, `net_staking_flow_7d`, `daily_trading_volume`

**Provider metrics:** `assets_under_management`, `commission`, `staking_wallets`, `provider_aum_change_7d`

**Validator metrics:** `commission`, `delegated_tokens`, `reward_rate`, `staked_tokens`, `staking_share`, `staking_wallets`

**Reward option metrics:** `reward_rate`, `commission`, `staked_tokens`, `staking_share`, `delegated_tokens`, `net_staking_flow_7d`, `staking_wallets`

**Global metrics:** `marketcap`, `staking_marketcap`, `benchmark_reward_rate`, `benchmark_staking_ratio`, `crypto_gdp`, `net_staking_flow_7d`, `pos_flippening_pow`

### Sorting

Use `order` argument:
- `{ name: asc }` or `{ name: desc }` — alphabetical
- `{ metricKey_desc: "staked_tokens" }` — by metric value
- `{ metricKey_desc: "staking_marketcap", changePercentagesKey: _30d }` — by metric change

Time periods for change sorting: `_24h`, `_7d`, `_30d`, `_90d`, `_1y`

### Date Filters (for historical)

- `createdAt_gt` — after date
- `createdAt_lt` — before date
- `createdAt_gte` / `createdAt_lte` — inclusive
- Format: `YYYY-MM-DD`

### Intervals (for historical)

`hour`, `day`, `week`, `month`, `quarter`

### Reward Option Type Keys

- `pos` — native proof-of-stake (direct delegation)
- `liquid-staking` — liquid staking tokens (stETH, rETH, etc.)
- `smart-contract` — DeFi protocol interactions
- `custodial` — custodial/exchange staking
- `staking-pool` — pooled staking services
- `dual-staking` — native restaking
- `lending` — lending/borrowing protocols
- `vault` — vault strategies
- `operator` — node operator staking

## Ratings API (REST)

**Base URL:** `https://api.stakingrewards.com`

**Auth:** `X-API-KEY` header (same key as GraphQL API)

### Endpoints

**DeFi Ratings:**

| Method | Path | Description |
|--------|------|-------------|
| GET | `/ratings/defi` | List all DeFi product ratings |
| GET | `/ratings/defi/{provider_slug}` | List DeFi ratings for a specific platform |
| GET | `/ratings/defi/{provider_slug}/{contract_address}` | Get single DeFi product rating |

**DeFi list query parameters:**
- `sort` — `rating`, `rated_at`, `rated_since`, `name`, `tvl`, `apy`, `users` (sort field)
- `order` — `asc` or `desc` (default: `asc`)
- `type` — `lending`, `vault`, `lst`, `liquid-staking`, `hosting`, `smart-contract`
- `chain` — blockchain name (e.g., `base`, `optimism`, `polygon`)
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

**DeFi rating object fields:**
- `name` — product name (e.g., "Aave v3 WETH")
- `provider_name` — protocol name (e.g., "Aave")
- `version` — rating version (e.g., "v0.1-alpha-preview")
- `type` — product type: `lending`, `vault`, `liquid-staking`, etc.
- `chain` — blockchain (e.g., "Ethereum")
- `contract_address` — on-chain contract address
- `rating` — overall letter grade (e.g., "A")
- `potential_rating` — best achievable rating (e.g., "AA+")
- `rated_at` / `rated_since` — dates (YYYY-MM-DD)
- `profile_url` — link to Staking Rewards profile page
- `risk_metrics` — object with `apy` (%), `tvl` (USD), `users` (count)
- `rating_data` — breakdown by category: `operations` (documentation, financial resilience, governance, team/legal), `security` (key management, smart contracts), `strategy` (collateral, liquidity, market, counterparty)
- `potential_rating_data` — same structure as `rating_data` with potential scores

**Infrastructure rating object fields:**
- `name` — provider slug (e.g., "a41")
- `version` — rating version (e.g., "v2.0")
- `rating` — overall letter grade
- `rated_at` / `rated_since` — dates (YYYY-MM-DD)
- `profile_url` — link to Staking Rewards profile page
- `report_url` — link to full report (if available)
- `rating_data` — breakdown: `business_operations` (delegator protection, general info, resilience), `reliability` (on-chain metrics, validator maintenance), `security_setup` (infrastructure, internal policies)

### Example Requests

**Top 5 DeFi products by rating (best first):**
```bash
curl -H "X-API-KEY: $STAKING_REWARDS_API_KEY" \
  "https://api.stakingrewards.com/ratings/defi?sort=rating&order=desc&limit=5"
```

**DeFi lending products on Ethereum with TVL > $1B:**
```bash
curl -H "X-API-KEY: $STAKING_REWARDS_API_KEY" \
  "https://api.stakingrewards.com/ratings/defi?type=lending&chain=Ethereum&tvl_gte=1000000000"
```

**All DeFi ratings for a specific platform:**
```bash
curl -H "X-API-KEY: $STAKING_REWARDS_API_KEY" \
  "https://api.stakingrewards.com/ratings/defi/aave"
```

**Single DeFi product rating by contract:**
```bash
curl -H "X-API-KEY: $STAKING_REWARDS_API_KEY" \
  "https://api.stakingrewards.com/ratings/defi/aave/0x4d5F47FA6A74757f35C14fD3a6Ef8E3C9BC514E8"
```

**Top infrastructure providers by rating:**
```bash
curl -H "X-API-KEY: $STAKING_REWARDS_API_KEY" \
  "https://api.stakingrewards.com/ratings/infra?sort=rating&order=desc&limit=10"
```

**Single infrastructure provider rating:**
```bash
curl -H "X-API-KEY: $STAKING_REWARDS_API_KEY" \
  "https://api.stakingrewards.com/ratings/infra/allnodes"
```

### Rating Grades

Both DeFi and Infrastructure use letter grades: AAA (best) → AA → A → BBB → BB → B → CCC → CC → C → D (worst). Plus/minus modifiers (e.g., "A-", "AA+") may appear.

DeFi ratings may be "preview" (version suffix `-preview`) or "full" (complete team review).

## Error Handling

| Code | Message | Fix |
|------|---------|-----|
| 401 | `you should be authenticated` | Check X-API-KEY header |
| 401 | `user not found` | Invalid API key |
| 400 | `limit field is required` | Add `limit` to query |
| 400 | `max limit is 500` | Reduce limit value |
| 400 | `query path is too long. Max depth is 2` | Reduce nesting |
| 429 | Rate limited | Wait, max 60 req/min |

## Presenting Results

When displaying API data to the user:
- Format numbers with appropriate precision (prices to 2 decimals, percentages to 2 decimals)
- Show metric labels not just keys (e.g., "Reward Rate" not "reward_rate")
- Include units where relevant (USD, %, tokens)
- For tables, sort by the most relevant metric
- **Always include VSP (Verified Staking Provider) status** when displaying provider data — include `isVerified` in provider queries and show it as a column in results tables
- Source attribution: "Data from [Staking Rewards](https://stakingrewards.com)"
