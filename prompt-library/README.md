# Ad Platoon — Agent Prompt Library

Representative prompts used by Ad Platoon's four agents, with notes on why each is structured the way it is. These are illustrative examples of the prompting patterns, not verbatim production strings.

## 1. Media Buyer Agent — Mutation Proposal
You are the Media Buyer Agent for Ad Platoon. Given the campaign
performance data below, determine whether any action is warranted.

CAMPAIGN DATA:
{campaign_json}

YIELD SCORE INPUTS:

Server-verified ROAS (last 72h): {roas}
Target ROAS: {target_roas}
CTR vs. 7-day rolling average: {ctr_delta}%
CPA (server-verified): {cpa}

RULES:
Only propose ONE action per campaign per cycle.
If no action is warranted, return {"action": "none"}.
Your rationale must cite the specific numbers above — never
a generic justification.
Output ONLY the JSON object below. No preamble, no explanation
outside the JSON.

OUTPUT SCHEMA:
{action_type, entity_id, current_value, proposed_value,
rationale, confidence_score}

**Why it's structured this way**: The "output ONLY the JSON object, no preamble" instruction exists because this is precisely the failure mode identified in the fine-tuning evaluation (see `llm-eval-notebook/`) — models default to wrapping structured output in conversational text unless explicitly and repeatedly constrained. The rule requiring rationale to cite specific numbers isn't a nicety — it's what makes the recommendation card auditable by a human reviewer in under 10 seconds, rather than requiring them to go verify the claim themselves.

## 2. Attribution Agent — Conversion Verification
Verify this incoming conversion event against the tracking token.

EVENT:
{webhook_payload}

STEPS (execute in order, halt on first failure):

Extract ap_token from event metadata.
Verify HMAC-SHA256 signature against tenant_secret_key.
Confirm event_time is within the 7-day attribution window.
Check deduplication store for this event_id.
If all checks pass: hash all PII fields (SHA-256), discard
raw values, mark as VERIFIED.
If any check fails: mark as REJECTED with the specific
failure reason.
Return ONLY: {status, failure_reason (if applicable),
verified_conversion_value}

**Why it's structured this way**: This isn't really a "creative" prompt — it's closer to a checklist the model executes deterministically, because attribution verification has zero tolerance for ambiguity. The explicit "halt on first failure" instruction prevents the model from creatively working around a failed check, which matters enormously here: a false-positive "verified" conversion directly corrupts the Media Buyer Agent's optimization data downstream.

## 3. Creative Agent — Copy Generation
Generate ad copy for the following campaign using the brand
context provided.

LANDING PAGE CONTEXT (retrieved via RAG):
{retrieved_brand_chunks}

AUDIENCE ANGLE: {angle} # one of: pain-aware, solution-aware, brand-aware

CONSTRAINTS:

Headline: max 40 characters
Description: max 125 characters
Do NOT use any word from this list: {prohibited_vocabulary}
Tone must match: {brand_tone_directive}
Generate 5 headline variants and 5 description variants for
this angle. Output as a JSON array — no commentary.
Generate 5 headline variants and 5 description variants for
this angle. Output as a JSON array — no commentary.

**Why it's structured this way**: The retrieved brand context is injected via RAG rather than relying on the model's general knowledge — this is what makes copy actually sound like the specific brand rather than generic ad copy. The prohibited vocabulary list is a hard constraint, not a preference, because a single off-brand or non-compliant word making it into a live ad is a real compliance failure, not a quality issue to catch on the next iteration.

## 4. Guardrail Agent — Mutation Validation
MUTATION: {proposed_mutation_json}
ACCOUNT STATE: {account_state_json}
CURRENT AUTONOMY TIER: {autonomy_level}

CHECK IN ORDER — reject on first violation:

Budget delta ≤ 20% in trailing 12h window?
Proposed budget ≥ $50/day floor?
No prohibited vocabulary in any text field?
Entity not in 72h post-deployment cooldown?
Current time outside 23:00–01:00 UTC blackout window?
Valid HMAC execution signature present?

Return: {approved: bool, failed_check: string|null,
rejection_code: string|null}

**Why it's structured this way**: Notably, the Guardrail Agent in production is actually hard-coded rule logic, not an LLM call at all — this prompt format is shown here mainly to illustrate the *rule set* clearly, since the same six checks apply whether they're expressed as code or as a prompt. The explicit "this is deterministic rule-checking, NOT a judgment call" framing matters if an LLM is ever used for a secondary review pass — it prevents the model from being "helpful" and creatively interpreting a borderline case in the mutation's favor.

## 5. Media Buyer Agent — Creative Fatigue Detection
Determine whether this ad has entered creative fatigue.

AD PERFORMANCE:

Current CTR: {current_ctr}
7-day rolling average CTR: {avg_ctr}
Impressions (last 24h): {impressions}

RULES:

Fatigue is only assessable with ≥500 impressions in the
evaluation window. Below that, return {"status": "insufficient_data"}.
WARNING threshold: current CTR < 80% of 7-day average.
CRITICAL threshold: current CTR < 65% of 7-day average.
Compute the exact percentage drop — do not round or estimate.

Return: {status, ctr_drop_pct, recommended_action}


**Why it's structured this way**: The minimum-impressions guard exists because low-traffic ads produce noisy CTR data that looks like fatigue but isn't — this prevents the agent from flagging false positives on ads that simply haven't accumulated enough data yet. Two explicit thresholds (not one) allow the downstream system to distinguish "worth watching" from "replace now," rather than forcing every fatigue signal into a single binary action.
