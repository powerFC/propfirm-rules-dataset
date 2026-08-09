# Prop firm drawdown types explained

Five mechanisms are in active use across the firms in this dataset. Two firms can both advertise "6% maximum drawdown" and be entirely different products. This page explains each one and what it means in practice.

---

## 1. Static

**The floor is a fixed percentage of the starting balance and never moves.**

$100,000 account with 10% static drawdown → the account fails if equity touches $90,000. That figure is the same on day one and on day ninety, whether the account is at $95,000 or $140,000.

*Practical effect:* every dollar of profit is a permanent buffer. This is the most forgiving structure and the reason forex firms using it feel easier than futures firms with the same headline number.

*Used by:* FTMO 2-Step, FundedNext Stellar 2-Step, FundingPips, The5%ers, Breakout 1-Step (all three variants), Bitfunded (assumed).

---

## 2. End-of-day trailing (`eod_trail`)

**The floor follows the highest end-of-day closed balance.**

$50,000 account with a $2,000 EOD trailing drawdown. Floor starts at $48,000. Close the day at $51,000 and the floor moves to $49,000. Intraday equity spikes are ignored entirely — only the closing balance counts.

*Practical effect:* profits tighten your room. Being up $3,000 at some point during a session means nothing if you close flat. The floor only moves when you bank it.

*Used by:* TopStep, Apex 4.0 EOD variant, MyFundedFutures Core and Pro.

---

## 3. Intraday trailing (`intraday_trail`)

**The floor follows the highest unrealised equity peak.**

$50,000 account with a $2,000 intraday trailing drawdown. Floor starts at $48,000. An open position runs to $53,000 unrealised, then you close it at breakeven. The floor is now $51,000 — and your balance is $50,000. You are already breached, having never closed a losing trade.

*Practical effect:* the harshest mechanism in common use, and the one that produces the most confused support tickets. Letting a winner run and then giving it back is fatal here in a way it isn't anywhere else.

*Tracking implication:* this cannot be computed from daily closing balances. You have to log the session's peak unrealised equity, which most journals and trackers do not capture.

*Used by:* Apex 4.0 Intraday variant, MyFundedFutures Rapid.

---

## 4. Trailing that locks (`trailing_high_water_mark` with `locks_at: start_balance`)

**The floor trails the high water mark, then freezes once it reaches the starting balance.**

Breakout's 2-Step uses 8% trailing on the high water mark and states that it will not trail past the starting balance. Official worked example: a high water mark of $105,000 gives a floor of $97,000. Once the high water mark is high enough that the trailing floor would exceed the starting balance, the floor stops there.

TopStep's Combine works the same way from the other direction: the trailing drawdown converts to a fixed floor at the starting balance once the account exceeds starting balance + drawdown amount.

*Practical effect:* punishing early, forgiving later. The dangerous window is the first stretch of profit, before the lock engages. Once locked, the worst case is returning to breakeven rather than being breached.

*Used by:* Breakout 2-Step, TopStep Trading Combine.

---

## 5. End-of-day fixed buffer (`eod_fixed_buffer`)

**A fixed dollar buffer below the starting balance, evaluated on end-of-day balances only.**

Behaves like static in that the floor doesn't move, but differs in that intraday excursions below the floor don't count — only the closing balance is checked.

MyFundedFutures Builder is the clearest case: the buffer size ($2,000 or $1,500 on a $50,000 account) is selected at checkout, so two traders on the same named plan can have different limits.

*Used by:* MyFundedFutures Flex and Builder.

---

## Daily loss limits are a separate axis

Drawdown type and daily loss limit are independent. The daily loss variants in this dataset:

| Type | Meaning |
|---|---|
| `pct_initial` | A fixed dollar amount derived from the initial balance. Does not grow with the account. (FTMO) |
| `pct_prior_day` | A percentage of the balance at the daily reset. Grows and shrinks with the account. (Breakout, reset at 00:30 UTC) |
| `trailing_daily` | The daily limit itself trails. Unusual. (HyroTrader) |
| `eod_soft_limit` | Hitting it pauses trading for the day but does **not** fail the account. (Apex EOD variant) |
| `none` | No daily limit at all. (MyFundedFutures, all plans; Apex Intraday; TopStep Combine) |

Two details that catch people out:

**What the limit is measured on.** Breakout evaluates the daily loss on equity *including open positions*, not on closed balance. An open drawdown at the wrong moment can breach you.

**Whether breaching it actually fails you.** Apex's EOD daily loss limit does not. Any tracker treating it as a hard breach will report failures that didn't happen.

---

## Consistency rules

Not a loss mechanism, but the most common cause of a *payout* being denied rather than an evaluation being failed. Three distinct formulations appear in this dataset:

| Type | Meaning |
|---|---|
| `best_day_pct_of_total` | No single day above X% of total profit. (MyFundedFutures Core 50%, The5%ers 50%) |
| `best_day_pct_of_positive_days` | No single day above X% of profit from winning days only. Denominator excludes losing days, so it is stricter than it looks. (FTMO 1-Step, 50%) |
| `best_trade_pct_of_total` | No single **trade** above X% of total profit. (HyroTrader, 40%) |

The per-trade variant is the awkward one for tooling: it can't be derived from daily balance entries, so any tracker working from daily P&L can only approximate it.

Consistency rules are typically evaluated at pass-request or payout-request time rather than continuously — which is why traders discover them after they've already made the money.

And note The5%ers: its 50%-per-day rule keeps applying once you're funded. Most firms drop consistency requirements after the evaluation.

---

*Part of the [Prop Firm Rules Dataset](../README.md). Maintained by [FuturesEdge](https://futuresedgetrade.app).*
