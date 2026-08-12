# Schema reference

`data/prop-firm-rules.json`, schema version **2.0**.

This file is **generated** from the FuturesEdge rules engine rather than hand-maintained, so field names mirror the engine's own contract. Corrections go into the engine and are re-exported — see [`../CONTRIBUTING.md`](../CONTRIBUTING.md).

## Top level

| Field | Type | Notes |
|---|---|---|
| `schema_version` | string | Bumped on breaking structural changes. 2.0 renamed fields to match the engine exactly. |
| `last_updated` | date | Date of the last export. |
| `generated_from` | string | Provenance statement. |
| `firms` | array | Engine-backed firms. Every plan here is `verified: true`. |
| `research_only` | array | Firms documented from research but NOT encoded in the engine, so not covered by its tests. Prose `key_facts` rather than structured phases, deliberately — so they can't be mistaken for engine data. |
| `defunct_firms` | array | Firms recorded as collapsed or no longer trustworthy. |

## Firm object

| Field | Type | Notes |
|---|---|---|
| `firm` | string | Firm name as it trades. |
| `asset_classes` | array | `forex`, `cfd`, `futures`, `crypto`. |
| `owner` | string, optional | Parent company where notable (Breakout → Kraken). |
| `execution_venue` | string, optional | Where trades execute (HyroTrader → Bybit). |
| `firm_note` | string, optional | Scope caveat, e.g. FundedNext's separate futures line not being covered. |
| `plans` | array | Firms A/B test rules per plan, so the **plan** is the unit of truth — never the firm. |

## Plan object

| Field | Type | Notes |
|---|---|---|
| `plan_id` | string | Stable identifier, matches the engine's preset id. |
| `plan_name` | string | As marketed by the firm. |
| `market` | string | `forex`, `futures` or `crypto`. |
| `verified` | boolean | `true` only if read from the firm's own documentation. |
| `rules_verified` | date | Date of that reading. |
| `account_sizes_usd` | array or null | `null` means not captured, never "none offered". |
| `notes` | array of strings | Caveats and gotchas. **Read these** — anything unconfirmed is stated here rather than guessed at in a field. |
| `phases` | array | Ordered. Present on most plans. |
| `phases_by_account_size` | object | Present *instead of* `phases` where the numbers don't reduce to one clean percentage across sizes (Apex). Keys are account sizes as strings. |

Consumers must handle both shapes:

```python
phases = plan.get("phases") or list(plan["phases_by_account_size"].values())[0]
```

## Phase object

| Field | Type | Notes |
|---|---|---|
| `profit_target_pct` | number | Percent of initial balance. `0` means no target — the phase represents a funded stage. |
| `daily_loss` | object | `{ type, value }`. See below. |
| `max_drawdown` | object | `{ type, value, locks_at }`. See below. |
| `min_trading_days` | integer or null | `0` = confirmed none. `null` = not captured. |
| `min_profitable_days` | integer, optional | Only present where a firm requires it (FundingPips Zero: 7). |
| `time_limit_days` | integer or null | `null` = no limit. |
| `consistency` | object | `{ type, value }`. See below. |
| `restrictions` | array of strings | Tags, listed below. |

## `daily_loss.type`

| Value | Meaning |
|---|---|
| `none` | No daily limit. `value` omitted. |
| `pct_initial` | Fixed dollar amount derived from the initial balance. Does not grow with the account. |
| `pct_prior_day` | Percentage of the balance at the daily reset. |
| `trailing_intraday_high` | The limit itself trails the intraday high. Harshest daily form. |

## `max_drawdown.type`

| Value | `value` unit | Meaning |
|---|---|---|
| `static` | percent | Fixed % of starting balance. Never moves. |
| `eod_trail` | percent | Trails the highest end-of-day closed balance. |
| `intraday_trail` | percent | Trails the highest unrealised equity peak. |
| `eod_fixed_buffer` | **dollars** | Fixed dollar buffer, evaluated end-of-day. Not currently used by any engine-backed plan. |

`locks_at` is either `none` or `start_balance`. `start_balance` means the floor stops trailing once it reaches the starting balance — see [`../docs/drawdown-taxonomy.md`](../docs/drawdown-taxonomy.md).

Note the unit inconsistency on `eod_fixed_buffer` is deliberate and inherited from the engine: a plan quoting a flat percentage uses `static`, while a plan quoting a flat dollar figure regardless of account size uses `eod_fixed_buffer`. They compute an identical fixed floor; the distinction records how the firm expresses it.

## `consistency.type`

| Value | Meaning |
|---|---|
| `none` | Confirmed no consistency rule. `value` omitted. |
| `best_day_pct_of_total` | No day above X% of total profit. |
| `best_day_pct_of_positive_days` | No day above X% of profit from winning days only. Stricter than it appears. |

There is no per-trade consistency type. A per-trade rule is frequently attributed to HyroTrader and is incorrect — see the note in the taxonomy doc.

## `restrictions` tags

| Tag | Meaning |
|---|---|
| `inactivity_60d` | Account closed after 60 days without trading. |
| `max_loss_per_trade_3pct` | No single trade may lose more than 3% of the account. |
| `no_martingale` | Martingale / grid position stacking prohibited. |
| `no_cross_account_hedging` | Cannot hedge one funded account against another. |
| `lowcap_exposure_5pct` | Low-capitalisation asset exposure capped at 5%. |

## Conventions

- Percentages are of the **initial** account balance unless the type says otherwise.
- `null` means *not captured*. It never means *zero* or *none*.
- `0` means *confirmed zero*.
- Fields are never filled with a plausible guess. If a value couldn't be confirmed, the plan's `notes` say so.
