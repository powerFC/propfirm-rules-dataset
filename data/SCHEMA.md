# Schema reference

`data/prop-firm-rules.json` structure and permitted values.

## Top level

| Field | Type | Notes |
|---|---|---|
| `schema_version` | string | Bumped on breaking structural changes. |
| `last_updated` | date | Last edit to the file. |
| `firms` | array | One object per firm. |
| `defunct_firms` | array | Firms recorded as collapsed or no longer trustworthy. |

## Firm object

| Field | Type | Notes |
|---|---|---|
| `firm` | string | Firm name as it trades. |
| `asset_classes` | array | `forex`, `cfd`, `futures`, `crypto`, `indices`, `commodities`. |
| `owner` | string, optional | Parent company where notable (e.g. Breakout → Kraken). |
| `platform` / `execution_venue` | string, optional | Where trades execute. |
| `plans` | array | One object per plan. Firms A/B test rules per plan, so plans are the unit of truth — not firms. |

## Plan object

| Field | Type | Notes |
|---|---|---|
| `plan_name` | string | As marketed by the firm. |
| `verified` | boolean | `true` only if read from the firm's own documentation. |
| `verified_date` | date or null | Date of that reading. |
| `source` | string | Where it was read from. |
| `account_sizes_usd` | array or null | `null` means not captured, not "none offered". |
| `phases` | array | Ordered. Funded stages are included as a final phase where the firm publishes rules for them. |
| `notes` | array of strings | Caveats, gotchas, and anything that doesn't fit a field. |

## Phase object

| Field | Type | Notes |
|---|---|---|
| `phase` | integer | 1-indexed. |
| `phase_name` | string | e.g. `Challenge`, `Verification`, `Trader Stage (funded)`. |
| `profit_target` | object | See units below. |
| `daily_loss` | object | See types below. |
| `max_drawdown` | object | See types below. |
| `min_trading_days` | integer or null | `null` = not captured. `0` = confirmed none. |
| `time_limit_days` | integer or null | `null` = no limit. |
| `consistency` | object | See types below. |
| `restrictions` | array of strings | Tags, listed below. |

## Value units

`profit_target`, and any dollar-denominated limit, uses one of:

| `unit` | Shape |
|---|---|
| `pct_of_initial` | `{ "unit": "pct_of_initial", "value": 10 }` |
| `usd_by_account_size` | `{ "unit": "usd_by_account_size", "values": { "50000": 3000 } }` |
| `usd_selectable_at_checkout` | `{ "options_usd": [2000, 1500] }` — the trader picks. |
| `none` | No target for this phase (funded stages). |
| `unconfirmed` | Not established. Do not substitute a guess. |

## `daily_loss.type`

| Value | Meaning |
|---|---|
| `none` | No daily limit. |
| `pct_initial` | Fixed dollar amount derived from the initial balance. Does not grow with the account. |
| `pct_prior_day` | Percentage of the balance at the daily reset. |
| `trailing_daily` | The daily limit itself trails. |
| `eod_soft_limit` | Pauses trading for the day; does **not** fail the account. |
| `eod` | Evaluated on end-of-day balances; value not captured. |
| `unconfirmed` | Not established. |

Additional keys: `failing` (boolean — whether breaching fails the account), `measured_on` (`equity_including_open_positions` where the firm specifies it), `reset_time_utc`.

## `max_drawdown.type`

| Value | Meaning |
|---|---|
| `static` | Fixed % of starting balance. Never moves. |
| `eod_trail` | Trails highest end-of-day closed balance. |
| `intraday_trail` | Trails highest unrealised equity peak. |
| `trailing_high_water_mark` | Trails the high water mark; see `locks_at`. |
| `eod_fixed_buffer` | Fixed dollar buffer, evaluated end-of-day. |
| `trailing` | Trails, but the exact basis is not confirmed. |
| `unconfirmed` | Not established. |

Additional keys: `locks_at` (`null` or `start_balance`), `confidence` (`unconfirmed`, `assumed_static`, `varies_by_plan`).

Full explanation of each type: [`../docs/drawdown-taxonomy.md`](../docs/drawdown-taxonomy.md).

## `consistency.type`

| Value | Meaning |
|---|---|
| `none` | Confirmed no consistency rule. |
| `best_day_pct_of_total` | No day above X% of total profit. |
| `best_day_pct_of_positive_days` | No day above X% of profit from winning days only. Stricter than it appears. |
| `best_trade_pct_of_total` | No single trade above X% of total profit. Cannot be derived from daily balances. |
| `unconfirmed` | Not established. |

Additional keys: `value_pct`, `applies_to` (`evaluation_only`, `all_stages_including_funded`).

## `restrictions` tags

| Tag | Meaning |
|---|---|
| `no_weekend_holding` | Positions must be closed before the weekend. |
| `mandatory_stop_loss_within_5_minutes` | Every position needs a stop within 5 minutes of entry. |
| `all_positions_closed_to_advance_stage` | Cannot progress a phase with open positions. |
| `daily_profit_cap_10000_usd` | Maximum profit per day. |
| `per_trade_profit_cap_10000_usd` | Maximum profit per trade. |

## Conventions

- Percentages are of the initial account balance unless the unit says otherwise.
- `null` means *not captured*. It never means *zero* or *none*.
- `0` means *confirmed zero*.
- A field is marked `unconfirmed` rather than filled with a plausible value. If you need a number that isn't here, get it from the firm.
