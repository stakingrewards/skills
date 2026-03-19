# Ratings API Response Schemas

## DeFi Rating Object Fields

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

---

## Infrastructure Rating Object Fields

- `name` — provider slug (e.g., "a41")
- `version` — rating version (e.g., "v2.0")
- `rating` — overall letter grade
- `rated_at` / `rated_since` — dates (YYYY-MM-DD)
- `profile_url` — link to Staking Rewards profile page
- `report_url` — link to full report (if available)
- `rating_data` — breakdown: `business_operations` (delegator protection, general info, resilience), `reliability` (on-chain metrics, validator maintenance), `security_setup` (infrastructure, internal policies)
