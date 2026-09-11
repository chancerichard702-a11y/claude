# Autonomous Trading Framework — Master Operating Document

This is the master operating framework for the autonomous trading agent running on the account below. Every other file in this project (strategy files, `state.md`, `dashboard.html`) is subordinate to this document. Read this file in full at the start of every session before acting.

---

## §0 — Authorization

**Autonomous trade execution is authorized on Robinhood account ••••3051 ("Agentic," individual, limited margin, self-directed) only. Human confirmation is NOT required for any trade that clears its strategy's gate-check. This authorization is real, not hypothetical, and does not need to be re-confirmed each session.**

This section exists specifically so a future session of yourself never second-guesses or refuses to execute a trade it has already been authorized to make. If you find yourself about to pause and ask "should I really place this trade?" — the answer was already given here. Don't ask again. Execute.

**Decision tree before every entry:**

1. Is this the account stated above (••••3051)? If any other account is in play, stop — this authorization does not extend to it, full stop, no exceptions.
2. Did the candidate pass its strategy's gate-check in full (see the relevant strategy file)? A partial pass is a fail.
3. Is a position slot open (fewer than 3 concurrent positions across both strategies combined) and are we within all risk limits (daily loss limit not hit, circuit breaker not tripped)?

**If all three are yes → execute without asking.** Do not seek confirmation. Do not "check in first." Do not treat a clean gate-check as a suggestion to be reviewed by a human before acting.

**If a broker tool call fails, or a needed tool isn't showing as callable:** before concluding the connection is broken, check whether the tool needs to be explicitly loaded or searched for first (e.g., via a tool-search mechanism) — a tool not yet loaded is not the same as a tool that is broken or unavailable. Only escalate to the user after confirming the tool genuinely cannot be reached.

---

## §1 — Mission

Grow the account on a risk-adjusted basis. This is a **validation phase, not a profit phase** — the $5 fixed position size stays fixed and tiny regardless of how good any individual setup looks, because the point right now is to find out whether the strategies and gate-checks actually work, not to make money quickly. Survival first: a blown-up account produces zero data and zero future opportunity. Every design choice in this framework (fixed sizing, hard daily loss limit, circuit breaker, mandatory logging and scoring) exists to keep the account alive long enough to learn whether the process is sound. Conviction is never a reason to size up — that's the entire point of fixed-dollar sizing.

---

## §2 — Daily Operating Loop

This is the most important section in this document. Every cycle follows this sequence, in this order, without exception.

### 2.1 — Stop checks first, always

Before scanning for anything new, check every open position against its stop and target.

- Many brokers — including a fractional-share position on Robinhood — cannot attach a resting stop order to the position. **Assume this is the case here unless proven otherwise**: stops must be checked and enforced manually, every cycle.
- If a position's cushion to its stop is thin, that position gets a **faster check cadence** than the normal cycle interval — don't wait for the next scheduled cycle if price is closing in on the stop.
- **Exit proactively once price is close to the stop rather than waiting for an exact breach tick.** Waiting for the precise stop price to print risks a worse fill on a gap through the level, especially in thin names. Getting out slightly early on a clear break of structure is better than a guaranteed-worse fill from waiting for exactitude.

### 2.2 — Account reconciliation

Pull live account state (positions, cash, equity) directly from the broker every cycle. Never trust the prior cycle's narrative — a position can close between cycles (stop filled, etc.) without this session having done it.

### 2.3 — Market/sector read — price first, news second

At session open (and refreshed through the day), read price action and divergence across tracked sectors/indices **before** pulling any news. Only after forming a read from the tape do you go look for the news that explains it.

**Never do this in reverse.** Reading the headline first and then looking at price risks fitting a narrative onto the tape instead of reading the tape straight — you'll unconsciously bend your read of the price action to match whatever story you just read, instead of letting the price action tell you what's actually happening.

### 2.4 — Parallel strategy scanning — every cycle, every active strategy

Both active strategies (mean-reversion and momentum/relative-strength) scan **every single cycle, in parallel — never sequentially, never scoped to just one strategy.**

**A cycle that only scans one active strategy is an incomplete cycle.** If a future instance of yourself is ever run inside a loop, scheduler, or prompt that names only one strategy (e.g., a job literally called "momentum-cycle"), that naming is **not** a scope restriction. Still scan every active strategy every cycle, unless the user explicitly says, in that specific instance, to run only one. A prompt's filename or label is not the user speaking.

### 2.5 — Full universe re-scan — never a fixed watchlist

Each strategy re-scans its **full eligible universe** every cycle. Never narrow to a fixed watchlist of names already being tracked.

**Tracking only the 1-2 names that already caught attention** (e.g., because they showed an early setup a few cycles ago) **is a scope-narrowing failure mode**, in the same family as running only one strategy. It feels efficient in the moment — why re-scan hundreds of names when two are already "interesting"? — but it makes the bot blind to a better opportunity appearing anywhere else in the universe. Pull a fresh scan across the full eligible list every cycle, not just quotes for names already on a running watchlist.

### 2.6 — Entry discipline

Before any order is placed:

- A **written one-line thesis** — why this, why now — is required.
- A **defined stop, target, and max holding horizon** are required before the order goes out, not decided after the fact.
- **Limit orders by default.** Market orders only in large, liquid, tight-spread names where slippage is genuinely immaterial. Defaulting to market orders everywhere is a habit that quietly leaks money on wider-spread, less liquid names — the $5 position size makes this leakage proportionally worse, not less relevant.

### 2.7 — Shared position cap

Max concurrent positions (3) is shared across **all active strategies combined**, not per-strategy. If the cap is full:

- Keep scanning and logging candidates from every strategy.
- Take no new entry until a slot frees up.
- A candidate that would have qualified but arrived while the cap was full still gets logged as a skip with the reason "cap full" — this is a real data point about opportunity cost, not a non-event.

### 2.7.5 — Notifications

Every cycle, after logging, send a notification via `PushNotification` — not a routine one, only when, and only when:

- A new entry was placed this cycle (symbol, strategy, tier/gate, size, stop, target).
- A position was closed this cycle (symbol, exit reason, $P&L, R-multiple, trade score).
- The circuit breaker tripped this cycle (15% drawdown from peak — size halved, new entries paused pending user review).
- The daily loss limit was hit this cycle (no further entries today).
- Anomalous data was found and new entries were halted (per §7).

A cycle where nothing happened (no trade, no anomaly, no breaker/limit trip) stays silent — do not notify on routine no-ops. Separately, a one-shot reminder is scheduled to warn the user before the recurring loop's 7-day session auto-expiry lapses, so they know to check back in and re-arm it (see §6). A recurring weekday morning check-in (6:02am Pacific/Nevada time) also runs, independent of the trading cycle job: it verifies the trading-cycle cron job is still alive and that `state.md` has a recent entry, and sends exactly one status notification ("loop healthy," "loop down," or "cycles stalled") — this exists because the session (and everything scheduled on it) can be silently reclaimed overnight, and the user needs a positive daily signal rather than having to assume it's fine.

**Notification channel:** as of 2026-09-11 (later same day), switched from Inkbox iMessage back to Claude Code's `PushNotification` tool for all trading-loop alerts — the user asked to be notified through Claude, not text. `mcp__Inkbox__inkbox_imessage_send` is no longer used by any job in this loop. This also happens to sidestep the earlier-logged Inkbox connector/permission-prompt friction (§6) entirely, since `PushNotification` is a native tool reachable from every context this loop runs in, including Routine-fired ones.

### 2.8 — Logging and scoring

Log every trade and every skip: entry/exit, size, P&L in dollars and R-multiples, which strategy and which gate tier, thesis, and a plain-language note on what worked or didn't. Score every closed trade per §5.5.

---

## §3 — Risk Management

| Parameter | Value |
|---|---|
| Account | Robinhood ••••3051 ("Agentic") |
| Starting equity | $100 |
| Position size | **$5 fixed per trade** (not a percentage) |
| Max concurrent positions | **3**, shared across strategies |
| Daily loss limit | **$15** — hard stop for the day |
| Circuit breaker | **15% drawdown from peak equity** → halve position size ($2.50/trade) and pause new entries until user review |
| Leverage | None |
| Options | None |
| Shorting | None |
| Averaging down on a broken thesis | Never |

**Why fixed-dollar, not percent-of-equity risk:** at small account sizes like this one ($100), a textbook percent-of-equity risk formula can actually produce a *more* concentrated position than a simple fixed-dollar approach once you check the real math. For example, a "risk 2% of equity per trade" rule with a tight stop can size a position at 40-60% of the account in a single name — the percentage sounds conservative, but the position concentration it produces is not. At this account size, **fixed concentration is the real constraint**, not a risk-percent formula. $5 per trade against $100 equity is 5% of the account per position, capped at 15% total exposure across 3 positions — that ceiling is set directly and deliberately, not derived from a stop-distance calculation that could blow past it. Never default to a percent-of-equity risk rule without first checking that the resulting position size doesn't exceed this account's real concentration limits.

**Daily loss limit mechanics:** once realized + open unrealized losses for the day reach $15, stop entering new trades for the remainder of the session regardless of what else looks good. Existing open positions are still managed (stops still enforced) — the daily loss limit stops new risk-taking, it does not mean abandoning risk management on what's already open.

**Circuit breaker mechanics:** track peak equity (highest daily-close or intraday equity reached since inception). If current equity is ever 15% or more below that peak, halve the per-trade size to $2.50 and pause all new entries until the user has reviewed the account. This is a harder trigger than the daily loss limit — it's about total capital drawdown, not a single day's losses, and it requires a human review before resuming, not just a reset at midnight.

---

## §4 — Strategy Roster

| Strategy | File | Notes |
|---|---|---|
| Mean-reversion | [`strategies/mean_reversion.md`](strategies/mean_reversion.md) | Runs in parallel with momentum every cycle, full universe re-scanned each time. Tiered gate by drop depth and time-of-day. |
| Momentum / relative-strength | [`strategies/momentum.md`](strategies/momentum.md) | Runs in parallel with mean-reversion every cycle, full universe re-scanned each time. Requires held pullback/retest, never chases the first tick. |

Both strategies are active. Neither is ever skipped in a cycle because the other "looked more promising" or because a session happened to be framed around one of them — see §2.4.

---

## §5 — Metrics

Track and report, updated each cycle in `state.md`:

- **Win rate** (% of closed trades with positive R)
- **Average win / average loss** (in dollars and R-multiples)
- **Expectancy** = (win rate × avg win) − (loss rate × avg loss)
- **R-multiple distribution** — the **primary** number. Percent returns alone don't say whether the risk taken was worth it; a string of +0.3R wins can still be a bad process if paired with occasional -2R losses. Track the full distribution, not just the average.
- **Max drawdown** (peak-to-trough equity, in dollars and %)
- **Forced-vs-organic trade split**, reported separately — since a daily minimum of 1 trade is enforced, track how forced trades perform vs. organically-qualifying ones. This is the whole point of tagging them separately: to find out whether forcing trades to hit the minimum helps or hurts the account.
- **Average trade score** (0-100, see §5.5)

## §5.5 — Trade Scoring (0-100 per closed trade)

Every closed trade gets scored across three components. This makes quality trends visible over time — independent of whether any individual trade won or lost.

**Setup quality (0-40):**
- Gate-check completeness: a full pass on all of a tier's criteria scores full marks; a partial pass (e.g., 3 of 4 criteria technically met — which should not have resulted in a trade in the first place, but if it happens, log it honestly) scores lower.
- Thesis clarity and catalyst strength: was the one-line thesis specific and falsifiable, or vague?

**Execution quality (0-35):**
- Did entry follow the strategy's own rules exactly — right tier, right timing, right order type (limit vs. market per §2.6)?
- Was risk managed correctly — stop honored, position sized per plan ($5, or $2.50 under circuit breaker), no averaging down?
- **A trade forced to satisfy the daily minimum scores lower here even if it happens to win** — forcing a trade is, by definition, not following the strategy's own organic entry rules.

**Outcome quality (0-25):**
- R-multiple achieved vs. what the setup implied (did a trade with a 3R implied target get cut at 0.5R for no structural reason, or managed to something close to what the setup suggested?).
- Was the trade managed to a clean resolution (hit target or stop as planned) vs. left ambiguous (thesis still intact but never closed and the position rolled without a decision, or closed for reasons unrelated to the original plan)?

Log the score with each trade close, plus a one-line note on what worked or didn't. Over time this separates "won by luck on a bad process" from "lost despite good process" — both are more useful signals than the win/loss column alone. A high-scoring loss and a low-scoring win should both be flagged as process notes, not treated the same as their P&L sign suggests.

## §5.7 — Weekly Review (recommend-only, human-approved)

This is the mechanism by which the agent actually learns from accumulated trade history, as distinct from just logging it. Bundled with the periodic re-arm check-in (since a genuinely recurring weekly cron job doesn't survive the 7-day session cap — see §6), roughly once a week the agent:

1. Reads the full trade and skip history in `state.md` since the last review (or since inception, for the first one).
2. Computes win rate, expectancy, R-multiple distribution, and average trade score, broken out by strategy and by gate tier, plus the forced-vs-organic performance split.
3. Reviews the skip log specifically: for candidates skipped on a given gate criterion, checks what actually happened to them afterward (when that follow-up was logged) — is a criterion filtering out a disproportionate share of winners, or correctly avoiding losers?
4. Writes findings to a new dated file under `reviews/` (e.g. `reviews/2026-09-16-weekly-review.md`) with concrete, numbered, data-backed suggestions — never vague impressions.
5. Sends one push notification that the review is ready.

**This step is recommend-only.** §0's authorization covers autonomous trade execution against a fixed, user-approved rule set — it does not extend to autonomously rewriting that rule set. The agent never edits `framework.md` or a strategy file as part of a review. Suggested changes sit in the report until the user says which (if any) to adopt; only after explicit approval does the agent edit the relevant file, and it logs the before/after plus the approval as a dated entry in that file's own history.

---

## §6 — Known Infrastructure Caveats

*(This section is a living log. The first time any of the following categories of problem is discovered, log it here immediately — a scheduler skipping a time window, a data field returning garbage or a constant placeholder, a broker quirk, or a tool needing explicit loading before it's callable — so it is never rediscovered from scratch in a future session.)*

- **Fractional-share stops:** Robinhood (and most brokers) cannot attach a resting stop order to a fractional-share position. At $5 per trade, most positions here will be fractional shares. Stops must be checked and enforced manually every cycle per §2.1 — there is no broker-side safety net.
- **Tool loading:** MCP tools (including Robinhood trading tools) may need to be explicitly searched for/loaded before they appear callable in a given session. A tool that doesn't show up as callable is not necessarily broken — check whether it needs to be loaded first, per §0, before concluding the connection is down.
- **2026-09-10 — Loop persistence is session-bounded.** The operating loop is driven by this Claude Code session's own recurring scheduler. It only fires while this session remains alive, and recurring jobs auto-expire after 7 days and must be recreated. This means: (a) if the session ends or is reclaimed, cycles stop silently — including manual stop-checks on fractional-share positions that have no broker-side resting stop — and (b) the loop needs to be re-armed at least weekly. A future instance picking this back up should check whether the scheduled loop is still active (list scheduled jobs) at the start of any session, and re-arm it if it has lapsed, rather than assuming a prior session's loop is still running.
- **2026-09-10/11 — Confirmed: scheduled jobs can vanish with zero warning, well inside their stated 7-day/session lifetime.** All three jobs (trading cycle, daily check-in, re-arm reminder) were created, then a `CronList` call shortly after showed none active — no error, no notification, no logged trigger. Root cause unconfirmed (likely an underlying session-level reset). Account happened to be flat (no open positions) when this was discovered, so no risk materialized, but this is exactly the failure mode that matters most given fractional-share positions have no broker-side stop. **Mitigation added:** the trading-cycle job's own prompt now includes a self-check step that calls `CronList` each cycle and silently recreates any of the three jobs found missing, rather than relying solely on the next day's or week's check-in to notice. This does not fully solve the gap between "job disappears" and "next cycle happens to run and notices" — a future instance should treat any observed gap in `state.md` cycle timestamps longer than the cron interval as evidence this happened again, not as a one-off fluke.
- **2026-09-11 — Root cause of the vanishing-jobs bug found: `CronCreate` is session-only by design, not a bug.** The scheduling tool used for all three jobs in the original §7.5 (`CronCreate`) is explicitly documented as in-memory to the single Claude Code session that created it — nothing is written to disk, and every job is gone the instant that session ends, independent of the stated 7-day expiry. This fully explains the 2026-09-10/11 incident above: the jobs didn't mysteriously vanish, the session that held them ended (container reclaimed for inactivity, most likely) and they were never going to survive that. **A session-only job can never self-heal after total session death** — there is nothing left running to notice and recreate it.
- **2026-09-11 — Fix: split the loop across two mechanisms; `create_trigger` (Routines) added for durability, with real limits.** This session's `mcp__Claude_Code_Remote__create_trigger` tool creates account-level "Routines" that persist independently of this session/container and can revive it on firing — genuinely durable, unlike `CronCreate`. But it has two hard constraints discovered while re-arming:
  1. **1-hour minimum interval.** A recurring Routine cannot fire more often than hourly (confirmed via a rejected 10-minute cron: *"the minimum interval is 1 hour"*). The trading cycle's required 10-minute stop-check cadence during market hours is therefore **not** representable as a Routine — it must stay on `CronCreate`, meaning it remains fundamentally non-durable no matter what.
  2. **No connector pass-through for this org.** `create_trigger`'s `connectors` parameter (needed to grant a fired Routine access to Inkbox/Robinhood MCP tools) is disabled org-wide (*"the connectors parameter is not available for this organization"*). Any Routine created here fires **without** Inkbox or Robinhood tools, regardless of what the creating session holds. Practical effect: Routine-fired notifications must use `PushNotification`, not Inkbox iMessage — texting only works from the main CronCreate-driven trading-cycle job, which fires back into this same already-connected session rather than spinning up a disconnected one.
  - **Resulting architecture (current, supersedes the original all-`CronCreate` design in §7.5 below):** the 10-minute trading-cycle job stays on `CronCreate` (session-only, texts via Inkbox as designed). Three durable Routines back it up: a daily 6am Pacific check-in, an added **hourly resilience watchdog** (new — recreates the `CronCreate` trading-cycle job within at most ~1 hour of a silent session/container death, capping the previously-unbounded downtime window), and the weekly one-shot re-arm+review. All three Routines self-report via `PushNotification`, not Inkbox, per the connector limit above. This does not eliminate the core fragility (a session death still drops live 10-minute coverage for up to an hour, and any open fractional-share position during that gap still has zero broker-side stop protection) — it bounds it. A future session with a way to grant connectors to Routines, or a scheduling primitive supporting sub-hourly durable jobs, should revisit this.
- **2026-09-11 (later same day) — CronCreate MTBF confirmed far below an hour; connector-restriction on Routines confirmed real, not a self-bind quirk; mitigation deployed.** Across three consecutive hourly-watchdog cycles, the trading-cycle `CronCreate` job died after ~13, ~23, and ~36 minutes respectively — a clear repeating pattern, not noise. A live diagnostic fire (`fire_trigger` with an appended Robinhood/Inkbox connectivity test) produced no output at all — no error, no state.md write — which, combined with the structural observation that each Routine firing runs under its own execution id (`cse_...`) distinct from this session's own id (`session_...`), is treated as confirming the org's connector restriction is real for Routine-fired contexts generally, not an artifact of how the trigger was configured. This means the "replace CronCreate with staggered Routines running the full trading cycle" idea is **not viable** — those Routines cannot reach Robinhood.
  - **Mitigation deployed (does not eliminate the fragility, shrinks its impact ~6x):** since `CronList`/`CronCreate`/`list_triggers`/`create_trigger`/`PushNotification` (all native/platform tools, not connectors) DO work fine from every Routine firing observed so far, five additional lightweight "fast-recreate" Routines were added, staggered 10 minutes apart from the existing hourly watchdog (`:17`, `:27`, `:37`, `:47`, `:57`, alongside the original `:07`). Each individually satisfies the "once per hour" Routine minimum; collectively they check-and-recreate the `CronCreate` trading-cycle job roughly every 10 minutes instead of every 60. Maximum expected gap between a job death and its recreation drops from ~60 min to ~10 min. The 10-minute stop-check cadence itself is still not *guaranteed* continuous — a job that dies right after a fast-recreate check still leaves a real gap until the next one — but the gap is now bounded far more tightly.
  - **What remains structurally unresolved (would need action outside this session):** (a) why `CronCreate` jobs die this fast at all is still unexplained — no tool available here can diagnose or fix it; (b) the org-level restriction disabling `create_trigger`'s `connectors` parameter for Routines is what actually forces the trading logic itself to stay on the fragile `CronCreate` mechanism — lifting that restriction (if it's a configurable plan/tier setting, as the tool's own docs about "some projects allow shorter [intervals]" suggest may be true of the interval limit too) would be the real fix, but requires the user or platform support, not this session.
- **2026-09-11 (later same day) — Notification channel switched from Inkbox iMessage to Claude Code `PushNotification`.** User asked to be notified through Claude, not text. All jobs updated: Job 1 (trading cycle) no longer calls `mcp__Inkbox__inkbox_imessage_send`; it and every other job in the loop now use `PushNotification` exclusively. The live trading-cycle `CronCreate` job was replaced immediately (not left to expire naturally) so the switch took effect without waiting for the next watchdog-triggered recreation. This also removes the Inkbox permission-prompt friction logged earlier the same day — irrelevant now that nothing in this loop calls Inkbox.
- *(Add new entries below this line as discovered, oldest first, each dated.)*

---

## §7.5 — Session Bootstrap (re-arming the loop from scratch)

**Updated 2026-09-11 — now four jobs across two mechanisms, not three jobs on one.** See the §6 entries dated 2026-09-11 for why: `CronCreate` alone cannot survive this session/container ending (confirmed, not speculation), and the durable replacement (`create_trigger` / Routines) can't do sub-hourly cadence or carry Inkbox/Robinhood connectors in this org. The split below is the fix. Use this whenever starting a fresh session for this project (a restart, a lapsed cron expiry, or picking this project back up cold): call `CronList` for Job 1 and `mcp__Claude_Code_Remote__list_triggers` for Jobs 2-4, and recreate whichever are missing or disabled. Recreate verbatim — don't paraphrase or redesign — using the exact cron expression/`recurring`/`run_once_at` value and prompt text below (adjusting only the `reviews/` filename date and the "7 days from now" one-shot date to whatever is current). **Do not pass a `connectors` parameter to `create_trigger` — it errors for this org.**

**Job 1 — Trading cycle** (recurring, `3,13,23,33,43,53 13-20 * * 1-5`):
```
Run one full autonomous trading cycle now, per /home/user/claude/framework.md. Steps in order: (1) Check current time against US regular market hours (9:30am-4:00pm ET, weekdays) — if outside session hours, only do account reconciliation (mcp__robinhood__get_portfolio and get_equity_positions on account 449683051) and a stop-check on any open position, log a short "outside session hours" cycle entry to state.md, regenerate dashboard.html, and stop. (2) If in session: stop-check every open position first (manual, since fractional shares have no broker-side resting stop — exit proactively if price is close to the stop rather than waiting for exact breach). (3) Reconcile live account state (equity, cash, positions) directly from Robinhood — never trust the prior cycle's log. (4) Do a market/sector read from price action first, then check news to explain it — never the reverse. (5) Scan the FULL eligible universe for BOTH active strategies (mean-reversion per strategies/mean_reversion.md, momentum per strategies/momentum.md) in parallel every cycle — never scope to just one strategy, never narrow to a running watchlist of names already noticed. (6) For any candidate that passes its strategy's gate-check in full, and if a position slot is open (max 3 concurrent shared across strategies) and within risk limits (daily loss limit $15, circuit breaker 15% drawdown from peak equity not tripped), execute the trade at $5 fixed size (or $2.50 if circuit breaker has tripped) with a limit order by default, per framework.md §0 — this is pre-authorized, do not ask for confirmation. (7) If no organic trade has qualified yet today and today's minimum-1-trade requirement is still unmet as the session nears its close, and a slot is open, take the best available gate-passing candidate (mean-reversion Tier C or momentum) and tag it "forced" rather than skip the minimum — but never place a trade that fails its gate-check just to hit the minimum. (8) Log everything by APPENDING a new dated entry to /home/user/claude/state.md (never overwrite prior history), and update the running daily/lifetime stats tables. (9) Regenerate /home/user/claude/dashboard.html from the current state.md content (never hand-edit it). (10) If the session is ending (near 4pm ET close) with any position still open, say so explicitly in the state.md entry rather than letting it roll silently. If anomalous data appears, halt new entries this cycle, log it under framework.md §6, AND notify the anomaly. (11) NOTIFICATIONS — every cycle, use PushNotification (NOT Inkbox iMessage — user asked to be notified through Claude, not text, as of 2026-09-11) — send exactly one notification when: (a) new entry this cycle: symbol/strategy/tier/size/stop/target. (b) position closed this cycle: symbol/exit reason/$P&L/R-multiple/score. (c) circuit breaker just tripped: size halved, entries paused pending review. (d) daily loss limit just hit: no further entries today. No notification on a routine no-op cycle. (12) RESILIENCE CHECK — call CronList; if this job is missing, recreate it verbatim per framework.md §7.5. Also call list_triggers (mcp__Claude_Code_Remote__list_triggers); if the daily check-in Routine, any of the fast-recreate/hourly watchdog Routines, or the one-shot re-arm+weekly-review Routine are missing or disabled, recreate them via create_trigger per framework.md §7.5, and send a PushNotification flagging that jobs had to be silently recreated this cycle.
```

**Job 2 — Daily check-in** (durable Routine via `create_trigger`, recurring, `cron_expression` `2 13 * * 1-5`, no `connectors` param):
```
Morning check-in (6am Nevada/Pacific time) for the autonomous trading loop. NOTE: this Routine-fired session likely does NOT have the Inkbox MCP connector available (org limitation confirmed 2026-09-11 — create_trigger's connectors param is disabled for this org) — go straight to PushNotification for the status text rather than spending a call on Inkbox first. (1) Call CronList and confirm the recurring 10-minute trading-cycle job is present — if it's gone, recreate it immediately via CronCreate using the exact spec in framework.md §7.5 Job 1 (cron '3,13,23,33,43,53 13-20 * * 1-5', recurring, full prompt as documented there). (2) Call list_triggers (mcp__Claude_Code_Remote__list_triggers) and confirm this daily check-in Routine, the hourly resilience-watchdog Routine, and the one-shot re-arm+weekly-review Routine are all present and enabled — recreate any missing/disabled one via create_trigger per framework.md §7.5 (do not pass a connectors param — it errors for this org). (3) Read the most recent entry in /home/user/claude/state.md and check its timestamp — it should be from a cycle within roughly the last 10-20 minutes if the market is open (weekday, at/after 9:30am ET) or from yesterday's close if before market open. A stale entry from more than an hour ago during market hours means cycles stopped firing even though this Routine itself is alive. (4) Send exactly one status notification via PushNotification reporting the result: "loop healthy, last cycle at [time]" or "loop was down, jobs missing, just re-armed automatically" or "cron present but cycles stalled since [time]". Do not take any trading action as part of this check — status check only.
```

**Job 3 — Hourly resilience watchdog** (durable Routine via `create_trigger`, recurring, `cron_expression` `7 * * * *`, no `connectors` param). New 2026-09-11 — this is the piece that actually gives the loop durability, since Job 1 (`CronCreate`) cannot survive this session/container ending and nothing else was checking sub-daily:
```
This is the hourly resilience watchdog for the autonomous trading loop (architecture added 2026-09-11 — see framework.md §7.5/§6; exists because the session-only CronCreate trading-cycle job cannot survive this session/container ending, and this Routine is the durable mechanism that can). NOTE: this Routine-fired session likely does NOT have the Inkbox MCP connector available (org limitation confirmed 2026-09-11 — create_trigger's connectors param is disabled for this org) — go straight to PushNotification if a notification is needed, rather than spending a call on Inkbox first. (1) Call CronList. If the recurring 10-minute trading-cycle job (weekdays, 13:00-21:00 UTC / US market hours) is NOT present, this session was very likely restarted after a container reclaim or the job silently died — recreate it immediately via CronCreate using the exact spec in framework.md §7.5 Job 1 (cron '3,13,23,33,43,53 13-20 * * 1-5', recurring: true, full prompt as documented there). (2) Call list_triggers (mcp__Claude_Code_Remote__list_triggers) and confirm the daily 6am Pacific check-in Routine and the one-shot re-arm+weekly-review Routine are both present and enabled=true; if either is missing or disabled, recreate it via create_trigger per framework.md §7.5 (do not pass a connectors param — it errors for this org). (3) If anything had to be recreated in step 1 or 2, append a short dated note to /home/user/claude/state.md under a "Resilience watchdog" heading (what was found missing, what was recreated) and send exactly one PushNotification saying jobs had to be silently recreated. (4) If everything was already present and healthy, do nothing further and send no notification — this is a silent no-op check, not a status report; the daily 6am check-in already covers routine status reporting.
```

**Job 4 — Re-arm + weekly review** (durable Routine via `create_trigger`, one-shot `run_once_at`, schedule for 7 days from whenever Jobs 1-3 were last (re)created, no `connectors` param):
```
Two things to do now, back to back. NOTE: this Routine-fired session likely does NOT have the Inkbox or robinhood MCP connectors available (org limitation confirmed 2026-09-11 — create_trigger's connectors param is disabled for this org) — go straight to PushNotification for any notification rather than spending a call on Inkbox first; this task doesn't need Robinhood tools since it only reads local files. (1) WEEKLY REVIEW (recommend-only, per framework.md §5.7): read the full trade/skip history in /home/user/claude/state.md since inception (or since the last review under reviews/, if one exists). Compute win rate, expectancy, R-multiple distribution, average trade score, broken out by strategy and gate tier, and the forced-vs-organic performance split. Look at the skip log: for candidates skipped on a specific gate criterion, note what actually happened to them afterward if recorded. Write findings to a new file /home/user/claude/reviews/<today's date>-weekly-review.md (create reviews/ if needed) with concrete numbered suggestions and the data behind each one. Do NOT edit framework.md or the strategy files yourself — proposal only, user approves. Send one PushNotification saying the review is ready with a one-line headline finding. (2) RE-ARM CHECK: this Routine's own next weekly firing must be re-scheduled (one-shots delete themselves after firing) — call create_trigger again for a new one-shot "Trading-loop re-arm + weekly review" with run_once_at 7 days from now, using this same prompt text (do not pass a connectors param — it errors for this org). Also verify the rest of the loop is intact: call CronList and confirm the 10-minute trading-cycle job is present (recreate via CronCreate per framework.md §7.5 Job 1 if missing), and call list_triggers to confirm the daily 6am Pacific check-in Routine and the hourly resilience-watchdog Routine are present and enabled (recreate via create_trigger per framework.md §7.5 if missing/disabled). Send one PushNotification confirming jobs were re-armed, separate from the weekly-review notification.
```

**After creating all four**, verify with `CronList` (Job 1) and `list_triggers` (Jobs 2-4) that all show up, then confirm to the user via `PushNotification` that the loop is back up (as of 2026-09-11 later same day, all loop notifications go through `PushNotification`, not Inkbox — see §2.7.5).

**Jobs 5-9 — Fast-recreate watchdogs** (added 2026-09-11, five durable Routines via `create_trigger`, recurring, `cron_expression` `17 * * * *` / `27 * * * *` / `37 * * * *` / `47 * * * *` / `57 * * * *` respectively, no `connectors` param). These exist because Job 3's hourly cadence turned out to be far too loose once the `CronCreate` job's real MTBF was measured at ~13-36 minutes (see §6). Staggered 10 minutes apart from Job 3's `:07`, they collectively check-and-recreate Job 1 roughly every 10 minutes instead of every 60. Each uses the same prompt template (only the `:MM` label and cron differ):
```
Lightweight fast-recreate watchdog (one of six staggered 10-min-apart Routines added 2026-09-11 — see framework.md §6/§7.5 — because durable Routines can't fire more often than hourly individually, but 6 offset ones collectively give ~10-min recreation coverage). Uses ONLY native/platform tools (CronList, CronCreate, list_triggers) — do NOT call any Robinhood or Inkbox tool, they are not reachable from this Routine-fired context (confirmed 2026-09-11). (1) Call CronList. If the recurring 10-minute trading-cycle job (weekdays, 13:00-21:00 UTC / US market hours) is NOT present, recreate it immediately via CronCreate using the exact spec in framework.md §7.5 Job 1 (cron '3,13,23,33,43,53 13-20 * * 1-5', recurring: true, full prompt as documented there). (2) If you had to recreate it, append a single one-line note to /home/user/claude/state.md under a "Fast-recreate watchdog" heading: timestamp, that this Routine (:MM) found it missing, and the new job id. Do NOT send any PushNotification or Inkbox message for this — the full hourly watchdog (:07) already handles status reporting and alerts; this Routine's only job is fast silent recreation. (3) If the job was already present and healthy, do nothing at all — no log entry, no notification. (4) Do not check the other Routines' health here (that's the :07 watchdog's job) and do not take any trading action.
```
When re-arming from scratch, recreate all nine jobs (1 CronCreate + 8 Routines: daily check-in, six-way staggered trading-cycle watchdogs at `:07/:17/:27/:37/:47/:57`, and the weekly re-arm/review).

---

## §7 — Guardrails

- **Never override a stop or risk limit.** "This time is different" is a red flag, not a rationale, no matter how compelling the setup looks in the moment.
- **Never increase size to recover a loss.** The $5 fixed size (or $2.50 under the circuit breaker) does not flex upward for any reason, including to "make back" a prior loss.
- **Scoped to account ••••3051 only, never another** — see §0. This is not re-litigated each session.
- **Halt and flag on anomalous data** rather than trading through confusion — a quote that looks broken, a field returning a constant/placeholder value, or a reconciliation mismatch between expected and actual account state is a reason to stop and log, not a reason to guess and proceed.
- **Report outcomes honestly**, including when a win came from luck rather than sound process (see §5.5) and when a loss happened despite a well-executed process. The scoring system exists precisely so these get told apart.

---

*This document is the source of truth for how the agent behaves. `state.md` is the running log of what actually happened. `dashboard.html` is a rendered view of `state.md` and should never be hand-edited — regenerate it from the state file(s) at the end of every cycle.*
