# Prop Firm Rules Dataset

An open, machine-readable dataset of proprietary trading firm evaluation rules — profit targets, daily loss limits, drawdown mechanics, consistency rules and trading restrictions — with a verification date on every entry.

**9 firms. 29 plans. Every one read directly from the firm's own rules page or help centre, not from a comparison site.**

Most published prop firm comparisons are affiliate listicles built from each other. Numbers get copied forward for years after the firm has changed them, and the single most important field — *what type of drawdown the firm uses* — is usually reduced to a percentage with no mention of whether it trails, when it trails, or what it trails on. That distinction is the difference between passing and being breached.

**This file is generated, not hand-written.** It's exported directly from the rules engine behind the [Prop Firm Challenge Tracker](https://play.google.com/store/apps/details?id=com.elitetrendhub.proptracker&referrer=utm_source%3Dgithub%26utm_medium%3Ddataset), which is covered by a unit test suite. The engine is the single source of truth, so the dataset can't drift from the code that's actually being tested against real challenges.

---

## Why drawdown type matters more than drawdown size

The most common way traders lose an evaluation is not hitting the daily loss limit. It's misunderstanding which balance the maximum drawdown is measured against.

| Type | What the floor follows | Consequence |
|---|---|---|
| `static` | A fixed % of the starting balance. Never moves. | Most forgiving. Profit becomes a permanent buffer. |
| `eod_trail` | The highest **end-of-day closed** balance. | Profit raises the floor overnight. Intraday spikes are ignored. |
| `eod_trail` + `locks_at: start_balance` | Trails end-of-day, then freezes once the floor reaches the starting balance. | Punishing early, forgiving later. Worst case becomes breakeven rather than breached. |
| `intraday_trail` | The highest **unrealised equity** peak. | Harshest in use. An open trade that goes green then reverses can breach you with no losing closed trade. |

Two firms can both advertise "6% max drawdown" and be completely different products. Full explanation in [`docs/drawdown-taxonomy.md`](docs/drawdown-taxonomy.md).

Daily loss limits are a separate axis with four variants in this dataset: `pct_initial` (fixed dollar amount from the starting balance), `pct_prior_day` (recalculated at each daily reset), `trailing_intraday_high` (the limit itself trails your intraday high — the harshest form), and `none`.

---

## Coverage

| Firm | Market | Plans | Last verified |
|---|---|---|---|
| FTMO | Forex / CFD | 2-Step, 1-Step | 2026-07-16 |
| TopStep | Futures | 50K, 100K, 150K Combine | 2026-07-11 |
| Apex Trader Funding | Futures | EOD Trail, Intraday Trail | 2026-07-11 |
| FundedNext | Forex / CFD | Stellar 2-Step, 1-Step, Lite, Instant | 2026-07-16 |
| FundingPips | Forex / CFD | 1 Step, 2 Step Standard, Flex, Pro, Zero | 2026-07-16 |
| The5%ers | Forex / CFD | 1-Step, 2-Step (8/5) | 2026-07-16 |
| Breakout (Kraken) | Crypto | 1-Step Classic, Pro, Turbo, 2-Step | 2026-07-15 |
| HyroTrader | Crypto | 1-Step & 2-Step, Standard & Swing | 2026-07-16 |
| Bitfunded | Crypto | 2-Step, 1-Step, Instant | 2026-07-23 |

Three further firms — MyFundedFutures, Crypto Fund Trader and Tradeify Crypto — appear under `research_only`. They're documented but not encoded in the engine, so they haven't been through its tests. The distinction is kept explicit rather than blurred.

---

## Corrections to widely-published errors

Cases where figures circulating on comparison sites disagree with the firm's own documentation as of the verification date:

- **FundingPips' 2 Step Standard Phase 2 target is 5%, not 10%.** Widely listed as 10%.
- **The5%ers' daily loss is 3%, not 5%.**
- **The5%ers' consistency rule is plan-specific.** The 1-Step carries a 50% best-day rule that continues to apply *on the funded stage*, not just during evaluation. The 2-Step (8/5) has no consistency rule during evaluation at all. Treating the firm as having one rule is wrong.
- **HyroTrader's consistency rule is measured per DAY, not per trade** — no single day above 40% of total profit. The per-trade version is widely repeated and incorrect. (It was in this dataset's first release too; corrected here.)
- **HyroTrader's 2-Step max loss is 10%, not 6%.** The 6% figure belongs to the 1-Step.
- **HyroTrader's Standard plans use a trailing intraday daily loss limit**, which tracks your intraday high rather than a fixed figure. The Swing upgrade replaces it with a conventional prior-day limit. Same plan family, materially different risk.
- **HyroTrader's rules changed materially between April and July 2026.** Any third-party data on this firm older than that is unreliable.
- **Apex's two variants differ on daily loss, not just drawdown.** The EOD Trail variant has a daily loss limit; the Intraday Trail variant has none at all.
- **TopStep and Breakout's 2-Step both lock their trailing drawdown at the starting balance.** Once locked, the worst case is returning to breakeven — a fundamentally different risk profile from drawdown that trails forever.
- **FundedNext's Stellar Instant carries a trailing drawdown that locks at the starting balance**, which is unusual for an instant-funding product and easy to miss.
- **Breakout's 1-Step Turbo pairs a 9% target with a 3% static drawdown** — the tightest target-to-buffer ratio in the dataset.

---

## Using the data

```bash
curl -O https://raw.githubusercontent.com/powerFC/propfirm-rules-dataset/main/data/prop-firm-rules.json
```

```python
import json

data = json.load(open("data/prop-firm-rules.json"))

# Every plan with no daily loss limit
for firm in data["firms"]:
    for plan in firm["plans"]:
        phases = plan.get("phases") or list(plan["phases_by_account_size"].values())[0]
        if all(p["daily_loss"]["type"] == "none" for p in phases):
            print(firm["firm"], "-", plan["plan_name"])

# Every plan whose drawdown follows unrealised equity
for firm in data["firms"]:
    for plan in firm["plans"]:
        phases = plan.get("phases") or list(plan["phases_by_account_size"].values())[0]
        if any(p["max_drawdown"]["type"] == "intraday_trail" for p in phases):
            print(firm["firm"], "-", plan["plan_name"])
```

Note the two shapes: most plans have a flat `phases` array where every value is a percentage that scales to any account size. A few — where target, drawdown and daily loss don't reduce to one clean percentage across sizes — use `phases_by_account_size` instead. Handle both.

Field-by-field reference: [`data/SCHEMA.md`](data/SCHEMA.md).

---

## FAQ

**Which prop firms have no daily loss limit?**
All three TopStep Combines, Apex's Intraday Trail variant, and FundedNext's Stellar Instant. Among the research-only firms, MyFundedFutures has no daily loss limit on any plan — that's its main structural differentiator.

**Which prop firms use static drawdown?**
FTMO's 2-Step; FundedNext's 2-Step, 1-Step and Lite; all five FundingPips plans; both The5%ers plans; Breakout's three 1-Step plans; all four HyroTrader plans; and all three Bitfunded plans.

**Which prop firm has the harshest drawdown?**
Apex's Intraday Trail variant, because the floor follows unrealised equity — an open position that moves into profit and back out can breach the account without a single losing closed trade. HyroTrader's Standard plans apply the same idea to the *daily* limit via `trailing_intraday_high`.

**Which prop firms have a consistency rule?**
Only four: FTMO's 1-Step (50% of profit from positive days), FundingPips' Zero (15% best day, plus 7 profitable days required), The5%ers' 1-Step (50% best day, funded stage included), and all four HyroTrader plans (40% best day). Every other plan in the dataset has none.

**Which prop firm has the tightest drawdown?**
Breakout's 1-Step Turbo — 3% static against a 9% profit target.

**Do prop firm rules change?**
Constantly, and often without announcement. Firms also A/B test rules between plans, which is why this dataset is structured per plan rather than per firm. Always confirm against the firm's own rules page before paying for an evaluation.

**Is this dataset affiliated with any prop firm?**
No. It contains no referral links and no rankings. It's a rules reference.

---

## Verification method

A plan is marked `verified: true` only when its values were read from the firm's own rules page, help centre or plan comparison table on the recorded date — not from a review site, a Discord, or an affiliate blog. Screenshots were taken at verification time.

Where a field couldn't be confirmed, the plan's `notes` say so explicitly rather than the value being filled with a plausible guess. Every known gap is listed in [`docs/known-gaps.md`](docs/known-gaps.md).

Re-verification sweeps run roughly quarterly. Next due around October 2026.

---

## Contributing

Corrections are welcome — see [`CONTRIBUTING.md`](CONTRIBUTING.md). One hard requirement: a correction must cite the firm's own documentation. A link to a comparison site isn't evidence.

Note that `data/prop-firm-rules.json` is **generated**, so corrections are applied to the engine and re-exported rather than edited in the JSON directly. Open an issue with the firm, plan, field and source and it'll be carried through.

---

## Licence and attribution

[CC BY 4.0](LICENSE). Free to use commercially, including in apps and comparison tools, provided you credit the source with a link:

> Prop firm rules data from [FuturesEdge](https://futuresedgetrade.app)

---

## Who maintains this

Maintained by **FuturesEdge**, which builds trading tools for prop firm, futures, forex and crypto traders. This dataset is the research layer behind those apps.

- **[futuresedgetrade.app](https://futuresedgetrade.app)** — firm-by-firm breakdowns and comparisons
- **[Prop Firm Challenge Tracker](https://play.google.com/store/apps/details?id=com.elitetrendhub.proptracker&referrer=utm_source%3Dgithub%26utm_medium%3Ddataset)** — applies these rules to a live challenge and tells you how close you are to breaching. Free for two challenges.
- **[Prop Firm Calculator](https://play.google.com/store/apps/details?id=com.elitetrendhub.compound&referrer=utm_source%3Dgithub%26utm_medium%3Ddataset)** — expected value of a challenge, and a Monte Carlo survival simulator
- **[Position Size Calculator](https://play.google.com/store/apps/details?id=com.elitetrendhub.calculator&referrer=utm_source%3Dgithub%26utm_medium%3Ddataset)** — free
- **[Trading Journal](https://play.google.com/store/apps/details?id=com.elitetrendhub.journal&referrer=utm_source%3Dgithub%26utm_medium%3Ddataset)**

Contact: futuresedgetrade@gmail.com

---

## Disclaimer

Provided for information only. Not financial advice, not a recommendation of any firm, and no warranty is given as to accuracy or completeness. Proprietary trading firm rules change without notice. Verify with the firm before risking money. Trading involves substantial risk of loss.
