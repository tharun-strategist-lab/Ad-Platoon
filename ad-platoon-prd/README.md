# Ad Platoon

**Autonomous AI marketing platform — designed and specified end-to-end as an AI Product Manager.**
Ad Platoon manages digital ad campaigns across Meta, Google, and TikTok using a coordinated multi-agent AI system, replacing platform-estimated performance data with server-verified truth and human-supervised autonomous execution.

I'm Tharun Prakash, an AI Product Manager with 6+ years in growth and brand strategy. This repo documents the full product thinking behind Ad Platoon — problem framing, architecture decisions, trust/safety design, and a working prototype — written entirely by me, no templates or ghostwriting.

---

## Start Here

| If you want to see... | Go to |
|---|---|
| The full product spec | [`ad-platoon-prd/`](./ad-platoon-prd/) |
| The working prototype | [`ad-platoon-prototype/`](./ad-platoon-prototype/) |
| How I evaluated the AI architecture | [`llm-eval-notebook/`](./llm-eval-notebook/) |
| How I think through product decisions | [`pm-case-studies/`](./pm-case-studies/) |
| The actual agent prompts | [`prompt-library/`](./prompt-library/) |

## The Problem, in One Paragraph

Since Apple's iOS 14 privacy changes, platform-reported ad conversions are inflated 15–40% versus what actually happened — brands are optimizing budgets against numbers that were never real. Meanwhile, human media buying teams check performance once or twice a day, so budget keeps flowing into underperforming campaigns for hours between reviews. Ad Platoon closes both gaps: verifying every conversion server-side against real Shopify/Stripe revenue, and optimizing hourly through a coordinated system of AI agents operating inside hard safety guardrails.

## System at a Glance

Four agents, one shared state layer, one non-negotiable safety rule: nothing executes without passing through the Guardrail Agent.

Media Buyer Agent → hourly yield optimization
Creative Agent → on-brand copy + image generation
Attribution Agent → server-verified conversion truth
Guardrail Agent → deterministic safety validation on every action

Full architecture breakdown in [`ad-platoon-prd/README.md`](./ad-platoon-prd/README.md).
## What This Repo Demonstrates

- **Zero-to-one product specification** — a complete PRD covering architecture, API strategy, MLOps, and compliance
- **Technical decision-making with evidence** — the LoRA/DPO fine-tuning decision was made against a measured reliability delta (12–18% → <0.5% schema error rate), not intuition
- **Trust and safety design for autonomous AI** — a 3-tier autonomy framework built around how much risk a human is actually willing to extend, not just what the AI is technically capable of
- **Working software, not just specs** — a functional prototype validating the core interaction loops with a real backend

## About Me

[LinkedIn](#) · [Portfolio](#) · [Résumé](#)

Currently building Ad Platoon full-time while leading product strategy for Fortune 500 martech clients as COO at The Creed. Open to Senior AI Product Manager roles.
  
