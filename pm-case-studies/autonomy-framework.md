# Case Study: Designing the 3-Tier Autonomy Framework

**Product**: Ad Platoon
**Decision owner**: Tharun Prakash, Co-Founder & CPO
**Category**: Trust & safety architecture for autonomous AI

## Problem

Ad Platoon's core value proposition requires AI agents to execute budget and campaign changes without waiting for human approval on every action — that's what makes it faster than a human media buying team. But the person authorizing that AI is a Commander (CMO/VP of Growth) putting real career risk on the line: if an autonomous system misallocates budget or damages brand reputation, that decision reflects on them personally, not on the AI.

The product had to solve two conflicting requirements simultaneously: genuine autonomy (the whole point of the product) and a trust on-ramp gradual enough that no Commander would ever feel forced into a leap of faith.

## Options Considered

**Option A — Full autonomy from day one.**
Simplest to build. Rejected: no enterprise buyer signs a contract that removes them from the execution loop before they've seen the system perform even once. This would have killed every sales conversation at the first objection.

**Option B — Recommendation-only, forever.**
Safest, easiest to sell initially. Rejected: this eliminates the core value proposition. A recommendation-only tool is a dashboard, not an autonomous platform — it doesn't solve the decision-latency problem that's central to the product thesis.

**Option C — Time-based automatic escalation** (e.g., auto-upgrade to full autonomy after 30 days).
Rejected: removes the human's agency over *when* they're ready to extend trust. Trust that's granted on a timer rather than a decision doesn't actually feel safer to the person granting it — it just feels like the system deciding on their behalf, which reintroduces the exact anxiety the framework is meant to resolve.

**Option D — Three explicit tiers, human-controlled progression, hard-coded circuit breakers.**
Selected.

## Decision

Three tiers, each a genuinely different operating mode rather than a cosmetic label:

- **Auditor** (read-only) — the AI generates recommendations and logs them, but has zero write access to any ad platform. This lets a Commander build a track record of "would I have approved these?" before any real risk exists.
- **Copilot** (human-in-the-loop) — the AI proposes, a human approves each action, execution is instant on approval. Recommendations expire after 4 hours to prevent stale-data execution.
- **Autopilot** (bounded autonomy) — the AI executes without per-action approval, but only after 30+ consecutive approved Copilot actions, a written guardrail acknowledgment, and a second Commander's co-signature with a mandatory 24-hour confirmation window before activation.

Autopilot is never *unbounded* autonomy — hard circuit breakers (max 20% budget delta per 12 hours, automatic downgrade on >35% spend anomaly, immediate auto-revert to Copilot on any guardrail violation) mean the system can't compound a bad decision even once activated.

## Outcome

The framework reframes the Commander's decision from "do I trust an AI with my budget" to "do I trust a process I fully control the pace of." The co-signature and 24-hour window specifically borrow the shape of financial dual-authorization controls that enterprise buyers already recognize and trust from other systems — it wasn't designed to be persuasive marketing language, it was designed to *feel structurally familiar* to someone whose job is assessing organizational risk.

The unresolved trade-off: this framework adds real friction to reaching full autonomy, which is a deliberate cost against speed-to-value. I don't yet have production usage data on how many design partners actually progress to Autopilot versus staying in Copilot indefinitely — that's the open question the next phase of the product needs to answer.

## What I'd Reconsider

If early usage data showed most users plateau at Copilot and never activate Autopilot, that would suggest the trust on-ramp is calibrated too conservatively relative to what users actually need to feel safe — and the co-signature/24-hour requirement might be over-engineered friction rather than appropriately-placed friction. I don't have that signal yet; it's the first thing I'd want to measure post-launch.
