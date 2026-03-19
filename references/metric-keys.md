# Metric Keys & Query Reference

## Common Metric Keys

**Asset metrics:** `price`, `reward_rate`, `real_reward_rate`, `staked_tokens`, `staking_marketcap`, `staking_ratio`, `marketcap`, `inflation_rate`, `active_validators`, `total_validators`, `net_staking_flow_7d`, `daily_trading_volume`

**Provider metrics:** `assets_under_management`, `commission`, `staking_wallets`, `provider_aum_change_7d`

**Validator metrics:** `commission`, `delegated_tokens`, `reward_rate`, `staked_tokens`, `staking_share`, `staking_wallets`

**Reward option metrics:** `reward_rate`, `commission`, `staked_tokens`, `staking_share`, `delegated_tokens`, `net_staking_flow_7d`, `staking_wallets`

**Global metrics:** `marketcap`, `staking_marketcap`, `benchmark_reward_rate`, `benchmark_staking_ratio`, `crypto_gdp`, `net_staking_flow_7d`, `pos_flippening_pow`

---

## Reward Option Type Keys

Full live list (from `rewardOptionTypes` query as of 2026-03-19):

`liquid-staking`, `solo-staking`, `smart-contract`, `lending`, `custodial`, `liquidity-pool`, `hosting`, `pos`, `operator`, `actively-validated-service`, `restaking`, `babylon-staking`, `dual-staking`, `borrowing`, `partial-staking`, `hyperliquid`, `staking-pool`, `distributor`, `partner`, `vault`

---

## Sorting

Use `order` argument:
- `{ name: asc }` or `{ name: desc }` — alphabetical
- `{ metricKey_desc: "staked_tokens" }` — by metric value
- `{ metricKey_desc: "staking_marketcap", changePercentagesKey: _30d }` — by metric change

Time periods for change sorting: `_24h`, `_7d`, `_30d`, `_90d`, `_1y`

---

## Date Filters (for historical queries)

- `createdAt_gt` — after date
- `createdAt_lt` — before date
- `createdAt_gte` / `createdAt_lte` — inclusive
- Format: `YYYY-MM-DD`

---

## Intervals (for historical queries)

`hour`, `day`, `week`, `month`, `quarter`

---

## Advanced `metrics` Query Args

Beyond `where`, `order`, `limit`, `offset`, and `interval`, the `metrics` query supports:

- `showAll` (Boolean) — include metrics that would otherwise be filtered out
- `pickItem` (`first` | `last`) — when multiple data points exist within an interval, pick the first or last one
- `backfillInterval` (Boolean) — fill gaps in time series with the last known value
