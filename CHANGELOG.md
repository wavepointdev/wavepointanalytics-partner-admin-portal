# CHANGELOG — Partner & Admin Portal Project

All notable changes to the `wavepointanalytics-partner-admin-portal` project will be documented in this file.

The format follows [Keep a Changelog](https://keepachangelog.com/), and this project adheres to semantic versioning for major artifacts.

---

## [Unreleased]

*Items being worked on but not yet ratified by the board.*

### Pending

- Board review of Partner Portal and Admin Portal mockups
- Board decisions on items listed in `docs/Partner-Portal-Architecture.md` §6
- Legal review of Privacy Policy v0.1, Terms of Service v0.1, and Partner DPA Template v0.1
- Partner Agreement draft (commercial terms document — separate from DPA)
- Admin Portal secondary screen mockups (Customer Detail, Partner Detail with custom commission/coupon/seat configuration)
- Partner Portal Team Member view mockup variant
- Cookie consent banner implementation spec
- `CLAUDE.md` update for this repo

---

## [0.1.0] — 2026-04-19

Initial planning and mockup phase. No code committed yet — all artifacts are documents and design mockups intended for board discussion.

### Added — Architecture & Decisions

- Established that this project will run on a hybrid **buy + build** model: Rewardful as the commission engine, custom-built portals for everything else
- Confirmed deployment topology: subdomains (`partners.wavepointanalytics.com`, `admin.wavepointanalytics.com`) on a single codebase with Cloudflare Worker hostname dispatch
- Set target scale: ~5,000 end-user customers, ~10 B2B content-creator partners with up to ~3 team members each
- Confirmed data sources of truth: Stripe (revenue, billing), Supabase (identity, audit, overrides), Rewardful (attribution, commissions), Cloudflare R2 (marketing assets)
- Selected **ECharts** (replacing Chart.js from Gemini's original spec) for premium financial-dashboard aesthetics
- Confirmed worldwide / GDPR-ready scope from Day 1
- Confirmed NinjaTrader as initial platform with TradingView as future Phase 2 (data model supports both from start)
- Hourly data refresh cadence (no real-time infrastructure required)
- Monthly partner payout cadence
- Two-role partner team model: **Owner** + **Team Member**
- Three-founder admin model with future role expansion in mind (RLS policies built to accommodate)

### Added — Documents

- **`README.md`** — repo orientation document with architecture diagram, tech stack, repo structure, and backup notes
- **`docs/Partner-Portal-Architecture.md`** — board briefing covering all decisions, requirements, assumptions, and open questions
  - *Originally created as `WavePoint-Board-Briefing-Portal-Architecture.md`, renamed by Jason to `Partner-Portal-Architecture.md` for repo placement*
- **`docs/Privacy-Policy-v0.1.md`** *(working name `WavePoint-Privacy-Policy-v0.1.md`)* — first draft covering US + GDPR + UK GDPR + CCPA/CPRA, with 15 lawyer-review notes flagged
- **`docs/Terms-of-Service-v0.1.md`** *(working name `WavePoint-Terms-of-Service-v0.1.md`)* — first draft covering subscription, refunds, IP, acceptable use, dispute resolution, with 25 lawyer-review notes flagged
- **`docs/Partner-DPA-Template-v0.1.md`** *(working name `WavePoint-Partner-DPA-Template-v0.1.md`)* — first draft as Controller-to-Controller DPA template for content-creator partners, with 24 lawyer-review notes flagged

### Added — Mockups

- **`mockups/partner-portal-dashboard-mockup.html`** — Owner view of Partner Portal dashboard with:
  - 4-card KPI row (Lifetime Earned, Available to Pay Out, In Refund Hold, Paid Conversions MTD)
  - Conversion funnel (Clicks → Trials → Paid) with stage-rate annotations
  - Upcoming Payout card with approval/clawback/adjustment math
  - 6-month earnings trend (ECharts area chart, paid-out vs. pending)
  - Performance by Product chart (ECharts horizontal bar by SKU)
  - Recent referrals table demonstrating anonymization rules (anonymized for trial/visit, full email after paid)
  - Referral link with copy-to-clipboard, "Manage coupon codes" CTA
  - Marketing toolkit preview (3 latest assets)
  - Team panel (Owner-only with green accent)
  - **Coupon management section** with admin-configurable guardrails (25% max discount, 90-day expiry, 50 uses, 3 concurrent active)
  - **Free Access Seats section** with quota visualization, grant/revoke actions
  - Coupon creation modal with discount slider and guardrail enforcement
  - Free seat grant modal with quota math
  - Annotated "Mockup notes" panel with 9 numbered design notes, role-difference documentation, and 8 open questions for board

- **`mockups/admin-portal-dashboard-mockup.html`** — Founders view of Admin Portal dashboard with:
  - Orange visual distinction (banner, badge, accents) marking privileged-access surface
  - System Status strip with 4 traffic lights (Stripe webhooks, Rewardful sync, NinjaTrader delivery, Risk alerts)
  - 4-card hero KPI row (MRR, Active Subscribers, Active Trials, Monthly Churn)
  - 6-month MRR growth chart (ECharts stacked area by plan tier: Standard, Pro, Terminal)
  - Revenue Recognition card showing Booked vs. Net MRR with refund liability and chargeback reserves
  - All-business trial funnel (Visits → Signups → Trials → Paid)
  - Product Mix donut chart (NinjaTrader subscribers by SKU) with TradingView Phase 2 placeholder
  - Partner Leaderboard (top 5 by performance, internal-only)
  - Geographic Top 5 table (replacing Gemini's heatmap concept)
  - Risk Alerts panel with 4 categories (chargebacks, account sharing, machine-ID resets, coupon velocity)
  - Audit Log feed showing all admin actions with reason strings
  - Annotated "Mockup notes" panel with 9 numbered design notes, manual-override toolkit documentation, and 8 open questions for board

### Added — Features Identified Beyond Original Scope

These were added to the Phase 1 scope during design review, beyond the original Gemini-prompted feature list:

- Immutable audit log
- Webhook health monitoring with paging
- Soft-launch toggle per partner (`is_active` with expiry)
- Content access log for indicator delivery
- Partner-side support ticket queue
- Revenue recognition view (booked vs. net MRR)
- Cohort retention view
- Free Access Seats program (admin-configurable per partner)
- Coupon velocity anomaly detection
- Custom commission terms per partner (percentage, time-based offers, finder's fees, per-product rates)

### Changed (vs. original Gemini brief)

- Product scope corrected from "Stocks, Options, Crypto" to **futures-only**, replacing Gemini's incorrect framing
- "Niche heatmap" feature replaced with **product-SKU adoption view** — meaningful for a futures-only product where SKU is the relevant slicing dimension
- "Stripe Connect" recommendation replaced with **Rewardful as commission engine** — at 10-partner scale, Stripe Connect's KYC/payout complexity isn't justified
- "Xero + Airbyte" pipeline dropped from architecture — Stripe's native Xero integration sufficient at this scale
- "Geographic heatmap" downgraded to top-5 country/state table — at 5,000 users a heatmap shows population centers, not actionable signal
- "Chart.js-style visualizations" replaced with **ECharts** for ultra-premium feel
- "Email anonymization" rule clarified — anonymized for trial/visit, full email post-conversion, with user opt-out

### Notes

- All four primary documents (Architecture brief, Privacy Policy, ToS, Partner DPA) have lawyer-review notes that should be removed before publication and used as a starting checklist for counsel
- Mockup notes panels in HTML are designed to be viewable by other founders without a walkthrough — they document every design decision in-place
- Working filenames in `/mnt/user-data/outputs/` differ from final repo paths (`docs/`); this CHANGELOG uses repo paths going forward

---

## Document Provenance

| Artifact | Working Filename (this session) | Final Repo Path |
|---|---|---|
| README | `README.md` | `/README.md` |
| Architecture Brief | `WavePoint-Board-Briefing-Portal-Architecture.md` | `/docs/Partner-Portal-Architecture.md` |
| Privacy Policy | `WavePoint-Privacy-Policy-v0.1.md` | `/docs/Privacy-Policy-v0.1.md` |
| Terms of Service | `WavePoint-Terms-of-Service-v0.1.md` | `/docs/Terms-of-Service-v0.1.md` |
| Partner DPA Template | `WavePoint-Partner-DPA-Template-v0.1.md` | `/docs/Partner-DPA-Template-v0.1.md` |
| Partner Portal Mockup | `partner-portal-dashboard-mockup.html` | `/mockups/partner-portal-dashboard-mockup.html` |
| Admin Portal Mockup | `admin-portal-dashboard-mockup.html` | `/mockups/admin-portal-dashboard-mockup.html` |
