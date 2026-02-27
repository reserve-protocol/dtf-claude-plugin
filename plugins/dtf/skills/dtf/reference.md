# DTF CLI — Full Command Reference

Complete reference for all `@reserve-protocol/dtf-cli` commands. Use `--json` for all agent interactions.

## Global Options

| Flag | Default | Description |
|------|---------|-------------|
| `--chain <id\|all>` | `8453` (Base) | Chain ID: `1` (Ethereum), `56` (BSC), `8453` (Base), or `all` |
| `--rpc <url>` | public RPCs | Custom RPC endpoint |
| `--json` | off | JSON output for structured consumption |
| `--subgraph <index\|yield>` | auto | Force subgraph type (for `query` command) |
| `--sort <mcap\|fee\|performance>` | `mcap` | Sort discover results |
| `--help` | — | Show help text |

## Symbol Resolution

Use DTF symbols instead of 0x addresses. Resolution is case-insensitive.

```bash
dtf info cmc20               # → 0x2f8a... on BSC (chain 56)
dtf basket lcap --json       # → 0x4da9... on Base (chain 8453)
dtf info 0xe469...f2         # Direct address also works
```

Symbols auto-set the chain. Explicit `--chain` overrides auto-detection.

If a partial name matches multiple DTFs, the CLI lists all matches and asks for the exact symbol.

## Config Precedence

1. CLI flags (`--chain 1`, `--rpc https://...`)
2. Environment variables (`CHAIN_ID`, `RPC_URL`, `WALLET_KEY`)
3. `.dtfrc` file in current directory (JSON)

---

## Commands

### `discover`

List DTFs across chains with market data.

```bash
dtf discover --json                           # Index DTFs (default)
dtf discover --type yield --json              # Yield DTFs ($210M+ TVL)
dtf discover --all --json                     # Both index + yield
dtf discover --chain all --json               # All chains
dtf discover --performance 3m --limit 10 --json
```

| Flag | Description |
|------|-------------|
| `--type <index\|yield>` | DTF type filter (default: index). Use `--all` for both. |
| `--performance <30d\|3m\|6m\|1y>` | Include return over period |
| `--limit <n>` | Limit number of results |
| `--sort <mcap\|fee\|performance>` | Sort results (default: mcap descending) |

**Unified output** (same schema for index AND yield — discriminate via `dtfType`): `dtfType`, `address`, `name`, `symbol`, `chainId`, `chainLabel`, `price`, `priceHuman`, `marketCap` (null if unavailable), `marketCapHuman`, `tvl` (null for yield), `tvlFeeAnnualPercent` (null for yield), `tvlFeeAnnualPercentHuman` (null for yield), `return7d` (null for yield), `return7dPercent` (human-readable with +/- sign), `stakingTvlUsd` (null if no staking), `stakingApr` (null if no staking), `stakingAprPercent`, `tokens[]` (empty for yield), `mandate` (truncated to 100 chars with "...", null if absent), `investUrl`

- `investUrl` — Direct link to the DTF on Register (app.reserve.org). Always use this when directing users to invest.
- **0% APR filtered out**: Yield entries with `stakingApr: 0` are excluded (staking vault exists but no rewards — misleading).
- **`--all` deduplication**: When showing both types, yield staking data (`stakingTvlUsd`, `stakingApr`) is merged onto the corresponding index entry. No duplicate rows.

**Sort modes**: `mcap` (market cap, descending — default), `fee` (TVL fee, ascending — cheapest first), `performance` (return, descending — best first, requires `--performance`).

**Note**: Default type is index only. Use `--type yield` to see Index DTFs with staking rewards, or `--all` for merged view.

---

### `compare <dtf1> <dtf2> [dtf3...]`

Side-by-side DTF comparison with basket overlap analysis.

```bash
dtf compare lcap bgci --json
dtf compare cmc20 lcap bgci --chain 8453 --json
```

**Output**: `comparison` (`dtfCount`, `totalUniqueTokens`, `overlapTokenCount`, `addressOverlapPercent`, `totalUniqueCanonical`, `canonicalOverlapCount`, `canonicalOverlapPercent`), `mintFees` (per-DTF mint fee % from RPC), `tvlFees` (per-DTF annual TVL fee % from RPC), `dtfs[]` (each with `address`, `name`, `symbol`, `chainId`, `chainLabel`, `investUrl`, `price`, `marketCap`, `mintFee`, `mintFeePercent`, `tvlFeeAnnualPercent`, `tvlFeeAnnualPercentHuman`, `totalSupply`, `tokenCount`, `hhi`, `effectiveN`, `tokens[]` each with `canonicalAsset`), `pairs[]` (each pair with `dtf1`, `dtf1Symbol`, `dtf2`, `dtf2Symbol`, `sharedTokens`, `overlapWeight`, `economicOverlap`), `weightMatrix[]` (per canonical asset: `asset`, `weights` per DTF, `maxDelta` sorted by largest difference), `sharedTokens[]`, `sharedCanonicalAssets[]`
- **Fees**: `mintFee`/`mintFeePercent` (one-time entry fee from RPC) AND `tvlFeeAnnualPercent`/`tvlFeeAnnualPercentHuman` (annual management fee from RPC). `mintFees` and `tvlFees` top-level objects for quick side-by-side comparison.
- **Weight matrix**: `weightMatrix[]` shows per-canonical-asset weight comparison across all DTFs, sorted by largest weight delta. E.g. `{ asset: "BTC", weights: { CMC20: 70.13, LCAP: 45.00 }, maxDelta: 25.13 }`. Only includes assets shared by 2+ DTFs.
- **Cross-chain economic overlap**: BTCB (BSC), cbBTC (Base), WBTC (ETH) all resolve to canonical "BTC". `economicOverlap` in pairs catches this; `overlapWeight` uses strict address matching only.
- Token symbols resolved via RPC multicall when API doesn't return them
- Each DTF includes `name` and `symbol` from registry

**Overlap metrics**: HHI (Herfindahl-Hirschman Index) measures concentration — lower = more diversified. `effectiveN` = 1/HHI = effective number of holdings. `overlapWeight` = sum of minimum weights for shared tokens (higher = more correlated).

---

### `info <address>`

Full DTF configuration from subgraph + RPC.

```bash
dtf info cmc20 --json
dtf info 0x2f8a339b5889ffac4c5a956787cda593b3c36867 --chain 56 --json
```

**Output fields**: `type`, `name`, `symbol`, `dtf` (address), `chainId`, `chainLabel`, `price`, `priceHuman`, `marketCap`, `marketCapHuman`, `tvl`, `tvlHuman`, `totalSupply`, `investUrl`, `ownerGovernance`, `tradingGovernance`, `stTokenGovernance`, `stToken`, `tokens[]`, `auctionLength`, `auctionLengthHuman`, `mandate` (null if absent), `tvlFeeAnnualPercent`, `tvlFeeAnnualPercentHuman`, `mintFeePercent`, `mintFeePercentHuman`, `bidsEnabled`, `rebalanceControl`, `auctionLaunchers[]`, `brandManagers[]`

- `price`/`marketCap`/`tvl` — Market data from Reserve API. For Index DTFs, TVL = marketCap (all assets locked). Null if API unavailable.
- `tvlFeeAnnualPercent` — Annualized TVL fee (e.g. `1.50` = 1.5%/year). `tvlFeeAnnualPercentHuman` — e.g. `"1.50%"`.
- `mintFeePercent` — Mint fee as number (e.g. `0.30`). `mintFeePercentHuman` — e.g. `"0.30%"`.
- `investUrl` — Direct link to the DTF on Register app
- `bidsEnabled` — Whether permissionless auction bids are enabled

**Requires subgraph** — Works on all chains (Ethereum, Base, BSC). BSC has an index subgraph only.

---

### `basket <address>`

Basket composition with token details and USD values.

```bash
dtf basket cmc20 --json
dtf basket lcap --json
```

**Index output**: `type: 'index'`, `tokens[]` (each with `symbol`, `address`, `decimals`, `price`, `weight`, `weightPercent`, `amount`, `amountRaw`, `value`), `totalSupply`, `tvl`, `sharePrice`, `marketCap`, `concentration` (`hhi`, `effectiveN`, `top3Weight`, `top5Weight`, `isConcentrated`)

**Yield output**: `type: 'yield'`, `sharePrice`, `tvl`, `totalSupply` (from subgraph, null if unavailable), `tokens[]` (each with `symbol`, `address`, `targetUnit`, `uoaSharePercent`, `usdValue`)

---

### `fees <address>`

Fee information: pending fees, recipients, fee rates.

```bash
dtf fees cmc20 --json
```

**Output fields**: `dtf`, `type`, `chainId`, `chainLabel`, `investUrl`, `pending` (`total`, `totalHuman`, `folio`, `folioHuman`, `dao`, `daoHuman`), `mintFeePercent`, `mintFeePercentHuman`, `tvlFeeAnnualPercent`, `tvlFeeAnnualPercentHuman`, `recipients[]` (each with `recipient`, `recipientLabel`, `portion`, `portionPercent`)

---

### `quote <address> <amount>`

Mint or redeem quote with exact token amounts.

```bash
dtf quote cmc20 100 --json
dtf quote cmc20 50 --action redeem --json
```

| Flag | Default | Description |
|------|---------|-------------|
| `--action <mint\|redeem>` | `mint` | Quote type |

**Mint output**: `dtf`, `type`, `chainId`, `chainLabel`, `investUrl`, `action: 'mint'`, `shares`, `sharesHuman`, `mintFee` (raw bigint string), `mintFeePercent` (number), `mintFeePercentHuman` (string like "1.50%"), `minSharesOut`, `slippageBps`, `slippagePercent`, `totalUsd`, `totalUsdHuman`, `navPerShare`, `navPerShareHuman`, `pricesAvailable`, `tokens[]` (each with `address`, `symbol`, `amount`, `amountHuman`, `usdValue` — amounts needed to deposit)

**Redeem output**: `dtf`, `type`, `chainId`, `chainLabel`, `investUrl`, `action: 'redeem'`, `shares`, `sharesHuman`, `totalUsd`, `totalUsdHuman`, `navPerShare`, `navPerShareHuman`, `pricesAvailable`, `tokens[]` (each with `address`, `symbol`, `amount`, `amountHuman`, `usdValue` — amounts received back)

- `investUrl` — Direct link to the DTF issuance page on Register. Use this to direct users to mint/redeem.
- `totalUsd` / `totalUsdHuman` — Total USD cost/value of the quote
- `mintFee` — Raw bigint mint fee amount (string). `mintFeePercent` — As number (1.5), `mintFeePercentHuman` — As string ("1.50%")
- `minSharesOut` — Minimum shares after slippage (for mint transactions)
- `slippageBps` / `slippagePercent` — Default 1% slippage protection
- `pricesAvailable` — Whether USD price data was available (false = prices API failed, USD values are 0)
- Yield DTFs include token `symbol` and `depositUoAHuman` fields

---

### `prices <address>`

Token prices and volatility classifications.

```bash
dtf prices cmc20 --json
dtf prices cmc20 --performance 30d --json
dtf prices lcap --performance 3m --json
```

| Flag | Description |
|------|-------------|
| `--performance <30d\|3m\|6m\|1y>` | Include per-token return over period, sorted best-first |

**Output fields**: `tokens[]` (each with `symbol`, `address`, `price`, `volatility`, `auctionPriceError`, `proposalPriceError`), `btcUsd` (Chainlink)

With `--performance`: adds `return_{period}` to each token (e.g. `return_30d: 12.5`). Tokens sorted by return descending. Also adds `totalReturn_{period}` — weighted portfolio return (yield DTFs: weighted by UoA share; index DTFs: equal-weight approximation).

Works for both index and yield DTFs. **Note**: Chainlink BTC/USD reads from ETH mainnet regardless of `--chain`.

---

### `governance <address>`

Governance settings for all three governors (owner, trading, lock/stToken).

```bash
dtf governance cmc20 --json
```

**Index output**: For each governor (owner + trading + stToken/lock): `votingDelay`, `votingDelayHuman`, `votingPeriod`, `votingPeriodHuman`, `proposalThreshold`, `proposalThresholdHuman`, `quorumNumerator`, `quorumDenominator`, `quorumPercent`, `timelockMinDelay`, `timelockMinDelayHuman`

**Yield output**: `guardians[]`, `delegateStats` (`currentDelegates`, `totalDelegates`, `proposalsQueued`, `proposalsExecuted`), `governances[]` (each with `label`, `governor`, `timelock`, `votingDelay`, `votingDelayHuman`, `votingPeriod`, `votingPeriodHuman`, `quorumPercent`, `executionDelay`, `executionDelayHuman`)

---

### `staking <address>`

Vote-lock (stToken) information and optionally account balances.

```bash
dtf staking cmc20 --json
dtf staking cmc20 --account 0xYOUR_ADDRESS --json
```

| Flag | Description |
|------|-------------|
| `--account <address>` | Show account-specific balances |

**Index output**: `stToken`, `underlyingAsset`, `rewardTokens[]`, `unstakingDelay`, `unstakingDelayHuman`, `earn` (when available: `apr`, `lockedAmountUsd`, `avgDailyRewardAmountUsd`)

With `--account`: adds `balance`, `balanceHuman`, `delegatee`, `votingPower`, `votingPowerHuman`, `maxWithdraw`, `maxWithdrawHuman`

**Yield output**: `stRSR`, `exchangeRate`, `exchangeRateHuman`, `totalSupply`, `totalSupplyHuman`, `unstakingDelay`, `unstakingDelayHuman`

With `--account`: adds `balance`, `balanceHuman`, `votingPower`, `votingPowerHuman`, `delegatee`

---

### `roles <address>`

Role holders for the DTF.

```bash
dtf roles cmc20 --json
```

**Index output**: `roles[]` (each with `address`, `roles[]` listing role names like `ADMIN`, `AUCTION_LAUNCHER`, `BRAND_MANAGER`, `REBALANCE_MANAGER`)

**Yield output**: `owners[]`, `pausers[]`, `freezers[]`, `longFreezers[]` — flat arrays of addresses per role

---

### `proposals [address] [id...]`

Governance proposals. Without an address, fetches across all DTFs.

```bash
dtf proposals cmc20 --json
dtf proposals cmc20 123 456 --json
dtf proposals --json
dtf proposals --chain 1 --json
```

**Output fields**: `proposals[]` (each with `id`, `proposer`, `state`, `stateHuman`, `startBlock`, `endBlock`, `forVotes`, `againstVotes`, `abstainVotes`, `actions[]` with decoded calldata, `type` index or yield)

Decodes calldata into human-readable function calls with parameter names. Works for both index and yield DTFs. Without an address, fetches proposals from both subgraphs.

---

### `rebalance <address>` | `rebalance --all`

Active rebalance state (single DTF) or batch scan (all DTFs).

```bash
dtf rebalance cmc20 --json              # Single DTF
dtf rebalance --all --json              # Scan all known DTFs for active rebalances
dtf rebalance --all --chain 8453 --json # Scan Base DTFs only
```

**Single DTF output**: `nonce`, `priceControl`, `bidsEnabled`, `isActive`, `isExpired`, `isInLauncherWindow`, `isInCommunityWindow`, `statusLabel`, `timeRemaining`, `tokens[]` (each with `symbol`, `address`, `weight` low/spot/high, `price` low/high, `inRebalance`), `auction` (if active: `auctionId`, `startTime`, `endTime`, `isActive`). Returns `null` if no rebalance has ever started.

**`--all` output**: `scan` (`chains`, `dtfsScanned`, `activeRebalances`, `liveAuctions`), `dtfs[]` (each with `symbol`, `name`, `address`, `chainId`, `status` (active/expired/none/error), `nonce`, `window`, `hasAuction`, `bidsEnabled`)
- `bidsEnabled: true` = permissionless (any bot can fill). `bidsEnabled: false` = trusted fillers only (CoW Swap).

**Yield output**: `rebalanceType: 'automatic'`, `backingPercent`, `message`. Yield DTFs rebalance automatically — no manual state.

---

### `rebalance-history <address>` (alias: `history`)

Rebalance history from the Reserve API.

```bash
dtf history cmc20 --json
dtf history cmc20 --nonce 3 --json
```

| Flag | Description |
|------|-------------|
| `--nonce <n>` | Get detail for a specific rebalance |

**Summary output**: `rebalances[]` (each with `nonce`, `startedAt`, `endedAt`, `status`, `numAuctions`)

**Detail output** (with `--nonce`): adds `auctions[]` with full bid details

---

### `earn`

Vote-lock yield opportunities across all chains with risk scoring.

```bash
dtf earn --json
dtf earn --chain 1 --json
dtf earn --underlying RSR --json
dtf earn --sort tvl --json
dtf earn --sort risk --json
```

| Flag | Default | Description |
|------|---------|-------------|
| `--underlying <symbol>` | — | Filter by underlying token (e.g. RSR, WETH) |
| `--sort <apr\|tvl\|risk>` | `apr` | Sort: APR descending, TVL descending, or risk (low first) |

**Output fields**: `type: 'vote-lock'`, `count`, `positions[]` (each with `vault`, `vaultAddress`, `underlying`, `underlyingAddress`, `chainId`, `chainLabel`, `investUrl`, `apr`, `aprPercent`, `lockedAmountUsd`, `lockedAmountUsdHuman`, `avgDailyRewardAmountUsd`, `avgDailyRewardHuman`, `rewardTokens[]`, `dtfCount`, `dtfSymbols[]`, `riskTier`, `riskFactors[]`)

- `investUrl` — Link to the primary DTF's governance/staking page on Register

**Risk tiers** (`low`, `medium`, `high`) based on weighted scoring: TVL size (critical), APR sustainability (critical), chain risk (moderate). `riskFactors` explains each flag. 0% APR positions are filtered out.

**`--sort risk`**: Groups by risk tier (low first), then by APR descending within each tier.

---

### `revenue <address> | --all`

Revenue breakdown.

```bash
dtf revenue cmc20 --json
dtf revenue --all --json
```

| Flag | Description |
|------|-------------|
| `--all` | Ecosystem-wide revenue |

**Single DTF output** (index only): `dtf`, `type: 'index'`, `chainId`, `chainLabel`, `investUrl`, `symbol`, `price`, `revenuePeriod: 'cumulative'`, `revenuePeriodNote`, `revenue` (`totalUsd`, `totalUsdHuman`, `protocolUsd`, `protocolUsdHuman`, `governanceUsd`, `governanceUsdHuman`, `externalUsd`, `externalUsdHuman`, `totalShares`, `protocolShares`, `governanceShares`, `externalShares`), `fees` (`mintFeePercent`, `mintFeePercentHuman`, `tvlFeeAnnualPercent`, `tvlFeeAnnualPercentHuman`)

**Yield DTFs**: Returns a clear message directing to `dtf fees` and `dtf info` instead. Revenue model differs for yield DTFs (automatic yield distribution vs fee accrual).

**Ecosystem output**: `type: 'index'`, `revenuePeriod: 'cumulative'`, `revenuePeriodNote`, `note`, `ecosystem` (`totalUsd`, `protocolUsd`, `governanceUsd`, `externalUsd`, `*Percent`, `dtfCount`, `topDtfs[]`), `fees` (`weightedMintFeePercent`, `weightedMintFeePercentHuman`, `weightedTvlFeePercent`, `weightedTvlFeePercentHuman`, `averageMintFeePercent`, `averageMintFeePercentHuman`, `averageTvlFeePercent`, `averageTvlFeePercentHuman`)

---

### `rsr-burns`

RSR burn analytics and projections.

```bash
dtf rsr-burns --json
dtf rsr-burns --chain 1 --json
```

**Output fields**: `burns[]`, `monthlySnapshots[]`, `totalBurned`, `totalBurnedUsd`, `projection` (annualized burn rate)

---

### `deploy`

Deploy a new DTF.

```bash
dtf deploy --name "My DTF" --symbol MDTF --basket "50% WETH, 30% USDC, 20% WBTC" --amount 0.1 --json
dtf deploy --config deploy.json --json
dtf deploy --list-tokens --chain 8453
dtf deploy --help
```

| Flag | Description |
|------|-------------|
| `--name <name>` | DTF name |
| `--symbol <symbol>` | Ticker symbol |
| `--basket <spec>` | Inline `"50% WETH, 30% USDC"` or path to CSV |
| `--amount <eth>` | ETH to deposit |
| `--mandate <text>` | DTF description |
| `--config <file>` | JSON config file |
| `--ungoverned` | Deploy without DAO governance |
| `--dry-run` | Preview without sending tx |
| `--list-tokens` | Show available tokens |
| `--wallet-key <key>` | Private key (prefer `WALLET_KEY` env var) |

---

### `forum`

Reserve governance forum.

```bash
dtf forum --json
dtf forum search rebalance --json
dtf forum topic 1234 --json
```

Subcommands: `search <query>`, `topic <id>`, or no args for monthly top topics.

---

### `query '<graphql>'`

Raw subgraph query with auto-detection of index vs yield subgraph.

```bash
dtf query '{ dtfs(first: 3) { id token { symbol } } }' --json
dtf query '{ rtokens(first: 3) { id token { symbol } pausers } }' --chain 1 --json
dtf query '{ rtokens(where: { pausers_contains: ["0x..."] }) { id } }' --chain all --json
dtf query '{ proposals(where: { state: "ACTIVE" }) { id } }' --subgraph yield --json
```

| Flag | Description |
|------|-------------|
| `--subgraph <index\|yield>` | Force subgraph type (auto-detected from entity names) |
| `--chain all` | Fan out query to all chains |

Auto-detection uses entity names: `dtf`, `rebalance`, `auction` → index; `rtoken`, `protocol`, `entry` → yield. **Shared entities** (proposals, delegates, token, account) default to the **index** subgraph — use `--subgraph yield` to override.

**Single-chain output**: `{ "subgraph": "index", "chain": 8453, "chainLabel": "Base", "data": { ... } }`

**Multi-chain output** (`--chain all`): `{ "subgraph": "index", "results": [{ "chain": 1, ... }, { "chain": 8453, ... }] }`

See `subgraph-schema.md` for entity reference and query patterns.

---

### `holders <address>`

Top token holders with balances and USD values.

```bash
dtf holders cmc20 --json
dtf holders lcap --limit 50 --json
dtf holders eth+ --chain 1 --json
```

| Flag | Description |
|------|-------------|
| `--limit <n>` | Number of holders (default: 20) |

**Output fields**: `holders[]` (each with `account`, `balance`, `balanceHuman`, `balanceUsd`, `supplyPercent`, `rank`), `totalHolders`, `totalSupply`, `tokenPrice`, `type` (index or yield), `concentration` (`top5Percent`, `top10Percent`)

- `supplyPercent` — Holder's % of total supply
- `concentration` — Top 5 and top 10 holders' combined supply share

Index DTFs fetch price via Reserve API. Yield DTFs use `lastPriceUSD` from subgraph. Auto-detects DTF type.

---

### `delegates <address>`

Governance delegation graph.

```bash
dtf delegates cmc20 --json
dtf delegates eth+ --chain 1 --json
dtf delegates lcap --limit 50 --json
```

| Flag | Description |
|------|-------------|
| `--limit <n>` | Number of delegates (default: 20) |

**Output fields**: `delegates[]` (each with `address`, `delegatedVotes`, `delegatedVotesHuman`, `representedHolders`, `votesCast`, `rank`), `totalDelegates`, `governanceToken`, `type` (index or yield)

Index DTFs query the stToken's delegation. Yield DTFs query the governance entity. Auto-detects DTF type.

---

### `cache-clear`

Clear local disk cache (`~/.dtf/cache`).

```bash
dtf cache-clear
```

---

## Error Handling

All errors in `--json` mode return:

```json
{ "error": "Error message here" }
```

Common errors:
- `"Error: Unknown DTF \"xyz\""` — Invalid symbol, use `dtf discover` to see available DTFs
- `"Error: fetchDtfConfig failed"` — Subgraph unavailable (common on BSC)
- `"Error: getRebalance() reverted"` — No rebalance has ever started for this DTF
- Network timeouts — Retry once, then check RPC availability

## Tips for Agents

1. **Start with `discover --json`** to get the list of all DTFs
2. **Use `basket` over `info`** when you only need token composition — it's faster
3. **Batch related queries** — run `info`, `basket`, `fees` in parallel for a full picture
4. **Check `isActive` in rebalance** before reporting auction state
5. **Use `*Human` fields** for user-facing responses, raw fields for calculations
6. **Add disclaimer** when discussing investment recommendations: "Not financial advice. DYOR!"
7. **Check `type` field** in JSON output — every command includes `type` (`'index'`, `'yield'`, or `'vote-lock'`) for routing
8. **Use `tokens[]`** as the normalized array in quote output — works for both index and yield. Old fields (`amounts`, `deposit`, `withdrawal`) kept for backwards compatibility
9. **Use `earn --sort risk`** for safer recommendations — low risk first, filtered by TVL/APR sustainability
