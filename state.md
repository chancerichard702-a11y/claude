# State Log — Running Cycle History

Shared log across both active strategies (mean-reversion, momentum/relative-strength). Subordinate to `framework.md`.

**Rules for future instances of yourself:**
- **Always append a new entry. Never overwrite or delete history.** Each cycle gets its own dated, timestamped entry below, oldest first.
- **Pull live data every cycle** — account state, quotes, positions — rather than trusting a prior cycle's narrative. A position can close between cycles (e.g., a manually-enforced stop fills) without this session having initiated it in the current cycle.
- Every entry should contain, in this order: timestamp, stop-check result, account reconciliation, market/sector read, scan results with gate-check verdicts for anything new, entry/exit decisions with trade scores on any close, and a running daily-stats summary.
- `dashboard.html` is rendered from this file (and this file alone, plus its own static shell). Regenerate `dashboard.html` from current state at the end of every cycle, or on request — never hand-edit the dashboard directly.

---

## Account Baseline

| Field | Value |
|---|---|
| Account | Robinhood ••••3051 ("Agentic") |
| Framework initialized | 2026-09-10 |
| Starting equity | $100.00 |
| Peak equity (for circuit breaker) | $100.00 |
| Position size | $5.00 fixed (halves to $2.50 if circuit breaker trips) |
| Circuit breaker status | Not tripped |
| Max concurrent positions | 3 |
| Daily loss limit | $15.00 |

---

## Cycle Log

*(No cycles have run yet. The first cycle entry will be appended below this line when the operating loop starts. Do not remove this placeholder note until the first real entry exists — it documents that the log is intentionally empty at framework-initialization time, not that logging was skipped.)*

---

## Running Daily Stats (resets each session/trading day)

| Date | Trades (organic) | Trades (forced) | Realized P&L | Open Unrealized P&L | Daily loss limit hit? | Circuit breaker status |
|---|---|---|---|---|---|---|
| — | 0 | 0 | $0.00 | $0.00 | No | Not tripped |

---

## Running Lifetime Stats

| Metric | Value |
|---|---|
| Total closed trades | 0 |
| Win rate | — |
| Average win ($ / R) | — |
| Average loss ($ / R) | — |
| Expectancy | — |
| Max drawdown | $0.00 (0.0%) |
| Average trade score | — |
| Forced-trade win rate vs. organic-trade win rate | — (no trades yet) |
