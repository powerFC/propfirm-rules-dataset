# Prop Firm Rules Dataset

An open, machine-readable dataset of proprietary trading firm evaluation rules — profit targets, daily loss limits, drawdown types, consistency rules and trading restrictions — with a verification date on every entry.

**12 firms. 26 plans. 22 of them read directly from the firm's own rules page rather than a third-party comparison site.**

Most published prop firm comparisons are affiliate listicles built from each other. Numbers get copied forward for years after the firm has changed them, and the single most important field — *what type of drawdown the firm uses* — is usually reduced to a percentage with no mention of whether it trails, when it trails, or what it trails on. That distinction is the difference between passing and being breached.

This dataset exists to be checkable. Every plan carries `verified: true|false` and a `verified_date`. Anything not confirmed against the firm's own documentation is explicitly marked unconfirmed rather than guessed at.

---

## Why drawdown type matters more than drawdown size

The most common way traders lose an evaluation is not hitting the daily loss limit. It is misunderstanding which balance their maximum drawdown is measured against. Five distinct mechanisms are in active use across the firms in this dataset:

| Type | What the floor follows | Consequence |
|---|---|---|
| `static` | A fixed % of the starting balance. Never moves. | Most forgiving. Profits build a permanent buffer. |
| `eod_trail` | The highest **end-of-day closed** balance. | Profits raise the floor overnight. Intraday spikes are ignored. |
| `intraday_trail` | The highest **unrealised equity** peak. | Harshest in common use. An open trade that goes green then reverses can breach you even if every trade closes profitable. |
| `trailing_high_water_mark` + `locks_at: start_balance` | Trails the high water mark, then freezes once the floor reaches the starting balance. | Trails early, stops trailing later. |
| `eod_fixed_buffer` | A fixed dollar buffer below the start, evaluated end-of-day. | Behaves like static but checked on closed balances only. |

Two firms can advertise "6% max drawdown" and be entirely different products. Full explanation in [`docs/drawdown-taxonomy.md`](docs/drawdown-taxonomy.md).

---

## Coverage

| Firm | Asset class | Plans | Verified | Last verified |
|---|---|---|---|---|
| FTMO | Forex / CFD | 2-Step, 1-Step | Yes | 2026-07-16 |
| TopStep | Futures | Trading Combine | Yes | 2026-07-11 |
| Apex Trader Funding | Futures | Apex 4.0 EOD, Apex 4.0 Intraday | Yes | 2026-07-11 |
| MyFundedFutures | Futures | Core, Rapid, Pro, Flex, Builder | Yes | 2026-07-11 |
| FundedNext | Forex / CFD | Stellar 2-Step (+2 partial) | Partial | 2026-07-16 |
| FundingPips | Forex / CFD | 2-Step | Yes | 2026-07-16 |
| The5%ers | Forex / CFD | High Stakes | Yes | 2026-07-16 |
| Breakout (Kraken) | Crypto | 1-Step Classic / Pro / Turbo, 2-Step | Yes | 2026-07-15 |
| HyroTrader | Crypto | 1-Step, 2-Step | Yes | 2026-07-16 |
| Bitfunded | Crypto | 2-Step, 1-Step, Instant | Yes | 2026-07-23 |
| Crypto Fund Trader | Crypto | Standard | No | — |
| Tradeify Crypto | Crypto | Standard | No | — |

---

## Corrections to widely-published errors

These are cases where the numbers circulating on comparison sites disagree with the firm's own documentation as of the verification date:

- **The5%ers daily loss is 3%, not 5%.** Commonly listed as 5%.
- **FundingPips Phase 2 target is 5%, not 10%.** Several comparison sites list 10%.
- **Apex's EOD daily loss limit does not fail the account.** It pauses trading for the day. Trackers that treat it as a failing breach will report false breaches. The intraday variant has no daily loss limit at all.
- **MyFundedFutures has no daily loss limit on any plan.** This is structural, not a per-plan quirk.
- **The5%ers' 50%-per-day consistency rule continues to apply in the funded stage**, not only during evaluation.
- **HyroTrader's consistency rule is per-trade, not per-day** — no single trade above 40% of total profit. It cannot be computed from daily balance entries alone.
- **HyroTrader's rules changed materially between April and July 2026.** Older third-party data on this firm is unreliable.

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
        for phase in plan["phases"]:
            if phase["daily_loss"]["type"] == "none":
                print(firm["firm"], plan["plan_name"])

# Only trust plans confirmed against the firm's own rules page
verified = [
    (f["firm"], p["plan_name"], p["verified_date"])
    for f in data["firms"] for p in f["plans"] if p["verified"]
]
```

Field-by-field reference: [`data/SCHEMA.md`](data/SCHEMA.md).

---

## FAQ

**Which prop firms have no daily loss limit?**
All five MyFundedFutures plans (Core, Rapid, Pro, Flex, Builder), Apex's Intraday Trailing variant, and TopStep's Trading Combine. Apex's EOD variant has a limit that pauses trading rather than failing the account.

**Which prop firms use static drawdown?**
FTMO 2-Step, FundedNext Stellar 2-Step, FundingPips, The5%ers, all three Breakout 1-Step plans, and Bitfunded (assumed static — the published tables do not state the type).

**Which prop firm has the harshest drawdown?**
Intraday trailing is the harshest mechanism, used by Apex's Intraday variant and MyFundedFutures Rapid. It follows unrealised equity, so an open position that moves into profit and back out can breach the account without a single losing closed trade.

**Which prop firms have no consistency rule?**
Breakout (all plans), Bitfunded (all plans), Apex during evaluation, FTMO's 2-Step, FundedNext's Stellar 2-Step, and MyFundedFutures Rapid, Pro, Flex and Builder.

**Which prop firms allow weekend holding?**
Breakout permits weekend holding and news trading. The5%ers prohibits holding over the weekend.

**Do prop firm rules change?**
Constantly, and often without announcement. Firms also A/B test rules between plans, which is why this dataset is structured per plan rather than per firm. Always confirm against the firm's own rules page before paying for an evaluation.

**Is this dataset affiliated with any prop firm?**
No. It contains no referral links and no rankings. It is a rules reference.

---

## Verification method

A plan is marked `verified: true` only when its values were read from the firm's own rules page, help centre or plan comparison table on the recorded date — not from a review site, not from a Discord, not from an affiliate blog. Screenshots were taken at verification time.

Fields that could not be confirmed are marked `unconfirmed` or carry a `confidence` key rather than being filled with a plausible guess. Known gaps are listed in [`docs/known-gaps.md`](docs/known-gaps.md).

Re-verification sweeps are run roughly quarterly. The next is due around October 2026.

---

## Contributing

Corrections are welcome and appreciated — see [`CONTRIBUTING.md`](CONTRIBUTING.md). The only hard requirement is that a correction cites the firm's own documentation. A link to a comparison site is not evidence.

---

## Licence and attribution

[CC BY 4.0](LICENSE). Free to use commercially, including in apps and comparison tools, provided you credit the source with a link:

> Prop firm rules data from [FuturesEdge](https://futuresedgetrade.app)

---

## Who maintains this

Maintained by **FuturesEdge**, which builds trading tools for prop firm, futures, forex and crypto traders. This dataset is the research layer behind those apps.

- **[futuresedgetrade.app](https://futuresedgetrade.app)** — firm-by-firm rule breakdowns and comparisons
- **[Prop Firm Challenge Tracker](https://play.google.com/store/apps/details?id=com.elitetrendhub.proptracker&referrer=utm_source%3Dgithub%26utm_medium%3Ddataset)** — applies these rules to a live challenge and tells you how close you are to breaching. Free for two challenges.
- **[Position Size Calculator](https://play.google.com/store/apps/details?id=com.elitetrendhub.calculator&referrer=utm_source%3Dgithub%26utm_medium%3Ddataset)** — free
- **[Trading Journal](https://play.google.com/store/apps/details?id=com.elitetrendhub.journal&referrer=utm_source%3Dgithub%26utm_medium%3Ddataset)**

Contact: futuresedgetrade@gmail.com

---

## Disclaimer

This dataset is provided for information only. It is not financial advice, not a recommendation of any firm, and no warranty is given as to accuracy or completeness. Prop firm rules change without notice. Verify with the firm before risking money. Trading involves substantial risk of loss.
