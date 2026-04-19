# wavepointanalytics-partner-admin-portal

Back-end portals for WavePoint Analytics: a multi-tenant **Partner Portal** for B2B content-creator partners, and an **Admin Portal** for internal operations and business intelligence.

> **Status:** 🟡 Planning & mockups. Not yet implemented. See [`docs/Partner-Portal-Architecture.md`](docs/Partner-Portal-Architecture.md) for the authoritative scope.

---

## What This Repo Is

This repo is the home for two sibling portals that sit alongside the customer portal (`app.wavepointanalytics.com`):

| Portal | Subdomain | Audience | Purpose |
|---|---|---|---|
| **Partner Portal** | `partners.wavepointanalytics.com` | B2B content-creator partners | Track conversions, commissions, marketing toolkit, self-serve coupons & free access seats |
| **Admin Portal** | `admin.wavepointanalytics.com` | WavePoint founders (and future hires) | Executive BI, customer/partner management, risk detection, manual overrides, audit log |

Both portals share a single codebase, single deploy, and single design system. A Cloudflare Worker routes requests to the right portal based on hostname.

---

## Target Scale

Designed for:

- ~5,000 end-user customers
- ~10 B2B content-creator partners
- Up to ~3 team members per partner
- 3 internal admin users (WavePoint founders), with future support for additional hire roles

---

## Architecture at a Glance

```
                ┌─────────────────────────────────────────┐
                │          User's Browser                 │
                └────────────────┬────────────────────────┘
                                 │
              partners.wavepointanalytics.com
              admin.wavepointanalytics.com
                                 │
                ┌────────────────▼────────────────────────┐
                │  Cloudflare Worker (hostname dispatch)  │
                └────────────────┬────────────────────────┘
                                 │
                ┌────────────────▼────────────────────────┐
                │  Cloudflare Pages  (HTML/CSS/JS)        │
                │  Single codebase, subdomain routing     │
                └────────────────┬────────────────────────┘
                                 │
         ┌───────────────────────┼──────────────────────────┐
         │                       │                          │
  ┌──────▼──────┐         ┌──────▼──────┐           ┌───────▼─────┐
  │  Supabase   │         │   Stripe    │           │  Rewardful  │
  │             │         │             │           │             │
  │ Identity,   │         │ Billing,    │           │ Attribution,│
  │ RLS, audit, │◄───────►│ subs, tax,  │◄─────────►│ commissions,│
  │ toolkit meta│webhooks │ webhooks    │ 2-way sync│ payouts     │
  └─────────────┘         └─────────────┘           └─────────────┘
         │
         │ R2 (marketing assets)
         ▼
  ┌─────────────┐
  │ Cloudflare  │
  │     R2      │
  └─────────────┘
```

### Tech Stack

- **Frontend:** HTML + CSS (Tailwind) + vanilla JS — consistent with the marketing site and customer portal
- **Charts:** ECharts (SVG render) — chosen over Chart.js for premium financial-dashboard aesthetics
- **Fonts:** Manrope (display) + Inter (body)
- **Database / Auth:** Supabase (PostgreSQL + Row-Level Security + magic-link/Google OAuth)
- **Billing:** Stripe (Subscriptions + Tax + Customer Portal)
- **Commission engine:** Rewardful (SaaS, ~$49–$149/mo depending on affiliate revenue)
- **Hosting:** Cloudflare Pages + Workers + R2
- **Email:** [TBD — transactional/marketing provider]

### Data Sources of Truth

| Concern | System of Record |
|---|---|
| User identity, TradingView username, NinjaTrader Machine ID | Supabase |
| Partner team memberships, roles, settings | Supabase |
| Toolkit assets & metadata | Cloudflare R2 + Supabase |
| Support tickets | Supabase |
| Audit log | Supabase (append-only via RLS) |
| Subscription status, billing events, tax/geographic data | Stripe |
| Referral attribution, commission balances, payouts | Rewardful |

---

## Repo Structure (planned)

```
/
├── docs/
│   ├── Partner-Portal-Architecture.md      # Authoritative scope doc (board briefing)
│   ├── Privacy-Policy.md                   # Current Privacy Policy (customer-facing)
│   ├── Terms-of-Service.md                 # Current ToS (customer-facing)
│   ├── Partner-DPA-Template.md             # Data Processing Agreement for partners
│   └── ADR/                                # Architecture Decision Records
├── src/
│   ├── partner/                            # Partner Portal pages
│   ├── admin/                              # Admin Portal pages
│   ├── shared/                             # Shared components, styles, utilities
│   └── worker/                             # Cloudflare Worker (hostname dispatch + API)
├── mockups/
│   ├── partner-portal-dashboard-mockup.html
│   └── admin-portal-dashboard-mockup.html
├── supabase/
│   ├── migrations/                         # SQL migrations
│   └── seed/                               # Local dev seed data
├── CLAUDE.md                               # AI session context (machine-readable)
├── README.md                               # You are here
└── .gitignore
```

---

## Development Status

### Completed

- ✅ Partner Portal dashboard mockup (Owner view) with annotated design notes
- ✅ Admin Portal dashboard mockup with annotated design notes
- ✅ Board briefing doc (`Partner-Portal-Architecture.md`) with decisions, assumptions, open questions
- ✅ Privacy Policy v0.1 draft (pending legal review)
- ✅ Decision: buy Rewardful + custom-build everything else
- ✅ Decision: subdomain topology with Cloudflare Worker dispatch

### In Progress

- 🟡 Terms of Service draft (pending legal review)
- 🟡 Partner DPA template (pending legal review)

### Pending Board Decisions

See `docs/Partner-Portal-Architecture.md` §6 for the full list. Highest priority:

- Refund clawback window (30 / 45 / 60 days)
- Default commission rate & whether to display prominently
- Coupon max discount ceiling
- Free access seat quota defaults & auto-expiry policy
- Future admin role tiers
- Legal review timeline & budget

### Not Yet Started

- Admin Portal mockups for secondary screens (Customer Detail, Partner Detail, etc.)
- Partner Portal Team Member view mockup
- Actual implementation (Phase 1 build)
- Supabase schema & migrations
- Cloudflare Worker
- Rewardful integration
- Cookie consent banner (required before EU/UK launch)

---

## Related WavePoint Repos

| Repo | Purpose |
|---|---|
| `wavepointanalytics.com` | Marketing site (public-facing) |
| `wavepointanalytics-portal` | Customer portal (`app.wavepointanalytics.com`) |
| **`wavepointanalytics-partner-admin-portal`** | **This repo — partner & admin portals** |

All three share the same design language, color tokens, and typography.

---

## Deployment

*Not yet deployed.* Planned flow:

1. Push to `main` on GitHub
2. Gitea pulls and mirrors within 8 hours for backup (`gitea.yingson.com/wavepointdev/`)
3. Cloudflare Pages auto-builds and deploys the static assets
4. Cloudflare Worker serves the two subdomains with hostname-based routing

### Pre-Push Reminders (will apply once built)

- Replace `YOUR_PUBLISHABLE_KEY_HERE` with Supabase anon public key
- Replace `YOUR_REWARDFUL_KEY` placeholder with Rewardful public key
- Do **not** commit Stripe secret keys, Supabase service role keys, Rewardful API keys, or Web3Forms keys
- Service-level secrets live in Cloudflare Workers secrets (`wrangler secret put`)

---

## Backup & Source Control

- **Primary:** GitHub (`github.com/wavepointdev/wavepointanalytics-partner-admin-portal`)
- **Backup mirror:** Gitea at `gitea.yingson.com/wavepointdev/wavepointanalytics-partner-admin-portal`
- **Mirror direction:** GitHub → Gitea (pull mirror, read-only on Gitea side)
- **Mirror frequency:** Every 8 hours (configurable)

> GitHub is the source of truth for this repo. The Gitea mirror is backup-only. Do not commit to Gitea directly.

---

## Security & Compliance

This codebase will handle personally identifiable information, payment metadata, and partner financial records.

- **Row-Level Security (RLS)** enforced on every table touching user or partner data
- **Audit log** append-only (no delete/edit even for admins)
- **GDPR / UK GDPR / CCPA** support built in from Day 1 (consent banner, data export, right to deletion)
- **Partner access** scoped by `partner_id` via RLS; partners cannot see each other's data
- **Cookie consent** required before any non-essential cookies (EU/UK users)

Partners sign a Data Processing Agreement (see `docs/Partner-DPA-Template.md`) before portal access is granted.

---

## Contributing

This repo is maintained by the WavePoint founders. External contributions are not accepted at this time.

For internal workflow:
- `main` is the deploy branch
- Feature work happens on `feature/<short-name>` branches
- Mockup iterations go on `mockup/<topic>` branches
- Commits by AI assistants are prefixed `[claude]`

---

## Questions?

- Technical & architecture questions: Jason
- Partner program & commission questions: Derek
- Brand, marketing, content questions: George

---

*© 2026 WavePoint Analytics LLC. All rights reserved.*
