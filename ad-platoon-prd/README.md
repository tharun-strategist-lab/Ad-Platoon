# AD PLATOON — Omnichannel Multi-Agent Performance Platform
> **Enterprise Product Requirements Document (PRD)**  
> **Document Version:** 1.0.0-Prod  
> **Target Release:** Q3 2026  
> **Target Audience:** Enterprise Growth Teams ($100K–$5M/month media spend)  
> **Deployment Architecture:** Single-Tenant VPC Infrastructure ($10K–$50K+/month)

---

## 1. Executive Summary & Business Objectives

Ad Platoon is an autonomous, multi-agent AI marketing engine engineered for high-scale performance advertising across Meta, Google Ads, and TikTok. Enterprise brands operating at scale face structural capital efficiency barriers:

* **Operational Creative Latency:** Creative iteration cycles take 48–72 hours. Ad Platoon compresses this cycle to under 5 minutes from brief to live deployment via automated interpretation, asset synthesis, compliance verification, and execution.
* **Agency & Headcount Overhead:** Over 60–70% of media buying workload consists of deterministic management tasks (bidding adjustments, budget reallocations, creative fatigue monitoring). Ad Platoon automates these workflows, delivering a projected 60% reduction in agency/headcount overhead (~$378K annual savings on a $500K/month spend profile).
* **Cross-Platform Misallocation:** Siloed multi-platform budgets allow capital to remain trapped in underperforming channels between manual review cycles. Ad Platoon rebalances capital hourly based on verified first-party yield.

---

## 2. Core Multi-Agent Architecture

The system operates as an integrated multi-agent platform running within isolated single-tenant VPC environments. Operations are driven by four autonomous agents, an asynchronous state machine layer, a deterministic rule validator, and an Anti-Ban Circuit Breaker Gateway.

┌──────────────────────────────────────────────────────────────────────────────────┐
│                    AD PLATOON — SINGLE-TENANT VPC LAYER                          │
│                                                                                  │
│   ┌──────────────────┐    ┌─────────────────────┐    ┌──────────────────────┐    │
│   │ Media Buyer      │    │ Copywriter/Creative │    │ Attribution Agent    │    │
│   │ Agent (Alloc)    │    │ Agent (RAG/vLLM)    │    │ (First-Party Truth)  │    │
│   └────────┬─────────┘    └──────────┬──────────┘    └──────────┬───────────┘    │
│            │                         │                          │                │
│            └─────────────────────────┼──────────────────────────┘                │
│                                      ▼                                           │
│                     ┌──────────────────────────────────┐                         │
│                     │ Deterministic Guardrail Agent    │                         │
│                     │ (Rule Validation & Delta Caps)   │                         │
│                     └────────────────┬─────────────────┘                         │
│                                      ▼                                           │
│                     ┌──────────────────────────────────┐                         │
│                     │  Circuit Breaker API Gateway     │                         │
│                     │  (Rate Limit / Jitter / Redlock) │                         │
│                     └────────────────┬─────────────────┘                         │
│                                      ▼                                           │
│                 Meta Graph / Google Ads API / TikTok API                         │
└──────────────────────────────────────────────────────────────────────────────────┘

### Architecture Topology

* **Media Buyer Agent (Alloc):** Evaluates server-verified yield hourly.
* **Copywriter / Creative Agent (RAG/vLLM):** Ingests brand directives and landing page context.
* **Attribution Agent (First-Party Truth):** Tracks server-to-server transaction webhooks.
* **Deterministic Guardrail Agent:** Validates rules and enforces budget delta caps.
* **Circuit Breaker API Gateway:** Controls rate limits, execution jitter, and Redlock distributed mutex.
* **Target Ad APIs:** Executes mutations directly on Meta Graph, Google Ads API, and TikTok API.

### Agent Roles & Specifications

* **Media Buyer Agent:** Executes hourly budget allocation adjustments based on server-verified yield. Evaluates performance using a composite score ($Score = server\_ROAS \times 0.60 + CTR\_baseline \times 0.25 + CPA\_eff \times 0.15$) and flags creative fatigue when 3-day rolling CTR falls below 65% of the 7-day average.
* **Copywriter & Creative Agent:** Extracts landing page context via Playwright, processes brand assets via Vision LLM, and retrieves historical winning hooks using hierarchical RAG over `pgvector`. Generates copy variants and localized creative assets using tenant-isolated LoRA fine-tuned adapters.
* **Attribution Agent (First-Party Truth Engine):** Bypasses client-side pixel vulnerabilities by appending HMAC-SHA256 tokens (`ap_token`) to destination URLs. Matches attribution directly against server-to-server transaction webhooks (Shopify, Stripe, custom CRMs) and hashes all PII within 1 second of receipt.
* **Guardrail Agent:** Serves as a deterministic, non-LLM safety proxy that inspects every mutation payload prior to platform submission. Enforces strict hard constraints including a maximum 20% budget variance per 12-hour window, $50/day ad set budget floors, IP similarity checks, and 72-hour cooldown locks.

---

## 3. Anti-Ban & API Translation Gateway

To safeguard advertising assets from platform bans, API suspensions, and rate limits, all outbound requests pass through the Circuit Breaker API Gateway:

* **Rate Limiting:** Enforces strict execution throttles (maximum 15 mutation requests per minute per ad account across Meta Graph, Google Ads, and TikTok APIs).
* **Behavioral Jitter:** Injects randomized execution delays (60s to 900s) to prevent predictable API call patterns.
* **Distributed Mutex Locking:** Uses Redis Redlock to enforce single-flight API updates per entity, preventing concurrent agent state collisions.
* **Circuit Breakers:** Suspends API execution pipelines immediately upon detecting HTTP 429, platform rate limit headers, or account verification warnings.

---

## 4. Progressive Autonomy Framework

Operating risk is managed via a tiered autonomy architecture:

| Autonomy Level | API Write Permission | Execution Rules & Safety Triggers |
| :--- | :--- | :--- |
| **Level 1: Auditor** | Read-Only (0 Writes) | Recommendations generated in `PENDING` state. Visualized on dashboard only; no network mutations permitted. |
| **Level 2: Copilot** | Conditional API Writes | Recommendations auto-expire after 4 hours. Requires explicit manual confirmation ("Approve & Deploy") from authorized users. |
| **Level 3: Autopilot** | Autonomous API Execution | Requires dual-executive cryptographic co-signatures to activate. Executes variations autonomously within 15 mins subject to 20% delta caps, jitter delays, and blackout windows. Automatically downgrades to Copilot upon anomaly detection. |

---

## 5. MLOps & Creative Generation Pipeline

* **Model Fine-Tuning Pipeline:** Employs low-rank adaptation (LoRA) on base open-weights LLMs/VLMs. Fine-tuning is isolated per tenant using historical high-ROAS copy and high-CTR asset pairs.
* **Generative Creative Engine:** Decouples text generation from visual rendering. Generates copy, overlays assets onto brand design templates using canvas manipulation engines (e.g., Sharp), and routes final compositions to a VLM QA gate for brand safety scoring before deployment.
* **Hierarchical RAG:** Embeds campaign history, customer demographic data, and winning ad scripts into `pgvector` to ensure generated assets adhere strictly to tenant-specific brand guidelines.

---

## 6. Success Metrics & Guardrail Policy

### Primary Performance Targets
* **Execution Latency:** $< 5$ minutes P95 from brief ingestion to live multi-platform deployment.
* **First-Party Attribution Match:** $\ge 98\%$ server-to-server transaction reconciliation rate.
* **Infrastructure Efficiency:** Cloud compute cost capped at $< 15\%$ of tenant monthly recurring revenue (MRR).
* **CPA Optimization:** Target $\ge 20\%$ reduction in effective CPA vs baseline over a 90-day window.
* **Copilot Adoption:** Target $\ge 70\%$ recommendation acceptance rate by Month 5.

### Zero-Tolerance Guardrail Violations (P0 Incidents)
* **Platform Bans:** Account ban rate must remain at 0.00%. Any ad account restriction triggers an immediate downgrade to Auditor mode, PagerDuty P0 alerts, and BullMQ task queue cancellation.
* **Budget Drift:** Maximum allowable budget change is capped at 20% over any 12-hour rolling window per entity without explicit cryptographic sign-off.
* **Data Security & Isolation:** Zero cross-tenant data leakage; zero unauthenticated or unauthorized mutation operations on the immutable `audit_log` database; zero plaintext PII storage exceeding 1 second.

---

## 7. Release & Implementation Roadmap

* **Phase 1 (Months 1–2) — Core Infrastructure & SFT Baseline:** Provision single-tenant VPC environments, set up OAuth connections to Meta/Google/TikTok APIs, build Circuit Breaker API Gateway and Guardrail Agent, and deploy SFT/DPO base model pipelines with LoRA fine-tuning support.
* **Phase 2 (Months 3–4) — Copilot Dashboard & Attribution Engine:** Deploy Next.js operations dashboard, launch server-to-server Attribution Agent telemetry, enforce SCOUTING phase campaign validation, and integrate compute metering systems.
* **Phase 3 (Months 5–6) — Guarded Autopilot, GRPO & Generative Creative:** Enable Level 3 Autopilot execution engine, launch visual generative pipeline with automated VLM QA gates, perform offline reinforcement learning via GRPO, finalize audit logging infrastructure, and open General Availability (GA).
