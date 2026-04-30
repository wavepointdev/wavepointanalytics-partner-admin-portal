# CHANGELOG entry — append to existing /CHANGELOG.md

The following entry should be added to the top of the existing `CHANGELOG.md` in the `wavepointanalytics-partner-admin-portal` repo.

---

## [0.2.0] — 2026-04-30

PRD scoping session. No code changes; this version delivers the canonical Product Requirements Document that supersedes the v0.1 architecture brief and locks every decision needed to begin Phase A foundation work.

### Added — PRD

- **`docs/Partner-Admin-Portal-PRD-v0.1.md`** — comprehensive PRD covering 21 sections: roles & access, partner/affiliate models, data architecture, widget library, design tokens, mockup deliverable plan, payouts, promo codes, QR codes, statements, audit log, monitoring, onboarding, co-branding & segmentation, support tickets, operational features, environments, forward-looking unification, and open items requiring board ratification.
- Document includes LOCKED / PROVISIONAL / REVISIT markings on every decision for traceability.

### Decisions locked (15 topics)

- **Roles model:** four-role system (admin / partner with owner-or-member sub-roles / affiliate / user) with multi-role stacking allowed except admin/partner-affiliate exclusion. User role implicit on every account.
- **Partner vs Affiliate differentiation:** feature matrix locked; partners have multi-user teams, promo codes, free seats, custom commission, full email visibility post-paid; affiliates are solo, flat rate (15% provisional), masked emails always.
- **Data architecture:** two separate tables (`partner_orgs`, `affiliates`) with shared SQL view for cross-cutting queries. Multiple Owners per partner_org allowed. Owner-approves-only team management with cap of 5.
- **Frontend stack:** React + Vite + Tailwind + shadcn/ui + ECharts + TanStack Router + TanStack Query. TypeScript throughout. Storybook for visual review. WCAG 2.1 AA accessibility floor from Day 1.
- **Widget architecture:** component library approach (no runtime config). Hybrid data access (presentation widgets receive props; feature widgets self-fetch). Base `WidgetShell` provides loading/error/empty states for free.
- **Design tokens:** single `design-tokens.json` source of truth consumed by Tailwind, shadcn/ui, ECharts, and asset paths. Light/dark both built from Day 1 with user toggle in top nav.
- **Mockup delivery:** phased build to `portal-preview.wavepointanalytics.com` behind Cloudflare Access. Partner Portal first, then Affiliate, then Admin, then Customer Portal nav refresh, then polish. Stub data layer mimics future Supabase client interface.
- **Payouts:** Manual ACH from business bank for v1. Per-partner stored payout method. 30-day clawback window (provisional). $25 minimum payout. USD-only display.
- **Promo codes:** uniqueness among active codes only with 12-month cooldown after deactivation. Globally case-insensitive, alphanumeric 4-20 chars, uppercase canonical. Admin-managed reserved word list. Partner self-deactivation allowed; concurrent-slot counter refills immediately.
- **QR codes:** self-hosted via npm `qrcode` library on Cloudflare Worker, both SVG and PNG (1024×1024) outputs, logo-centered with white circle background, error correction H, stored in R2 with public read URLs. Admin-only regeneration.
- **Statements:** monthly only, HTML email + PDF attachment via `@react-pdf/renderer`, opt-in default OFF, past statements always downloadable from portal.
- **Audit log:** Postgres table with RLS-enforced append-only, dual write paths (app-level + DB triggers), reason text required for sensitive actions only. Database retention indefinite, backup retention 7 years. Supabase Pro daily + Cloudflare Worker weekly export to R2.
- **Monitoring:** Uptime Kuma at Yingson Labs for v1 (with documented migration trigger). Cloudflare Worker polls every 60s, caches in Cloudflare D1. Public status page at `status.wavepointanalytics.com` with name-rewrite mapping that hides Yingson Labs origin.
- **Onboarding:** single-page form (no wizard). Magic-link invitation via Resend with 7-day expiry. Slug reserved at form submit. Welcome screen + setup summary on activation. BoldSign for embedded e-sign (provisional).
- **Co-branding & segmentation:** customer-facing co-branding default ON for first 30 days then auto-hide. Partner toggle to disable own co-branding. Internal segmentation via admin filters; marketing automation deferred (Option A only at launch).
- **Support tickets:** in-portal threaded UI with email notifications. 9 categories for partners, 7 for affiliates. Unassigned queue model. No SLA shown to user. No internal notes or attachments in v1. 30-day reopen window.
- **Operational features:** admin impersonation with 60-min time-limited sessions, mandatory banner, hard restrictions on destructive actions during impersonation. Suspension state distinct from deactivation. Risk alerts page for chargebacks/account sharing/coupon velocity/etc.
- **Environments:** Preview → Production at launch (staging deferred). Multi-environment readiness baked into Phase A foundation so staging is later a config flip.

### Architectural pivots vs v0.1

- **Stripe-only payouts replaces Rewardful-with-Wise approach** from v0.1 — manual ACH at current scale, Wise removed entirely.
- **In-portal tickets with email notifications replaces v0.1 implicit assumption** — tickets are now a fully designed feature with schema, UX, and lifecycle.
- **Multi-Owner partner_org model replaces single-Owner** — multiple Owners can co-administer with equivalent permissions.
- **Owner-approves-only team management replaces admin-gated** — partners self-serve up to a 5-member cap, admin gate only above cap.
- **Status page architecture replaces direct iframe** — Cloudflare Worker caches Kuma data in D1 every 60s; public page reads from D1, never from Yingson Labs directly.
- **Co-branding feature added** — Possibility 1+2+3 from the conversation (attribution + customer-facing acknowledgment + admin segmentation), with 30-day auto-hide window.
- **Admin impersonation feature added** — full spec for "View as user" support tooling.
- **Suspension state added** — distinct from deactivation; freezes partner access while preserving data for investigation.
- **All e-signing moves to embedded provider (BoldSign provisional)** — replaces v0.1 implicit assumption of out-of-band signature.

### Pending / Provisional / Revisit

The PRD's §21 consolidates all open items requiring board ratification, legal counsel review, or designed-to-be-revisited decisions. Notable items:

- Refund clawback window (30 days, provisional)
- Commission rates (provisional pending Derek/George)
- Affiliate DPA requirement (conservative; may relax after counsel)
- E-sign provider (BoldSign default; Jason still considering)
- Email delivery service (Resend default; Jason still considering)
- Customer co-branding window length (30 days, open question)

### Notes

- This PRD is intended as the canonical reference for Phase A onwards.
- Future updates should increment the version number (v0.2, v0.3, etc.) and add a CHANGELOG entry describing what changed and why.
- The PRD does not replace the existing v0.1 legal documents (Privacy Policy, ToS, Partner DPA), which still need counsel review per their flagged notes.
