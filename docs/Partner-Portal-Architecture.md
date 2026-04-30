# WavePoint Portal Architecture — Board Briefing

**Prepared by:** Jason
**Date:** April 19, 2026
**Status:** Draft for Board Discussion
**Supporting Materials:** Partner Portal mockup · Admin Portal mockup · Privacy Policy v0.1 (draft)

---

## Purpose

This document summarizes the scope, decisions, assumptions, and open questions that emerged from a design session on the WavePoint back-end portals. It is intended to frame discussion at the next board meeting so we can confirm direction, resolve open questions, and align on priorities before building begins.

Two mockups accompany this brief: a **Partner Portal** dashboard showing how our B2B content-creator partners will interact with WavePoint, and an **Admin Portal** dashboard showing how the founders will run the business. Both are static mockups with annotated "design notes" panels that explain every design decision.

---

## Executive Summary

We are building a two-portal system on top of our existing customer portal:

1. **Partner Portal** (`partners.wavepointanalytics.com`) — a multi-tenant portal supporting up to ~10 B2B content-creator partners, each with up to ~3 team members. Partners see conversions, commissions, marketing assets, and can self-serve coupon codes and free access seats for their communities.

2. **Admin Portal** (`admin.wavepointanalytics.com`) — an internal business-intelligence and operations view for the three founders. Covers MRR/churn/conversion analytics, partner management, risk detection, manual overrides, and an immutable audit log.

Both portals share a single codebase, single deploy, and single design system consistent with the marketing site and customer portal. A Cloudflare Worker routes traffic to the right portal based on subdomain. We are targeting ~5,000 end-user customers as the design capacity.

**Recommended approach: buy + build hybrid.** Use Rewardful ($49–$149/month) as the commission and payout engine; build everything else custom (partner UX, admin BI, risk detection, audit log, toolkit delivery). Total ongoing SaaS cost at scale is estimated at under $400/month.

---

## 1. Architecture Decisions — Confirmed

These items are settled and will drive the build.

| # | Decision | Notes |
|---|---|---|
| 1.1 | **Deployment topology:** subdomains (`app`, `partners`, `admin`) on a single codebase, Cloudflare Worker dispatches based on hostname | Simplest to operate, one deploy, clean URLs |
| 1.2 | **Partner program engine:** Rewardful (SaaS), paid monthly | Starter $49/mo up to $7.5k affiliate rev, Growth $99/mo up to $15k, Enterprise $149+ beyond |
| 1.3 | **Custom-built layer:** partner portal UI, admin portal, toolkit delivery, risk detection, audit log, support tickets, team management | Everything Rewardful doesn't cover |
| 1.4 | **Data sources of truth:** Stripe (revenue, billing, tax) · Supabase (identity, platform IDs, overrides, audit) · Rewardful (attribution, commissions) | No Xero or Airbyte in scope |
| 1.5 | **Accounting integration:** Stripe's native Xero integration when/if needed; no custom pipeline | Defer until volumes justify it |
| 1.6 | **Charting library:** ECharts (free, premium aesthetic, financial-industry quality) | Replaces Chart.js for ultra-premium feel |
| 1.7 | **Platform support:** NinjaTrader first (all initial products); TradingView as a future phase | Data model supports both from Day 1 |
| 1.8 | **Payout cadence:** monthly | Simplicity, predictability for partners |
| 1.9 | **Data refresh cadence:** hourly (or less frequent if it reduces cost meaningfully) | No real-time infrastructure needed |
| 1.10 | **Geographic scope:** worldwide from Day 1, with GDPR/EU compliance built in | Privacy policy + consent banner + DPA templates required |
| 1.11 | **Partner archetype focus:** content creators and educators | Brokers and strategic partners are future considerations |
| 1.12 | **Commission structure:** flat percentage of billed net revenue (after Stripe fees) on paid conversions only | With structural support for one-time finder's fees on future products |
| 1.13 | **Attribution model:** Rewardful's cookie + coupon-code hybrid | 60-day default cookie window |
| 1.14 | **Email anonymization for partners:** anonymized for trial/visit activity; revealed once the user becomes a paying subscriber | Users can opt out of partner visibility entirely |

---

## 2. Partner Portal — Requirements

### 2.1 Functional Scope

The partner portal is the primary interface for our B2B partners to track their performance with WavePoint. Partners are content creators who drive referral traffic to us through their audience.

**Core features:**

- **Dashboard** with money-first KPIs (lifetime earnings, available payout, refund hold, paid conversions)
- **Conversion funnel** visualization (clicks → trials → paid) with conversion rates at each stage
- **Earnings trend chart** showing paid-out vs. pending commissions over time
- **Product performance chart** showing conversions broken out by indicator SKU
- **Recent referrals table** with anonymization rules applied
- **Upcoming payout card** with approval/clawback/adjustment breakdown (transparency drives trust)
- **Marketing toolkit** — branded assets, copy, B-roll, disclosure templates (served from Cloudflare R2)
- **Support ticket system** scoped to partner
- **Referral link + coupon code** self-service management
- **Free access seat** management (grant, revoke, track usage)
- **Team management** (Owner-only)

### 2.2 Role Structure — Confirmed

Two roles per partner account:

| Role | Capabilities |
|---|---|
| **Owner** | Full access. Sees commissions, payouts, and financials. Manages team members. Manages payout settings. Can create coupons and grant free seats. One per partner account. |
| **Team Member** | Sees funnel, conversions, anonymized user list, toolkit, and support tickets. **Does not see commission dollar amounts, payout history, or payment settings.** Cannot manage team or create/revoke coupons and seats. |

Rationale for starting with two roles: finer granularity (separate Marketing, Support, Analytics roles) adds complexity without clear benefit at this partner scale. We can split further when partners specifically ask for it.

### 2.3 Privacy Rules for Partner Visibility

- **Before a user becomes a paying subscriber:** partner sees anonymized email (`j***@g***.com`), subscription tier if any, date of click/trial/conversion events. No NinjaTrader Machine ID, TradingView username, IP, payment info, or usage data.
- **After a user becomes a paying subscriber:** partner sees full email address to enable legitimate customer-relationship and commission-audit purposes. Still no platform IDs, IPs, payment info, or usage data.
- **Users can opt out** of partner visibility at any time without affecting their subscription. Opted-out users disappear from partner view.

Partners agree to a Data Processing Agreement (DPA) as part of their partner contract, restricting how they may use this data.

### 2.4 Coupon Codes — Partner Self-Service with Guardrails

Partners can create and deactivate their own promo codes inside the portal. Guardrails are admin-configurable **per partner**:

| Guardrail | Default | Admin-configurable? |
|---|---|---|
| Max discount percentage | 25% | Yes, per partner |
| Maximum uses per code | 50 | Yes, per partner |
| Code expiration | 90 days from creation | Yes, per partner |
| Concurrent active codes | 3 | Yes, per partner |
| Code format | 4–20 chars, alphanumeric | Fixed |

A deactivated code does not refund to the partner's quota (prevents create/deactivate abuse). All create/deactivate events sync to Rewardful and are logged to the audit trail.

### 2.5 Free Access Seats — New Concept

Each partner gets a quota of "free access seats" they can grant to named emails for community demos, giveaways, reviewer accounts, or testimonial seeds. A seat provides full indicator access without creating a Stripe customer.

| Seat Control | Default | Admin-configurable? |
|---|---|---|
| Seats per partner | 5 | Yes, per partner |
| Auto-expire seats | Off (permanent until revoked) | Yes — can be set to 30/60/90 days |
| Revoke refills quota | Yes | Fixed |
| Grant/revoke logs to audit | Yes | Fixed |

Each seat grant captures recipient email, product assigned, and a free-text "purpose" field for the partner's own records.

### 2.6 Custom Commission Terms

While the standard offering is a flat percentage of net revenue, the admin portal's partner detail page supports custom arrangements on a per-partner basis:

- **Percentage commissions** at any rate the admin sets (default 30%)
- **Alternative structures** such as "first two months free for the referrer" instead of a percentage
- **One-time finder's fees** for future products where we incentivize partners for single referrals
- **Commission duration** — for-life, first-year-only, first-N-months, etc.
- **Per-product commission rates** — different rates for VWAP Wave vs. EMA Ribbons, etc.

These customizations live in the partner record and are surfaced to the partner transparently in their portal (so they never wonder why their commission looks different from someone else's).

---

## 3. Admin Portal — Requirements

### 3.1 Functional Scope

The admin portal is the founders' business-intelligence and operations view. It has nine functional areas:

| Section | Purpose |
|---|---|
| **Dashboard** | Executive overview: system health, hero KPIs, MRR growth, revenue recognition, trial funnel, product mix, partner leaderboard, geographic distribution, risk alerts, audit log |
| **Customers** | Search, filter, and drill into any customer account; full lifecycle view and manual override tools |
| **Revenue** | MRR waterfall, cohort retention heatmap, recognition detail, plan-tier and per-product breakdowns |
| **Products** | SKU catalog, adoption per product, pricing experiments, platform delivery status |
| **Partners** | Full partner list, onboarding queue, per-partner profile + performance + custom settings, payout approvals |
| **Risk** | Alert queue, chargeback workflow, account-sharing investigation, machine-ID reset history |
| **Operations** | Webhook delivery log, sync job history, content access log, feature flags |
| **Audit Log** | Full immutable audit log with filters, exportable for compliance |
| **Settings** | Admin roles, global thresholds, tax settings, email templates |

### 3.2 System Status Strip — Critical Infrastructure Monitoring

A dedicated strip on the admin dashboard shows four traffic lights for the infrastructure paths that cost the business money silently if they break:

1. **Stripe Webhooks** — if these fail, paid customers don't get access or canceled customers keep access
2. **Rewardful Sync** — if this breaks, commissions are miscalculated
3. **NinjaTrader Delivery** — if this fails, paying customers have no product
4. **Open Risk Alerts** — unreviewed fraud signals

Red or yellow states should page founders via email and Discord. These are not dashboard curiosities — they are P1 operational incidents.

### 3.3 Risk Signal Categories — Four Flavors

| Category | What it watches for | Typical response |
|---|---|---|
| Chargebacks & disputes | Stripe dispute notifications | Submit evidence within Stripe's window (7–21 days) |
| Account sharing | Multi-IP, multi-country, rapid-session patterns | Human review before any suspension |
| Machine-ID reset abuse | Users exceeding the 1-per-30-day reset limit | Investigate; may indicate shared account or legitimate support need |
| Coupon velocity anomaly | A code used at 10×+ historical rate, suggesting public leak | Auto-suspend code; notify partner |

Thresholds for each category are admin-configurable. The system surfaces signals to humans rather than auto-suspending — wrongly suspending a paying customer is worse than letting an actual abuser run one more day while a human reviews.

### 3.4 Manual Override Toolkit

Available from customer and partner detail pages. All actions require a reason string and are logged to the immutable audit trail.

- Grant lifetime access
- Extend trial
- Force machine-ID reset (override the 1-per-30-day limit)
- Issue refund (triggers Stripe refund + automatic commission clawback)
- Adjust partner commission (retroactive correction, signing bonus, clawback waiver)
- Bump partner coupon/seat quotas
- Suspend partner
- Revoke free seat (cross-partner)
- Resend webhook (replay Stripe event if sync broke)

### 3.5 Revenue Recognition

The admin dashboard deliberately surfaces the gap between **Booked MRR** (what Stripe shows gross) and **Net MRR** (booked minus refund liability minus chargeback reserves). This prevents the common early-SaaS surprise of "we thought we were bigger than we were" when refunds settle.

### 3.6 Audit Log

Every admin action that affects a customer, partner, or financial record is logged with actor, action, target, reason, and timestamp. **The log is append-only** — even admins cannot delete or edit entries (enforced at the database RLS level). This protects against compromised admin accounts, internal disputes over "who made that change," and external compliance requests.

---

## 4. Additional Features Added Beyond Original Scope

During design, the following features were identified as either critical additions or quick wins. All are in scope for Phase 1 unless noted.

| Feature | Source | Rationale |
|---|---|---|
| **Immutable audit log** | Added in review | Protects us against compromised accounts, internal disputes, compliance requests. Cheap to add now, painful to retrofit. |
| **Webhook health monitoring** | Added in review | Silent webhook failures = lost revenue and broken customer experiences. A must-have for peace of mind. |
| **Soft-launch toggle per partner** | Added in review | `is_active` boolean with expiry — pause a partner's tracking/commission without deleting them |
| **Content access log** | Added in review | Every indicator grant/revoke event with trigger reason. Required for account-sharing detection and support investigations. |
| **Partner-side support ticket queue** | Added in review | Better than an email thread. Partners' top question ("why didn't this referral credit?") gets a dedicated channel. |
| **Revenue recognition mode** | Added in review | Booked vs. net-of-refund-window. Prevents founder misreading of the numbers. |
| **Cohort retention view** | Added in review | LTV is the lagging indicator; cohort retention is the leading one. |
| **Free Access Seats** | Added in review | Community demos, giveaways, reviewer accounts, testimonial seeds. |
| **Coupon velocity anomaly detection** | Added in review | Catches public leaks of partner coupon codes before they blow through the uses-per-code limit. |

---

## 5. Key Assumptions

These are working assumptions the design is built on. Any change invalidates parts of the design and should be explicitly addressed at the board meeting.

| # | Assumption | Impact if changed |
|---|---|---|
| 5.1 | Target scale: ~5,000 end-user customers, ~10 B2B partners with up to ~3 team members each | Infrastructure sizing, feature complexity, buy-vs-build tipping point |
| 5.2 | All partners are content creators / educators (not brokers, not strategic resellers) | Role structure, commission model, portal feature emphasis |
| 5.3 | Trial structure: 2-week free trial, self-service, partner commission paid only on paid conversions | Funnel design, commission logic |
| 5.4 | Partners will accept Rewardful as the commission engine of record | Buy-vs-build tradeoff; if not, we build in-house (4–8 weeks dev) |
| 5.5 | 30-day refund clawback window on partner commissions | Must be confirmed or set — affects every partner relationship |
| 5.6 | All three founders share equal admin access (no hierarchy) | Role modeling; needs reconsideration when team grows |
| 5.7 | Hourly data refresh is acceptable; no partner expects real-time numbers | Infrastructure cost, complexity |
| 5.8 | NinjaTrader-only Phase 1 delivery; TradingView is Phase 2 | Data model supports both; architecture ready |
| 5.9 | Indicator delivery starts manual, becomes programmatic in a later phase | System status strip anticipates the programmatic model |
| 5.10 | Marketing, partners, and customer portals use the same design language and brand colors | Consistency drives "ultra-premium" feel |

---

## 6. Open Questions for the Board

These need explicit decisions, ordered by priority. Most affect contract language or customer-facing commitments, so changing them later creates friction.

### 6.1 Commission & Partner Economics — HIGH PRIORITY

| # | Question | Why it matters |
|---|---|---|
| 6.1.1 | **Refund clawback window: 30, 45, or 60 days?** | Once set and shown to partners, retroactive changes create significant trust friction |
| 6.1.2 | **Minimum payout threshold: none, $25, $50, $100?** | Balance admin overhead vs. partner frustration at small balances being held |
| 6.1.3 | **Default commission rate: 30% standard?** | Sets expectations for all partner negotiations |
| 6.1.4 | **Should we show the commission rate prominently in the portal** ("30% for life"), or keep it contractual-only? | Transparency vs. flexibility to negotiate custom deals |
| 6.1.5 | **Finder's-fee structure for future products** — what does a one-time payout look like (flat $? % of first payment?) | Affects data model now even if not used yet |

### 6.2 Partner Self-Service Controls — MEDIUM PRIORITY

| # | Question | Why it matters |
|---|---|---|
| 6.2.1 | **Default coupon max discount: 25%?** | Protects margin; too restrictive limits partner creativity |
| 6.2.2 | **Default free seat quota: 5 per partner?** | Balance community-demo use cases against real cost per seat |
| 6.2.3 | **Should free seats auto-expire?** (e.g., 30/60/90 days) | Permanent seats accumulate over time; auto-expiry forces intentional renewals |
| 6.2.4 | **Anonymization for refunded users: keep anonymized or reveal?** | Partner takes the clawback — do they deserve full visibility? Privacy tradeoff |
| 6.2.5 | **Live activity feed for partners** ("Someone just converted!"): include or exclude? | Some partners love it, some find it noisy and anxiety-inducing |

### 6.3 Admin Operations — MEDIUM PRIORITY

| # | Question | Why it matters |
|---|---|---|
| 6.3.1 | **Future admin role tiers** — what does "Support Agent" see? "Analyst"? "CS Manager"? | Define now so RLS policies are built with future expansion in mind |
| 6.3.2 | **Lifetime access ceiling** — company-wide cap on free lifetime grants? | Prevents accumulation over years; enforces intentionality |
| 6.3.3 | **Chargeback handling SLA** — who owns responding to disputes within Stripe's 7–21 day window? | Missed deadlines = automatic loss |
| 6.3.4 | **Founder alert routing** — who gets paged for which alert class? | Avoid everyone ignoring everything; give each founder clear ownership |
| 6.3.5 | **Risk alert thresholds** — how many IPs in how many hours triggers account-sharing flag? | Too tight = noise; too loose = missed fraud |

### 6.4 Legal, Privacy, and Compliance — HIGH PRIORITY (Timeline-Dependent)

| # | Question | Why it matters |
|---|---|---|
| 6.4.1 | **Engage privacy counsel for Privacy Policy + ToS review** — budget and timeline | Cannot launch to EU/UK without this done |
| 6.4.2 | **EU representative (GDPR Article 27)** — approve ~$100–$400/year service | Required unless "occasional processing" exception clearly applies |
| 6.4.3 | **Partner Data Processing Agreement template** — counsel-drafted | Cannot onboard partners to the portal without this signed |
| 6.4.4 | **Does WavePoint want to preserve a broader right to use aggregated usage data for marketing?** (e.g., "85% of Pro users trade ES") | Affects the Privacy Policy language on usage data |
| 6.4.5 | **Terms of Service** — separate draft needed covering refunds, dispute resolution, arbitration, governing law | Parallel track to Privacy Policy |

### 6.5 Infrastructure & Tooling — LOWER PRIORITY

| # | Question | Why it matters |
|---|---|---|
| 6.5.1 | **Validate Rewardful fits our needs** — spend an hour in the trial before committing | Small chance their data model doesn't fit our finder's-fee requirement |
| 6.5.2 | **External BI tool** (e.g., Metabase) integration later, or keep all BI in-portal? | Not urgent; depends on how much ad-hoc analysis the founders do |
| 6.5.3 | **Discord integration for system alerts** — which channel, what detail level? | Nice-to-have; email is sufficient initially |

---

## 7. Cost Summary (Estimated, Monthly)

| Item | Cost | Scale |
|---|---|---|
| Rewardful | $49–$149/mo | Grows with affiliate revenue |
| Supabase Pro | ~$25/mo | Single flat fee sufficient at this scale |
| Cloudflare (Workers, Pages, R2, DNS) | $0–$20/mo | Free tier covers most use; small overage possible |
| Cloudflare Access (if used) | $0 | Free for small team |
| EU/UK GDPR representative | $10–$35/mo | Amortized annual cost |
| Cookie consent tool | $0–$20/mo | Self-hosted free options exist |
| Email delivery (transactional/marketing) | $10–$30/mo | Depends on provider |
| ECharts, Web3Forms, Rewardful Wise payouts | $0 | Free / included |
| **Total ongoing SaaS baseline** | **~$100–$280/mo** | At launch |
| **Total ongoing SaaS at scale** | **~$250–$400/mo** | Past 1,000 paying customers |

Legal fees (Privacy Policy, ToS, partner DPA template) are one-time (or periodic) costs and should be budgeted separately.

---

## 8. Build Phasing

Suggested phase split for the build itself, assuming board approval of direction.

### Phase 1 — Foundation (4–6 weeks)

- Extend existing customer portal codebase
- Set up Rewardful, connect to Stripe
- Build admin portal MVP: Dashboard, Customers, Partners, Audit Log
- Build partner portal MVP: Dashboard, Conversions, Toolkit, Support
- Cloudflare Worker subdomain routing
- Cookie consent banner (EU/UK)

### Phase 2 — Operational Depth (3–4 weeks)

- Coupons and free seats for partners
- Admin risk detection signals and review workflow
- Revenue recognition view
- Manual override toolkit (full set)
- Partner team management

### Phase 3 — Polish and Expansion (ongoing)

- Cohort retention analytics
- Programmatic NinjaTrader delivery
- TradingView support when products launch
- Customer data export (GDPR Article 20)
- Per-partner custom commission structures

---

## 9. Artifacts Produced from This Session

1. **Partner Portal Dashboard mockup** (Owner view) — `partner-portal-dashboard-mockup.html`
2. **Admin Portal Dashboard mockup** (Founder view) — `admin-portal-dashboard-mockup.html`
3. **Privacy Policy v0.1 draft** (pre-legal-review) — `WavePoint-Privacy-Policy-v0.1.md`
4. **This board briefing** — `WavePoint-Board-Briefing-Portal-Architecture.md`

All four are intended to be reviewed together. Each mockup includes an on-page "Mockup notes" panel with design rationale and open questions specific to that portal.

---

## 10. Recommended Board Discussion Agenda

Suggested ~60-minute agenda for the next board meeting:

1. **Walkthrough of mockups** (15 min) — all three founders view both portals and the notes panels
2. **Confirm Section 1 architecture decisions** (5 min) — quick go/no-go on each item
3. **Work through Section 6.1 commission economics** (15 min) — this is the most consequential block
4. **Work through Section 6.4 legal/privacy** (10 min) — timeline and budget commitment
5. **Work through Section 6.2–6.3 operational questions** (10 min) — set defaults that can be tuned later
6. **Phasing and timeline** (5 min) — when does Phase 1 start?

Any question not resolved in the meeting can be parked in a follow-up doc with an owner assigned.

---

*Prepared for internal WavePoint Analytics board discussion. Not for external distribution.*
