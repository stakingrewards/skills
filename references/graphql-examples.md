# GraphQL API Examples

## Schema Introspection

**Discover fields on a type:**
```bash
curl -X POST https://api.stakingrewards.com/public/query \
  -H "Content-Type: application/json" \
  -H "X-API-KEY: $STAKING_REWARDS_API_KEY" \
  -H "X-Agent: claude-skill" \
  -d '{"query": "{ __type(name: \"Provider\") { name fields { name type { name kind ofType { name } } } } }"}'
```

**Discover arguments on a query:**
```bash
curl -X POST https://api.stakingrewards.com/public/query \
  -H "Content-Type: application/json" \
  -H "X-API-KEY: $STAKING_REWARDS_API_KEY" \
  -H "X-Agent: claude-skill" \
  -d '{"query": "{ __type(name: \"Query\") { fields { name args { name type { name kind ofType { name } } } } } }"}'
```

Available type names: `Asset`, `Provider`, `Validator`, `RewardOption`, `Metric`, `Query`

---

## Common Queries

**Get asset data with metrics:**
```bash
curl -X POST https://api.stakingrewards.com/public/query \
  -H "Content-Type: application/json" \
  -H "X-API-KEY: $STAKING_REWARDS_API_KEY" \
  -H "X-Agent: claude-skill" \
  -d '{"query": "{ assets(where: { symbols: [\"ETH\"] }, limit: 1) { name symbol slug metrics(where: { metricKeys: [\"reward_rate\", \"price\", \"staked_tokens\"] }, limit: 3) { metricKey defaultValue } } }"}'
```

**Get providers with AUM:**
```bash
curl -X POST https://api.stakingrewards.com/public/query \
  -H "Content-Type: application/json" \
  -H "X-API-KEY: $STAKING_REWARDS_API_KEY" \
  -H "X-Agent: claude-skill" \
  -d '{"query": "{ providers(where: { isVerified: true }, order: { metricKey_desc: \"assets_under_management\" }, limit: 10) { name isVerified metrics(where: { metricKeys: [\"assets_under_management\"] }, limit: 1) { defaultValue } } }"}'
```

**Get reward options for an asset:**
```bash
curl -X POST https://api.stakingrewards.com/public/query \
  -H "Content-Type: application/json" \
  -H "X-API-KEY: $STAKING_REWARDS_API_KEY" \
  -H "X-Agent: claude-skill" \
  -d '{"query": "{ rewardOptions(where: { inputAsset: { symbols: [\"ETH\"] }, typeKeys: [\"liquid-staking\"] }, limit: 10) { providers(limit: 1) { name isVerified } metrics(where: { metricKeys: [\"reward_rate\", \"staked_tokens\", \"commission\"] }, limit: 3) { metricKey defaultValue } } }"}'
```

**Get validators for an asset via reward options:**
```bash
curl -X POST https://api.stakingrewards.com/public/query \
  -H "Content-Type: application/json" \
  -H "X-API-KEY: $STAKING_REWARDS_API_KEY" \
  -H "X-Agent: claude-skill" \
  -d '{"query": "{ rewardOptions(where: { inputAsset: { slugs: [\"cosmos\"] }, typeKeys: [\"pos\"] }, limit: 10) { providers(limit: 1) { slug } validators(limit: 5) { address metrics(where: { metricKeys: [\"staked_tokens\", \"commission\"] }, limit: 2) { metricKey defaultValue } } } }"}'
```

**Get historical data:**
```bash
curl -X POST https://api.stakingrewards.com/public/query \
  -H "Content-Type: application/json" \
  -H "X-API-KEY: $STAKING_REWARDS_API_KEY" \
  -H "X-Agent: claude-skill" \
  -d '{"query": "{ assets(where: { slugs: [\"ethereum-2-0\"] }, limit: 1) { metrics(where: { metricKeys: [\"reward_rate\"], createdAt_gt: \"2024-01-01\" }, interval: day, limit: 100, order: { createdAt: asc }) { defaultValue createdAt } } }"}'
```

**Get global ecosystem metrics:**
```bash
curl -X POST https://api.stakingrewards.com/public/query \
  -H "Content-Type: application/json" \
  -H "X-API-KEY: $STAKING_REWARDS_API_KEY" \
  -H "X-Agent: claude-skill" \
  -d '{"query": "{ metrics(where: { asset: null, provider: null, rewardOption: null, validator: null, metricKeys: [\"marketcap\", \"staking_marketcap\", \"benchmark_reward_rate\"] }, limit: 3) { metricKey defaultValue changePercentages } }"}'
```

---

## Search & Discovery

**Search across assets, providers, and validators by name:**
```bash
curl -X POST https://api.stakingrewards.com/public/query \
  -H "Content-Type: application/json" \
  -H "X-API-KEY: $STAKING_REWARDS_API_KEY" \
  -H "X-Agent: claude-skill" \
  -d '{"query": "{ search(query: \"ethereum\", limit: 5) { entityType entityID entityMeta } }"}'
```
Returns `entityType` (`asset`, `provider`, `validator`), `entityID`, and `entityMeta` (JSON string with `name`, `slug`, `logo_url`, and type-specific fields like `isVerified`). Use this to resolve a name to a slug before querying.

**Filter search to verified providers only:**
```bash
curl -X POST https://api.stakingrewards.com/public/query \
  -H "Content-Type: application/json" \
  -H "X-API-KEY: $STAKING_REWARDS_API_KEY" \
  -H "X-Agent: claude-skill" \
  -d '{"query": "{ search(query: \"lido\", limit: 5, providerIsVerified: true) { entityType entityID entityMeta } }"}'
```

**Get live list of all reward option type keys:**
```bash
curl -X POST https://api.stakingrewards.com/public/query \
  -H "Content-Type: application/json" \
  -H "X-API-KEY: $STAKING_REWARDS_API_KEY" \
  -H "X-Agent: claude-skill" \
  -d '{"query": "{ rewardOptionTypes }"}'
```

---

## Billing & Credit Checks

**Check billing status:**
```bash
curl -H "X-API-KEY: $STAKING_REWARDS_API_KEY" \
  -H "X-Agent: claude-skill" \
  https://api.stakingrewards.com/public/billing/status
```

**Check credits used per request** — use `-w "%header{x-used-credits}"` to get body and credit cost in a single call:
```bash
curl -s -H "X-API-KEY: $STAKING_REWARDS_API_KEY" \
  -H "X-Agent: claude-skill" \
  "https://api.stakingrewards.com/ratings/defi?limit=3" \
  -w "\nCredits used: %header{x-used-credits}"
```
