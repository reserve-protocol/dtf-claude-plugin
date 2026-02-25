# DTF Protocol Plugin for Claude Code

Query and interact with [Reserve Protocol](https://reserve.org) Index DTFs directly from Claude Code.

## Install

```bash
# In Claude Code
/plugin install github:reserve-protocol/dtf-claude-plugin
```

Or test locally:

```bash
claude --plugin-dir ./dtf-plugin
```

## What It Does

Gives Claude the ability to:

- **Discover DTFs** across Ethereum, Base, and BSC
- **Inspect baskets** — token composition, weights, USD values, TVL
- **Monitor rebalances** — active auctions, progress, time windows
- **Check governance** — proposals, voting settings, roles
- **Get quotes** — mint/redeem with exact token amounts
- **Analyze revenue** — fee breakdowns, RSR burn projections
- **Compare yield** — vote-lock staking opportunities across chains

All data is live from the blockchain via the [`@reserve-protocol/dtf-cli`](https://www.npmjs.com/package/@reserve-protocol/dtf-cli).

## Requirements

- Node.js >= 18
- The CLI is installed automatically via `npx` (no manual setup needed)

## Commands Available

The plugin enables 18 CLI commands. See `skills/dtf/reference.md` for the full reference.

| Command | Description |
|---------|-------------|
| `discover` | List DTFs across chains |
| `info` | DTF config and metadata |
| `basket` | Basket composition and TVL |
| `fees` | Fee info and recipients |
| `quote` | Mint/redeem quotes |
| `prices` | Token prices and volatility |
| `governance` | Voting settings and timelock |
| `staking` | Vote-lock info and rewards |
| `roles` | Role holders |
| `proposals` | Governance proposals |
| `rebalance` | Active rebalance state |
| `history` | Rebalance history |
| `earn` | Yield opportunities |
| `revenue` | Revenue breakdown |
| `rsr-burns` | RSR burn analytics |
| `deploy` | Deploy a new DTF |
| `forum` | Governance forum |
| `cache-clear` | Clear disk cache |

## Links

- [DTF SDK](https://www.npmjs.com/package/@reserve-protocol/dtf-sdk)
- [DTF CLI](https://www.npmjs.com/package/@reserve-protocol/dtf-cli)
- [Reserve Protocol](https://reserve.org)
- [Register App](https://app.reserve.org)
