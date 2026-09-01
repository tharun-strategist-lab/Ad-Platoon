# Model Architecture Decision: Fine-Tuning vs. Prompt Engineering

A case study in how I evaluated whether Ad Platoon's AI agents needed a fine-tuning investment, or whether prompt engineering on an off-the-shelf model was sufficient.

## The Question

Ad Platoon's agents need to produce structured, machine-executable output (JSON mutation manifests that get parsed and sent directly to platform APIs) — not conversational text. The question: is prompt engineering on GPT-4o/Claude reliable enough for this, or does the reliability requirement justify a fine-tuning investment (LoRA + SFT + DPO)?

This matters because a malformed API call on a live ad account isn't a cosmetic bug — it can misconfigure or halt a real campaign spending real money.

## What I Tested

Three architecture options, evaluated against the same task: given a campaign performance snapshot, produce a structured budget-mutation recommendation in a fixed JSON schema.

| Approach | Method |
|---|---|
| **Baseline** | Off-the-shelf model + prompt engineering (system prompt + few-shot examples enforcing JSON output) |
| **SFT only** | LoRA fine-tune (r=64) on a performance-marketing corpus — domain vocabulary, no explicit output-format training |
| **SFT + DPO** | Adds a preference-pair training stage: "chosen" = valid JSON manifest, "rejected" = conversational prose response to the same prompt |

## How I Measured It

Ran each configuration against a held-out set of 200 realistic campaign scenarios (varied performance patterns, edge cases like zero-spend days and missing conversion data) and scored:

- **Schema validity rate** — did the output parse as valid JSON matching the required schema, with no retry?
- **Field completeness** — were all required fields present and correctly typed?
- **Directional correctness** — did the recommended action match the expected direction (increase/decrease/pause) given the input data?

## Results

| Configuration | Schema Validity Rate | Notes |
|---|---|---|
| Baseline (prompt engineering only) | ~82–88% | Failures were mostly conversational preambles ("Sure, here's the recommendation:") wrapping otherwise-valid JSON, plus occasional missing fields on edge-case inputs |
| SFT only | ~91% | Better domain reasoning, marginal improvement on format reliability — SFT alone doesn't specifically train output *format* preference |
| SFT + DPO | **99.5%+** | DPO's chosen/rejected pairing directly targets the failure mode — the model learns to prefer structured output as its default register, not just as an instruction it sometimes follows |

**The gap between SFT-only and SFT+DPO was the deciding data point.** SFT improves domain knowledge; DPO is what actually fixes the specific failure mode (conversational wrapper text around valid JSON) that a retry-and-hope approach can't reliably catch in production.

## Decision

Fine-tune with SFT + DPO. The reliability delta (roughly 12–18% failure rate down to under 0.5%) is the difference between a system that occasionally needs a human to catch a malformed API call, and one trustworthy enough to run autonomously within guardrails. Given Ad Platoon's Autopilot tier requires unattended execution, an ~85% baseline schema validity rate was disqualifying on its own — the question was never "is fine-tuning nice to have," it was "what's the minimum reliability bar to responsibly ship autonomous execution."

## What I'd Test Next

- **GRPO on a simulated ad environment** — the current eval measures format reliability and directional correctness, not whether the *recommended action itself* was actually the right call. A reinforcement-learning pass against an offline Ad Sandbox Simulator (rewarding trajectories that improved real CPA/ROAS) is the next layer, planned for v2.
- **Per-tenant adapter regression testing** — as per-tenant LoRA adapters get fine-tuned on individual brand data, I'd want a regression suite confirming schema validity doesn't degrade as adapters specialize.
- **Adversarial input testing** — deliberately malformed or contradictory campaign data, to see where the 99.5% floor actually breaks.
