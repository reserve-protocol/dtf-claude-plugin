# Subgraph Schema Reference

Use with `dtf query '<graphql>' --json` for flexible subgraph access.

## GraphQL Filter Syntax

```graphql
# Exact match
where: { id: "0x..." }

# Comparison
where: { amount_gt: "0", timestamp_gte: "1700000000" }

# Set membership
where: { id_in: ["0xa...", "0xb..."] }

# Array contains (for roles: pausers, freezers, admins)
where: { pausers_contains: ["0x..."] }

# Negation
where: { amount_not: "0" }

# Pagination (max 1000 per query)
first: 100, skip: 0

# Sorting
orderBy: amount, orderDirection: desc
```

**Important:** All BigInt/BigDecimal values are strings in GraphQL. Entity IDs are always lowercase.

---

## Index DTF Subgraph

**Chains:** Ethereum (1), Base (8453), BSC (56)
**No USD prices** — combine with `dtf prices <address> --json` for USD conversion.

### Core Entities

| Entity | ID | Key Fields |
|--------|-----|------------|
| `dtf` | DTF address | `token { symbol }`, `mandate`, `mintingFee`, `tvlFee`, `auctionLength`, `bidsEnabled`, `admins[]`, `auctionLaunchers[]`, `brandManagers[]`, stToken, ownerGovernance, tradingGovernance |
| `rebalance` | `{dtf}-{nonce}` | `nonce`, `priceControl`, `tokens[]`, weight/price limits, `startedAt`, `restrictedUntil`, `availableUntil` |
| `auction` | `{dtf}-{rebalance}-{id}` | `startTime`, `endTime`, weight/price limits, `bids[]` |
| `rebalanceAuctionBid` | `{dtf}-{auction}-{bidder}-{ts}` | `bidder`, `sellToken`, `buyToken`, `sellAmount`, `buyAmount`, `filler` |

### Token & Analytics

| Entity | ID | Key Fields |
|--------|-----|------------|
| `token` | address | `symbol`, `decimals`, `totalSupply`, `currentHolderCount`, `totalMinted`, `totalBurned` |
| `tokenDailySnapshot` | `{token}-{day}` | `dailyRevenue`, `dailyTotalSupply`, `currentHolderCount`, `dailyMintAmount`, `dailyBurnAmount` |
| `tokenHourlySnapshot` | `{token}-{hour}` | Same as daily, hourly granularity |
| `tokenMonthlySnapshot` | `{token}-{month}` | `monthlyRevenue`, `monthlyMintAmount`, `cumulativeRevenue` |

### Accounts & Transfers

| Entity | ID | Key Fields |
|--------|-----|------------|
| `account` | address | (derived: balances, transfers, locks) |
| `accountBalance` | `{account}-{token}` | `amount`, `delegate { address }`, `firstHoldTimestamp` |
| `accountBalanceDailySnapshot` | `{account}-{token}-{day}` | `amount`, `timestamp` |
| `transferEvent` | `{token}-{txHash}-{logIndex}` | `amount`, `to`, `from`, `type` (MINT/BURN/TRANSFER) |
| `minting` | `{account}-{token}` | `amount`, `firstMintTimestamp` |

### Governance

| Entity | ID | Key Fields |
|--------|-----|------------|
| `governance` | governor address | `name`, `votingDelay`, `votingPeriod`, `proposalThreshold`, `quorumNumerator`, `proposalCount` |
| `governanceTimelock` | timelock address | `executionDelay`, `guardians[]`, `type` |
| `proposal` | auto-increment ID | `description`, `state`, `proposer`, votes (for/against/abstain weighted), `voteStart`, `voteEnd`, `executionTime`, `targets[]`, `calldatas[]` |
| `vote` | `{delegate}-{proposal}-{token}` | `choice` (FOR/AGAINST/ABSTAIN), `weight`, `reason` |
| `voteDailySnapshot` | day | `forWeightedVotes`, `againstWeightedVotes`, `totalWeightedVotes` |

### Staking & Delegation

| Entity | ID | Key Fields |
|--------|-----|------------|
| `stakingToken` | stToken address | `underlying { symbol }`, `token { symbol }`, `currentDelegates`, `totalDelegates`, `delegatedVotesRaw`, `delegates[]` |
| `delegate` | `{stToken}-{address}` | `address`, `delegatedVotesRaw`, `tokenHoldersRepresentedAmount`, `numberVotes` |
| `delegateChange` | unique | `delegator`, `delegate`, `previousDelegate` |
| `delegateVotingPowerChange` | unique | `delegate`, `previousBalance`, `newBalance` |
| `lock` | `{stToken}-{lockId}` | `amount`, `account`, `unlockTime`, `createdTimestamp` |
| `stakingTokenRewards` | `{stToken}-{rewardToken}` | `active` |

### RSR Burns (Index-only)

| Entity | ID | Key Fields |
|--------|-----|------------|
| `rsrBurn` | `{txHash}-{logIndex}` | `amount`, `burner`, `timestamp` |
| `rsrBurnGlobal` | "1" (singleton) | `totalBurned`, `totalBurnCount` |
| `rsrBurnDailySnapshot` | day | `dailyBurnAmount`, `dailyBurnCount`, `cumulativeBurned` |
| `rsrBurnMonthlySnapshot` | month | `monthlyBurnAmount`, `monthlyBurnCount`, `cumulativeBurned` |

### Other

| Entity | ID | Key Fields |
|--------|-----|------------|
| `version` | hash | `address` (deployer), `timestamp` |
| `unstakingManager` | address | `token` (stToken) |
| `timelockOperation` | bytes32 | `proposal`, `transactionHash` |

---

## Yield DTF Subgraph

**Chains:** Ethereum (1), Base (8453) — **NOT on BSC**
**Has USD prices** — `lastPriceUSD`, `rsrStakedUSD`, `totalValueLockedUSD`, `amountUSD` built-in.

### Core Entities

| Entity | ID | Key Fields |
|--------|-----|------------|
| `protocol` | main address | `rsrStaked`, `totalValueLockedUSD`, `cumulativeVolumeUSD`, `rTokenCount` |
| `rtoken` | RToken address | `backing`, `rsrStaked`, `rsrExchangeRate`, `holdersRewardShare`, `stakersRewardShare`, `owners[]`, `pausers[]`, `freezers[]`, `longFreezers[]`, `targetUnits`, `collateralDistribution` |
| `collateral` | ERC20 address | `symbol` |
| `trade` | auction address | `selling`, `buying`, `amount`, `boughtAmount`, `startedAt`, `endAt`, `isSettled` |
| `rtokenContract` | address | `name` |

### Token (with prices!)

| Entity | ID | Key Fields |
|--------|-----|------------|
| `token` | address | `symbol`, `decimals`, `totalSupply`, `holderCount`, `lastPriceUSD`, `lastMarketCapUSD`, `basketRate` |
| `tokenDailySnapshot` | `{token}-{day}` | `dailyTotalSupply`, `dailyVolume`, `priceUSD`, `basketRate`, `dailyMintAmount`, `dailyBurnAmount` |
| `tokenHourlySnapshot` | `{token}-{hour}` | Same, hourly granularity |
| `rewardToken` | `{type}-{address}` | `type` (DEPOSIT/BORROW), `token { symbol }` |

### Accounts

| Entity | ID | Key Fields |
|--------|-----|------------|
| `account` | address | (derived: entries, balances) |
| `accountBalance` | `{account}-{token}` | `amount`, `transferCount` |
| `accountBalanceDailySnapshot` | `{account}-{token}-{day}` | `amount`, `amountUSD` |
| `accountRToken` | `{account}-{rToken}` | `stake`, `totalStaked`, `pendingUnstake`, `totalRSRStaked`, `totalRSRUnstaked` |
| `accountStakeRecord` | `{account}-{rToken}-{txHash}` | `amount`, `rsrAmount`, `rsrPriceUSD`, `exchangeRate`, `isStake` |
| `activeAccount` | composite | Helper for unique user counting |

### Events

| Entity | ID | Key Fields |
|--------|-----|------------|
| `entry` | `{token}-{txHash}-{logIndex}` | `type`, `amount`, `amountUSD`, `to`, `stAmount` (stake/unstake) |
| `revenueDistribution` | `{account}-{rToken}` | `destination`, `rTokenDist`, `rsrDist` |

### Governance

| Entity | ID | Key Fields |
|--------|-----|------------|
| `governance` | RToken address | `currentDelegates`, `totalDelegates`, `delegatedVotesRaw`, `proposalsQueued`, `proposalsExecuted`, `guardians[]` |
| `governanceFramework` | governor address | `timelockAddress`, `executionDelay`, `votingDelay`, `votingPeriod`, `proposalThreshold`, `quorumNumerator` |
| `proposal` | auto-increment | `description`, `state`, `proposer`, votes, timing, execution data |
| `vote` | `{delegate}-{proposal}` | `choice`, `weight`, `reason` |
| `delegate` | address | `delegatedVotesRaw`, `tokenHoldersRepresentedAmount`, `numberVotes` |
| `delegateChange` | unique | `delegator`, `delegate`, `previousDelegate` |
| `delegateVotingPowerChange` | unique | `delegate`, `previousBalance`, `newBalance` |
| `tokenHolder` | address | `tokenBalanceRaw`, `totalTokensHeldRaw` |

### Timeseries

| Entity | ID | Key Fields |
|--------|-----|------------|
| `financialsDailySnapshot` | day | `totalValueLockedUSD`, `dailyVolumeUSD`, `cumulativeRTokenRevenueUSD`, `cumulativeRSRRevenueUSD` |
| `rtokenDailySnapshot` | `{rToken}-{day}` | `rsrStaked`, `rsrExchangeRate`, `dailyRTokenRevenueUSD`, `dailyRSRRevenueUSD`, `rsrPrice` |
| `rtokenHourlySnapshot` | `{rToken}-{hour}` | Same, hourly |
| `rtokenHistoricalBaskets` | RToken address | `targetUnits`, `collateralDistribution`, `rTokenDist`, `rsrDist` |
| `usageMetricsDailySnapshot` | day | `dailyActiveUsers`, `dailyTransactionCount`, `dailyRSRStaked` |
| `usageMetricsHourlySnapshot` | hour | Same, hourly |
| `stTokenDailySnapshot` | day | `totalSupply`, `tokenHolders`, `delegates` |
| `voteDailySnapshot` | day | `forWeightedVotes`, `againstWeightedVotes`, `totalWeightedVotes` |
| `accountRTokenDailySnapshot` | `{account}-{rToken}-{day}` | `stake`, `totalStaked`, `totalRSRStaked` |

---

## Common Agent Query Patterns

### Find DTFs by role holder

```graphql
# Index: Who has admin role on any DTF?
{ dtfs(where: { admins_contains: ["0x..."] }) { id token { symbol } admins } }

# Yield: Find pausers/freezers
{ rtokens(where: { pausers_contains: ["0x..."] }) { id token { symbol } pausers freezers } }
```

### Top holders by balance

```graphql
# Works on both subgraphs
{
  accountBalances(
    where: { token: "0x...", amount_gt: "0" }
    orderBy: amount, orderDirection: desc
    first: 20
  ) { account { id } amount }
}
```

### Delegation rankings

```graphql
# Index
{
  stakingToken(id: "0x_stToken") {
    currentDelegates delegatedVotesRaw
    delegates(first: 10, orderBy: delegatedVotesRaw, orderDirection: desc) {
      address delegatedVotesRaw tokenHoldersRepresentedAmount numberVotes
    }
  }
}
```

### Proposal voting analysis

```graphql
{
  votes(where: { proposal: "123" }) {
    voter { address }
    choice
    weight
    reason
  }
}
```

### Recent mints/burns

```graphql
# Index
{
  transferEvents(
    where: { token: "0x...", type: "MINT" }
    orderBy: timestamp, orderDirection: desc
    first: 20
  ) { amount to { id } timestamp }
}

# Yield
{
  entries(
    where: { token: "0x...", type: "MINT" }
    orderBy: timestamp, orderDirection: desc
    first: 20
  ) { amount amountUSD timestamp }
}
```

### Revenue snapshots over time

```graphql
# Index — daily token revenue
{
  tokenDailySnapshots(
    where: { token: "0x..." }
    orderBy: timestamp, orderDirection: desc
    first: 30
  ) { dailyRevenue dailyTotalSupply timestamp }
}

# Yield — protocol-level financials
{
  financialsDailySnapshots(
    orderBy: timestamp, orderDirection: desc
    first: 30
  ) { totalValueLockedUSD dailyVolumeUSD cumulativeRTokenRevenueUSD timestamp }
}
```

### Account staking positions (yield)

```graphql
{
  accountRTokens(where: { account: "0x..." }) {
    rToken { id token { symbol } }
    stake totalStaked totalRSRStaked
  }
}
```

### Historical basket (yield only)

```graphql
# Singleton per RToken — query by ID (= RToken address)
{ rtokenHistoricalBaskets(id: "0x_rtoken_address") {
    targetUnits collateralDistribution rTokenDist rsrDist
} }
```

### Protocol-level TVL (yield only)

```graphql
{ protocol(id: "0x_main_contract") { totalValueLockedUSD rTokenCount rsrStaked } }
```

### RSR burn tracking (index only)

```graphql
{
  rsrBurnDailySnapshots(orderBy: timestamp, orderDirection: desc, first: 30) {
    dailyBurnAmount dailyBurnCount cumulativeBurned timestamp
  }
}
```

### Rebalance auction bids (index only)

```graphql
{
  rebalanceAuctionBids(
    where: { dtf: "0x..." }
    orderBy: timestamp, orderDirection: desc
    first: 20
  ) {
    sellToken { symbol } buyToken { symbol }
    sellAmount buyAmount bidder timestamp
  }
}
```

### What DTFs does an account hold? (index)

```graphql
{
  accountBalances(
    where: { account: "0x...", amount_gt: "0" }
  ) { token { id symbol } amount }
}
```

### Active proposals across all DTFs

```graphql
{
  proposals(
    where: { state: "ACTIVE" }
    orderBy: creationTime, orderDirection: desc
    first: 20
  ) { id description governance { id } forWeightedVotes againstWeightedVotes voteEnd }
}
```

---

## Key Differences Between Subgraphs

| Feature | Index DTF | Yield DTF |
|---------|-----------|-----------|
| USD prices | No — use `dtf prices` | Yes — `lastPriceUSD`, `amountUSD` |
| Rebalance data | Full (rebalance, auction, bids) | Automatic (no user-facing) |
| RSR burns | Yes (rsrBurn, snapshots) | No |
| Basket history | No | Yes (rtokenHistoricalBaskets) |
| Protocol metrics | No | Yes (protocol entity) |
| Revenue snapshots | tokenDailySnapshot.dailyRevenue | financialsDailySnapshot |
| Transfer events | `transferEvent` entity | `entry` entity |
| Staking entity | `stakingToken` | `accountRToken` |
| Roles in entity | DTF has admins[], launchers[] | RToken has owners[], pausers[], freezers[] |

## Tips

- Keep `first` values reasonable (20-100). Max is 1000.
- Entity IDs are always lowercase addresses.
- **Shared entities** (proposals, delegates, token, accountBalance, vote, governance) exist in both subgraphs. `dtf query` defaults to the **index** subgraph for these — use `--subgraph yield` to override.
- For cross-chain queries, use `--chain all`.
- Subgraph introspection: `dtf query '{ __schema { types { name } } }' --json` to explore schema directly.
- **Cross-DTF role search**: Use `dtf query` to find DTFs where a specific address holds a role — e.g. `dtf query '{ dtfs(where: { admins_contains: ["0x..."] }) { id token { symbol } } }' --chain all --json` for index DTFs, or `dtf query '{ rtokens(where: { pausers_contains: ["0x..."] }) { id } }' --subgraph yield --chain all --json` for yield DTFs.
