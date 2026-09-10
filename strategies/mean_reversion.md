# Strategy — Mean Reversion

Subordinate to `framework.md`. Runs in parallel with `momentum.md` every cycle, full universe re-scanned each cycle — never a fixed watchlist.

## Concept

Buy oversold names where the drop is driven by external, sector/macro-level pressure — **not** company-specific bad news — tiered by how deep and fresh the drop is, and by how far into the session it is. The tiering exists because the highest-quality setups are more likely to be found early in the day; as the day goes on without a qualifying setup, criteria loosen in a defined, pre-committed way rather than being loosened in the moment out of impatience.

## Universe

- US-listed equities only.
- Market cap floor: $2B (avoid microcap names where "external cause" is hard to distinguish from idiosyncratic risk and liquidity is unreliable).
- Liquidity floor: average daily dollar volume ≥ $20M, so a $5 position can enter/exit without meaningfully moving the quote.
- No sector restriction — the strategy is explicitly about buying weakness that hits a sector or the broader tape, so the eligible universe is broad by design.

## Gate Tiers

Every tier requires answering all four questions below **in writing**, logged before entry. **One FAIL on any question means no trade, regardless of how good anything else looks.** There is no "close enough" — a partial pass is a fail (see §5.5 scoring in the framework: this is graded, not vibes-based).

1. **What caused the drop?** Must be external — sector rotation, macro data, index-level flow, broad market selloff. A company-specific cause (earnings miss, guidance cut, analyst downgrade tied to fundamentals, legal/regulatory action, management issue) is an **automatic fail**, no matter how attractive the technical setup looks.
2. **What's the most recent fundamental datapoint?** Must be neutral-or-better. A miss or a guidance cut is an **automatic fail** regardless of how oversold the technicals look — the technical setup does not override a real fundamental deterioration.
3. **Is there a genuine upside anchor?** An analyst price target well above current price, an intact structural growth driver, or a level the stock has held repeatedly on prior tests. Absence of any anchor is a fail — "it's down a lot" is not itself an anchor.
4. **Is the trend structure a dislocation, or a real downtrend?** A real downtrend — lower highs and lower lows over multiple weeks — is an **automatic fail** regardless of how far the stock has fallen today. This strategy buys sharp dislocations in names with an otherwise intact or neutral trend, not "buying the dip" in a name that has been grinding lower for weeks.

### Tier A — Early session, strictest (first ~90 minutes)

- All four gate questions pass cleanly, with a clearly external, high-conviction cause (e.g., a broad index/sector selloff on a macro print, not a diffuse "market's just soft today").
- Drop should be a **large, fresh, single/multi-day dislocation** — not a slow bleed.
- This is the highest-quality tier and should be preferred whenever it's available. Do not skip a Tier A setup in favor of waiting for something to "loosen into" later — take the clean setup when it's there.

### Tier B — Mid-session, looser (roughly midday through early afternoon)

- All four gate questions still pass — no exceptions on the gate itself — but the drop can be a bit shallower or the external cause a bit more diffuse (e.g., "risk-off day across growth names" rather than a single sharp catalyst).
- Only enter Tier B if no Tier A candidate qualified and a slot is open.

### Tier C — Late session, "take anything reasonable" (final ~60-90 minutes)

- All four gate questions **still must pass** — this tier does not relax the gate itself, only how deep/fresh the drop needs to be and how strong the anchor needs to be to count as "genuine."
- Only used if nothing qualified earlier in the day and a slot is still open. This tier exists so a quiet day for Tier A/B setups doesn't necessarily mean zero organic trades, but it is never a license to wave through a gate-check fail.
- If nothing clears even Tier C by end of session, log the day as having zero organic mean-reversion trades. Do **not** invent a passing gate-check to force a fill — that's what the separately-tracked daily-minimum/forced-trade mechanism in `framework.md` §2.4/§5 is for, and even a forced trade still must pass the actual gate (see below).

**Note on the daily minimum:** if the day's forced-trade minimum (framework.md) needs to be satisfied and no strategy has organically qualified, a forced trade must still pass its strategy's gate-check in full — the "forced" tag reflects that the trade was taken to satisfy the minimum rather than because it was the best opportunity available, not that the gate was skipped. Never place a trade that fails the gate.

## Position Management

- **Trim into 2R+ extensions** or ahead of a known upcoming catalyst (e.g., earnings) rather than holding for the full anticipated move unconditionally.
- **Exit on invalidation, thesis change, or relative-strength loss versus the stock's own sector — even above the stop.** A name that keeps bleeding while its own sector or peer group holds steady is an exit signal on its own, independent of the hard stop. Don't wait for the stop to be hit if the reversion thesis has clearly stopped working.
- **Prefer closing day-trades before the session ends** rather than holding overnight by default. Only hold overnight if the thesis is explicitly still intact and no overnight catalyst (earnings, macro print, etc.) is expected before the next session. If a cycle or session is ending with a mean-reversion position still open, **say so explicitly** in the log — don't let it roll to the next day silently.

## Logging

For every candidate scanned — whether it results in a trade or a skip — log:

- Ticker, timestamp, tier being evaluated against.
- Answers to all four gate questions, and which one(s) failed if it's a skip.
- If it's an entry: thesis (one line), stop, target, max holding horizon, tier, order type.
- If it's a skip: the specific failing criterion, so gate accuracy can be reviewed later against what the stock actually did.
- **Follow-up:** what actually happened to skipped candidates afterward (did the "failed" ones keep falling, validating the skip, or did they bounce, suggesting the gate is too strict or mis-specified?). This is what lets the gate-check's own accuracy be evaluated over time, not just the trades that were taken.
