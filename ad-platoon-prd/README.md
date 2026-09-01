# Ad Platoon — Product Requirements Summary

> Condensed from the full 900+ paragraph production PRD. This document covers the problem, architecture, and key decisions — see linked case studies for deep-dives on specific trade-offs.

## Problem Statement

Advertisers running campaigns across Meta, Google, and TikTok face three compounding failures:

- **Attribution inflation**: Post-iOS 14, platform-reported conversions are inflated 15–40% vs. server-verified truth (Shopify/Stripe), because platforms fill tracking gaps with modelled estimates rather than observed data.
- **Decision latency**: Human media buyers review performance once or twice daily. Budget continues flowing into underperforming campaigns for hours between reviews.
- **Creative production lag**: A 48-hour brief-to-live-ad cycle means fatiguing creative keeps burning budget for days before replacement.

Ad Platoon is an autonomous multi-agent system that closes all three gaps — verifying truth server-side, optimizing hourly instead of daily, and generating replacement creative in under 5 minutes.

## System Architecture

Four specialized agents, coordinated through a shared PostgreSQL/Redis state layer:

| Agent | Responsibility |
|---|---|
| **Media Buyer** | Hourly yield scoring (ROAS 60% / CTR 25% / CPA 15%), budget reallocation, fatigue detection |
| **Creative** | Landing-page + brand-book parsing (pgvector RAG), copy generation, image generation pipeline |
| **Attribution** | HMAC-tokenized tracking, server-to-server webhook verification, PII hashing, CAPI forwarding |
| **Guardrail** | Deterministic (non-LLM) rule engine — the safety layer every mutation passes through before execution |

All inter-agent communication runs through a Kafka event bus (10 topics), which decouples agent failure domains — a slow Attribution Agent never blocks the Media Buyer Agent's optimization cycle.

**Anti-ban infrastructure**: BullMQ leaky-bucket rate limiting (15 mutations/min/account), randomized execution jitter (60–900s), and a Circuit Breaker pattern that freezes writes and rolls back to a 6-hour-old stable snapshot after 3 consecutive API failures.

## API Integration Strategy

- **Meta**: System User Access Token (permanent, non-expiring), Graph API + Conversions API (CAPI)
- **Google**: OAuth 2.0 with 50-minute refresh cycle, batched `MutateJob` writes (4 fixed windows/day — Google penalizes high-frequency small writes)
- **TikTok**: App Access Token with daily refresh, Events API for server-side conversions

All three integrations sit behind the Circuit Breaker Gateway — no agent calls a platform API directly.

## MLOps Pipeline

The core reliability problem: pure prompt engineering produces unpredictable structured output (see `llm-eval-notebook/` for the actual measurement). The pipeline:

1. **SFT** — Llama-3-70B + LoRA (r=64), trained on performance-marketing corpus for domain vocabulary
2. **DPO** — preference-pair training (chosen: valid JSON manifest / rejected: conversational prose) to enforce structured, machine-executable output
3. **Per-tenant LoRA adapters** (<100MB, r=16) — hot-swapped via vLLM at inference time for brand-specific personalization without cross-tenant data leakage
4. **GRPO** (v2) — reinforcement learning against an offline Ad Sandbox Simulator, rewarding trajectories that actually improved CPA/ROAS, not just trajectories that looked plausible

## The 3-Tier Autonomy Framework

The core trust-architecture decision — full reasoning in [`pm-case-studies/autonomy-framework.md`](../pm-case-studies/autonomy-framework.md).

| Tier | Write Access | Trust Mechanism |
|---|---|---|
| **Auditor** | None | Read-only recommendation feed; builds a track record before any execution authority is granted |
| **Copilot** | Conditional | Human approves each recommendation; 4-hour expiry prevents stale-data execution |
| **Autopilot** | Full, within guardrails | Requires 30+ consecutive approved actions + dual Commander co-signature + 24-hour confirmation window |

Autopilot auto-downgrades to Copilot on any of: circuit breaker trip, >35% spend deviation in a 4-hour window, >15% attribution discrepancy, or billing pause.

## Compliance Approach

- **GDPR**: All PII (email/phone/IP) hashed via SHA-256 within 1 second of receipt; raw values never persist. 90-day retention on hashed values, then purged. 72-hour breach notification protocol.
- **SOC 2 Type I readiness**: Access control (RBAC — Commander/Strategist/Analyst), encryption at rest (AES-256) and in transit (TLS 1.3), documented incident response, immutable append-only audit log with SHA-256 hash chaining (tamper-evident — any modification breaks the chain at the next verification pass).
- **Platform policy compliance**: Automated 6-hour policy-drift scraper (HAG memory fabric) keeps agent context current with Meta/Google/TikTok ad policy changes without requiring model retraining.

## Related Documents

- Full PRD (v1.0.0-Prod) — available on request
- [Autonomy framework case study](../pm-case-studies/autonomy-framework.md)
- [Model architecture decision](../llm-eval-notebook/README.md)
- [Prototype documentation](../ad-platoon-prototype/README.md)
