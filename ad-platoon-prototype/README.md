# Ad Platoon — Prototype

A functional React + Supabase prototype validating the core Copilot approval workflow, autonomy controls, and attribution dashboard — built to test product decisions with real interaction before committing engineering time to production infrastructure.

## What It Does

The prototype simulates the full Ad Platoon user journey for a Commander/Strategist reviewing AI-generated campaign recommendations:

- View live campaign performance across mock Meta/Google/TikTok accounts
- Review AI-generated budget/creative/pause recommendations as approval cards, each with a rationale and confidence score
- Approve, reject, or modify recommendations — writes to a real Supabase backend, not static mock state
- Toggle the 3-tier Autonomy Slider, including the full multi-step Autopilot activation flow (guardrail acknowledgment → co-signature request → confirmation)
- View a append-only audit log of every action taken
- Compare platform-reported vs. server-verified attribution data

## Workflows Validated

| Workflow | What It Proves |
|---|---|
| Recommendation approve/reject | The core trust loop — does a card with rationale + confidence score give a user enough context to act confidently in under 10 seconds? |
| Autopilot activation | Does the multi-step confirmation (guardrail checklist → co-signature → 24h window) feel like appropriate friction, or too much? |
| Attribution comparison view | Does surfacing the platform-vs-verified discrepancy land as the "aha moment" it's designed to be? |
| Creative fatigue alert → replacement flow | Is the generate-and-review loop fast enough to feel like automation rather than a new manual task? |

## Tech Stack

- **Frontend**: React, TypeScript, Tailwind CSS
- **Backend**: Supabase (PostgreSQL + Row-Level Security + Auth + Realtime subscriptions)
- **Charts**: Recharts
- **Hosting**: [Lovable](https://lovable.dev) — built via 7 phased prompts rather than one monolithic spec (see `/docs/lovable-prompts.md`)

## Setup

```bash
git clone <this-repo>
cd ad-platoon-prototype
npm install

# Create a Supabase project, then run the schema:
# (see /supabase/schema.sql for full table definitions + RLS policies)

cp .env.example .env.local
# Add your Supabase URL + anon key to .env.local

npm run dev
```

Seed data for demo campaigns and mock recommendations is in `/supabase/seed.sql` — run it after schema setup and after creating your first user account.

## Screenshots / GIFs to Capture

> Replace these placeholders once the prototype is deployed:

- [ ] **Dashboard overview** — stat cards + Autonomy Slider in default (Copilot) state
- [ ] **Recommendation approval flow** — GIF showing a card being approved, confirmation modal, and the resulting audit log entry appearing in real time
- [ ] **Autopilot activation modal** — the 3-step guardrail checklist → co-signature screen sequence
- [ ] **Attribution comparison chart** — platform-reported vs. server-verified bar chart with the discrepancy callout

## Why I Built It This Way

**Prototype first, production infrastructure second.** Before committing engineering time to real Meta/Google API integration, rate limiting, and a production Kafka event bus, I needed to know whether the *interaction model* actually worked — could a media buyer look at a recommendation card and trust it enough to click approve? That's a design question, not an infrastructure question, so I validated it with mock data on Supabase rather than building against live ad accounts first.

**Phased AI-assisted build, not one prompt.** Lovable (and similar AI builders) degrade in output quality as prompt scope grows. I split the build into 7 sequential phases — shell/auth, campaigns table, recommendation cards, autonomy controls, creative studio, attribution/audit/settings, polish — testing each phase in the live preview before writing the next prompt. This is documented in full because it's a repeatable pattern for scoping AI-assisted builds, not just a one-off decision.

**Real Supabase writes, not static state.** Every approve/reject action actually persists to a database with Row-Level Security, not a `useState` mock. This was deliberate — it meant the prototype could be handed to a real design partner for a working demo, not just clicked through by me.

**Autopilot friction is intentional, not a placeholder.** The multi-step activation flow isn't unfinished polish — the friction itself is the product decision being tested. Full reasoning in [`pm-case-studies/autonomy-framework.md`](../pm-case-studies/autonomy-framework.md).
