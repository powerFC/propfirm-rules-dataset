# Known gaps

Fields that are not confirmed, listed openly so the dataset can be trusted where it doesn't carry a caveat. Corrections citing firm documentation are welcome — see [`../CONTRIBUTING.md`](../CONTRIBUTING.md).

## Unconfirmed drawdown mechanics

| Firm / plan | Gap |
|---|---|
| FTMO 1-Step | Whether the 10% max loss is static or trailing. Marked `trailing` with `confidence: unconfirmed`. |
| Bitfunded (all plans) | The published plan tables state percentages but not whether max loss is static or trailing. Static assumed. |
| FundingPips | Static assumed for the 2-Step; not explicitly stated in the plan table. |
| The5%ers | Static assumed; not explicitly stated. |

## Missing account sizes

`account_sizes_usd: null` on FTMO, FundedNext, FundingPips, The5%ers and HyroTrader. The plans exist; the size tiers were not captured at verification time.

## Missing targets

| Firm / plan | Gap |
|---|---|
| HyroTrader 1-Step and 2-Step | Profit target not captured. Loss limits, minimum days, consistency and restrictions are confirmed. |

## Unverified firms

**Crypto Fund Trader** — the profit caps and general structure come from secondary sources. Targets and drawdown limits are unknown. Operating since November 2022 with a long payout history, so the firm itself is well established; only the numbers here are unconfirmed.

**Tradeify Crypto** — launched February 2026. Structure (EOD daily loss, instant funding option, account caps, 80% split) from secondary sources. No specific rule values confirmed. No crypto payout history yet.

## Partially encoded plan lineups

Several firms run more plans than are encoded here:

- **FundedNext** — a separate futures line (Flex, Legacy, Rapid) is not encoded. Stellar Instant is not encoded.
- **FundingPips** — five plans exist; only the 2-Step is encoded. Daily loss varies 3–5% across them.
- **MyFundedFutures** — funded-phase rules not encoded, only evaluation.
- **Apex** — funded (PA) phase rules summarised in notes rather than encoded as a phase.

## Funded-phase coverage generally

The dataset is strongest on evaluation rules and weakest on funded-phase rules. Payout minimums, qualifying-day requirements, payout caps and funded-stage consistency rules are where most payout denials originate, and they are the least consistently documented by firms. Bitfunded and The5%ers are the only entries with funded stages encoded as phases.

This is the clearest area for contribution.

## Verification cadence

Sweeps are run roughly quarterly. Last full sweep: 16 July 2026, with Bitfunded added 23 July 2026. Next due around October 2026.

Prop firms change rules without announcement. Anything more than three months past its `verified_date` should be re-checked before you rely on it.
