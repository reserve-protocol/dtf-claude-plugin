# DTF CLI — Full Command Reference

Complete reference for all `@reserve-protocol/dtf-cli` commands. Use `--json` for all agent interactions.

## Global Options

| Flag | Default | Description |
|------|---------|-------------|
| `--chain <id\|all>` | `8453` (Base) | Chain ID: `1` (Ethereum), `56` (BSC), `8453` (Base), or `all` |
| `--rpc <url>` | public RPCs | Custom RPC endpoint |
| `--json` | off | JSON output for structured consumption |
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
dtf discover --json
dtf discover --chain 8453 --json
dtf discover --performance 3m --limit 10 --json
dtf discover --chain all --json
```

| Flag | Description |
|------|-------------|
| `--performance <30d\|3m\|6m\|1y>` | Include return over period |
| `--limit <n>` | Limit number of results |

**Output fields**: `address`, `name`, `symbol`, `chainId`, `marketCapUsd`, `marketCapHuman`, `performance` (when requested)

---

### `info <address>`

Full DTF configuration from subgraph + RPC.

```bash
dtf info cmc20 --json
dtf info 0x2f8a339b5889ffac4c5a956787cda593b3c36867 --chain 56 --json
```

**Output fields**: `address`, `name`, `symbol`, `governor`, `tradingGovernor`, `timelock`, `tradingTimelock`, `stToken`, `tokens[]`, `auctionLength`, `mandate`, `mintFee`, `tvlFee`, `feeRecipients[]`

**Requires subgraph** — Ethereum and Base only. Throws on BSC (chain 56).

---

### `basket <address>`

Basket composition with token details and USD values.

```bash
dtf basket cmc20 --json
dtf basket lcap --json
```

**Output fields**: `tokens[]` (each with `symbol`, `address`, `balance`, `balanceHuman`, `weight`, `weightPercent`, `usdValue`, `usdValueHuman`), `totalSupply`, `tvl`, `tvlHuman`, `sharePrice`, `sharePriceHuman`

---

### `fees <address>`

Fee information: pending fees, recipients, fee rates.

```bash
dtf fees cmc20 --json
```

**Output fields**: `pendingFees` (`total`, `folio`, `dao`, all with `Human` variants), `mintFee`, `mintFeeHuman`, `tvlFee`, `tvlFeeHuman`, `recipients[]` (each with `address`, `portion`, `portionPercent`)

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

**Mint output**: `shares`, `sharesHuman`, `fee`, `feeHuman`, `tokens[]` (each with `symbol`, `amount`, `amountHuman` — amounts needed to deposit)

**Redeem output**: `shares`, `sharesHuman`, `tokens[]` (each with `symbol`, `amount`, `amountHuman` — amounts received back)

---

### `prices <address>`

Token prices and volatility classifications.

```bash
dtf prices cmc20 --json
```

**Output fields**: `tokens[]` (each with `symbol`, `address`, `price`, `priceHuman`, `volatility`), `btcUsd`, `btcUsdHuman` (Chainlink)

**Note**: Chainlink BTC/USD reads from ETH mainnet regardless of `--chain`.

---

### `governance <address>`

Governance settings for both governors.

```bash
dtf governance cmc20 --json
```

**Output fields**: For each governor (owner + trading): `votingDelay`, `votingDelayHuman`, `votingPeriod`, `votingPeriodHuman`, `proposalThreshold`, `proposalThresholdHuman`, `quorumNumerator`, `quorumDenominator`, `quorumPercent`, `timelockMinDelay`, `timelockMinDelayHuman`

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

**Output fields**: `stToken`, `underlying`, `unstakingDelay`, `unstakingDelayHuman`, `rewardTokens[]`

With `--account`: adds `balance`, `balanceHuman`, `lockedBalance`, `lockedBalanceHuman`, `withdrawable`, `withdrawableHuman`

---

### `roles <address>`

Role holders for the DTF.

```bash
dtf roles cmc20 --json
```

**Output fields**: `roles[]` (each with `address`, `roles[]` listing role names like `ADMIN`, `AUCTION_LAUNCHER`, `BRAND_MANAGER`, `REBALANCE_MANAGER`)

---

### `proposals [address] [id...]`

Governance proposals. Without an address, fetches across all DTFs.

```bash
dtf proposals cmc20 --json
dtf proposals cmc20 123 456 --json
dtf proposals --json
dtf proposals --chain 1 --json
```

**Output fields**: `proposals[]` (each with `id`, `proposer`, `state`, `stateHuman`, `startBlock`, `endBlock`, `forVotes`, `againstVotes`, `abstainVotes`, `actions[]` with decoded calldata)

Decodes calldata into human-readable function calls with parameter names.

---

### `rebalance <address>`

Active rebalance state.

```bash
dtf rebalance cmc20 --json
```

**Output fields**: `nonce`, `priceControl`, `isActive`, `isExpired`, `isInLauncherWindow`, `launcherWindowEnd`, `availableUntil`, `tokens[]` (each with `symbol`, `address`, `weight` low/spot/high, `price` low/high, `inRebalance`), `auction` (if active: `auctionId`, `sellToken`, `buyToken`, `sellAmount`, `buyAmount`, `startPrice`, `endPrice`)

Returns `null` if no rebalance has ever started.

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

Vote-lock yield opportunities across all chains.

```bash
dtf earn --json
dtf earn --chain 1 --json
```

**Output fields**: `positions[]` (each with `token`, `underlying`, `chainId`, `apr`, `aprPercent`, `lockedAmount`, `lockedAmountUsd`, `rewards[]`, `dtfs[]`)

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

**Single DTF output**: `tvlFeeRevenue`, `mintFeeRevenue`, `totalRevenue`, all with `Human` and `Usd` variants

**Ecosystem output**: `dtfs[]` with per-DTF revenue, `totals`

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
