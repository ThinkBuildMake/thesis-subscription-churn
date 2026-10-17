# Handoff: design and requirement content removed from USERS.md

These items came from the PRD, the brief and the interview but describe how the product is built, measured or verified, not who it serves. Carry them into requirements, key metrics and architecture. Each maps to a use case ID.

## Targets the brief set (for the metrics step)
- Monthly churn reduction of at least 2–3 percentage points within 90 days. Must be measured per archetype, per the builder's request to track all four (UC-1–4).
- Precision above 70% in the top risk decile; recall above 60% for pre-cancellation churn (UC-1–4).
- Intervention coverage above 85% of identified at-risk subscribers; autonomous action rate above 70% of dispatched interventions; 100% of actions logged (UC-1–4).

## Access and trust
- Approvers and escalation recipients need a way to see the case and decide; thresholds must be adjustable without code changes (UC-1, UC-4).
- No raw PII in logs or training data (all).

## Verification and honesty
- Each intervention must be traceable to the score, archetype and factors that chose it (all).
- Offers must not go to subscribers classified outside the price archetype (UC-1).
- Failure and retry must not double-send or lose state (all).

## Capabilities traced to use cases
| Capability | Required by |
|---|---|
| Per-subscriber risk score with top factors | UC-1–4 |
| Archetype classifier | UC-1–4 |
| Email send; offer; blend suggestion; escalation | UC-1, UC-2, UC-3, UC-4 |
| Approval queue with configurable thresholds | UC-1, UC-4 |
| Handling of missing quiz and short history | UC-1–4 |
| Outcome logging feeding scoring | UC-1–4 |

## Scope boundaries with their design reasons
- No dashboard deliverable: the brief wants action; results go to Snowflake and Metabase.
- No direct API connectors: data is in Snowflake.

## Quality bars not asked (for the requirements step)
These were not stated by the builder or the brief and are left open: consistency, latency per run, scale beyond "whole active base", fault-tolerance specifics, send-time windows, and email-consent compliance.

## Open questions for the audit / research step
- **Archetype labels:** does any ground truth for the four archetypes exist (cancellation reasons, survey tags)? Without labels, archetype classification can't be evaluated; this is the biggest unknown behind UC-1–4.
- **Life-change signal (UC-4):** does support text or any field carry it? If not, UC-4 reduces to a generic human escalation.
- **Efficacy and life-change shares:** no external source found; measure from Thesis data if available.
- **Data access:** the PRD implies real Snowflake views, with tables confirmed at onboarding; if access fails, precision and recall are measured on synthetic data and prove little.
- **Churn definition:** what counts as churn (cancel, skip streak, no order in N days) is not in the brief.
- **Leading indicators:** no primary study confirms skips or engagement decline predict churn in consumable subscriptions; the model must test this.
- **Actions available:** whether Klaviyo sends, offer creation and blend changes can be executed from the system, and in what form, is not stated.
- **Discount training:** no field evidence found on whether offers train customers to cancel again.

## Existing products in the same space
- Recharge, Smartrr, Loopwork, Churnkey and ProsperStack sell cancel-flow offers at about $99–$750/month; none listed predictive scoring on the pricing pages checked.
- Klaviyo's churn-risk feature needs at least 500 ordering customers and 180 days of history.
