# Ratings API Example Requests

**Get all valid chain slugs for the `chain` filter:**
```bash
curl -H "X-API-KEY: $STAKING_REWARDS_API_KEY" \
  -H "X-Agent: claude-skill" \
  "https://api.stakingrewards.com/ratings/defi/chains"
```
Returns `{ "chains": [ { "slug": "...", "name": "..." } ] }`. Use `slug` as the `chain` query param value. Note: the `chain` field in DeFi rating responses returns the display `name`, not the slug.

---

**Top 5 DeFi products by rating (best first):**
```bash
curl -H "X-API-KEY: $STAKING_REWARDS_API_KEY" \
  -H "X-Agent: claude-skill" \
  "https://api.stakingrewards.com/ratings/defi?sort=rating&order=desc&limit=5"
```

**DeFi lending products on Ethereum with TVL > $1B:**
```bash
curl -H "X-API-KEY: $STAKING_REWARDS_API_KEY" \
  -H "X-Agent: claude-skill" \
  "https://api.stakingrewards.com/ratings/defi?type=lending&chain=ethereum-2-0&tvl_gte=1000000000"
```

**All DeFi ratings for a specific platform:**
```bash
curl -H "X-API-KEY: $STAKING_REWARDS_API_KEY" \
  -H "X-Agent: claude-skill" \
  "https://api.stakingrewards.com/ratings/defi/aave"
```

**Single DeFi product rating by contract:**
```bash
curl -H "X-API-KEY: $STAKING_REWARDS_API_KEY" \
  -H "X-Agent: claude-skill" \
  "https://api.stakingrewards.com/ratings/defi/aave/0x4d5F47FA6A74757f35C14fD3a6Ef8E3C9BC514E8"
```

**Top infrastructure providers by rating:**
```bash
curl -H "X-API-KEY: $STAKING_REWARDS_API_KEY" \
  -H "X-Agent: claude-skill" \
  "https://api.stakingrewards.com/ratings/infra?sort=rating&order=desc&limit=10"
```

**Single infrastructure provider rating:**
```bash
curl -H "X-API-KEY: $STAKING_REWARDS_API_KEY" \
  -H "X-Agent: claude-skill" \
  "https://api.stakingrewards.com/ratings/infra/allnodes"
```
