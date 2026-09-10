# Strategy — Momentum / Relative Strength

Subordinate to `framework.md`. Runs in parallel with `mean_reversion.md` every cycle, full universe re-scanned each cycle — never a fixed watchlist.

## Concept

Buy strength: a leading name pulling back to a rising short-term average and holding, then continuing. This is not a breakout-chasing strategy and not a "buy because the sector is up" strategy — it requires a specific name separating itself from its peers, then requires that name to prove the pullback is being bought (a held retest) before entry.

## Universe

- US-listed equities only.
- Market cap floor: $2B; liquidity floor: average daily dollar volume ≥ $20M (same rationale as mean-reversion — a $5 position needs to enter/exit cleanly).
- Focus on names showing leadership within an active or emerging theme/sector — but the eligible universe for scanning is the full liquid, large/mid-cap list each cycle, not a pre-selected set of "known leaders." A new leader can emerge in any cycle.

## Gate Checks

All of the following must pass. **One FAIL means no trade, regardless of how good anything else looks.**

1. **Genuine relative-strength divergence versus the stock's own peer complex.** The candidate must be visibly separating from its peer group — outperforming names in the same sector/theme — not just moving with a uniform sector-wide rally. A uniform sector-wide move with no standout name is a sign the whole group may still be moving together (including potentially reversing together) — it is **not** a signal to chase any name in that group. Wait for a specific name that clearly separates from its peers before treating it as a momentum candidate.
2. **A real catalyst, confirmed via more than a same-day price move alone.** The move needs an identifiable reason (news, sector rotation into a specific theme, a data point, an upgrade, sustained volume pattern, etc.) — "it's up today" is not itself a catalyst.
3. **A held pullback/retest** — not the first tick down, and never the first tick of a bounce off a low. Require rising short-term support (e.g., a rising short-term moving average or a rising trendline of recent lows) and a genuine base: a **higher low versus the prior low**, not just a single green candle. A name that has pulled back and immediately printed one green bar has not yet proven the pullback is held — wait for actual structure (a base, a stall-then-turn, a higher low) before entering.
4. **Never enter within the first 30–60 minutes of the session open.** Let the opening range establish first — early-session price action is disproportionately noisy and prone to failed moves that reverse once real participation shows up.

There are no time-of-day tiers for this strategy the way mean-reversion has them — the gate is constant throughout the day, but the "never in the first 30-60 minutes" rule effectively means momentum candidates are only evaluated starting after the opening range is established.

## Position Management

- **Trim into 2R+ extensions** or ahead of a known upcoming catalyst rather than holding for the full move unconditionally.
- **Exit on invalidation, thesis change, or relative-strength loss versus the stock's own sector — even above the stop.** If the name starts lagging its own peer group after having been the leader, that's an exit signal independent of the hard stop — the entire premise of the trade was relative strength, so losing that relative strength is a thesis failure even if the stop hasn't been touched yet.
- **Prefer closing day-trades before the session ends** rather than holding overnight by default. Only hold overnight if the thesis is explicitly still intact (still showing relative strength, base still holding) and no overnight catalyst is expected. If a cycle or session is ending with a momentum position still open, **say so explicitly** in the log rather than letting it roll silently to the next day.

## Logging

For every candidate scanned — whether it results in a trade or a skip — log:

- Ticker, timestamp.
- Answers to all four gate questions, and which one(s) failed if it's a skip.
- If it's an entry: thesis (one line), stop, target, max holding horizon, order type.
- If it's a skip: the specific failing criterion.
- **Follow-up:** what actually happened afterward to skipped candidates (did a name skipped for "no held pullback yet" go on to establish one and run, or did it fail as expected?) — this is what allows the gate-check's own accuracy to be reviewed over time against real outcomes, not just judged by the trades actually taken.
