# USERS.md — Thesis Churn Prevention Agent

## Scope

**This document covers:** who the product is for and who it is not for; what goes wrong for them today, with sources; the moments the product reaches them; the four use cases it serves and why an agent beats the obvious alternatives for each; the minimum first iteration; what is out of scope; and the constraints the project was given.

**This document does not cover:** how the product is built, verified, measured or hosted. Requirements, key metrics, stack and architecture are separate documents that trace back to the use case IDs (UC-n) and the constraints here.

**Status:** provisional draft, 2026-10-16. Source material is the partner brief (`docs/Thesis - Churn Project.docx`, R. Cattan, 2026-04-15) and `docs/Portal PRD.md`. No stakeholder is available, so nothing here was confirmed with Thesis. Claims not backed by the brief, a cited source, or the builder's own answers are marked *(assumption)*. The four load-bearing statistics [R1]–[R4] were re-opened and confirmed on 2026-10-17; the rest came from a research pass (Section 9).

---

## 1. Target user

**A Thesis subscriber (a person paying monthly for a personalized nootropic blend ordered through a quiz) who is drifting toward cancelling and has not cancelled yet.** Subscriptions cost $59–$79 a month on a 25-, 50- or 75-day shipment cycle, and subscribers can skip, reschedule or cancel after the second order [R6].

The brief names four reasons a subscriber drifts, and this document treats each as its own moment for the same person: price (UC-1), the blend doesn't seem to work (UC-2), too much product or tired of the routine (UC-3), and a change in their life (UC-4). The subscriber never opens this product. It reaches them as a message, an adjusted offer or blend, or a person's reply; the product exists in the minutes before they would have pressed cancel.

**Not the target user:** the Thesis staff who approve high-stakes actions or take escalations, and the engineer who maintains the system. They matter downstream, but this document is written for the person receiving the intervention.

---

## 2. The problem, in numbers

Each source below is its own study and its numbers are not combined.

### Churn is a normal, constant loss

Recurly's July 2026 network benchmark reports median annual churn of **4.25% for ecommerce**, split into **2.87% voluntary** and **1.38% involuntary** (failed payment) [R1]. Sample size and sampling are not stated. This says how much subscribers leave and that most of it is chosen, not that it is avoidable.

### Why people say they cancel (two separate consumer surveys)

- A Closer Look, December 2024, n=1,091 US adults: cost **63%**, lack of usage or no longer needing it **57.8%**, dissatisfaction with content or service **15.4%** [R2]. The reason shares add to about 158%, which suggests respondents could pick several, though the page doesn't say.
- PYMNTS and sticky.io, May 2022, n=1,919 US consumers: reducing expenses **65%** (55% said it was the most important reason), no longer needed **29%**, received more than used **17%**, product quality declined **13%** [R3].

These map to our archetypes as follows: price is well supported (UC-1); stockpile and fatigue is supported (UC-3); **no source gave a share for "it didn't work" (UC-2) or for life change (UC-4)**, so those two rest on the brief and on Thesis's own reviews. Both are stated reasons, not verified causes, and neither is supplement-specific.

### What Thesis subscribers say (anecdotal)

Thesis has a 4.0 Trustpilot rating on 129 reviews. Recurring complaints are that cancelling is hard, that shipments arrive faster than the product is used, and that the price is high [R7]. A second review site shows the same price complaint on 81 reviews [R8]. These are unrepresentative self-selected reviews.

### Harm from getting the response wrong

A randomized test on 65,000 customers of a wireless carrier found that proactively recommending a cheaper plan raised churn to **10%, against 6% for the control group** (time window not stated) [R4]. It is a different industry and a different offer, but it is the strongest evidence found that a "helpful" retention message can cause the cancellation it was meant to prevent. It is the reason the product must choose its action by cause, not blast every at-risk subscriber with the same offer.

---

## 3. Why we target the moment before the cancel, by reason

Retention today is entirely reactive: signals sit in subscription, email, support and quiz data with no connection between them and no automatic response (brief). The goal is not to message more subscribers; it is to reach the right subscriber, with the right response for their reason, while there is still something to change.

**Reason 1: Existing save tools act only at the cancel button.** Cancel-flow products (Recharge, Smartrr, Loopwork, Churnkey, ProsperStack) present an offer once the subscriber has already decided, and the ones checked list no predictive scoring on their pricing pages [R5]. The brief asks for action *before* the cancel.

**Reason 2: The cost of getting the reason wrong is real.** A discount sent to someone who is stocked up, or a "try a new blend" nudge sent to someone who simply can't afford the plan, is a miss, and the carrier experiment shows a miss can raise churn [R4].

**Reason 3: Cancelling is already friction-heavy.** 40.8% of respondents in one survey had trouble finding the cancel option [R2], and Thesis reviews say the same. A product that responds to a person drifting away has to keep their way out easy.

### Out of scope: the moment of cancellation itself, and involuntary churn

Once a subscriber has pressed cancel, or when a card fails, the work is different (a save flow, a payment retry) and a different set of tools exists for it. This product acts earlier.

---

## 4. Workflow: where the product reaches the subscriber

Four subscribers, four evenings. In each, the product has already scored the subscriber and decided before they think about cancelling.

**Mon 8:10 PM, Dana, month 5.** She is looking at the next-charge email: $79. Her budget is tight this month and she has been putting off the decision for a week. She wouldn't open an app. What reaches her is a note that offers a way to lower the cost or the cadence. → **UC-1**

**Tue 7:45 AM, Marcus, month 2.** He took the quiz to improve focus. Eight weeks in he can't tell if the blend does anything, has skipped the last shipment, and wrote to support once asking whether it works. He sees an email that acknowledges his goal and suggests a change to the blend, based on his quiz answers. → **UC-2**

**Wed 9:30 PM, Priya, month 6.** She has three unopened boxes. She isn't unhappy; she just has too much. She sees a message offering to slow her shipments, not a discount. → **UC-3**

**Thu 12:15 PM, Sam, month 4.** Sam wrote support that a new baby, a move and a new job have changed everything. The product doesn't send an offer. A person gets the case, with context, so that Sam doesn't have to repeat it. → **UC-4**

*(The four people, the clock times and the messages are illustrations built from the brief's archetype names and the survey reasons, not observed subscribers.)*

---

## 5. Use cases

Across all four use cases, a subscriber with little data (no quiz, a short history) must still get a sensible response and not an error or a generic blast. *(Brief requirement.)*

### UC-1: Price sensitivity

| | |
|---|---|
| **Trigger** | Rising risk with signals that point to cost: a recent price or plan change, skips around the charge date, price-related support text *(assumption)* |
| **Input** | Subscription, order, email-engagement and support history; quiz data if any |
| **What they need to see** | A concrete way to lower what they pay or pay less often, stated plainly, with no pressure to decide today |
| **What they do with it** | Accept, adjust the plan, or ignore it |
| **Time available** | Until their next charge; the survey data says cost is the most-cited reason [R2, R3] |
| **Must never** | Offer a discount to someone whose reason isn't price; make cancelling harder; hide the real price |

**Why an agent, not "if risk > 0.7, send discount":** A rule can send a coupon to high-risk subscribers, and for subscribers who are clearly price-driven it will often be enough. That is exactly the rule the brief calls out as insufficient. What a rule can't do: (1) tell price-driven subscribers from fatigued or unhappy ones, when the evidence is mixed across support text, skips and engagement; (2) scale the response (a pause, a lighter cadence, a smaller offer) to the subscriber; (3) avoid the backfire documented in [R4] by withholding an offer when the reason isn't cost. The value is in deciding when *not* to discount.

### UC-2: Efficacy dissatisfaction

| | |
|---|---|
| **Trigger** | Rising risk with signals that the blend isn't delivering: skips, a quiz retake or a gap since the last one, complaints about results, switching between blends |
| **Input** | Quiz responses and recommendations (when present), purchase history, support text |
| **What they need to see** | Acknowledgement of their original goal and a specific alternative blend or adjustment based on what they told Thesis |
| **What they do with it** | Try the adjustment or keep their current plan |
| **Time available** | Within their next order cycle |
| **Must never** | Claim a medical or cognitive result; suggest a change based on data they didn't give; contradict what they were told before |

**Why an agent, not a fixed "recommended blend" list:** A rule can map quiz answers to a blend. What it can't do: (1) use the unstructured text of a support conversation to see that the complaint is about results; (2) fall back sensibly when there is no quiz data (purchase history and engagement instead); (3) explain the suggestion in the subscriber's own terms. *(This rests on the assumption that support text carries the signal; it weakens if the data has little free text.)*

### UC-3: Subscription fatigue

| | |
|---|---|
| **Trigger** | Rising risk with signals of too much product: repeated skips, delay requests, "I have enough" language |
| **Input** | Order and skip history, shipment-modification history, support text |
| **What they need to see** | An easy way to slow down or pause shipments without cancelling |
| **What they do with it** | Reschedule, lengthen the cycle, or ignore it |
| **Time available** | Before the next shipment ships |
| **Must never** | Ship more product than they asked for; offer a deeper discount in place of a slower cadence |

**Why an agent:** Skipping is already visible to a rule, so a rule can detect it. The agent's use is telling "stocked up and happy" from "disengaging", which look the same in order data and call for different responses. *(A research-pass note: Thesis does not support an indefinite pause, so the response may be a longer cadence or rescheduling [R6].)*

### UC-4: Life change

| | |
|---|---|
| **Trigger** | Rising risk with signals of a circumstance change: support text about a move, a baby, a new job, a health event *(assumption: this signal exists in the data)* |
| **Input** | Support history and text, any account changes |
| **What they need to see** | A human who already knows the situation and offers an easy pause or a graceful exit |
| **What they do with it** | Pause, adjust, or leave on good terms |
| **Time available** | The next support reply |
| **Must never** | Send an automated offer; make them repeat their story; make the exit harder |

**Why an agent:** A rule can't read a support message and recognize that this is not a price or product problem. The agent's value is to recognize it and hand a person the context, a case where automation should *not* act. No source found a share for this archetype, so its size is unknown *(assumption)*.

---

## 6. The minimum product

**First iteration.** The brief and PRD set the floor and the builder confirmed "no cuts": the whole listed set is the minimum. A subscriber's experience is only coherent if the system scores, tells the reasons apart, chooses the right response, and records what happened; without any of these the system collapses into a blast or a dashboard, both of which the brief rules out.

| # | Feature | Use case | What it does for the subscriber | Acceptable in v1 |
|---|---|---|---|---|
| 1 | Score every active subscriber's churn risk, on a schedule | UC-1–4 | Nobody drifting is missed | Whole active base, not a sample; precision > 70% in the top risk decile and recall > 60% (PRD targets) |
| 2 | Show the top reasons behind each score | UC-1–4 | Responses fit the reason | Top contributing factors per subscriber |
| 3 | Classify the churn archetype | UC-1–4 | The response matches why they're leaving | Four archetypes route to different paths |
| 4 | Personalized check-in email | UC-1–3 | A human-sounding note that fits their situation | No raw PII in logs |
| 5 | Formulation adjustment suggestion | UC-2 | A blend change that references their quiz when available | Falls back to purchase and engagement history when there is no quiz |
| 6 | Targeted retention offer | UC-1 | A real way to pay less | Never sent when the reason isn't price |
| 7 | Escalation to a human | UC-4 and others | A person gets the case with context | Case includes score, reason and history |
| 8 | Approval before high-stakes actions, thresholds adjustable without code | UC-1, UC-4 | Risky actions are checked by a person | Over 70% of interventions go out without approval (PRD target) |
| 9 | Handle incomplete data | UC-1–4 | New or quiet subscribers still get a sensible response | Works with no quiz and little history |
| 10 | Log every action with context, response and outcome | UC-1–4 | The system learns what worked | 100% of actions logged; coverage of at-risk subscribers > 85% (PRD target) |
| 11 | Recover from failure without losing state | UC-1–4 | A broken run doesn't double-send or skip anyone | Resumes where it stopped |
| 12 | Use logged outcomes to improve scoring and selection | UC-1–4 | Responses get better over time | The brief calls a static model incomplete |

*(Acceptance values are the PRD's own targets. They are not validated, and the builder did not state any other quality bars: latency, consistency and similar are open, see the handoff.)*

**Later iterations.** None named. The builder asked to focus only on the requirements, so the wish-list pass was skipped on purpose.

---

## 7. Explicitly out of scope

| Not doing | Why |
|---|---|
| A dashboard or report as the deliverable | The brief says the deliverable is action; staff already have Metabase for results |
| Direct connections to Shopify, Klaviyo or Zendesk | The data is in Snowflake; the brief forbids it unless real-time data is required |
| A static rules engine presented as an agent | The brief forbids it |
| Fine-tuned models where simpler approaches work | The brief forbids it |
| Raw customer PII in logs or training data | The brief forbids it |
| Cancel-button save flows and failed-payment recovery | Different moment and different tools; this product acts earlier |

---

## 8. Constraints we were given

| Constraint | What it is | Given by |
|---|---|---|
| Timeline | Due Tuesday 2026-10-20, midnight Central (the builder said "CST"; read as end of day, 11:59 PM Central). The brief says 3–5 days. Nothing was named as droppable first because "no cuts" | Builder; brief |
| Team | One person | Builder |
| Spend | No budget for now. Anything outside the approved list may be used only on a free tier. Confirmed by the builder; whether existing credit covers LLM API calls was not stated | Builder |
| Services / vendors | Approved whitelist the builder may choose from: Anthropic, Aqua Voice, Cohere, Cursor, DigitalOcean, ElevenLabs, Google One, Midjourney, OpenAI, OpenRouter, Railway, Vultr, Whisper. Free tiers of other services are also allowed. The brief requires Python, SQL, an LLM API (Anthropic Claude or OpenAI), Snowflake and agent orchestration | Builder; brief |
| Hardware in hand | None bounding the first iteration | Builder |
| Must integrate with | Snowflake views for subscriptions, orders, email engagement (Klaviyo sync), support (Zendesk sync) and quiz data; outputs must be queryable in Metabase | Brief |
| Intended use | A demo product for Thesis's evaluation. The brief says it should be deployable to real data with minimal change | Builder; brief |
| Data access | The PRD says all data comes from existing Snowflake production views and that the specific tables are reviewed during onboarding. Real access for this build is therefore assumed, not confirmed *(assumption)*; if it falls through, accuracy can only be shown on synthetic data | Brief |
| Regulatory / contractual | The brief's rule against raw PII in logs and training data. The PRD and brief state no email-consent or unsubscribe rule, so none is given here; see the handoff | Brief |

---

## 9. Evidence limitations

- **[R1] Recurly** (re-opened 2026-10-17; figures confirmed) gives annual medians across its network with no sample size, and the page's text and chart disagree on some other rows (not ecommerce). It is annual and not comparable to a monthly churn target such as the brief's 2–3 points. It shows how much subscribers leave, not that it is avoidable.
- **[R2]** (re-opened; figures confirmed) is a 2024 blog post summarizing a survey of 1,091 US adults. Recruitment, screening, fieldwork dates and question wording aren't stated. **[R3]** (re-opened; figures confirmed) is a 2022 survey of 1,919 US consumers whose page mixes "in March" and "over the last 12 months" for different reasons. Both are stated reasons, neither is supplement-specific, and they are different studies, not combined.
- **[R4]** (re-opened; figures confirmed) is a trade-press account (2015) of a randomized field test at an unnamed South American wireless firm. The article gives no time window, group sizes or significance test, and I did not read the paper. It shows a mechanism, not that it applies here.
- **[R5]** competitor pricing was read from pricing pages by a research pass; features not listed there may exist.
- **[R7], [R8]** are self-selected reviews: anecdotal.
- No source gave a share for the efficacy or life-change archetypes, and none tested skips or engagement decline as predictors in consumable subscriptions.
- [R5]–[R8] (competitor pricing, Thesis pages, reviews) come from the research pass and were not re-opened by the author. Vendor figures (e.g. Churnkey pause and discount shares) have no stated control group and are not used here.
- None of these sources measures whether this product reduces churn.

---

## References

1. Recurly. Churn rate benchmarks (July 2026). https://www.recurly.com/research/churn-rate-benchmarks
2. A Closer Look. Subscription Exit Strategies (2024-12-20), n=1,091. https://www.a-closer-look.com/market-research/subscription-cancellation-customer-experience-study
3. PYMNTS and sticky.io. Subscription Commerce Conversion (2022-05), n=1,919. https://www.pymnts.com/?p=1307161
4. Ascarza, Iyengar, Schleicher. The Perils of Proactive Churn Prevention Using Plan Recommendations. Journal of Marketing Research (2016); summary: https://www.mediapost.com/publications/article/248672/customer-retention-efforts-can-often-backfire.html
5. Recharge, Smartrr, Loopwork, Churnkey, ProsperStack pricing pages: https://getrecharge.com/pricing, https://smartrr.com/pricing, https://www.loopwork.co/pricing, https://churnkey.co/pricing, https://prosperstack.com/pricing/
6. Thesis. Clarity product page, terms of service and refund policy. https://takethesis.com/products/clarity, https://takethesis.com/policies/terms-of-service, https://takethesis.com/policies/refund-policy
7. Trustpilot. takethesis.com reviews. https://www.trustpilot.com/review/takethesis.com
8. Thingtesting. Thesis reviews. https://www.thingtesting.com/brands/thesis/reviews
