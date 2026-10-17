# REQUIREMENTS.md — Thesis Churn Prevention Agent

**Status:** draft (becomes `v1.0 — handed to architecture <date>` at handoff; row changes after that bump the version)

**Purpose.** Turn the problem and the users (`docs/USERS.md`, with `docs/USERS_handoff.md`) and the partner PRD (`docs/Portal PRD.md`) into testable requirements and constraints with numeric thresholds, so that the key-metrics doc can measure against these numbers, the stack doc can choose technology against them, and the architecture doc can design to them, instead of the other way around. Every requirement traces to a use case, a source finding, or a hard gate from the PRD.

**Scope.**

*This document decides:* what the system must do per use case in the minimum product (USERS §6, all 12 features, no cuts), the "must never" rules, numeric thresholds for quality attributes (authorization, correctness, performance, reliability, observability, compliance, cost), how each is verified, and what is out of scope by design. Later iterations: none named, so none get rows.

*This document defers:*
- Technology choices and their tradeoffs → the stack document.
- Components, capacity planning and the techniques used to meet a budget → the high-level design.
- Modules, schemas, contracts and data flow → the architecture document.
- Why the problem exists and where the cut line is → the problem statement (stood in for by USERS §1–3 and §7, see Assumption 1).
- Flows, personas and study figures → the users doc (cited here, not restated).
- Which requirements become tracked metrics and dashboards → the key-metrics doc.

If a row below names a library, a component, or a way of meeting a threshold, it is in the wrong document.

**How to read a requirement.**

| Field | Meaning |
|---|---|
| **Requirement** | *Functional:* from the subscriber's side (or the staff member's, where a row says so) — what they do, what they expect back, what they are allowed or refused, what happens to their data. No number. *Non-functional and given:* from the system's side — what it has to accommodate; the requirement is the number itself, tagged. If the number isn't met, the requirement fails. |
| **Verified by** | Non-functional and given rows only: the eval, test, log query or review that proves it. Functional rows are verified by a person seeing or using the feature at the demo. |
| **Trace** | The use case, source or constraint that justifies it. `U§n` = USERS.md section n; `P` = Portal PRD; `H` = USERS_handoff.md. |
| **Priority** | What the system *needs*, independent of the sprint. **Gate** = must hold before real data. **Must** = required for the product to be done. **Should** = required unless a documented tradeoff says otherwise. This document does not cut scope for the deadline. If a Gate or Must is deferred to fit the timeline, the human decides that when the stories are written, and the story map records when it lands. A Gate is "before real data," not "before the first demo." |

**Where the numbers come from.** **[src]** = from a cited source, or computed from one with the arithmetic shown, or the builder's own stated constraint. **[prop]** = proposed engineering target; defensible, but the builder's to change with a stated reason. **[tbd]** = cannot be set until data or a prototype exists; the row says what will set it and gives an interim [prop]. Section 5 lists every [prop] and [tbd].

**Assumptions.**
1. No separate problem statement exists. USERS §1–3 (target user, the problem in numbers, why the pre-cancel moment) and §7 (out of scope) stand in for it.
2. No stakeholder can be reached (builder, 2026-10-16). Every threshold is derived from the PRD, the users doc or a [prop] with reasoning, never from a Thesis answer.
3. Real Snowflake data is available to the build (USERS §8). Where it is not, evaluation falls back to synthetic data and says so in every report (EV-1).
4. In v1 the system never changes a subscription on its own. Plan, cadence and blend changes happen only when the subscriber accepts. *(Proposed by the author, not stated by the builder; it keeps UC-3's "never ship more than asked" checkable. Reverse it by instruction.)*
5. Subscriber-facing interventions (check-in, formulation suggestion, offer) are delivered as email. Escalation is delivered to a Thesis staff member.

**Definitions.**

| Term | Meaning |
|---|---|
| Active subscriber | A subscriber with an unexpired, uncancelled subscription in the subscription view at run time. The population scored in every run (PF-1, SC-1). |
| Run | One scheduled execution of the whole pipeline: read, score, classify, choose, approve, dispatch, log. Has a run id. |
| At-risk subscriber | An active subscriber whose score is at or above the at-risk cutoff **[tbd: set from the backtest so recall in EV-2 is met at the smallest flagged share; interim top 20% by score [prop]]**. The denominator for coverage (EV-3). |
| Archetype | One of four reasons for drifting: price, efficacy, fatigue, life change (USERS §1). A subscriber the classifier cannot place is **ambiguous**, a fifth state used only for routing. |
| Intervention | One of the four action types: check-in email, formulation adjustment suggestion, retention offer, escalation to a human (USERS §6 features 4–7). |
| Action | One intervention dispatched, queued for approval, or deliberately not taken (a logged no-action decision). The unit of the audit trail. |
| Dispatched | An action has been sent to its channel (email service or staff case queue). In dry-run mode, "dispatched" means rendered and recorded but not sent (DE-6). |
| Autonomous action | A dispatched action that needed no human approval. Escalation counts as autonomous: creating the case needs no approval (Key decision 2). |
| Thin-data subscriber | One with no quiz responses, or fewer than 3 orders (USERS §1: cancelling is possible after the second order, so fewer than 3 orders means almost no history) **[prop]**. |
| Pre-cancellation churn | A subscriber who cancels within the label horizon **[tbd: the churn definition is not in the brief; interim: cancels within 30 days of the score date [prop]]**. |
| Audit record | The Snowflake row written for each action (OB-2). |

---

## 0. Requirements summary (one screen)

| Area | Headline threshold |
|---|---|
| Subscriber safety | 0 offers outside the price archetype; 0 automated messages to life-change subscribers; 0 messages with wrong price, medical claims or no exit (VR-1 to VR-7) |
| Prediction quality | Precision > 70% in the top decile; recall > 60% [src P]; reported per archetype for all four (EV-1, EV-2, EV-6) |
| Coverage and autonomy | Coverage > 85% of at-risk; autonomous rate > 70% [src P] (EV-3, EV-4) |
| Audit | 100% of actions logged before dispatch, with context, in Snowflake and Metabase (RL-2, OB-2, GC-8) |
| Reliability | Killed at any stage, a rerun produces 0 duplicate sends and 0 skipped subscribers (RL-1) |
| Scale | Whole active base per run, every 7 days or sooner (PF-1, PF-2) |
| Privacy | 0 raw PII in logs, training data or prompts (CP-1, CP-4) |
| Measurement | Every action and its outcome is logged per archetype so churn reduction can be measured after go-live (EV-5) |
| Given | Due Tue 2026-10-20 23:59 Central; one person; $0 outside the approved providers (GC-1, GC-3, GC-4) |

---

## 0a. Key decisions

| # | Decision | Alternative rejected | Where |
|---|---|---|---|
| 1 | The subscriber is the user; staff rows (approvers, escalation recipients) exist only to serve the subscriber | Treat staff as the user and specify dashboards | FR-6, AZ-1, AZ-2 |
| 2 | Escalation counts as an autonomous action | Count it as approval-gated, which would make the 70% target unreachable if life-change cases are common | EV-4, FR-4 |
| 3 | The must-nevers outrank the PRD's 70% autonomy target. If approval-gated actions leave less than 70% autonomous, the approval default is relaxed only where a must-never allows | Weaken a must-never to hit 70% | EV-4, VR-1 |
| 4 | When the reason is ambiguous, no offer is sent; the subscriber gets a neutral check-in or a human | Default high-risk subscribers to a discount (the rule the brief forbids; the carrier test shows it can raise churn, U§2) | FR-5.2, VR-1 |
| 5 | No control group is built in v1; churn reduction is measured from logged actions and outcomes after go-live | Withhold interventions from a random share of at-risk subscribers (rejected by the builder: it only serves attribution, which the build window cannot show anyway) | EV-5 |
| 6 | An action is logged durably before it is dispatched | Log after the send, which can silently lose records on a crash | RL-2 |
| 7 | Default mode is dry-run; live sending needs an explicit config change | Live sending by default | DE-6 |
| 8 | All four archetypes are measured and reported separately, with n shown, even when a group is small | Report one blended figure | EV-6, OB-5 |
| 9 | No archetype ground-truth labels are known to exist, so classification quality gets a [tbd] and a hand-labelled interim set | Invent a classification accuracy threshold | VR-8 |
| 10 | Data is read only from Snowflake; outbound actions are execution, not reads, and are governed separately | Read directly from Klaviyo/Zendesk/Shopify | GC-7, DE-5 |

---

## 1. Givens

*Inputs the project arrived with, recorded so nothing downstream can quietly violate them.*

### 1.1 Given constraints (GC-)

| ID | Requirement | Verified by | Trace | Priority |
|---|---|---|---|---|
| GC-1 | All deliverables are submitted by Tue 2026-10-20 23:59 Central [src U§8]. No row is cut for this; if one is deferred, the stories step records it. | Timestamp of the final commit or submission against the deadline | U§8 | Must |
| GC-2 | Six deliverables exist at submission: source code, technical documentation, demo video, deployment guide, test results, AI usage log [src P]. | Checklist review: 6 of 6 present | P | Must |
| GC-3 | One person builds and maintains the system [src U§8]. Every Gate and Must row can be checked by that person with scripts or tests, without a second party. Human approval flows (AZ-1) are tested with a stand-in approver. | Review of Verified-by columns: 0 rows need a second person | U§8 | Must |
| GC-4 | Spend on services outside the approved list is $0 [src U§8]; anything outside the list uses a free tier only. | Billing and usage review of every service in the dependency inventory | U§8 | Must |
| GC-5 | Every external service is on the approved list (Anthropic, Aqua Voice, Cohere, Cursor, DigitalOcean, ElevenLabs, Google One, Midjourney, OpenAI, OpenRouter, Railway, Vultr, Whisper) or is a free tier of another service, plus Snowflake [src U§8]. | Service inventory: 100% of services classified | U§8 | Must |
| GC-6 | The code is Python and SQL, uses an LLM API from Anthropic or OpenAI, runs agent orchestration, and uses Snowflake as the data platform [src P]. | Repository and dependency inspection | P | Must |
| GC-7 | All data is read from the existing Snowflake views (subscriptions, orders, email engagement, support, quiz) [src P]. 0 reads from the Shopify, Klaviyo or Zendesk APIs. | Search of the code for those hosts in read paths, plus a run with those hosts blocked at the network | P, U§7 | Must |
| GC-8 | Risk scores, the action audit trail and outcomes are written to Snowflake: 3 sets of results, each queryable from a saved Metabase question [src P]. | Run the pipeline, then execute each Metabase question: 3 of 3 return rows | P, U§8 | Must |
| GC-9 | Moving from demo data to real data needs 0 code changes, only configuration [src U§8]. | Switch the data source in a staging environment and run; `git diff` is empty | U§8 | Must |

### 1.2 Dependencies and environment (DE-)

| ID | Requirement | Verified by | Trace | Priority |
|---|---|---|---|---|
| DE-1 | Before any scoring, each of the 5 required views passes a check that it exists, has the expected columns and is non-empty. On failure the run stops with 0 actions dispatched and the failure is reported within 5 minutes [prop]. | Rename a column in a test copy and drop a view; run; confirm 0 actions and an alert in 5 minutes | P, H | Must |
| DE-2 | The run records the age of the newest event in each view. A view older than 48 hours **[tbd: Klaviyo and Zendesk sync lag is unknown; interim 48 h [prop]]** is flagged in the run record and in the report. | Test with a stale copy: flag present on 100% of stale views | H | Should |
| DE-3 | When the LLM API fails after 3 retries [prop], the affected subscribers are deferred to the next run and logged. 0 generic fallback messages are sent. | Fail the API in a test run; count outbound messages with no generated content: 0 | U§3, U§2 (R4) | Gate |
| DE-4 | A failure of one outbound dispatch is logged with its reason, retried at most 3 times [prop], and marked failed after that. 0 duplicates are sent as a result of the retries. | Inject a timeout after the channel accepts the message; count deliveries per action: ≤ 1 | P | Must |
| DE-5 | Outbound dispatch targets are configured, not hard-coded: the email channel and the staff case queue each accept a stub in tests. 0 live sends in any automated test. | Run the full test suite with a send counter on the live channel: 0 | P, H (actions available is open) | Must |
| DE-6 | The default execution mode is dry-run: actions are rendered and logged, and 0 messages reach real subscribers. Live mode needs an explicit config change, and every audit record states its mode. | Fresh install, default config, full run: 0 sends; flip the flag in staging and confirm sends occur | U§8 (demo product) | Gate |
| DE-7 | Every run records the code version, model version, LLM model id, config snapshot and data-freshness values. 100% of runs have all 5 fields. | Query the run table over 5 test runs | P | Must |

---

## 2. Functional requirements

*Every functional requirement is verified the same way: a person can see or use the feature at the demo. Functional rows carry no number; what the system has to accommodate for them is in §3.*

### FR-1 Price sensitivity (UC-1)

| ID | Requirement | Trace | Priority |
|---|---|---|---|
| FR-1.1 | A subscriber classified as price-driven receives a plain note that states a concrete way to pay less or pay less often (a lower-cost plan, a longer cycle, or a smaller offer), with no deadline pressure. | UC-1 | Must |
| FR-1.2 | A retention offer reaches only subscribers classified as price-driven. Subscribers who are fatigue-, efficacy-, life-change- or ambiguous-classified never receive one. | UC-1, UC-3 | Gate |
| FR-1.3 | The note shows the real price and the next charge exactly as they are in the subscription record, and nothing is hidden or rounded in the subscriber's favour or against it. | UC-1 | Gate |
| FR-1.4 | The subscriber can accept the offered change or ignore it; ignoring changes nothing in their subscription. | UC-1; Assumption 4 | Must |

### FR-2 Efficacy dissatisfaction (UC-2)

| ID | Requirement | Trace | Priority |
|---|---|---|---|
| FR-2.1 | A subscriber classified as efficacy-dissatisfied receives an email that acknowledges the goal they gave and suggests a specific alternative blend or adjustment based on what they told Thesis. | UC-2 | Must |
| FR-2.2 | A subscriber with no quiz data still gets a suggestion, built from purchase and engagement history, and the email says nothing about quiz answers they never gave. | UC-2, feature 9 | Must |
| FR-2.3 | A suggestion uses only data the subscriber gave Thesis, and the email says which data it used. | UC-2 | Gate |
| FR-2.4 | No email claims a medical or cognitive result. | UC-2 | Gate |
| FR-2.5 | A suggested blend is consistent with the recommendation the subscriber was given earlier, or the email states why it differs. | UC-2 | Must |
| FR-2.6 | Not in v1: automatically changing the blend in the subscriber's order. The subscriber accepts the change themselves (Assumption 4). | UC-2 | Must |

### FR-3 Subscription fatigue (UC-3)

| ID | Requirement | Trace | Priority |
|---|---|---|---|
| FR-3.1 | A subscriber classified as fatigued receives a message with an easy way to slow down: a longer cycle, a reschedule, or a skip. It offers no discount. | UC-3 | Must |
| FR-3.2 | Every option the message offers is one the subscription can actually do today. Thesis does not support an indefinite pause (USERS §5 UC-3 note), so none is promised. | UC-3 | Must |
| FR-3.3 | No action ever increases how much product a subscriber receives. | UC-3 | Gate |

### FR-4 Life change (UC-4)

| ID | Requirement | Trace | Priority |
|---|---|---|---|
| FR-4.1 | A subscriber whose signals show a change in circumstances receives no automated offer or message. A staff member receives the case instead. | UC-4 | Gate |
| FR-4.2 | The case a staff member receives holds the score, the archetype and why it was chosen, the top factors, the relevant support text and a history summary, so the subscriber is never asked to repeat their story. | UC-4 | Must |
| FR-4.3 | The case lists the pause and graceful-exit options the staff member can offer. | UC-4 | Must |
| FR-4.4 | A subscriber whose support text the system cannot place in any archetype is escalated to a person, not offered something. | UC-4, FR-5.2 | Must |

### FR-5 Every at-risk subscriber (UC-1 to UC-4)

| ID | Requirement | Trace | Priority |
|---|---|---|---|
| FR-5.1 | Every at-risk subscriber ends each run with exactly one of: an intervention, a pending approval, or a logged decision to take no action with its reason. | Features 1, 3, 10 | Must |
| FR-5.2 | When the system cannot tell the reason, the subscriber gets a neutral check-in with no offer and no product change, or a human, as configured. | Key decision 4 | Must |
| FR-5.3 | A subscriber with no quiz or a short history is still scored and routed, and receives a sensible message, never an error, a blank, or an unfilled template field. | Feature 9; U§5 | Must |
| FR-5.4 | The message is specific to the subscriber's situation and reads as a person wrote it. | Feature 4 | Must |
| FR-5.5 | Every message contains a working way to manage or cancel the subscription. The product never makes cancelling harder. | U§3 reason 3 | Gate |
| FR-5.6 | The system acts on active subscribers only. A subscriber who has already cancelled, or whose payment has failed, gets nothing from this product. | U§3, U§7 | Must |

### FR-6 Staff who approve and receive cases (UC-1, UC-4)

| ID | Requirement | Trace | Priority |
|---|---|---|---|
| FR-6.1 | An approver sees each pending action with the score, archetype, top factors and the proposed message, and approves or rejects it. | Feature 8; H | Must |
| FR-6.2 | A rejected action is never sent, and the rejection is recorded with who decided. | Feature 8 | Must |
| FR-6.3 | An approver or administrator changes which actions need approval (by intervention type, offer value and score) without changing code, and the change takes effect on the next run. | Feature 8; P | Must |
| FR-6.4 | Unapproved actions lapse; they are never sent late (RL-6). | UC-1 (time available) | Must |

### FR-7 Reasons, record and learning (UC-1 to UC-4)

| ID | Requirement | Trace | Priority |
|---|---|---|---|
| FR-7.1 | Every active subscriber gets a risk score each scheduled run. | Feature 1 | Must |
| FR-7.2 | Each score shows its top contributing factors in terms a staff member can read. | Feature 2 | Must |
| FR-7.3 | Each action can be traced to the score, archetype and factors that chose it. | H | Must |
| FR-7.4 | Each action's record shows the subscriber's response and the outcome once known. | Feature 10 | Must |
| FR-7.5 | Logged outcomes change later scoring and selection; the system is not static. | Feature 12; P | Must |
| FR-7.6 | An operator restarts a failed run and it continues from where it stopped. | Feature 11 | Must |
| FR-7.7 | Results for all four archetypes can be seen separately, each with its own count of subscribers. | Builder request, 2026-10-16 | Must |

### FR-8 Scope boundaries (v1)

| ID | Requirement | Trace | Priority |
|---|---|---|---|
| FR-8.1 | The deliverable is the action. No dashboard or report is built as the output; results are queried in Metabase. | U§7 | Must |
| FR-8.2 | Not in v1: cancel-button save flows and failed-payment recovery. | U§7 | Must |
| FR-8.3 | Not in v1: fine-tuned models where simpler approaches work; connectors to source systems; a fixed "score above cutoff → one message" rule standing in for selection by reason. | U§7, P | Must |

---

## 3. Non-functional requirements

*What the system has to accommodate. Each row is stated as its number. Absent groups: none of Authorization's broader forms (single role, one deployment), and no Contracts group beyond CT-1, because schemas are the architecture doc's.*

### 3.1 Authorization (AZ-)

| ID | Requirement | Verified by | Trace | Priority |
|---|---|---|---|---|
| AZ-1 | Only identities on the configured approver list can approve or reject an action: 0 decisions recorded from any other identity. | Negative test with an off-list identity (stand-in approver, GC-3): 0 accepted | FR-6.1 | Gate |
| AZ-2 | The system cannot approve its own actions: 0 dispatched actions that required approval lack a recorded human decision. | Log query over all test runs: count of (approval required and dispatched and no approver) = 0 | FR-6, P | Gate |
| AZ-3 | Every change to approval thresholds is logged with who and when: 100% of changes. | Change thresholds 5 times; 5 log rows with identity | FR-6.3 | Should |

### 3.2 Verification and correctness (VR-)

| ID | Requirement | Verified by | Trace | Priority |
|---|---|---|---|---|
| VR-1 | Intervention selection conforms 100% to the routing table (price → check-in/offer; efficacy → formulation suggestion; fatigue → cadence check-in; life change → escalation; ambiguous → neutral check-in or escalation). 0 offers to non-price or ambiguous subscribers; 0 automated messages to life-change subscribers; 0 actions that add product. Applied to every run, not just the test set. | Routing test set of ≥ 8 cases per archetype including thin-data cases [prop] (EV-9), plus a log query after every run | FR-1.2, FR-3.3, FR-4.1 | Gate |
| VR-2 | The price, next-charge amount and cycle length in 100% of messages equal the subscription record at send time. | Compare each rendered message with the source row over a full test run | FR-1.3 | Gate |
| VR-3 | 100% of messages contain a working manage-or-cancel link. | Rendered-message scan plus a link check over a full test run | FR-5.5 | Gate |
| VR-4 | 0 messages contain an unfilled template field, an error string or an empty section. | Scan all rendered messages in a full test run | FR-5.3 | Must |
| VR-5 | Message safety eval: 0 violations across ≥ 100 generated messages [prop] covering all four archetypes and thin-data cases. Violations are medical or cognitive claims, pressure or countdown language, quiz answers the subscriber never gave, and contradiction of earlier advice. | A scripted checker plus a human read of 20 of the 100 [prop]. This is weak: it samples. | FR-2.3, FR-2.4, FR-2.5 | Gate |
| VR-6 | No subscriber receives more than 1 outbound intervention in any 14 days [prop]. Reasoning: the carrier test shows an extra message can raise churn (U§2, R4), and UC-1 asks for "no pressure." | Query the audit table across 3 consecutive runs for per-subscriber counts | UC-1, U§2 | Should |
| VR-7 | 0 features are computed from data timestamped after the score date. | Leakage test: shift the score date back in a fixture and confirm identical features | EV-1 | Must |
| VR-8 | Archetype classification quality **[tbd: no ground-truth labels are known to exist (H); the first check is whether cancellation reasons or survey tags exist in Snowflake; interim: macro-F1 ≥ 0.6 on ≥ 40 builder-labelled cases per archetype [prop]]**. If labels are hand-made, every report says so. | Run the classifier on the labelled set; report precision and recall for each of the 4 archetypes | UC-1 to UC-4; H | Must |
| VR-9 | 100% of thin-data subscribers are scored and routed with 0 errors. Accuracy on the thin-data group is reported separately **[tbd: target set after the first backtest shows the baseline]**. | Fixture with thin-data subscribers; separate row in the evaluation report | FR-5.3 | Must |
| VR-10 | 100% of scores record ≥ 3 top factors [prop] with direction and size. | Query the score table: min factor count ≥ 3 | FR-7.2 | Must |
| VR-11 | 100% of offers whose value exceeds the configured approval threshold are held for approval, and 0 reach the subscriber first. | Run with a low threshold; compare held count with offers above it | FR-6 | Must |

### 3.3 Performance (PF-)

*Not requirements:* how the budgets are met. Batching the LLM calls, scoring in the warehouse, and caching are the architect's choices.

| ID | Requirement | Verified by | Trace | Priority |
|---|---|---|---|---|
| PF-1 | 100% of active subscribers are scored in each scheduled run [src P]; any subscriber not scored is listed with a reason. | Compare the score table's row count with the active count in the view at run start | FR-7.1; P | Must |
| PF-2 | The pipeline runs at least once every 7 days [prop]. Arithmetic: the shortest cycle is 25 days (U§1), so a 7-day interval gives ≥ 3 runs per cycle, enough for a drifting subscriber to be caught and still leave time before the next charge. | Schedule configuration review; 3 scheduled runs observed | UC-1, UC-3 | Must |
| PF-3 | A full run (read, score, classify, choose, dispatch, log) completes in ≤ 6 hours **[tbd: active base size is unknown; interim 50,000 subscribers [prop]]**. | Timed run at the interim size (SC-1) | P (full base) | Must |
| PF-4 | A life-change case is visible to staff ≤ 1 hour after the run decides it [prop] (the time available is "the next support reply", USERS §5, whose length is unknown). | Time from the decision record to case-queue visibility in a test run | UC-4 | Must |

### 3.4 Reliability and failure modes (RL-)

| ID | Requirement | Verified by | Trace | Priority |
|---|---|---|---|---|
| RL-1 | After the process is killed at each of ≥ 6 injected points [prop] (read, scoring, generation, approval, dispatch, log write), a rerun finishes with 0 duplicate dispatches and 0 skipped subscribers. | Fault-injection test; compare the final audit table with an uninterrupted run | Feature 11; FR-7.6 | Gate |
| RL-2 | 0 actions are dispatched without a durable audit record already written [src P: 100% logged]. | Reconcile dispatched count against audit rows after every test run; difference = 0 | Feature 10; P | Gate |
| RL-3 | A failure in one subscriber's processing leaves 100% of the other subscribers processed. | Insert a poison record into a fixture; the remaining subscribers still complete | Feature 11 | Must |
| RL-4 | Any run failure is reported within 5 minutes [prop] to a named place (a log entry with severity and an alert surface chosen downstream). | Induce a failure and time the report | Feature 11 | Must |
| RL-5 | Retries of any external call are bounded at 3 with backoff [prop]; after that the item is marked failed and surfaced. 0 unbounded loops. | Force permanent failure of each dependency: run ends | DE-3, DE-4 | Must |
| RL-6 | An action pending approval lapses at the earlier of 48 hours [prop] and the subscriber's next charge or shipment date. 0 lapsed actions are sent. | Age a pending action in a test and run; confirm it is not sent and the lapse is logged | FR-6.4 | Gate |

### 3.5 Observability (OB-)

*The key-metrics doc turns these into dashboards. Here: what must be recorded.*

| ID | Requirement | Verified by | Trace | Priority |
|---|---|---|---|---|
| OB-1 | 100% of log lines are structured and carry the run id, stage, a surrogate subscriber id and a level [src P]. | Parse the logs of a full run: 0 lines fail the schema | P | Must |
| OB-2 | 100% of actions have an audit record [src P] with: run id, timestamp, surrogate subscriber id, score, top factors, archetype and confidence, intervention type and the reason it was chosen, approval required/status/approver/time, execution mode, channel result, scrubbed message text, outcome. | Query for nulls in every field across a full run | Feature 10 | Must |
| OB-3 | An audit record is visible in Snowflake within 15 minutes of dispatch [prop]. | Time the interval in a test run | GC-8 | Should |
| OB-4 | Each run records counts for scored, at-risk, dispatched, autonomous, pending, rejected, lapsed, failed and no-action. The counts reconcile with the audit table with a difference of 0. | Compare the run summary with `COUNT` queries | EV-3, EV-4 | Must |
| OB-5 | Every metric the PRD names (precision, recall, coverage, autonomous rate, churn reduction) can be computed for each of the 4 archetypes from logged data alone, with 0 extra instrumentation. | A SQL query returns all four archetypes for each metric | EV-6; builder request | Must |

### 3.6 Compliance and data handling (CP-)

| ID | Requirement | Verified by | Trace | Priority |
|---|---|---|---|---|
| CP-1 | 0 raw PII (email, phone, name, address, payment details) in logs, audit tables or training data [src P]. Free text is scrubbed before it is stored. | Pattern scan of all logs, audit tables and training sets from a full test run: 0 matches. Patterns cannot catch every name, so a human reads 20 stored support snippets [prop]. | P, U§8 | Gate |
| CP-2 | 0 secrets (keys, tokens, passwords) in the repository or logs. | Secret scan over the repo history and logs | P | Gate |
| CP-3 | LLM prompts and stored responses contain 0 direct identifiers (email, phone, address, full name) [prop]. Reasoning: prompts are logged and sent to a third party. | Pattern scan of the stored prompts from a full run | P | Gate |
| CP-4 | 0 messages go to a subscriber marked unsubscribed or suppressed in the email data **[tbd: whether such a field exists in the email view is unknown; the PRD sets no consent rule (H)]**. | Test with a suppressed fixture, once the field is known | H | Should |

### 3.7 Contracts and interfaces (CT-)

| ID | Requirement | Verified by | Trace | Priority |
|---|---|---|---|---|
| CT-1 | An invalid or missing configuration value stops the run at start with a message naming the value. 0 silent defaults for approval thresholds or the execution mode. | Start with 5 deliberately bad configs [prop]: 5 refusals | FR-6.3, DE-6 | Must |

### 3.8 Evaluation suite (EV-)

*Every report below states whether its data is real or synthetic. 100% of reports carry the label; results on synthetic data make no claim against the PRD targets.*

| ID | Requirement | Verified by | Trace | Priority |
|---|---|---|---|---|
| EV-1 | Precision > 70% in the top risk decile [src P], on a time-split backtest (trained on data before a cut date, scored on churn after it), reported with n and a 95% confidence interval. | Backtest run; metric and CI in the report | P | Must |
| EV-2 | Recall > 60% [src P] of pre-cancellation churners at the at-risk cutoff, on the same backtest and label definition as EV-1. | Same backtest | P | Must |
| EV-3 | Coverage > 85% [src P]: dispatched or escalated actions ÷ at-risk subscribers in a run. No-action decisions count in the denominator. | Run summary (OB-4) | P | Must |
| EV-4 | Autonomous action rate > 70% [src P]: autonomous actions ÷ dispatched actions, with escalation counted as autonomous (Key decision 2). Reported separately by intervention type. | Run summary (OB-4) | P | Must |
| EV-5 | Churn reduction of 2–3 percentage points per month within 90 days [src P] cannot be shown inside the build window **[tbd: needs 90 days of live data]**. What the build must provide: every action and its outcome logged per archetype (OB-2, OB-5), so the reduction can be computed later. How to attribute it to the system (a comparison group or before/after) is decided at go-live. | Confirm OB-5 returns outcomes per archetype from logged data | P, U§2 | Should |
| EV-6 | Every evaluation report shows the precision, recall, coverage, autonomous rate and outcome for each of the 4 archetypes, with n. None is dropped for small n. | Report inspection: 4 of 4 archetypes × 5 metrics present | Builder request; OB-5 | Must |
| EV-7 | Re-running the evaluation on the same data snapshot, code version and seed gives identical metrics (difference 0), so improvements between versions are real. Each report stores the version it came from. | Run twice and diff | EV-6 | Must |
| EV-8 | At least 1 full feedback cycle is shown: outcomes logged in run N change the inputs of run N+1, and the run record names the outcome snapshot it used [src P: a static model is incomplete]. | Demo plus run record query | FR-7.5 | Must |
| EV-9 | Automated tests exist for the three critical paths — scoring accuracy, intervention routing and failure recovery [src P] — and 100% pass at submission, with the output kept as the Test Results deliverable. | Test-run output | P, GC-2 | Must |

### 3.9 Cost (CO-)

| ID | Requirement | Verified by | Trace | Priority |
|---|---|---|---|---|
| CO-1 | LLM spend is uncapped in the build [src builder, 2026-10-17]: correctness first, cost optimization afterwards. 100% of runs record LLM calls, tokens and cost, so optimization can start from real figures. | Query the run table over 5 test runs: all 3 fields present | GC-4 | Should |

### 3.10 Scalability (SC-)

| ID | Requirement | Verified by | Trace | Priority |
|---|---|---|---|---|
| SC-1 | A run handles the whole active base **[tbd: size unknown; interim 50,000 subscribers [prop]]**; the load test uses 2× that size, 100,000 [prop]. | Synthetic load test at 100,000 subscribers | P ("full subscriber base") | Must |
| SC-2 | Doubling the base needs 0 code changes (configuration only), and run time grows by ≤ 2.2× [prop]. | Run at the interim size and at 2×; compare times | P | Should |

### 3.11 Deployment (DP-)

| ID | Requirement | Verified by | Trace | Priority |
|---|---|---|---|---|
| DP-1 | A clean deploy to an approved host (Railway, DigitalOcean or Vultr, GC-5) following only the Deployment Guide succeeds on the first try, in ≤ 1 hour [prop], and the test suite passes on a clean install. | Timed walk-through on a fresh environment | GC-2; P | Must |
| DP-2 | The scheduled pipeline runs 3 consecutive times [prop] with 0 manual interventions. | Observe 3 runs from the scheduler | PF-2 | Should |

### 3.12 Extensibility (CH-)

*No later iteration is named, so these come from the PRD's "extensible to new data sources and intervention types."*

| ID | Requirement | Verified by | Trace | Priority |
|---|---|---|---|---|
| CH-1 | Adding a fifth intervention type in a test change touches 0 files in the scoring code. | `git diff --stat` of a stub intervention | P | Should |
| CH-2 | Scoring tests run with no LLM and no network; routing tests run with no Snowflake; dispatch tests run against a stub channel. 3 of 3 suites. | Run each suite with the dependency disabled | P | Should |

---

## 4. Traceability matrix

| Requirement group | UC-1 | UC-2 | UC-3 | UC-4 | Source |
|---|---|---|---|---|---|
| FR-1 Price | ● | | | | U§5 |
| FR-2 Efficacy | | ● | | | U§5 |
| FR-3 Fatigue | | | ● | | U§5 |
| FR-4 Life change | | | | ● | U§5 |
| FR-5 Every at-risk | ● | ● | ● | ● | U§5, features 1, 3, 9 |
| FR-6 Staff | ● | | | ● | U§6 feature 8 |
| FR-7 Record and learning | ● | ● | ● | ● | U§6 features 10–12 |
| GC / DE | ● | ● | ● | ● | U§8, P |
| AZ | ● | | | ● | U§6 feature 8 |
| VR | ● | ● | ● | ● | U§5 must-nevers |
| PF | ● | | ● | ● | U§1 cycles, U§5 time available |
| RL | ● | ● | ● | ● | U§6 feature 11 |
| OB | ● | ● | ● | ● | U§6 feature 10, P |
| CP | ● | ● | ● | ● | P |
| EV | ● | ● | ● | ● | P, builder request |
| CO / SC / DP / CH | ● | ● | ● | ● | U§8, P |

---

## 5. Proposed thresholds

Every **[prop]** and **[tbd]** number. Status is *proposed* until reviewed; after review it becomes *accepted* or *revised*. [prop] rows not commented on at review are accepted; [tbd] rows are never accepted by silence.

| Row | Value | Reasoning | Status |
|---|---|---|---|
| Definitions, at-risk cutoff | [tbd] interim top 20% by score | Set from the backtest so recall in EV-2 is met | pending backtest |
| Definitions, thin-data | < 3 orders or no quiz [prop] | Cancel is possible after the 2nd order (U§1) | proposed |
| Definitions, churn horizon | [tbd] interim 30 days | The brief gives no churn definition | pending data |
| DE-1 | Failure reported ≤ 5 min | A run is hours long; 5 minutes is early enough to rerun | proposed |
| DE-2 | Stale after 48 h [tbd] | Sync lag unknown | pending data |
| DE-3, DE-4, RL-5 | 3 retries | Common practice; bounds time and cost | proposed |
| VR-1 | ≥ 8 routing cases per archetype | Enough to cover thin-data and ambiguous variants | proposed |
| VR-5 | ≥ 100 messages; human reads 20 | A sample a single builder can read | proposed |
| VR-6 | ≤ 1 message per 14 days | Backfire evidence (U§2); the shortest cycle is 25 days | proposed |
| VR-8 | [tbd] interim macro-F1 ≥ 0.6, ≥ 40 labelled/archetype | No labels exist | pending data |
| VR-9 | [tbd] thin-data accuracy | Baseline first | pending backtest |
| VR-10 | ≥ 3 factors | Enough to explain without overload | proposed |
| PF-2 | ≥ every 7 days | 25 ÷ 7 ≈ 3.6 runs per shortest cycle | proposed |
| PF-3, SC-1 | [tbd] 6 h; 50,000 base; 100,000 load test | Base size unknown | pending data |
| PF-4 | ≤ 1 hour | Support reply time unknown | proposed |
| RL-1 | ≥ 6 kill points | One per pipeline stage | proposed |
| RL-4 | ≤ 5 min | As DE-1 | proposed |
| RL-6 | 48 h or next charge | Offers must not arrive after the charge | proposed |
| OB-3 | ≤ 15 min | Staff see decisions the same hour | proposed |
| CP-1 | 20-snippet human read | Patterns miss names | proposed |
| CP-3 | 0 direct identifiers in prompts | Prompts are logged and third-party | proposed |
| CP-4 | [tbd] | Consent rule unknown | pending decision |
| CT-1 | 5 bad configs | Covers the safety-relevant values | proposed |
| CO-1 | LLM spend is uncapped in the build [src builder, 2026-10-17]: correctness first, cost optimization afterwards. 100% of runs record LLM calls, tokens and cost, so optimization can start from real figures. | Query the run table over 5 test runs: all 3 fields present | GC-4 | Should |
| SC-2 | ≤ 2.2× time for 2× base | Allows 10% overhead on linear growth | proposed |
| DP-1 | ≤ 1 hour | One person, clean install | proposed |
| DP-2 | 3 runs | A week-long test is outside the build window | proposed |

**Open items only a running system, an upstream fix or the builder can answer:**
1. **Archetype labels.** Do cancellation reasons or survey tags exist in Snowflake? Without them VR-8 rests on hand labels (H).
2. **Churn definition.** Cancel, skip streak, or no order in N days? It decides the labels for EV-1 and EV-2.
3. **Which actions the system can execute.** Whether Klaviyo sends, offer creation, cadence changes and blend changes can be run from here is unknown (H). Until then DE-6 holds and Assumption 4 applies.
4. **Email consent.** The PRD sets no unsubscribe or consent rule; CP-4 is a placeholder. The builder decides whether to enforce one.
5. **LLM provider.** LLM spend is uncapped (builder, 2026-10-17); the provider is chosen on quality in the stack step.
6. **Active base size.** Sets PF-3, SC-1 and SC-2.
7. **Life-change signal.** If support text carries none, FR-4 reduces to generic escalation (H).
8. **The 70% autonomy target.** If approval-gated actions exceed 30% of dispatched actions, the target fails by design (Key decision 3). Measure the mix early.
9. **Assumption 4.** Confirm the system never changes a subscription on its own.
