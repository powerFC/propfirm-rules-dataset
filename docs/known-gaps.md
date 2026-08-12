# Known gaps

Listed openly so the rest of the dataset can be trusted where it doesn't carry a caveat. Every item below also appears in the relevant plan's `notes` in the JSON.

## Drawdown type not stated by the firm

Several firms publish a maximum-loss percentage without saying whether the floor is static or trailing. Where that's the case the value is encoded conservatively and the plan's notes say so.

| Firm / plan | Gap |
|---|---|
| FTMO 1-Step | Static vs trailing not confirmed on FTMO's official rules table. Encoded as `eod_trail` per secondary sources — treat the floor conservatively. |
| FundingPips (all five plans) | Static assumed pending confirmation at fundingpips.com. |
| The5%ers (both plans) | Static assumed pending confirmation. |
| HyroTrader (all four plans) | Overall max loss encoded as static; the official rules table doesn't indicate whether it trails. |
| Bitfunded (all plans) | The published plan tables state percentages but not the mechanism. Static assumed. |

## Account sizes not re-verified

**FundedNext** — the `account_sizes_usd` list on all four Stellar plans is inherited from an earlier preset and hasn't been re-confirmed against the current Stellar lineup. The rule values *are* verified; only the size tiers are stale.

## Modelling limitations rather than data gaps

These are cases where the firm's rule is known but can't be fully represented from daily closed P&L:

**Unrealised P&L in daily loss limits.** FundedNext and Breakout both evaluate the daily loss on equity including open positions. A day with a large intraday open drawdown that closed back out won't be reflected in daily-balance tooling.

**HyroTrader's "qualifying" trading day.** HyroTrader's own rules require a trade of at least 5% of the initial balance with P&L beyond ±1% for a day to count toward the minimum. Any tooling counting all logged days is looser than the official definition.

**TopStep payout eligibility.** Five winning days of at least $150 each is a payout condition, separate from evaluation pass/fail, and isn't modelled.

**The5%ers 2-Step funded stage.** The 2-Step (8/5) has no consistency rule during evaluation, but the funded stage carries a 50% best-day rule. Only the evaluation phases are encoded.

**The5%ers 2-Step target variant.** A 10%/5% variant exists at checkout alongside the 8%/5% encoded here.

## Plan lineups not fully covered

- **FundedNext** — the separate futures line (Flex, Legacy, Rapid) isn't covered. This dataset holds the CFD line only.
- **Apex** — funded (PA) phase rules are summarised in notes rather than encoded as a phase. A 50% consistency rule applies there but not during evaluation.
- **Bitfunded** — funded Trader Stage limits are documented but only the evaluation phases are encoded.

## Funded-phase coverage generally

The dataset is strongest on evaluation rules and weakest on funded-stage rules. Payout minimums, qualifying-day requirements, payout caps and funded-stage consistency rules are where most payout *denials* originate, and they're the least consistently documented by firms.

This is the clearest area for contribution.

## Research-only firms

**MyFundedFutures** — documented from help.myfundedfutures.com on 2026-07-11 but not encoded in the engine, so not covered by its tests. The figures are usable; they just haven't been through the same validation.

**Crypto Fund Trader** — secondary sources only. Targets and drawdown limits unknown. The firm itself is well established (operating since November 2022 with a long payout history); only the numbers here are unconfirmed.

**Tradeify Crypto** — secondary sources only. Launched February 2026 with no crypto payout history yet. No specific rule values confirmed.

## Verification cadence

Last full sweep: 16 July 2026, with Bitfunded added 23 July 2026. Next due around October 2026.

Prop firms change rules without announcement. Anything more than three months past its `rules_verified` date should be re-checked before you rely on it.
