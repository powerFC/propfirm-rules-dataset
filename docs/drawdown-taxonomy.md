# Prop firm drawdown and loss-limit mechanics

Two firms can both advertise "6% max drawdown" and be entirely different products. This page explains every mechanism that appears in the dataset, and what each one means in practice.

---

## Maximum drawdown

### `static`

**The floor is a fixed percentage of the starting balance and never moves.**

A $100,000 account with 10% static drawdown fails if equity touches $90,000. That figure is identical on day one and day ninety, whether the account sits at $95,000 or $140,000.

*Effect:* every dollar of profit is a permanent buffer. The most forgiving structure, and the reason forex firms using it feel easier than futures firms quoting the same headline number.

*In the dataset:* FTMO 2-Step, FundedNext (2-Step, 1-Step, Lite), all five FundingPips plans, both The5%ers plans, Breakout's three 1-Step plans, all four HyroTrader plans, all three Bitfunded plans.

---

### `eod_trail`

**The floor follows the highest end-of-day closed balance.**

A $50,000 account with a 4% trailing drawdown starts with a floor at $48,000. Close the day at $51,000 and the floor moves to $49,000. Intraday equity spikes are ignored entirely — only the closing balance counts.

*Effect:* profit tightens your room. Being up $3,000 at some point during a session means nothing if you close flat. The floor only moves when you bank it.

*In the dataset:* FTMO 1-Step, Apex's EOD Trail variant.

---

### `eod_trail` with `locks_at: start_balance`

**Trails end-of-day, then freezes once the floor reaches the starting balance.**

TopStep's Combine trails the highest end-of-day balance until the floor would exceed the starting balance, at which point it stops and becomes fixed there. Breakout's 2-Step works the same way — its official worked example gives a high water mark of $105,000 producing a floor of $97,000, and states the floor will not trail past the starting balance.

*Effect:* punishing early, forgiving later. The dangerous window is the first stretch of profit, before the lock engages. Once locked, the worst case is returning to breakeven rather than being breached.

*In the dataset:* TopStep (all three Combines), Breakout 2-Step, FundedNext Stellar Instant.

That last one is worth noting — an instant-funding product with a trailing drawdown is easy to miss when you're not expecting an evaluation-style rule.

---

### `intraday_trail`

**The floor follows the highest unrealised equity peak.**

A $50,000 account with a 4% intraday trailing drawdown starts with a floor at $48,000. An open position runs to $53,000 unrealised, then you close it at breakeven. The floor is now $51,000 and your balance is $50,000 — you are already breached, having never closed a losing trade.

*Effect:* the harshest mechanism in use, and the one that produces the most confused support tickets. Letting a winner run and then giving it back is fatal here in a way it isn't anywhere else.

*Tracking implication:* this cannot be computed from daily closing balances. You have to log the session's peak unrealised equity, which most journals and trackers never capture.

*In the dataset:* Apex's Intraday Trail variant.

---

### `eod_fixed_buffer`

**A fixed dollar buffer below the starting balance, evaluated on end-of-day balances only.**

Behaves like static in that the floor doesn't move, but differs in that intraday excursions below it don't count — only the closing balance is checked.

*In the dataset:* not currently present among the engine-backed firms. It appears in MyFundedFutures' Flex and Builder plans, which are documented under `research_only`. Builder is the clearest case: the buffer ($2,000 or $1,500 on a $50,000 account) is selected at checkout, so two traders on the same named plan can have different limits.

---

## Daily loss limits

Drawdown type and daily loss limit are independent axes. Four variants appear in the dataset:

| Type | Meaning | Where |
|---|---|---|
| `pct_initial` | A fixed dollar amount derived from the initial balance. Does not grow with the account. | FTMO, FundedNext, FundingPips, The5%ers, Bitfunded, Apex EOD |
| `pct_prior_day` | A percentage of the balance at the daily reset, so it moves with the account. | Breakout, HyroTrader Swing |
| `trailing_intraday_high` | The limit itself trails your intraday high. Harshest daily form. | HyroTrader Standard |
| `none` | No daily limit at all. | TopStep, Apex Intraday, FundedNext Stellar Instant |

Two details that catch people out:

**What the limit is measured on.** Breakout evaluates the daily loss on equity *including open positions*, at a reset from 00:30 UTC. FundedNext's daily loss also includes unrealised P&L, resetting at 00:00 server time. An open drawdown at the wrong moment can breach you even if you close the day flat.

**HyroTrader's Standard vs Swing distinction.** Same plan family, same target, same max loss — but Standard's daily limit trails your intraday high while Swing's is a conventional prior-day figure. Swing is a paid checkout upgrade, and the only thing it buys is the softer daily-loss mechanism. Worth knowing before paying for it, and worth knowing before assuming a "HyroTrader" figure you read somewhere applies to your plan.

---

## Consistency rules

Not a loss mechanism, but the most common cause of a *payout* being denied rather than an evaluation being failed. Two formulations appear:

| Type | Meaning |
|---|---|
| `best_day_pct_of_total` | No single day above X% of total profit. |
| `best_day_pct_of_positive_days` | No single day above X% of profit from winning days only. The denominator excludes losing days, so it's stricter than it first looks. |

Only four plans in the dataset have one at all:

- **FTMO 1-Step** — 50% of profit from positive days
- **FundingPips Zero** — 15% best day, and 7 profitable days required
- **The5%ers 1-Step** — 50% best day, and it **keeps applying on the funded stage**. Most firms drop consistency requirements after the evaluation; this one doesn't.
- **HyroTrader** (all four plans) — 40% best day

Everything else has none. That's worth stating plainly, because consistency rules are often described as universal in this industry when they aren't.

**A common error worth naming:** HyroTrader's rule is frequently reported as a per-*trade* limit — no single trade above 40% of total profit. It isn't; it's per-day. The first release of this dataset repeated that error. If a source tells you a firm's consistency rule is per-trade, check it, because daily-balance tooling can't compute a per-trade rule and the two produce very different answers.

Consistency rules are typically evaluated at pass-request or payout-request time rather than continuously, which is why traders discover them after they've already made the money.

---

## Restriction tags

| Tag | Meaning | Where |
|---|---|---|
| `inactivity_60d` | Account closed after 60 days without trading. | FundedNext (all plans) |
| `max_loss_per_trade_3pct` | No single trade may lose more than 3% of the account. | HyroTrader |
| `no_martingale` | Martingale and grid-style position stacking prohibited. | HyroTrader |
| `no_cross_account_hedging` | Cannot hedge one funded account against another. | HyroTrader |
| `lowcap_exposure_5pct` | Exposure to low-capitalisation assets capped at 5%. | HyroTrader |

---

*Part of the [Prop Firm Rules Dataset](../README.md). Maintained by [FuturesEdge](https://futuresedgetrade.app).*
