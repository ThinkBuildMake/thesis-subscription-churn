# KEY_METRICS.md — Thesis Churn Prevention Agent

## What success means

The user is a Thesis subscriber who is drifting toward cancelling for one of four reasons: price, the blend doesn't seem to work, too much product, or a life change (`USERS.md` §1). Subscribers leave steadily and most of it is their choice (voluntary churn is 2.87% of 4.25% annual in ecommerce, `USERS.md` §2 [R1]), and a wrong response can raise it (10% vs 6% churn, `USERS.md` §2 [R4]).

Success means fewer subscribers cancel each month, because someone noticed why they were drifting and answered in a way that fit.

## Success metric: Monthly churn rate

**What is it?** The percentage of active subscribers who cancelled by their own choice during the month. One subscriber counts if they were active on the first day of the month and cancelled before the last.

**How we measure it.** Subscribers active on day 1 who have a voluntary cancellation date inside the month ÷ subscribers active on day 1. Computed from the subscription view in Snowflake, monthly, and shown in Metabase. Failed-payment cancellations are excluded, because this product doesn't act on them (`USERS.md` §7). The baseline comes from the same query run over the months before go-live.

**Target.** Thesis's own pre-launch monthly rate minus 2 to 3 percentage points within 90 days of go-live [src PRD]. The baseline isn't known yet; it is read from history in the first query.

**Why it shows the need was met.** The need is that subscribers who would have left for a fixable reason stay. A lower monthly cancel rate is that outcome, and it can't go down because we sent more emails.

**What it can't tell you.** It doesn't prove this product caused the drop (seasonality or a price change would move it too), and the build window of 3–5 days is far too short to show it (`REQUIREMENTS.md` EV-5). Only go-live data can.

### Supporting metrics

| Metric | What it is | Recorded from | Why it explains the success metric |
|---|---|---|---|
| **Stay rate after an action, by archetype** | Subscribers who haven't cancelled 30 days after an action ÷ subscribers who got one, for each of price, efficacy, fatigue, life change | Audit record (archetype, action date) joined to the subscription view | If churn doesn't fall, this shows which of the four reasons isn't being solved. |
| **Acceptance rate, by intervention type** | Subscribers who took the offered change (cheaper plan, slower cycle, new blend) ÷ messages sent | Audit record joined to plan and cycle changes in the subscription view | A low rate means the response didn't fit the reason, even if nobody has cancelled yet. |
| **Cancelled within 7 days of a message** | Subscribers who cancelled within 7 days after an action ÷ actions sent | Audit record joined to cancellation dates | If churn rises, this shows whether our own messages are pushing people out (`USERS.md` §2 [R4]). |

## Guarantee metrics

| Metric | Requirement | Threshold | Measured on | Recorded from |
|---|---|---|---|---|
| **Precision in the top risk decile** (higher is better) | EV-1 | > 70% | Subscribers in the top 10% by score ÷ those who cancelled within the churn horizon; scores older than the horizon; rolling 3 months | Score table joined to cancellations in the subscription view |
| **Recall of churners** (higher is better) | EV-2 | > 60% | Subscribers who cancelled and were at-risk before they did ÷ all who cancelled; same window | Score table joined to cancellations |
| **At-risk subscribers acted on** (higher is better) | EV-3 | > 85% | Dispatched or escalated actions ÷ at-risk subscribers, per run; the lowest run in the month | Run summary (OB-3) |
| **Actions sent without approval** (higher is better) | EV-4 | > 70% | Autonomous actions ÷ dispatched actions, per run, escalations counted as autonomous; the lowest run in the month | Run summary (OB-3) |
| **Run time for the whole base** (lower is better) | SC-1 | ≤ 6 hours (interim 50,000 subscribers) | Longest run in the month, at the current active count | Per-stage durations in the run record (OB-4) |
| **Data age** (lower is better) | DE-2 | Newest event in any view ≤ 48 hours old (interim) | Oldest view at each run start | Data-freshness values in the run record (OB-4) |

## Upstream fixes and open questions

- Churn definition and a voluntary/involuntary flag: `REQUIREMENTS.md` open item 2 must say whether the subscription view can separate them. The success metric and the first three guarantees rest on it; the 30-day horizon (and the 7-day window above) is interim.
- Pre-launch baseline monthly churn: read once from Snowflake history, then written into the target.
- Acceptance of an offered change must be visible in the subscription view; add as a requirement if it isn't (FR-1.3 only says the subscriber can accept).
- LLM deferrals (DE-3) and approval lapses (RL-6) have no rate threshold, so they aren't tracked; add one if a rate matters.
- The 70% autonomous target can fail by design if approvals are common (`REQUIREMENTS.md` open item 8); watch that row first.
