# WavePoint Partner & Admin Portal — Product Requirements Document

**Version:** 0.3 (Phase reprioritization for compressed time-to-market)
**Status:** Draft — pending board ratification
**Authors:** Jason Johnson (COO) with structured input gathering via Claude
**Date:** May 3, 2026
**Supersedes:** v0.2 (April 30, 2026 PRD) — which superseded `Partner-Portal-Architecture.md` v0.1 (architecture brief, retained as historical reference)

---

## Changes in v0.3

This revision **restructures the build into five phases** to compress Partner Portal time-to-market. Most non-essential and back-end-only features have been pushed to Phase 3. Locked decisions from v0.2 are preserved; only their **phase placement** has changed unless explicitly noted.

**New in v0.3:**
- §1.5 Build phases and timelines (Phase 1a / 1b / 2 / 3 / Future)
- ⚠️ Launch dependencies callout in executive summary
- §22 Security and operational integrity (full new section)
- Companion `docs/Security-Overview.md`
- Companion `docs/Partner-Onboarding-Runbook.md`
- Companion `docs/reserved-promo-codes.md`

**Material decision changes vs v0.2:**
- Referral URL format: `wavepointanalytics.com/{slug}` (eliminates `/r/` prefix). Implemented via Cloudflare Worker dispatch with reserved slug list.
- E-signature provider: **SignWell** (locked, replaces provisional BoldSign).
- Statements in Phase 1a: manual-only (admin-portal button generates PDF; admin attaches to Gmail). Resend integration + cron + opt-in toggle moves to Phase 3.
- QR codes in Phase 1a: admin manually generates one PNG per partner via qr-code-generator.com and drops in R2. Self-hosted Worker pipeline moves to Phase 3.
- Partner self-service of promo codes: moves to Phase 3 (read-only stats list in Phase 1a; admin creates codes on partner request).
- Partner self-service of free access seats: moves to Phase 3 (read-only display in Phase 1a; admin grants manually).
- In-portal support tickets: moves to Phase 3. Phase 1a uses `support@wavepointanalytics.com` with prominent "Need help?" callout.
- Customer-facing co-branding display: moves to Phase 3. Attribution data field stays in Phase 1a.
- Admin impersonation full UI: moves to Phase 3. Phase 1a allows direct Supabase queries by admin.
- Onboarding wizard: moves to Phase 3. Phase 1a is fully manual, governed by `docs/Partner-Onboarding-Runbook.md`.
- Public status page (`status.wavepointanalytics.com`): moves to Phase 3. Phase 1a is internal Uptime Kuma only.
- Light mode: moves to Phase 3. Phase 1a ships dark-only.
- Multi-user partner team: Phase 1a supports multi-user teams with admin-set roles; partner Owner role-management UI moves to Phase 3.
- Storybook: moves to Phase 2.
- Formal WCAG 2.1 AA audit: moves to Phase 2 (build-to-WCAG patterns remain in Phase 1a).
- Notification preferences UI: minimized in Phase 1a (single statement-related toggle); full preferences page moves to Phase 3.
- Audit log DB triggers + R2 weekly export + searchable UI: moves to Phase 3 (basic app-level audit log + admin table view stays in Phase 1a).
- Risk Alerts page: moves to Phase 2.

**Net effect:** Phase 1a Partner Portal scope is reduced by approximately 40% relative to v0.2's all-up plan, in service of getting partners onboarded against working software faster.

---

## Document purpose

This PRD captures the complete set of requirements, architectural decisions, and design rules for WavePoint's Partner Portal and Admin Portal — including a refresh of the existing Customer Portal navigation. It is the canonical reference for engineering, design, legal review, and partner conversations.

It is intended to be readable cold by anyone joining the project — co-founders, contractors, counsel — without requiring institutional context.

Items marked **PROVISIONAL** are working defaults that need ratification by the founding team before launch. Items marked **REVISIT** are locked for v1 but intentionally flagged for later review under specified triggers.

---

## Table of contents

1. [Executive summary](#1-executive-summary)
   - 1.5 [Build phases and timelines](#15-build-phases-and-timelines)
2. [Glossary and conventions](#2-glossary-and-conventions)
3. [Roles, identity, and access](#3-roles-identity-and-access)
4. [Partner and affiliate models](#4-partner-and-affiliate-models)
5. [Data architecture](#5-data-architecture)
6. [Widget library and frontend architecture](#6-widget-library-and-frontend-architecture)
7. [Design tokens and brand](#7-design-tokens-and-brand)
8. [Mockup deliverable plan](#8-mockup-deliverable-plan)
9. [Payouts and commissions](#9-payouts-and-commissions)
10. [Promo codes](#10-promo-codes)
11. [QR codes](#11-qr-codes)
12. [Statements](#12-statements)
13. [Audit log](#13-audit-log)
14. [Monitoring and status](#14-monitoring-and-status)
15. [Onboarding flow](#15-onboarding-flow)
16. [Customer co-branding and segmentation](#16-customer-co-branding-and-segmentation)
17. [Support tickets](#17-support-tickets)
18. [Operational features](#18-operational-features)
19. [Environments and deployment](#19-environments-and-deployment)
20. [Forward-looking unification](#20-forward-looking-unification)
21. [Open items requiring board ratification](#21-open-items-requiring-board-ratification)
22. [Security and operational integrity](#22-security-and-operational-integrity)

---

## 1. Executive summary

The Partner Portal and Admin Portal are two purpose-built surfaces serving WavePoint Analytics's content-creator partner program and internal operations team. They share infrastructure, design language, and a unified widget-based component library, but expose different data and capabilities to different roles.

The system supports four authenticated role types: `admin` (WavePoint internal staff), `partner` (formal partner organizations with multi-user teams and self-service tooling), `affiliate` (simpler revenue-share relationships), and `user` (paying customers). Multiple roles can stack on a single account, except `admin` cannot also hold partner or affiliate roles (separation of duties).

Architectural foundation:

- **React + Vite + Tailwind + shadcn/ui + ECharts** as the frontend stack
- **Supabase** for identity, audit, and partner-specific data
- **Stripe** for revenue and customer billing; **Rewardful** for attribution and commission tracking
- **Cloudflare Pages** for hosting; **Cloudflare Workers** for the API layer and edge logic
- **Cloudflare R2** for marketing toolkit assets, QR codes, and statement archives
- **Cloudflare D1** for the cached status data driving the public status page
- **Resend** for transactional email (Phase 3); **SignWell** for embedded e-signature
- **Uptime Kuma** at Yingson Labs for v1 monitoring (with documented migration to a separate WavePoint instance for v2)

Build approach: phased development against a preview URL behind Cloudflare Access, starting with the Partner Portal (highest priority), then Affiliate Portal, then Admin Portal, with a final phase for the unified navigation refresh of the Customer Portal.

The widget-based architecture, design-tokens-as-single-source-of-truth pattern, and externalized configuration are designed to enable a future unification of all WavePoint web properties (marketing site, customer portal, partner portal, admin portal) into a single codebase. That unification is explicitly out of scope for this PRD.

### ⚠️ Launch dependencies (non-engineering)

Engineering completion of Phase 1a does **not** equal launch readiness. The following items must be resolved in parallel with engineering work or launch will be blocked. None of them is a code task.

| # | Dependency | Owner | Status |
|---|---|---|---|
| 1 | **CFTC / NFA Commodity Trading Advisor registration question** | Counsel + Founders | OPEN — flagged in ToS §19.2 item 8. Biggest regulatory risk; must be resolved before significant build investment. |
| 2 | Privacy Policy v0.1 counsel review (15 lawyer-review notes) | Counsel | Pending engagement |
| 3 | Terms of Service v0.1 counsel review (25 lawyer-review notes) | Counsel | Pending engagement |
| 4 | Partner DPA Template v0.1 counsel review (24 lawyer-review notes) | Counsel | Pending engagement |
| 5 | Partner Agreement (commercial terms — separate from DPA) drafting | Counsel + Jason | Not started |
| 6 | Affiliate Agreement drafting | Counsel + Jason | Not started |
| 7 | EU / UK Article 27 representative service procurement (~$100–$400/year) | Jason | Not procured |
| 8 | Cookie consent banner implementation on marketing site + customer portal | Jason | Separate workstream — not part of portal phase plan |
| 9 | Resend domain verification (DKIM/SPF/DMARC for `wavepointanalytics.com`) | Jason | Required before any production email send |
| 10 | SignWell account setup + Partner Agreement / DPA / Affiliate Agreement template upload | Jason | Required before first Phase 3 onboarding (manual signing OK in Phase 1a) |
| 11 | Initial signed Partner Agreements + DPAs with the first ~10 partners | Jason + Partners | Required before partners gain portal access |
| 12 | DNS configuration for production domains | Jason | Cloudflare-managed; quick once needed |

**Recommendation:** Start dependencies #1, #2, #3, #4, #7 in parallel with Phase 1a engineering. They have lead times measured in weeks and any one of them can block launch even with code complete.

---

## 1.5 Build phases and timelines

This PRD organizes work into five phases. The phase structure replaces the v0.2 single-package approach with a sequenced delivery that gets working software in front of partners faster.

**Timeline note:** estimates below are working-week ranges assuming Jason's typical AI-assisted development pace on a single project. They are not commitments. Calibrate against the actual cadence of Phase 1a once it begins; revise downstream estimates based on observed velocity.

### Phase 1a — Partner Portal MVP

**Goal:** ship a portal that lets a content-creator partner do their core job: see what they've earned, share their referral link, see who they've referred, find marketing assets.

**Estimated timeline:** ~3–4 weeks of focused work.

**In scope:**

- Foundation: React + Vite + Tailwind + shadcn/ui + ECharts + TanStack Router + TanStack Query, TypeScript throughout
- Synthetic data layer (`packages/data-stub`) per §8
- Design tokens single-source-of-truth (`packages/design-tokens`) per §7 — **dark mode only**
- Top-bar nav component with role switcher (foundation for all four portals)
- Cloudflare Worker hostname dispatch (the existing architectural plan)
- **Cloudflare Worker referral slug dispatch** at `wavepointanalytics.com/{slug}` per §10A
- Partner Portal pages:
  - Dashboard (KPI row, conversion funnel, earnings trend chart, performance by product, recent referrals table, "Need help? support@wavepointanalytics.com" callout, "💡 Suggest a feature" footer link)
  - Referrals (full table with filters)
  - Marketing Toolkit (asset gallery, including the partner's manually-generated QR code PNG)
  - Promo Codes (read-only list with stats; admin creates codes on partner request)
  - Free Access Seats (read-only display; admin grants manually)
  - Team (multi-user team with admin-set roles; visible roster + activity, no Owner role-management UI yet)
  - Settings — Profile (display name, bio, public link)
  - Settings — Payout (stored payout method, read-only commission rate display)
  - Settings — Notifications (single statement-related toggle for forward-compatibility)
  - Statements (list of past PDF downloads, no opt-in toggle)
- Light internal Admin Portal tooling (just enough to operate Phase 1a):
  - Create / edit partner_orgs and affiliates (manual SQL or simple form, runbook documents the process)
  - Set per-partner commission rate
  - Create / deactivate promo codes for partners on request
  - Grant / revoke Free Access Seats
  - View list of partners + payouts due (CSV export of monthly payouts)
  - Mark payouts complete
  - Generate statement PDF for a partner (button per partner per month — admin attaches to Gmail manually)
  - Basic audit log table view (no full search UI, no row-expansion JSON diff viewer)
- Manual partner onboarding (per `docs/Partner-Onboarding-Runbook.md`):
  - Direct DB row creation
  - Slug reservation
  - Manually-generated QR code via qr-code-generator.com
  - Manual Supabase Auth invite
  - Manual welcome email via Gmail
  - SignWell-or-PDF agreement signing out-of-band
- `feedback@wavepointanalytics.com` mailto link in footer of every page
- Phase 1a security baseline per §22 (CSP headers, HSTS, secure cookies, Zod validation, RLS reviewed by second person, MFA required for admins, Dependabot, npm audit in CI, secrets in Cloudflare Workers secrets, incident response runbook stub)

**Explicitly OUT of Phase 1a (deferred to later phases):**

- Partner self-service promo code creation/deactivation → Phase 3
- Partner self-service Free Access Seat granting → Phase 3
- Self-hosted Worker QR code generation pipeline → Phase 3
- Customer-facing co-branding display (`ReferredByCard` in customer portal) → Phase 3
- In-portal support tickets (entire §17 feature set) → Phase 3
- Public status page at `status.wavepointanalytics.com` → Phase 3
- Full admin impersonation UI → Phase 3
- Partner onboarding wizard with magic-link invite → Phase 3
- Resend email integration (statement automation, ticket emails, magic-link) → Phase 3
- Light mode → Phase 3
- Partner Owner role-management UI for team members → Phase 3
- Statement opt-in toggle + monthly cron + automatic PDF email → Phase 3
- Storybook → Phase 2
- Formal WCAG 2.1 AA audit → Phase 2
- Risk Alerts page → Phase 2
- Audit log DB triggers + R2 weekly export + full searchable UI → Phase 3
- User opt-out of partner visibility self-service toggle → Phase 3 (data field stays in 1a)

### Phase 1b — Affiliate Portal

**Goal:** ship the affiliate-facing variant of the partner portal.

**Estimated timeline:** ~1–2 weeks (most widgets are reused from Phase 1a; affiliate is essentially a stripped-down composition).

**In scope:**

- Affiliate Portal single-page UI per §4 and §8.7 (referrals, earnings, link, statements, no in-portal nav)
- Same dark-mode-only, same `support@` email pattern, same manual onboarding
- Manual statement PDF generation for affiliates (same flow as partners)
- Reuses the widget library + Worker referral dispatch + admin tooling already shipped in 1a

### Phase 2 — Admin Portal

**Goal:** replace the light internal admin tooling from Phase 1a with the full Admin Portal.

**Estimated timeline:** ~3–4 weeks.

**In scope:**

- Full Admin Portal pages per §8.7:
  - Dashboard (full BI KPIs, MRR growth, revenue recognition, all-business funnel, product mix, partner leaderboard, geographic top-5)
  - Customers (list, detail, segmentation filters per §16.3)
  - Partners (list, detail with all per-partner config: commission rate, alternative structures, promo code guardrails, seat quota, auto-expiry, suspension state)
  - Affiliates (list, detail)
  - Onboarding form replacement for the Phase 1a runbook (still admin-driven; not yet self-service)
  - Promo Codes — global view + uniqueness enforcement + reserved word config UI (admin-managed table replaces the Phase 1a hard-coded list)
  - **Risk Alerts** page (full feature per §18.4)
  - Audit Log — full searchable UI with filters + row-expansion JSON diff per §13.2 (DB triggers still deferred to Phase 3)
  - Revenue (recognition, projections)
  - System Health (Uptime Kuma integration via the existing Yingson Labs instance — internal access only)
  - Settings (admin team, branding tokens preview, reserved slugs config, environment config)
- **Storybook** deployment at `widgets-preview.wavepointanalytics.com` per §6.7
- **Formal WCAG 2.1 AA audit** of all widgets shipped in Phases 1a / 1b / 2

### Phase 3 — Partner / Affiliate Upgrades

**Goal:** restore everything punted from Phase 1a, in order of partner-perceived value. These can ship as a single phase or as a sequence of small releases — they are independent.

**Estimated timeline:** ~3–4 weeks total, but designed to be released individually as each completes.

**Sub-priorities (rough order):**

1. **Resend integration** — required for almost everything else in this phase
2. **Statements** — automated monthly cron, opt-in toggle UI, PDF + HTML email (replaces Phase 1a manual generation)
3. **Partner self-service promo code creation/deactivation** with admin-configurable per-partner guardrails
4. **In-portal support tickets** — full §17 feature set (categories, threading, status workflow, email notifications, reopen window)
5. **Self-hosted QR code Worker pipeline** — admin regeneration UI, R2 storage, automatic generation at onboarding (replaces Phase 1a manual generation)
6. **Free Access Seats self-service** — partner Owner can grant/revoke within quota
7. **Partner onboarding wizard** with magic-link invite + welcome screen + SignWell embedded signing
8. **Light mode toggle** — second theme variant of design tokens; user toggle in nav
9. **Customer-facing co-branding** — `ReferredByCard` widget in customer portal with 30-day window logic and partner toggle
10. **Public status page** at `status.wavepointanalytics.com` — Cloudflare Worker polling Kuma + D1 cache + name-rewrite mapping
11. **Admin impersonation UI** — full "View as user" with banners, time-limited sessions, hard restrictions, audit logging
12. **Partner Owner role-management UI** — Owners can invite, role-set, and remove team members within the cap
13. **User opt-out of partner visibility** self-service toggle in user portal
14. **Audit log enhancements** — DB triggers as second write path, weekly R2 exports, advanced filters
15. **In-portal feedback submission** — replaces `feedback@` mailto pattern with structured form (or folds into ticket category)

### Future phases (out of scope for this PRD)

- User Portal redesign with co-branding integration
- Marketing site rewrite on the same React stack
- Marketing site + customer portal + partner portal + admin portal unified into a single codebase per §20
- Stripe Connect for partner payouts (REVISIT trigger: ~30 partners)
- Marketing automation depth Option B / Option C per §16.4
- White-labeling of the partner portal for top-tier partners
- Multi-currency display
- File attachments on tickets
- Customer-side ticket system
- Bug bounty program
- SOC2 Type 1 prep (REVISIT trigger documented in §22)

### Phase entry/exit criteria

| Phase | Entry criteria | Exit criteria |
|---|---|---|
| 1a | This PRD ratified by founders; legal counsel engagement initiated | Partner Portal pages all functional against synthetic data; first ~3 partners onboarded against real data via runbook; basic admin tooling sufficient to operate the program |
| 1b | Phase 1a deployed; first partners using portal | Affiliate single-page UI deployed; first ~3 affiliates onboarded |
| 2 | Phase 1b deployed; volume of admin work in light internal tooling justifies full build | Full Admin Portal replaces light tooling; risk alerts surface real signals; audit log searchable |
| 3 | Phase 2 deployed; partners requesting self-service features that map to Phase 3 sub-items | Each sub-item ships independently; no big-bang Phase 3 release |

---

## 2. Glossary and conventions

### Roles and entities

- **User** — any authenticated person. The base role; everyone has it.
- **Customer** — a User with an active paid subscription to WavePoint indicators.
- **Admin** — internal WavePoint staff (currently the three founders: Jason, George, Derek). Has full operational control. Cannot also be a partner or affiliate.
- **Partner** — a formal business relationship with custom contractual terms. Partners are organizations (`partner_org`) that may have multiple team members.
- **Partner Owner** — a member of a partner_org with full self-service capabilities (financial visibility, team management, promo codes, free seats, payout settings). A partner_org may have multiple Owners.
- **Partner Member** — a member of a partner_org with limited capabilities (sees referral activity but not financial details, cannot manage team or codes).
- **Affiliate** — a simpler revenue-share relationship. Always a single individual, no team. Lower trust tier, lower onboarding friction. No promo codes, no free seats.

### Data and operational terms

- **Referral slug** — the unique short string identifying a partner_org or affiliate in URLs (e.g., `drysdale` in `wavepointanalytics.com/drysdale`). Globally unique across both partner_orgs and affiliates and namespace-isolated from marketing site routes via reserved slug list (see §10A).
- **Promo code** — a unique alphanumeric string that grants a discount at checkout and attributes the conversion to its owner. Replaces the term "coupon code" used in earlier drafts.
- **Free Access Seat** — an admin-configurable allowance that lets a partner grant free indicator access to a specific customer. Partners only.
- **Attribution** — the system's record of which partner_org or affiliate is credited for a customer's signup. Locked at first paid conversion; only one entity earns from a given customer.
- **Clawback** — the reversal of a previously-recorded commission when a customer refunds within the clawback window (30 days, provisional).

### Document conventions

- **LOCKED** — decision finalized for v1; not expected to change before launch.
- **PROVISIONAL** — working assumption for the PRD/mockup; requires board ratification before launch.
- **REVISIT** — locked for v1 with intentional flag for later review under specified triggers.
- **OUT OF SCOPE** — referenced for completeness but not part of this PRD's deliverable.


---

## 3. Roles, identity, and access

### 3.1 Role model

WavePoint defines four authenticated role types:

| Role | Issued by | Portal access |
|---|---|---|
| `user` | Automatic on account creation | User Portal |
| `affiliate` | WavePoint Admin (manual onboarding) | User Portal + Affiliate view |
| `partner_owner` | WavePoint Admin (initial Owner per partner_org) or other Owners (subsequent Owners) | User Portal + full Partner Portal |
| `partner_member` | Partner Owner (within their org) | User Portal + Partner Portal (limited view) |
| `admin` | Existing WavePoint Admin (manual addition) | User Portal + Admin Portal |

**Role stacking and exclusions (LOCKED):**

- A single account can hold multiple roles simultaneously.
- The `user` role is implicit on every authenticated account.
- Other roles stack additively: a partner team member who is also a paying customer holds `user + partner_member`. An affiliate who is also a customer holds `user + affiliate`. A user can hold both `partner_member` and `affiliate` simultaneously if appropriate.
- **Hard exclusion:** `admin` cannot coexist with `partner_owner`, `partner_member`, or `affiliate`. This separation of duties prevents the people controlling payouts from also receiving them.
- Founders (the three current Admins) never earn referral commission, ever — even if they personally refer someone. Personal referrals by founders are business development, not affiliate revenue.

### 3.2 Multi-portal navigation

When a user with multiple roles signs in, they see a unified top-bar navigation that displays only the portals they have access to, in fixed order:

1. **User Portal** (always visible — everyone has it)
2. **Affiliate** (if applicable)
3. **Partner** (if applicable)
4. **Admin** (if applicable, always last because it is the most privileged)

The default landing page on login is the User Portal Dashboard for everyone. The user clicks a portal in the top bar to switch context; in-portal navigation refreshes to show that portal's pages.

This top-bar nav is a single component dropped into all four portals (Customer, Partner, Affiliate, Admin). Refreshing the existing customer portal at `app.wavepointanalytics.com` to consume this component is part of this PRD's scope.

### 3.3 Owner vs Member within a partner_org

The Owner role is a **superset** of the Member role. Owners and Members see the same UI surface; Owners simply have more buttons enabled. Specifically, Owners can:

- View commission dollars, payout history, and financial details
- Manage team members (invite, remove)
- Create and deactivate promo codes
- Grant and revoke Free Access Seats
- Update payout method and partner_org settings
- Manage the partner_org's public profile and co-branding settings

Members see referral activity (count of attributed customers, conversion funnel, performance by product) but no commission dollar amounts, no payout details, no team management, no promo code creation.

A partner_org can have multiple Owners, all with equivalent permissions. An Owner can remove another Owner, but the system prevents removing the last Owner — admin must designate a replacement before the action completes.

### 3.4 Self-referral prevention (LOCKED)

An affiliate or partner cannot earn commission on their own subscription. Implementation:

- At attribution-recording time, the system checks if the referred user's email matches the affiliate's email or any team member email of the partner_org.
- If matched: attribution is recorded but commission is set to zero.
- The block also extends to: same payment method as the affiliate's payout method, and best-effort same-household checks (IP + name heuristic). Same-household matches are flagged for admin review rather than auto-blocked, to avoid false positives.

### 3.5 Attribution conflicts (LOCKED)

Only one role earns from a given customer. Resolution order at signup:

1. If a promo code is used at checkout → that promo code's owner gets credit (overrides cookie).
2. Else, if a Rewardful tracking cookie is set → that affiliate or partner_org gets credit.
3. Else, no attribution.

After attribution is locked at first paid conversion, it cannot change via cookie or promo code reuse. Admins retain the ability to manually override attribution with a logged reason (covered in §13).

### 3.6 Account state transitions (LOCKED)

- **Suspending a partner role** does NOT terminate the user's customer subscription. They lose partner dashboard access; their indicators continue working as long as they're paying.
- Same applies to suspending an affiliate role.
- Terminating the user/customer subscription is a separate action.
- Suspension state semantics are detailed in §18 (Suspension state).

### 3.7 Demotion (Partner → Affiliate)

When admin demotes a partner_org to affiliate status:

- Team members are removed (affiliates are solo).
- Existing promo codes are deactivated immediately; code strings remain reserved per the uniqueness rule (12-month cooldown).
- Existing free seats are revoked immediately; recipients lose access.
- Referral slug is preserved unchanged (attribution continuity).
- QR code is preserved unchanged.
- Commission rate resets to the default affiliate rate unless admin sets a custom override.
- Historical attribution data is preserved.
- All actions are audit-logged with required reason text from admin.

### 3.8 Multi-Owner ownership transfer

When an Owner leaves a partner_org:

- Their `user_role_assignments` row for `partner_owner` is revoked.
- They retain their `user` role; subscription unaffected if paying.
- The org keeps all historical attribution, referrals, commission records.
- If departing Owner is the last Owner, admin must designate a new Owner from existing team members or invite a new user before removal can complete. The action is blocked until succession is resolved.


---

## 4. Partner and affiliate models

### 4.1 Conceptual definitions

- **Partner:** A formal business relationship with custom terms, multi-user team, and self-service tooling. Onboarded via signed Partner Agreement + DPA. Higher trust tier, higher administrative overhead.
- **Affiliate:** A simpler revenue-share relationship — single person, standard terms, fewer self-service tools. Lower trust tier, lower onboarding friction.

### 4.2 Feature differential matrix (LOCKED)

| Feature | Partner | Affiliate |
|---|---|---|
| Referral link | ✅ | ✅ |
| QR code with WavePoint logo embed | ✅ | ✅ (same setup) |
| Promo codes | ✅ self-serve with admin guardrails | ❌ none — referral link only |
| Free Access Seats | ✅ | ❌ |
| Multi-user team | ✅ Owner + Members | ❌ solo only |
| Email visibility on referred users | Masked pre-paid (`j***@g***.com`) → full email post-paid | Always masked: first 3 + last 3 chars of local part + full domain (e.g., `jas______son@gmail.com`); never reveals even post-paid |
| Marketing toolkit | ✅ full access | ✅ full access (same toolkit) |
| Support tickets | ✅ 9 categories | ✅ 7 categories (no Content Request, no Team Management Help) |
| Statements opt-in (email summaries) | ✅ | ✅ |
| Earnings trend chart | ✅ | ✅ same |
| Conversion funnel | ✅ | ✅ same |
| Performance by Product chart | ✅ | ✅ same |
| Custom commission rate | ✅ admin-configurable per partner | ❌ flat rate only (15% PROVISIONAL default) |
| Total user count attributed (broken down by source) | ✅ Breakdown by source: referral link / promo code / free seat redemption / admin-manual. Sliced by product (NinjaTrader / TradingView). | ✅ Same breakdown view (free seat row will be empty since affiliates have no seats) |
| Onboarding | Admin-driven flow with Partner Agreement + DPA | Admin-driven flow, simpler — no team setup. DPA required (PROVISIONAL, REVISIT) |
| Public co-branding profile | ✅ display name + bio + avatar/logo + public link | ✅ display name + bio + public link (no avatar/logo at launch) |

### 4.3 Email masking rule (LOCKED)

- **Partners:** masked pre-paid (`j***@g***.com`); full email post-paid conversion.
- **Affiliates:** always masked, never reveals. Algorithm: first 3 + last 3 characters of the local part + full domain (e.g., `jasonjohnson@gmail.com` → `jas______son@gmail.com`).
- **Edge case:** if local part is shorter than 6 characters, the entire local part is masked (e.g., `bob@gmail.com` → `***@gmail.com`).

### 4.4 Affiliate-to-Partner promotion path (LOCKED)

- Admin can promote an affiliate to partner status.
- Attribution carries over (existing referrals continue earning).
- Referral slug + QR code preserved unchanged.
- Team setup begins fresh (Owner role assigned to the original affiliate user).
- Commission rate switches from flat-affiliate-rate to whatever partner deal is negotiated.
- Partner Agreement + DPA must be signed before promotion finalizes.


---

## 5. Data architecture

### 5.1 High-level approach (LOCKED)

- **Two separate tables for partner_orgs and affiliates**, with a SQL view `referral_entities` for cross-cutting queries.
- Promotion path implemented via INSERT into `partner_orgs` + lineage column `promoted_from_affiliate_id` + DELETE from `affiliates`.
- Both tables FK into a shared `referral_slugs` table for global namespace enforcement.
- Postgres in Supabase (`user_db` project, URL `https://eqozbdekcawelfbpgrit.supabase.co`).
- All sensitive data access governed by Postgres RLS policies; widget code does not implement authorization itself.

### 5.2 Core schema sketch

The following is illustrative of the structural approach. Final column-level details emerge during Phase A foundation work.

```sql
-- Identity layer
auth.users                  -- Supabase-managed
user_profiles               -- 1:1 with auth.users
  id (PK = auth.users.id)
  email
  display_name
  avatar_url
  opted_out_of_partner_visibility
  statement_email_opt_in
  statement_cadence
  light_dark_preference
  ...

user_role_assignments       -- many-to-many: user ↔ role
  id
  user_profile_id (FK)
  role (enum: admin, partner_owner, partner_member, affiliate)
  partner_org_id (FK, nullable)
  affiliate_id (FK, nullable)
  is_active
  suspended_at
  granted_by (FK)
  granted_at
  revoked_at

-- Referral entity layer
referral_slugs              -- shared namespace
  slug (unique PK)
  entity_type (enum: partner_org, affiliate)
  entity_id (uuid)
  is_active
  reserved_at

partner_orgs
  id
  display_name
  referral_slug (FK)
  qr_code_asset_url
  commission_rate_pct
  commission_structure (enum: percentage, alternative_offer, hybrid)
  alternative_offer_terms (jsonb)
  promo_max_discount_pct
  promo_max_uses_per_code
  promo_expiry_days
  promo_concurrent_active_max
  free_seat_quota
  free_seat_auto_expire_days
  team_member_cap (default 5)
  is_active
  suspended_at
  agreement_version_signed
  agreement_signed_at
  dpa_version_signed
  dpa_signed_at
  promoted_from_affiliate_id (nullable lineage FK)
  ...

affiliates
  id
  user_profile_id (FK, 1:1)
  display_name
  referral_slug (FK)
  qr_code_asset_url
  commission_rate_pct
  is_active
  suspended_at
  agreement_version_signed
  agreement_signed_at
  dpa_version_signed
  dpa_signed_at
  ...

-- Attribution and conversion layer
referral_attributions       -- one row per attributed customer
  id
  referred_user_id (FK)
  attribution_type (enum: partner, affiliate)
  partner_org_id (FK, nullable)
  affiliate_id (FK, nullable)
  attribution_source (enum: cookie, promo_code, free_seat, admin_manual)
  promo_code_id (FK, nullable)
  locked_at
  ...

-- Operational layer
promo_codes
  id
  code (uppercase, unique among active)
  owner_partner_org_id (FK)
  discount_type (enum: pct_off_first_month, dollar_off_first_month, ...)
  discount_value
  max_uses
  uses_so_far
  expires_at
  status (enum: pending, active, expired, deactivated)
  deactivated_at
  ...

reserved_promo_codes        -- admin-managed reserved word list
reserved_slugs              -- admin-managed reserved word list

free_access_seats
  id
  partner_org_id (FK)
  granted_to_user_id (FK)
  granted_by_user_id (FK)
  granted_at
  revoked_at
  expires_at
  ...

payouts
  id
  partner_org_id (FK, nullable)
  affiliate_id (FK, nullable)
  amount_cents
  currency (always 'USD' v1)
  payout_method (enum: ach, paypal, wise, manual_check)
  destination_last_4
  status (enum: pending, processing, completed, failed)
  initiated_at
  completed_at
  reason_text (nullable for routine)
  ...

-- Communication and tickets
tickets
ticket_messages

-- Audit and compliance
audit_log                   -- append-only, RLS-enforced
  id
  created_at
  actor_user_id (FK, nullable)
  actor_role
  action_type
  target_type
  target_id
  before_jsonb
  after_jsonb
  reason_text
  ip_address
  user_agent
  session_id
  metadata_jsonb

email_history               -- when users change emails

-- Status / monitoring (Cloudflare D1, separate from Postgres)
monitor_current_status
monitor_status_history
incidents
```

### 5.3 Key data integrity rules

- `referral_attributions.partner_org_id` and `affiliate_id` are mutually exclusive (CHECK constraint or trigger ensures exactly one is non-null).
- `referral_slugs.slug` is globally unique across all entity types.
- Promo code uniqueness is enforced via a CHECK that combines active codes plus deactivated codes within the 12-month cooldown window.
- `user_role_assignments` enforces the admin/partner-affiliate exclusion via CHECK constraint or trigger.
- Audit log is append-only; UPDATE and DELETE operations are blocked at RLS layer.

### 5.4 RLS policy examples

```sql
-- audit_log: admins see all; users see their own actor rows
CREATE POLICY audit_log_select ON audit_log FOR SELECT TO authenticated USING (
  EXISTS (
    SELECT 1 FROM user_role_assignments
    WHERE user_profile_id = auth.uid() AND role = 'admin' AND is_active
  )
  OR actor_user_id = auth.uid()
);

CREATE POLICY audit_log_insert ON audit_log FOR INSERT TO authenticated WITH CHECK (true);
CREATE POLICY audit_log_no_update ON audit_log FOR UPDATE USING (false);
CREATE POLICY audit_log_no_delete ON audit_log FOR DELETE USING (false);

-- partner-scoped visibility on referrals
CREATE POLICY referral_attributions_partner_view ON referral_attributions
FOR SELECT TO authenticated USING (
  partner_org_id IN (
    SELECT partner_org_id FROM user_role_assignments
    WHERE user_profile_id = auth.uid()
      AND role IN ('partner_owner', 'partner_member')
      AND is_active
  )
);
```


---

## 6. Widget library and frontend architecture

> **Phase placement (v0.3):** The widget library architecture, base widget wrapper, hybrid data access pattern, and most widget implementations all ship in **Phase 1a**. Two notable exceptions:
> - **Storybook (§6.7)** moves to **Phase 2**. In Phase 1a, widgets are reviewed inline within the running app (a `/dev/widgets` sandbox route is acceptable).
> - **Formal WCAG 2.1 AA audit (§6.8)** moves to **Phase 2**. Phase 1a builds *to* WCAG patterns (semantic HTML, focus rings, keyboard nav, contrast tokens) but does not run formal axe-core or screen reader audits.
> - The **build sequence in §6.3** (Phases A–F) is the v0.2 internal sub-phase plan and is now superseded by the Phase 1a / 1b / 2 / 3 structure in §1.5. The A–F labels are retained as internal sub-phases inside Phase 1a only (foundation → partner portal scaffolding → ...).

### 6.1 Approach (LOCKED)

The frontend is built as a **library of reusable widget components** composed onto pages. There is no runtime configuration UI (no drag-drop dashboard builder); pages are defined in code by importing widgets and arranging them.

Each widget receives at minimum two props:

- `role` — one of `admin / partner_owner / partner_member / affiliate / user`
- `dataScope` — an object identifying what data the widget should render (e.g., `{ partnerOrgId: 'abc' }`, `{ affiliateId: 'xyz' }`, or `{ scope: 'global' }` for admin)

Internal feature flags within each widget handle role-based visibility (e.g., `EarningsTrendChart` hides dollar amounts when `role === 'partner_member'`).

### 6.2 Tech stack (LOCKED)

- **React 18+** with **TypeScript** throughout — compile-time prop safety eliminates the `tvUsername||''` class of bug.
- **Vite** as build tool — fast dev server, fast production builds.
- **Tailwind CSS** for styling — utility-first, consumes design tokens at build time.
- **shadcn/ui** for primitives — modals, popovers, dropdowns, tooltips, form controls, all light/dark mode aware.
- **ECharts** for data visualization — premium financial-dashboard aesthetic (replaced Chart.js from earlier specs).
- **TanStack Router** for routing — typesafe routes, query params, navigation state.
- **TanStack Query** for data fetching, caching, and revalidation.
- **lucide-react** for icons.

### 6.3 Build sequence — phased delivery (LOCKED)

| Phase | Scope | Estimate |
|---|---|---|
| A | Foundation: design tokens, base components, widget shell, theme provider, light/dark, mobile breakpoints, role-switcher control, synthetic data fixture, environment config externalization | 1 session |
| B | **Partner Portal** (Owner + Member views, 9 pages) | 2-3 sessions |
| C | **Affiliate Portal** (1 page + tickets) | 1 session |
| D | **Admin Portal** (11 pages) | 3-4 sessions |
| E | User Portal nav additions for cross-portal navigation | <1 session |
| F | Storybook deployment + polish + a11y audit | 1-2 sessions |

Partner Portal is the highest-priority first deliverable. Admin Portal value unlocks fully when Stripe and Supabase are wired to real data.

### 6.4 Widget catalog

Approximately 30 widgets total, grouped by category:

**KPI / summary:**
- `KpiCard`, `KpiCardRow`, `SystemStatusStrip`

**Charts:**
- `EarningsTrendChart`, `MrrGrowthChart`, `ConversionFunnelChart`, `PerformanceByProductChart`, `ProductMixDonut`

**Tables:**
- `RecentReferralsTable`, `AttributedUsersBreakdown`, `CustomersTable`, `PartnersTable`, `AffiliatesTable`, `AuditLogFeed`, `RiskAlertsPanel`, `GeographicTop5Table`, `PartnerLeaderboard`, `RecentTicketsTable`

**Action / management widgets:**
- `ReferralLinkCard`, `PromoCodesManager`, `FreeSeatsManager`, `TeamMembersManager`, `MarketingToolkitGallery`, `SupportTicketsList`, `StatementsOptInToggle`, `ReferredByCard` (customer portal co-branding), `PublicProfileEditor`

**Admin-only operational:**
- `RevenueRecognitionCard`, `UpcomingPayoutsCard`, `PartnerOnboardingForm`, `ManualOverrideToolkit`, `UptimeKumaWidget`, `ImpersonationBanner`

### 6.5 Data access pattern (LOCKED)

**Hybrid model:**

- **Presentation widgets** (charts, tables, KPI cards, status strips) receive data as **props** from pages. Pure presentation, infinitely reusable.
- **Feature widgets** (`PromoCodesManager`, `FreeSeatsManager`, `TeamMembersManager`, `MarketingToolkitGallery`, `SupportTicketsList`, `StatementsOptInToggle`) **self-fetch and self-mutate**, encapsulating their full read+write lifecycle.

TanStack Query handles caching, dedup, and revalidation across widgets that fetch the same underlying data.

### 6.6 Base widget wrapper (LOCKED)

All widgets render inside a `<WidgetShell>` component that provides:

- **Loading state** — skeleton placeholder matching widget shape
- **Error state** — with retry button, structured error object passed in
- **Empty state** — with optional copy and CTA
- Optional title bar / actions slot
- Boundary error catching (a widget crash doesn't crash the page)

Widgets opt out of any state via flags (rare cases).

### 6.7 Storybook (LOCKED)

A Storybook deployment at `widgets-preview.wavepointanalytics.com` (behind Cloudflare Access) renders every widget in isolation across:

- Default state, loading, error, empty
- Key role variants (Owner vs Member, Partner vs Affiliate)
- Light and dark mode
- Mobile (768px) and desktop breakpoints

Storybook includes axe-core accessibility checks per story; CI fails on violations.

### 6.8 Accessibility floor (LOCKED)

**WCAG 2.1 AA** from Day 1. Specific requirements baked into every widget:

- Color contrast ≥ 4.5:1 for text, ≥ 3:1 for UI elements
- All interactive elements have visible focus rings
- Keyboard navigation works everywhere (no mouse-only)
- Screen reader semantics: proper headings, ARIA labels on icon buttons, aria-live regions for dynamic content
- Modals trap focus, restore on close
- Form fields have associated labels
- Color is never the only signal (red+icon for error, not just red)
- Tap targets ≥ 44×44 px on mobile
- Motion can be disabled via `prefers-reduced-motion`


---

## 7. Design tokens and brand

## 7. Design tokens and brand

> **Phase placement (v0.3):** The design tokens single-source-of-truth pattern, Tailwind / shadcn / ECharts theme integration, and aesthetic principles all ship in **Phase 1a**. Critical change: **Phase 1a ships dark mode only**. The light/dark toggle described in §7.4 moves to **Phase 3**. Light-mode token values are still defined in the tokens file (so the second variant can be activated quickly later), but the toggle UI and runtime theme switching are not built in Phase 1a.

### 7.1 Single source of truth (LOCKED)

All visual brand values live in a single canonical file: `design-tokens.json` (with a TypeScript wrapper for type safety) and a human-readable `BRANDING.md` companion.

This is the only place the following are defined:

- Colors (surface, accent, semantic, text)
- Typography (font families, weights, type scale)
- Spacing scale (4 / 8 / 12 / 16 / 24 / 32 / 48 / 64 etc.)
- Border radii (sm / md / lg)
- Animation timings (fast / base / slow)
- Logo paths and brand assets

### 7.2 Rules

- No CSS file, component, or page may hardcode brand values. No raw `#0b1326`, no `'Manrope'` string literals scattered across components.
- **Tailwind config** consumes the design tokens at build time (`tailwind.config.ts` imports the JSON).
- **shadcn/ui theme variables** map to the design tokens.
- **ECharts theme** is generated from the design tokens (custom theme JSON derived from the source).
- Logo, favicon, OG images live in `branding-assets/` directory referenced from the tokens file.
- A change to the tokens file triggers a rebuild and re-deploy that propagates the change everywhere.

### 7.3 Token file shape (proposed)

```json
{
  "color": {
    "surface": { "base": "#0b1326", "raised": "...", "glass": "..." },
    "accent": { "teal": "#4edea3", "primary": "#adc6ff" },
    "semantic": { "success": "...", "warn": "...", "danger": "..." },
    "text": { "primary": "...", "secondary": "...", "muted": "..." }
  },
  "typography": {
    "display": { "family": "Manrope", "weights": [400, 600, 700] },
    "body": { "family": "Inter", "weights": [400, 500, 600] },
    "scale": { "xs": "12px", "sm": "14px", "base": "16px", "lg": "18px", "xl": "24px", "2xl": "32px" }
  },
  "spacing": { "1": "4px", "2": "8px", "3": "12px", "4": "16px", "6": "24px", "8": "32px", "12": "48px" },
  "radius": { "sm": "6px", "md": "12px", "lg": "20px" },
  "motion": { "fast": "150ms", "base": "250ms", "slow": "400ms" },
  "logo": { "primary": "/branding-assets/logo.svg", "mark": "/branding-assets/mark.svg" }
}
```

### 7.4 Light/dark mode (LOCKED)

- Both modes built from Day 1.
- User-selectable toggle in the top navigation.
- Toggle preference persists per user (localStorage initially, eventually a column on `user_profiles`).
- If forced to choose a default, default is **dark**.
- Both modes consume the same design tokens (light and dark variants defined in the tokens file).

### 7.5 Aesthetic principles

What makes the portal feel "ultra premium":

1. **Typography rhythm** — Manrope display, Inter body, generous line-height, consistent type scale
2. **Color discipline** — limited palette, no bright accents
3. **Spacing system** — consistent 4/8 px grid, generous padding inside cards
4. **Animation restraint** — subtle hover transitions (200-250ms ease-out), no bouncy effects, no aggressive entrance animations
5. **Chart aesthetic** — ECharts with custom theme, thin lines, subdued grid, no chart junk, value labels on hover
6. **Glass / depth** — backdrop-filter blur on cards, subtle inner glow, never heavy shadows
7. **Mobile** — single-column layouts, larger tap targets (44px min), simplified KPI cards

### 7.6 Forward path to full unification (REVISIT)

The `design-tokens.json` is structured to be portable. When the marketing site and customer portal are eventually rewritten on the same React stack:

- They consume the same tokens
- They use the same widget library (or its successor packages)
- A change to brand colors becomes: edit one JSON file → all four properties rebuild and deploy

This unification is **out of scope** for this PRD but the architecture is designed to enable it.


---

## 8. Mockup deliverable plan

### 8.1 Approach (LOCKED)

Build the **production React stack from the start**. There is no separate "mockup" codebase that gets thrown away — the v0.2 mockup IS the production code, just running against a synthetic data layer instead of real Supabase/Stripe.

When real data sources are wired up (post-PRD, in a later session), the UI is preserved unchanged; only the data layer swaps from `data-stub` to a real Supabase client.

### 8.2 Phased deployment

Each phase deploys to `portal-preview.wavepointanalytics.com` behind Cloudflare Access for review. Each phase reviewed independently before the next begins. Bug fixes and feedback go in a backlog for Phase F polish.

### 8.3 Repo structure (LOCKED)

```
wavepointanalytics-partner-admin-portal/
├── apps/
│   ├── portal/              ← React app (partner + admin + affiliate, routed by subdomain)
│   └── storybook/           ← Storybook deploy
├── packages/
│   ├── widgets/             ← Widget library
│   ├── design-tokens/       ← Single source of truth for brand
│   └── data-stub/           ← Synthetic data layer mimicking future Supabase client interface
├── docs/
│   ├── Partner-Admin-Portal-PRD-v0.1.md   ← THIS DOCUMENT
│   ├── Partner-Portal-Architecture.md     ← v0.1 architecture brief (historical)
│   ├── Privacy-Policy-v0.1.md
│   ├── Terms-of-Service-v0.1.md
│   └── Partner-DPA-Template-v0.1.md
├── mockups/                 ← v0.1 HTML mockups (archived)
├── infra/
│   ├── cloudflare-worker/   ← API layer, hostname dispatch, status cache, QR generator
│   └── pages-config/
├── README.md
├── CHANGELOG.md
└── CLAUDE.md
```

- **pnpm workspaces** monorepo
- Designed to allow future migration of marketing site and customer portal as additional `apps/`

### 8.4 Stub data layer (LOCKED)

`packages/data-stub` returns **deterministic synthetic data** mimicking the future Supabase client interface. Drop-in replacement: when production-ready, swap `data-stub` for `data-supabase` via a single config change.

**Synthetic data fixture content:**

- 6 months of plausible referral history
- ~50 customers (named, varied demographics, mix of plan tiers and lifecycle states)
- 10 partners (including **Drysdale Trading Group** as Partner #1, with named team members)
- 5 affiliates (including **Sam Carter** as Affiliate #1)
- Mix of paid, trialing, refunded, free-seat customers to cover all UI states

### 8.5 Role switcher (LOCKED)

A persistent corner control on every page lets the viewer simulate any role for demo and review purposes.

- Available in Phase A onwards
- Visually distinct (small badge: "🔧 Demo Role: Owner") so it's never confused with production UI
- Removed automatically in production builds (env flag)

### 8.6 Mobile breakpoints (LOCKED)

- Mobile baseline: **768px** (tablet portrait and up)
- Desktop primary
- Smaller phone sizes (375px iPhone SE) skipped for v0.2; can be added in Phase F polish if needed.

### 8.7 Pages roster

**Partner Portal (Owner view, full-fledged):**
1. Dashboard
2. Referrals (full table, filters, search, export)
3. Promo Codes (list, create, manage)
4. Free Seats (list, grant, revoke)
5. Marketing Toolkit (asset gallery)
6. Team Management (invite, remove)
7. Support Tickets (list, create, reply thread)
8. Statements (history, opt-in toggle)
9. Settings (profile, public co-branding, notifications, payout)

**Affiliate Portal:**
- Single dashboard with referrals, earnings, link/QR, statements, ticket creation/list — no in-portal nav (mobile-friendly single-page model)

**Admin Portal:**
1. Dashboard
2. Customers (list, detail, segmentation filters)
3. Partners (list, detail with all per-partner config)
4. Affiliates (list, detail)
5. Onboarding (the single-page form for adding partner/affiliate)
6. Promo Codes (global view, uniqueness enforcement, reserved word config)
7. Risk Alerts (list, investigate)
8. Audit Log (full searchable view)
9. Revenue (recognition, projections)
10. System Health (Uptime Kuma integration)
11. Settings (admin team, branding tokens preview, reserved slugs config, environment config)

**User Portal:**
- Out of scope to redesign
- In scope: add the unified top-bar nav component (Phase E)
- In scope: add the `ReferredByCard` widget for co-branding (Phase E)


---

## 9. Payouts and commissions

### 9.1 Mechanism (LOCKED)

**Phase 1: Manual ACH from WavePoint's business bank account.**

- Admin portal generates a monthly payout list (CSV export + on-screen view) showing each partner: name, amount owed, payout method, destination details.
- Admin reads the list and executes payouts via business bank's bill-pay UI.
- Admin marks each payout as completed in the portal (status: `pending → processing → completed`).

**Future phases (deferred):**

- **Phase 2:** Add Rewardful PayPal payouts as an option for partners who prefer it.
- **Phase 3+:** Stripe Connect if partner volume grows past ~30 partners.
- **No Wise** unless meaningful international FX volume justifies it.

### 9.2 Stored payout method per partner (LOCKED)

Partner sets their preferred method at onboarding. Methods supported at launch:

- **ACH** — US bank account details (routing, account)
- **PayPal** — email
- **Wise** — email or account
- **Manual Check** — mailing address

Partner can update method via portal settings. Admin notified on change. Change logged to audit. Change takes effect next payout cycle.

Admin can override the partner's stored method only with a logged reason.

### 9.3 Refund clawback window (PROVISIONAL — 30 days)

- 30 days from initial paid conversion.
- During this window, the commission appears in "In Refund Hold" KPI, not yet payable.
- After 30 days, commission moves to "Available to Pay Out".
- Refund within window: commission reversed (clawback); partner balance reduced.
- Refund AFTER window: commission has already been paid out; future commissions debited to recover (negative balance allowed up to a limit before admin intervention).

**Note:** PROVISIONAL — Jason flagged that this is not yet finalized at board level. Locked to 30 days for the PRD/mockup; revisit if board changes.

### 9.4 Minimum payout threshold (LOCKED)

- **$25 minimum** payout
- Below threshold, balance rolls to next month
- Partner sees: "Your balance is $X — minimum payout is $25, paying out next when threshold is met"

### 9.5 Default commission rates (PROVISIONAL)

- **Partner commission rate:** set per partner contract — no system-wide default. Each partner's rate is configured by admin during onboarding based on the signed Partner Agreement.
- **Affiliate flat commission rate:** **15%** (system-wide default; admin can override per affiliate but discouraged).

**Note:** Both rates pending Derek/George conversation. Working assumptions for the PRD/mockup; revisit when board confirms.

### 9.6 Commission rate visibility (LOCKED)

- Visible in partner/affiliate portal UI as "Your commission rate: X%".
- For partners with custom alternative structures (e.g., "first 2 months free for the referrer instead of percentage"): "Your commission structure: Custom — see partner agreement".
- Builds trust through transparency.

### 9.7 Currency (LOCKED)

- **USD-only display in v1.**
- All amounts shown in USD across portal, statements, payout history.
- Multi-currency display deferred to a later phase if non-US partners report friction.
- Data model stores amounts in cents (USD) avoiding floating point issues.

### 9.8 Payout history detail (LOCKED)

Partner sees per past payout: date, amount, payout method label (e.g., "ACH", "PayPal"), last 4 digits of destination account for verification, status.

NOT shown in partner-facing UI:

- Full account numbers
- Full email addresses for PayPal/Wise
- Reference numbers

These are visible to admins for operational purposes (audit-logged). No banking info exposed in any partner-facing email or PDF statement.


---

## 10. Promo codes

> **Phase placement (v0.3):** The promo code feature ships in Phase 1a as **admin-creates-on-request** with a partner-facing read-only stats list. Partner self-service code creation, deactivation, and guardrail configuration moves to **Phase 3**. All locked decisions in §10.1–§10.11 stay valid; only the surface is gated by phase. See §10.12 for the per-subsection breakdown.

### 10.1 Uniqueness (LOCKED)

- **Unique among ACTIVE codes only**, globally across all partners.
- Deactivated/expired code strings can be reused after **12 months** from deactivation date.
- Implementation: collision check queries `WHERE status='active' OR (status IN ('deactivated','expired') AND deactivated_at > NOW() - INTERVAL '12 months')`.

### 10.2 Conflict UX (LOCKED)

When a partner tries to create a taken code:

- Message: "Code unavailable — try [SUGGESTION_1], [SUGGESTION_2], [SUGGESTION_3]"
- Suggestion algorithm: numeric suffix (CODE → CODE2, CODE10), or two-letter suffix (CODE → CODEXY), or insert separator-free variation
- Partner identity NEVER revealed in the conflict message

### 10.3 Format (LOCKED)

- 4-20 characters, alphanumeric only (a-z, A-Z, 0-9)
- Case-insensitive matching; canonical form is uppercase
- No hyphens, underscores, or special characters
- Validation regex: `^[A-Za-z0-9]{4,20}$`

### 10.4 Reserved word list (LOCKED)

- **Admin-managed config table** (`reserved_promo_codes`)
- Admins can add/remove entries via admin portal; all changes audit-logged
- Initial seed list:
  - Brand: `WAVEPOINT`, `WAVEPOINTANALYTICS`, `OFFICIAL`, `SUPPORT`, `ADMIN`, `HELP`
  - Generic: `FREE`, `TEST`, `DEMO`, `INVALID`, `EXAMPLE`
  - Profanity: TBD list before launch

### 10.5 Partner self-deactivation (LOCKED)

- Partner can deactivate their own active codes via portal
- Deactivation is **irreversible** — code cannot be reactivated; string reserved per uniqueness rule (12-month cooldown)
- Admin receives email notification when partner deactivates a code
- Audit log records: actor, code, timestamp, reason (free-text from partner, optional)

### 10.6 Concurrent slot accounting (LOCKED)

- Deactivation **refills** the concurrent-active-codes counter immediately
- Partner can create a new code right after deactivating an old one
- Anomaly detection: >3 deactivations in 30 days raises a risk alert for admin review (covers cycling abuse)

### 10.7 Discount types (PROVISIONAL)

At launch, supported discount types:

- **Percentage off first month** (e.g., 20% off first month)
- **Fixed dollar off first month** (e.g., $20 off first month)

NOT supported at launch (deferred):

- First N months free
- Lifetime percentage discount
- Free trial extension

**Note:** Founders have not finalized this — Jason flagged for future discussion. Working assumption; data model designed to support all five types when added.

### 10.8 Product applicability (LOCKED)

- Discounts apply universally to any product/SKU at launch
- Per-product / per-tier promo codes deferred to a later phase
- Data model includes `applies_to_product_ids` array (NULL = universal) so structure is ready

### 10.9 Stacking rules (LOCKED)

- Promo code stacks with **free trial only** (free trial runs first, discount applies to first paid period)
- Does NOT stack with another promo code (one code per checkout)
- Does NOT stack with free access seat (free seat is full access, no discount applicable)
- Does NOT stack with grandfather pricing or other admin discounts (admin discount wins)

### 10.10 Code lifecycle (LOCKED)

A code's status transitions:

- `pending → active` (when partner creates and confirms)
- `active → expired` (expiry date passes OR max uses reached)
- `active → deactivated` (partner or admin deactivates)

Once not `active`, code cannot be used at checkout.

String reserved for 12 months after the transition out of `active`, then can be reused by anyone.

Audit log captures every transition.

### 10.11 Partner per-code visibility (LOCKED)

**Phase 1a:** Read-only list. Columns shown but no actions:

- Code string (uppercase)
- Status (active / expired / deactivated)
- Discount value (e.g., "20% off first month")
- Uses so far / Max uses (e.g., "12 / 50")
- Created date
- Expiration date
- Total attributed revenue (USD)
- Total earned commission (USD)

**Phase 3:** Adds self-service actions: Deactivate (only on active), View details, "Create new code" button with admin-configured guardrails.

### 10.12 Phase placement summary

| Subsection | Phase 1a | Phase 2 | Phase 3 |
|---|---|---|---|
| 10.1 Uniqueness logic | ✅ enforced (admin-creates) | — | ✅ extends to partner-creates |
| 10.2 Conflict UX | — | — | ✅ shown to partner during create flow |
| 10.3 Format validation | ✅ enforced (admin-creates) | — | ✅ shown to partner during create flow |
| 10.4 Reserved word list | ✅ hard-coded constant from `docs/reserved-promo-codes.md` | ✅ admin UI to manage list | — |
| 10.5 Self-deactivation | — | — | ✅ |
| 10.6 Concurrent slot accounting | — | — | ✅ |
| 10.7 Discount types | ✅ admin-creates supports both types | — | — |
| 10.8 Product applicability | ✅ admin-creates supports universal | — | — |
| 10.9 Stacking rules | ✅ enforced at checkout | — | — |
| 10.10 Code lifecycle | ✅ admin-driven transitions | — | ✅ partner-driven transitions |
| 10.11 Partner per-code visibility | ✅ read-only list | — | ✅ self-service actions |


---

## 10A. Referral URL dispatch and reserved slug list

This section is new in v0.3 and reflects the URL format change from `wavepointanalytics.com/r/{slug}` to `wavepointanalytics.com/{slug}`.

### 10A.1 URL format (LOCKED)

- **Production format:** `https://wavepointanalytics.com/{slug}`
- The `/r/` namespace prefix is **eliminated** for cleaner, more brandable URLs (e.g., `wavepointanalytics.com/drysdale`)
- Slugs share the namespace with marketing site routes; collision prevention via reserved slug list (§10A.4)
- Phase 1a deliverable

### 10A.2 Worker dispatch logic (LOCKED — Phase 1a)

The existing Cloudflare Worker that handles hostname dispatch (per §1) gains a path-based dispatch layer for the marketing site:

```
1. Request arrives: GET https://wavepointanalytics.com/{path}
2. Worker checks: is {path} a reserved marketing route?
   - Static reserved list: /about, /pricing, /contact, /blog, /login, /signup,
     /terms, /privacy, /dpa, /products/*, /docs, /support, /careers, /api/*,
     /assets/*, /branding-assets/*, etc. (full list maintained in Worker code +
     docs/reserved-slugs.md)
   - If yes → pass through to marketing site, no slug lookup
3. If not reserved, Worker queries the slug cache (Cloudflare KV):
   - Cache key: slug
   - Cache value: { entity_type, entity_id, is_active }
   - Cache TTL: 5 minutes; invalidated on slug create/change in admin portal
4. If slug found and active:
   - Set Rewardful tracking cookie server-side
   - Record click event in Supabase via async fetch (don't block response)
   - 302 redirect to marketing site root (or /signup) with cookie set
5. If slug not found in KV:
   - Fall through to marketing site as a normal 404
   - Marketing site shows its standard 404 page
```

The Worker has a strict performance budget: dispatch decision must complete in < 50ms typical case (KV lookup is fast).

### 10A.3 Slug uniqueness rules (LOCKED — Phase 1a)

- Globally unique across both partner_orgs and affiliates (already locked in §5).
- Slug cannot match any entry in the reserved slug list (§10A.4).
- Slug regex: `^[a-z0-9]([a-z0-9-]{0,30}[a-z0-9])?$` — lowercase alphanumeric with optional hyphens, 1–32 chars.
- Slug case-insensitive at lookup; canonical form stored as lowercase.
- 1-year retention after slug change (existing slug 301-redirects to new slug for 1 year).

### 10A.4 Reserved slug seed list (LOCKED — Phase 1a)

Phase 1a hard-codes a reserved slug list as a TypeScript constant in the Worker. Phase 2 builds the admin UI to manage this list as a database table with audit logging.

The seed list is maintained in `docs/reserved-slugs.md` (mirror of `docs/reserved-promo-codes.md`) and includes at minimum:

**Marketing site routes (current and likely-future):**

`about`, `pricing`, `contact`, `blog`, `login`, `signup`, `signin`, `signout`, `register`, `dashboard`, `account`, `app`, `api`, `support`, `help`, `docs`, `documentation`, `terms`, `privacy`, `dpa`, `legal`, `cookies`, `careers`, `team`, `community`, `events`, `press`, `partners`, `affiliates`, `referral`, `referrals`, `r`, `track`, `redirect`, `link`, `links`, `landing`, `home`, `index`, `products`, `product`, `download`, `downloads`, `assets`, `static`, `branding-assets`, `branding`, `media`, `images`, `img`, `js`, `css`, `fonts`, `favicon`, `robots`, `sitemap`, `feed`, `rss`, `404`, `error`, `maintenance`, `status`, `health`, `ping`

**Brand and system terms:**

`wavepoint`, `wavepointanalytics`, `wp`, `wpa`, `official`, `staff`, `admin`, `administrator`, `root`, `owner`, `founder`, `ceo`, `cto`, `coo`, `team`, `wavepointteam`

**WavePoint indicator and product terms:**

`indicator`, `indicators`, `ninjatrader`, `nt`, `tradingview`, `tv`, `futures`, `nq`, `es`, `cl`, `gc`, `vix`

**Standard reserved technical terms:**

`null`, `undefined`, `none`, `test`, `demo`, `example`, `sample`, `tbd`, `placeholder`

**Trademark / competitor / regulatory terms (high-risk for false-endorsement claims):**

`thinkorswim`, `tos`, `interactivebrokers`, `ibkr`, `tdameritrade`, `schwab`, `etrade`, `robinhood`, `webull`, `tradestation`, `cme`, `ice`, `cftc`, `nfa`, `sec`, `fed`, `federalreserve`

**Misleading / regulatory-risk terms** (full list in `docs/reserved-slugs.md`):

`guaranteed`, `riskfree`, `noloss`, `secret`, `insider`, `winning`, `profitable`, `pro`, `expert`, `master`

The full list is maintained in `docs/reserved-slugs.md` and reviewed annually or whenever a partner reports a slug they wanted is blocked.

### 10A.5 Marketing site route additions (LOCKED — Phase 1a)

Marketing site adds new routes that conflict with existing partner slugs:

- **Conflict check at marketing-site-route-add time:** the team adding a new marketing route must check `docs/reserved-slugs.md` and the live `referral_slugs` table to confirm no existing partner has that slug
- If a conflict exists: either rename the marketing route, or contact the partner to negotiate a slug change with 30-day notice
- This responsibility documented in `docs/reserved-slugs.md` as a maintenance contract

### 10A.6 Phase placement summary

| Aspect | Phase 1a | Phase 2 | Phase 3 |
|---|---|---|---|
| Worker dispatch logic | ✅ | — | — |
| Reserved slug list as Worker constant | ✅ | — | — |
| Reserved slug admin UI to manage list | — | ✅ | — |
| KV cache for slug lookups | ✅ | — | — |
| Click tracking via async fetch to Supabase | ✅ | — | — |


---

## 11. QR codes

> **Phase placement (v0.3):** QR codes in **Phase 1a** are generated **manually by admin** via a free third-party tool (qr-code-generator.com), saved as a single PNG, and uploaded to the partner's R2 toolkit folder by the admin during onboarding (see `docs/Partner-Onboarding-Runbook.md`). The self-hosted Cloudflare Worker generation pipeline, SVG output, regeneration UI, and slug-change auto-regeneration described in §11.1–§11.8 all move to **Phase 3**. Partners get a working QR for marketing materials from Day 1 without the engineering investment.

### 11.1 Generation (LOCKED — Phase 3)

- **Self-hosted** using npm `qrcode` library
- Runs on Cloudflare Worker (admin-triggered endpoint, not partner-facing)
- Zero cost, zero rate limits, full control
- No third-party dependency

### 11.2 Output formats (LOCKED)

- **Both SVG and PNG** generated and stored
- SVG for: scaling, web embedding, print at any size
- PNG generated at **1024×1024px** — covers print up to ~3.4 inches at 300 DPI

### 11.3 Logo embedding (LOCKED)

- WavePoint logo centered in QR code
- ~20% of QR size
- White circular background behind logo for contrast
- Error correction level H (30% redundancy) so QR remains scannable

### 11.4 Color (LOCKED)

- **Black on white** (highest scan reliability)
- Single canonical version generated for all partners and affiliates
- Branded color variants deferred; can be added as a second download option later

### 11.5 Storage (LOCKED)

- Cloudflare R2, public read access
- Path structure:
  - `partners/{partner_org_id}/qr.svg`
  - `partners/{partner_org_id}/qr.png`
  - `affiliates/{affiliate_id}/qr.svg`
  - `affiliates/{affiliate_id}/qr.png`
- Public URLs (QR contains non-sensitive referral URL, no need for signed URLs)

### 11.6 Encoded URL (LOCKED)

- QR encodes: `https://wavepointanalytics.com/{slug}` (path-based, prettier for marketing — `/r/` prefix eliminated in v0.3)
- Cloudflare Worker dispatches `/{slug}` requests by looking up the slug in the partner_orgs / affiliates table (see §10A for the dispatch logic and reserved slug list)
- The Worker sets the Rewardful tracking cookie server-side before passing through to the marketing site
- Reserved slug list ensures slugs cannot collide with marketing site routes

### 11.7 Regeneration (LOCKED)

- Admin-only regeneration capability
- Triggered manually (e.g., after slug change, after logo refresh)
- Each regeneration audit-logged with reason text from admin
- Partners cannot self-regenerate (request via support ticket if needed)

### 11.8 Slug change behavior (LOCKED)

- When admin changes a slug, **both old and new QRs are preserved**
- Old QR continues to work via the 1-year slug 301 redirect (locked §3 / §15)
- Partner has both versions available in their portal during the redirect window
- After 1 year, old slug dies; admin should ensure partner has switched before that


---

## 12. Statements

> **Phase placement (v0.3):** Statements in **Phase 1a** are **manual**. Admin clicks a "Generate statement" button per partner per month in the admin portal (PDF generated via `@react-pdf/renderer` running in Cloudflare Worker), downloads the PDF, and attaches it to a Gmail message manually. This is ~2 minutes per partner per month at 10-partner scale. Past statements are stored in R2 and downloadable from the partner portal.
>
> The **automated cron + Resend integration + opt-in toggle UI + automatic PDF email** described in §12.1, §12.3, §12.5, §12.10, §12.11 all move to **Phase 3**. The PDF generator (§12.4), R2 storage path (§12.6), statement contents (§12.7), affiliate variants (§12.8), and multi-Owner delivery (§12.9) stay in Phase 1a (the data model and PDF format are needed for the manual flow to work).

### 12.1 Cadence (Phase 3)

- **Monthly only** (no after-payout cadence in v1)
- Sent on the **1st of each month at 9 AM Pacific**
- After-payout cadence may be added later if partners request it

### 12.2 Format (LOCKED)

- HTML email body PLUS PDF attachment
- HTML: summary inline with link to portal for live data
- PDF: full statement, archival format

### 12.3 Email delivery (PROVISIONAL — Resend)

- **Resend** as the working choice
- $20/mo for 50k emails (more than sufficient at current scale)
- Modern API, React Email template support, pairs with the locked React stack
- DKIM/SPF/DMARC must be configured for `wavepointanalytics.com` before launch

**Note:** PROVISIONAL — Jason wants to think on this. Postmark is the strong alternative if maximum deliverability matters more than DX. Switch is straightforward (single integration point in the codebase).

### 12.4 PDF generation (LOCKED)

- **`@react-pdf/renderer`**
- Runs in Cloudflare Worker
- Same React template logic generates both HTML email body and PDF attachment (DRY)
- Zero third-party dependency, zero added cost

### 12.5 Opt-in default (LOCKED)

- **Default OFF** — partner explicitly opts in via toggle in portal settings
- Onboarding flow surfaces the toggle but doesn't pre-check it
- Both partners and affiliates have access to the toggle

### 12.6 Past statement download (LOCKED)

- Partner can download a PDF of any past month's statement from the portal anytime
- Independent of opt-in status (even partners not subscribed to email can download from portal)
- Statements stored in R2:
  - `partners/{partner_org_id}/statements/{YYYY-MM}.pdf`
  - `affiliates/{affiliate_id}/statements/{YYYY-MM}.pdf`
- Generated and stored on the 1st of each month regardless of opt-in (so historical access is always available)
- Retrieved via signed URLs with 5-minute expiry, through portal session

### 12.7 Statement contents (LOCKED)

**Header:** Partner/affiliate display name + statement period (e.g., "April 2026 Statement")

**Summary section:**

- Total commission earned this period
- Number of new conversions
- Number of refunds (clawbacks)
- Net commission for period
- Available to pay out
- In refund hold
- Lifetime totals (earned, paid, in hold)

**Activity detail:**

- Each new conversion: date, customer email per masking rule, product, commission earned
- Each refund/clawback: date, customer email per masking rule, product, commission reversed

**Payout info:**

- Last payout: date, amount, method (no banking detail beyond last-4)
- Next payout: estimated date, projected amount

**Footer:**

- Link to portal for live data
- Unsubscribe link (compliance with CAN-SPAM, GDPR)
- WavePoint contact info

### 12.8 Affiliate statements (LOCKED)

- Same format as partner statements
- Referral data masked per the affiliate masking rule (first 3 + last 3 chars of local part + full domain)
- Affiliates do not see "free seats" rows (they have none)

### 12.9 Multi-Owner delivery (LOCKED)

- All opted-in Owners on a partner_org receive their own individual copy
- No designated "billing contact" concept needed
- Each Owner can unsubscribe individually without affecting other Owners

### 12.10 Sender configuration (LOCKED)

- From: `statements@wavepointanalytics.com`
- Reply-To: `support@wavepointanalytics.com`
- DKIM/SPF/DMARC required for wavepointanalytics.com before first send

### 12.11 Unsubscribe behavior (LOCKED)

- Each statement email includes a one-click unsubscribe link
- Clicking unsubscribe sets `statement_email_opt_in=false` for that user immediately
- Unsubscribed user can re-opt-in via portal settings


---

## 13. Audit log

> **Phase placement (v0.3):** A working audit log ships in **Phase 1a** with the `audit_log` table, RLS append-only enforcement, app-level write path, and a basic admin table view. The **DB trigger fallback** (write path #2 in §13.1), the **searchable Audit Log page** with filters and JSON-diff row expansion (§13.2), and the **weekly Cloudflare Worker R2 export for 7-year backup retention** (§13.7) all move to **Phase 3**.
>
> Reason-text policy (§13.5), action type enum (§13.1), and partner-visible audit subset (§13.4) stay in Phase 1a. The partner-facing "Account Activity" page (§13.4) is a Phase 2 deliverable since it depends on Admin Portal infrastructure.

### 13.1 Implementation approach (LOCKED)

- Single `audit_log` table in Postgres with the schema sketched in §5.2
- RLS policies enforce append-only (no UPDATE, no DELETE — even from app-level admin connection)
- Indexes on `(actor_user_id, created_at)`, `(target_type, target_id, created_at)`, `(action_type, created_at)`
- **Two write paths:**
  1. **App-level logging** — every admin write action wraps in a transaction that does the change + writes the audit row with rich `reason_text`, `before_jsonb`, `after_jsonb`, `metadata_jsonb`
  2. **Database trigger fallback** — on critical tables (`partner_orgs`, `affiliates`, `promo_codes`, `user_role_assignments`, `free_access_seats`, `payouts`, etc.), a trigger writes a baseline audit row on every UPDATE/INSERT/DELETE
- No Merkle chain hashing in v1 (overkill at current scale; revisit if SOC2 demands it)
- Action types defined as Postgres enum, with seed list:

```
role_assignment_granted, role_assignment_revoked, role_suspended, role_reactivated
partner_org_created, partner_org_updated, partner_org_suspended, partner_org_promoted_from_affiliate
affiliate_created, affiliate_updated, affiliate_suspended, affiliate_promoted_to_partner
team_member_invited, team_member_added, team_member_removed
promo_code_created, promo_code_deactivated, promo_code_admin_deactivated
free_seat_granted, free_seat_revoked, free_seat_redeemed
referral_attribution_locked, referral_attribution_overridden
payout_method_changed, payout_initiated, payout_completed, payout_failed, payout_reversed
manual_refund_issued, manual_chargeback_recorded, manual_clawback_applied
machine_id_reset
admin_impersonation_started, admin_impersonation_ended
slug_changed
ticket_created, ticket_replied, ticket_status_changed, ticket_reopened
admin_login, admin_login_failed
config_changed (reserved_slugs, reserved_promo_codes, etc.)
data_export_requested (GDPR)
data_deletion_requested (GDPR)
```

### 13.2 Reading the audit log (LOCKED)

The Admin Portal's Audit Log page provides:

- **Full-text search** across reason_text, target IDs, action types
- **Filters:** actor (specific user or role), target type, action type, time range
- **Sorted** reverse-chronologically by default
- **Row expansion:** click a row to see full before/after JSON diff (using a JSON diff viewer component)
- **Export:** CSV export of filtered results
- **Saved filters:** admin can save common queries

### 13.3 Reader policies (LOCKED)

- **All admins see all audit log rows** (across all actors and targets)
- **Plus** every authenticated user sees rows where they were the actor (so an admin who is also a customer sees their own user-level actions when reviewing their own activity)
- Implementation via the RLS policy combining admin-role-check OR `actor_user_id = auth.uid()`

### 13.4 Partner-visible audit subset (LOCKED)

Partners see audit rows in their portal that affect **their own org**:

**Includes:**

- Team adds/removes
- Promo code creations/deactivations
- Free seat grants/revocations
- Role assignments within the org
- Slug changes by admin affecting them
- Payout events
- Attribution overrides affecting them
- Refunds/clawbacks affecting them

**Excludes:**

- Cross-org actions
- Admin-only system events
- Other partners' activity
- Financial details about other entities

Filtered via RLS: `partner_org_id = (lookup of viewer's org)` AND `action_type` in the partner-visible subset list. Rendered in the partner portal as a "Account Activity" page (separate from admin-only audit log; uses same data source via filtered query).

### 13.5 Reason text policy (LOCKED)

**Required for sensitive actions:**

- Payout overrides (changing amounts before sending)
- Attribution overrides (manually moving a customer between referrers)
- Role changes (granting/revoking admin/partner/affiliate)
- Deactivations (partner_org, affiliate, promo code, free seat — anything turned off)
- Slug changes
- Refunds
- Manual clawbacks
- Free seat revocations
- Suspensions
- Admin-deactivating partner-facing items
- Admin impersonation start

**Optional everywhere else** (login events, viewing pages, routine config, etc.)

UI enforcement: actions in the required list show a modal that requires reason text before submit; backend validation enforces non-empty reason for these action types. Reason text minimum length: 5 characters (prevents "x", "ok", etc. as bypass).

### 13.6 Retention (LOCKED)

- **Database retention: indefinite**
  - audit_log rows never deleted from live Postgres
  - Storage volume negligible at WavePoint's scale (<1 GB at v1, <10 GB at 10x growth)
  - Indexes handle query performance
- **Backup retention: 7 years**
  - R2 weekly exports retained for 7 years (matches financial records standard)
  - After 7 years, expired backups automatically purged via R2 lifecycle policy
  - GDPR right-to-erasure handled separately: when a user requests deletion, audit_log rows referencing them have PII fields nulled (actor name, IP) but the action records remain (anonymized) for compliance integrity

### 13.7 Backup strategy (LOCKED)

**v1: Supabase Pro daily backups + Cloudflare Worker weekly export to R2.**

- R2 bucket `wavepoint-audit-archive` with versioning enabled
- Worker exports the audit_log table (and other critical tables: `partner_orgs`, `affiliates`, `referral_attributions`, `payouts`, `promo_codes`, `free_access_seats`, `tickets`, `user_role_assignments`) weekly
- Storage estimate: <1 GB at current scale, <10 GB at 10x; cost is pennies/month

**Documented trigger to add S3 Glacier Deep Archive layer:** any of (a) preparing for SOC2 audit, (b) crossing 100+ partners, (c) onboarding a partner whose contract requires cross-cloud backup, (d) annual review identifies it as worth the time.

**Annual integrity check:** pull a random monthly archive, verify it restores cleanly, log verification in runbook.


---

## 14. Monitoring and status

> **Phase placement (v0.3):** **Phase 1a uses internal Uptime Kuma only** (`uptime.yingson.com`). Monitors for WavePoint services are configured with the `wavepoint` tag and Discord-channel alerting (already established pattern from existing infrastructure). The public status page at **`status.wavepointanalytics.com`**, the Cloudflare Worker polling + D1 cache architecture (§14.2), the name-rewrite mapping (§14.3), and the public-facing status page UI (§14.4–§14.6) all move to **Phase 3**.
>
> If something breaks in Phase 1a, partners receive a Discord/email notification directly from the admin team. At 10-partner scale this is faster and more personal than a status page.

### 14.1 Deployment for v1 (LOCKED)

- **Path A: Use existing Yingson Labs Uptime Kuma (`uptime.yingson.com`) for v1.**
- **Documented migration trigger to Path B (separate WavePoint Kuma instance):** any of (a) preparing for SOC2 audit, (b) selling/restructuring WavePoint, (c) first enterprise partner asks about monitoring infra, (d) annual review identifies it as worth migrating.
- v2 plan: stand up a separate Uptime Kuma for WavePoint on a dedicated VPS at a WavePoint-owned domain.

### 14.2 Cached time-series architecture (LOCKED)

A Cloudflare Worker (cron) runs every 60 seconds:

- Fetches public status JSON from Kuma at `uptime.yingson.com`
- Filters to monitors tagged `wavepoint`
- Applies name-rewrite mapping (internal → public-friendly)
- Writes current snapshot to Cloudflare D1 + appends to history table

**Cloudflare D1 holds:**

- `monitor_current_status` (1 row per monitor, overwrite-on-poll, fast reads)
- `monitor_status_history` (append-on-poll, drives 90-day uptime charts and history)
- `incidents` (ranges of "not operational", manual or auto-detected)

Storage volume estimate: ~12 monitors × 1 poll/min × 525,600 minutes/year = ~6.3M rows/year of small data, well within D1 free tier.

**Benefits:**

- Resilience to Yingson Labs outages (last-known state served while home lab is down)
- No traffic from random visitors hitting Yingson Labs (public page reads from Cloudflare's edge)
- Custom retention/history independent of Kuma's settings
- Future-proofs Path B migration (only the cron Worker's source URL changes)

### 14.3 Two surfaces (LOCKED)

1. **Internal admin dashboard** — traffic-light strip + detailed status widget; shows technical monitor names; reads from D1.
2. **Public status page** at `status.wavepointanalytics.com` — fully WavePoint-branded, zero leak of Yingson Labs origin; shows rebranded names; reads from D1.

Both surfaces use the same data source (single source of truth).

### 14.4 Monitor list (LOCKED)

| Monitor (internal name) | Type | Public name |
|---|---|---|
| `stripe-webhook-handler` | HTTP probe | Payment Processing |
| `rewardful-sync` | Push (heartbeat) | Affiliate Tracking |
| `ninjatrader-delivery` | Push (heartbeat) | Indicator Delivery |
| `risk-alert-threshold` | Push (custom check) | Internal Alerting |
| `supabase-health` | HTTP probe | Authentication & Data |
| `cloudflare-worker-api` | HTTP probe | API Layer |
| `marketing-site` | HTTP probe | Marketing Site |
| `customer-portal` | HTTP probe | Customer Portal |
| `partner-portal` | HTTP probe | Partner Portal |
| `admin-portal` | HTTP probe | Admin Portal |
| `resend-email-status` | API check | Email Delivery |
| `r2-bucket-reachability` | Push from Worker | Asset Storage |

All monitors tagged `wavepoint` for filter eligibility. New monitors added as new services are introduced.

### 14.5 Alert routing (LOCKED)

- **Discord webhook** to a `#wavepoint-alerts` channel + **email** to all admins for all alerts in v1
- SMS / phone escalation deferred to later phase if real on-call rotations are established
- Critical-only filtering configured per-monitor

### 14.6 Push monitor pattern (LOCKED)

- App-internal scheduled jobs (Rewardful sync, NinjaTrader delivery, R2 verification) POST a heartbeat to a unique Kuma push URL after each successful run
- If the heartbeat stops arriving for the configured interval, Kuma alerts
- Implementation: one HTTP POST per job at the end of its success path (~5 lines of code per job)
- Push URLs are secrets stored in Worker env / job config; never exposed publicly

### 14.7 Public status page UX (LOCKED)

- Hosted at `status.wavepointanalytics.com`
- WavePoint-branded (logo, fonts, color tokens from the design-tokens file)
- Sections:
  - Current overall status banner (operational / degraded / outage)
  - Per-component status grid (the renamed monitors)
  - 90-day uptime history chart per component
  - Recent incidents log (last 30 days)
- No auth required
- Mobile-responsive (768px+ baseline)
- Linked from customer portal footer, partner portal footer, support ticket creation flow


---

## 15. Onboarding flow

> **Phase placement (v0.3):** **Phase 1a is fully manual onboarding**, governed by `docs/Partner-Onboarding-Runbook.md`. Admin creates partner_org / affiliate rows directly in the database (or via a minimal admin form), generates the slug, manually generates a QR code via qr-code-generator.com, manually invites the user via Supabase Auth, and signs the Partner Agreement / DPA / Affiliate Agreement out-of-band (SignWell or PDF-by-email).
>
> The **single-page onboarding form** (§15.1), **magic-link invitation via Resend** (§15.6, §15.8), **welcome screen + setup summary** (§15.6), **embedded SignWell signing** (§15.5), and **automated onboarding notifications** (§15.9) all move to **Phase 3**.
>
> The slug reservation rule (§15.4), affiliate-specific differential (§15.3), pre-existing user handling (§15.7), and team member approval policy (§15.10) describe the underlying behavior the runbook implements manually in Phase 1a.

### 15.1 Flow structure (Phase 3)

**Single-page form** — no wizard, all fields visible at once. Admin fills what's needed and submits.

Required fields enforced at submit; optional fields can be deferred. Form sections (visual grouping, not step-by-step):

1. **Identity** — org name, contact name, contact email, entity type, platforms (NinjaTrader / TradingView / both)
2. **Economics** — commission structure, rate, promo guardrails (Partners only), free seat quota (Partners only)
3. **Assets** — referral slug input with uniqueness check + suggest-button, optional starter promo codes (Partners only)
4. **Agreement** — e-sign trigger (see §15.5)
5. **Invite** — send button, sends magic link via Resend with 7-day expiry

### 15.2 Application intake (LOCKED)

- **No queue, no self-service application form**
- Admin manually adds new partners/affiliates from scratch
- Application path remains out-of-band: prospect emails/contacts WavePoint → admin decides yes/no externally → admin enters them into onboarding form
- Self-service application form deferred (could be added at `wavepointanalytics.com/partners/apply` later if volume warrants)

### 15.3 Affiliate-specific differential (LOCKED)

When entity type is "Affiliate" instead of "Partner":

| Field | Behavior for Affiliate |
|---|---|
| Commission structure | Only "percentage" — no alternative offers for affiliates |
| Commission rate | Pre-fills to 15% (default); admin can override |
| Promo code guardrails | Hidden — affiliates have no promo codes |
| Free seat quota | Hidden — affiliates have no free seats |
| Optional starter promo codes | Hidden |
| Agreement | Affiliate Agreement (simpler doc covering revenue share, conduct, termination, no-disparagement) |
| DPA | **Required (PROVISIONAL — REVISIT)** — same DPA template as partners initially; can be specialized later |

**Note:** Affiliate DPA requirement is conservative. Even though affiliates only see masked PII per §4, requiring DPA provides legal cover. Jason flagged for Derek/George discussion; the requirement may be relaxed after legal counsel engages.

### 15.4 Slug reservation (LOCKED)

- **Slug reserved at form submit** (the moment admin commits the onboarding record)
- Stays reserved through the entire invite period regardless of activation outcome
- If invite is revoked (admin deletes pending record) or expires-and-not-resent for an extended period, slug returns to pool only after admin explicit cancellation
- Multi-admin safe: George and Jason can be onboarding simultaneously without slug collision races
- Admin sees pending-record slugs in admin portal alongside active slugs (visually distinguished as "pending"); other admins can see slug is taken and pick a different one

### 15.5 E-sign approach (LOCKED — SignWell)

**SignWell as the locked e-signature provider** (changed from v0.2's provisional BoldSign default).

- Already in use at WavePoint for other documents — no new vendor onboarding, no new compliance review
- Embedded signing supported (signer never leaves the WavePoint portal in Phase 3 self-service onboarding)
- API integration: documented, REST-based, fits cleanly into the React stack
- Phase 1a: SignWell or PDF-by-email out-of-band signing is acceptable; portal does not yet integrate with SignWell programmatically. Admin records signed-date, document version, and counterparty in the partner_orgs / affiliates row manually per the runbook.
- Phase 3: SignWell embedded signing integration in the onboarding wizard.

**Documents to host as templates on SignWell (when Phase 3 begins):**

- Partner Agreement (latest version with version tracking)
- Partner DPA (latest version)
- Affiliate Agreement (latest version, simpler than Partner Agreement)

Portal records each signing event: signer name, IP, timestamp, document version, SignWell envelope ID.

### 15.6 Magic-link recipient experience (LOCKED)

When recipient clicks the magic link, they see a **welcome screen + setup summary**:

- "Welcome to WavePoint, [Display Name]"
- Role granted: "Partner Owner of [Org Name]" / "Affiliate" with a brief description
- Setup summary: their commission rate, referral link, QR preview, payout method status, agreement-signed status
- "Set up your account" button → if first time: prompts for password (or proceeds via passwordless if magic-link-only is configured) + display name confirmation
- After setup → redirects to their portal Dashboard

This serves as both onboarding affirmation and orientation; reduces "what is this?" support tickets.

### 15.7 Pre-existing user becoming Partner/Affiliate (LOCKED)

- Magic link adds the new role to their existing account; **no new account created**
- They retain their `user` role and customer subscription unchanged
- They gain `partner_owner` or `affiliate` role addition
- Login flow unchanged — same email, same password, just sees the new portal in their nav after next login
- Welcome screen still appears (shorter version: "You've been added as Partner Owner of [Org]" — skips password creation since account exists)

### 15.8 Re-sending invitations (LOCKED)

- Admin can re-send if expired (7-day window per send)
- Slug stays reserved during the entire invite period including re-sends
- No formal cap on re-sends (audit log captures each, so abnormal patterns visible)
- Admin can revoke a pending record to free the slug back to the pool

### 15.9 Onboarding notifications (LOCKED)

Email admin team when:

- Invitation accepted (recipient clicked magic link, completed setup)
- Invitation expired without acceptance (after 7 days)
- Each completed step of onboarding process (e.g., "Drysdale Trading Group has signed the Partner Agreement", "Drysdale Trading Group has set their payout method")

First-login is captured in audit log only, no separate email.

Notifications sent via Resend. Each notification is a passive heads-up, not actionable urgency.

### 15.10 Team member approval (LOCKED)

- **Owner-approves-only up to a cap of 5 team members per partner_org**
- Above the cap, admin approval is required to raise the cap for that specific partner
- Cap is admin-configurable per partner (default 5)
- All add/remove actions logged to audit log with timestamp, actor, target, and reason text
- All add events trigger email notification to admins (passive monitoring, not gating)
- An Owner cannot remove themselves if they are the last Owner (see §3.8)
- Removed team members lose `partner_member` role assignment but retain `user` role
- Inviting a brand new email (no existing user account) creates a pending invitation that expires in 7 days


---

## 16. Customer co-branding and segmentation

> **Phase placement (v0.3):** Of the three integrated capabilities in §16.1:
> - **Auto-attribution tagging** ships in **Phase 1a** (the `referral_attributions` table, the partner-facing referrals view scoped to their org)
> - **Customer-facing co-branding display** (the `ReferredByCard` widget in the customer portal, 30-day window logic, partner toggle) moves to **Phase 3**
> - **Internal segmentation** (admin filter by attribution) is part of the full Admin Portal in **Phase 2**
>
> The data fields needed for co-branding (`partner_org_public_profile`, `affiliate_public_profile`, `attributed_to_partner_id`, `referred_at`, `cobranding_dismissed_at`) ship in Phase 1a so the schema is forward-compatible. Only the customer-facing display surface is gated.

### 16.1 Three integrated capabilities (LOCKED)

When a customer is attributed to a partner_org or affiliate, three things happen:

1. **Auto-attribution tagging** — the system automatically writes a `referral_attributions` row identifying the partner_org or affiliate. No manual admin action needed. Partners see attributed customers in their portal automatically (RLS scoped to their org_id).

2. **Customer-facing co-branding** — the customer's user portal shows a subtle co-branding element acknowledging the referrer.

3. **Internal segmentation** — admin tooling can filter customer records by attribution metadata for ops/marketing purposes.

### 16.2 Customer-facing co-branding details (LOCKED)

**Visual treatment:**

- Customer's user portal dashboard shows a small "Referred by [Display Name]" line in a footer/sidebar area
- If the partner has uploaded a small avatar/logo (optional), it appears alongside the line
- Clickable link opens a non-intrusive bio popup (referrer's display name, optional bio, optional public link to their content)
- Affiliates: same treatment but uses affiliate display name; less prominent surface (no logo at launch, just text and bio)

**Partner controls:**

- Display name (required, set at onboarding)
- Optional bio (1-2 sentences, edited via portal settings)
- Optional avatar/logo upload (Partners only at launch)
- Optional public link (e.g., their own website, YouTube channel)
- All fields subject to admin moderation (admin can blank out a field with audit log if needed)

**Visibility default — opt-out for first 30 days, then auto-hide:**

- Default ON for first 30 days post-signup
- After 30 days, the card auto-hides automatically
- Customer can dismiss earlier via "X" button on the co-branding card
- Implementation: time-based suppression check on every page load (`now() - signup_date < 30 days AND not_dismissed_by_user`)
- This applies regardless of partner toggle (partner toggle OFF takes precedence — no co-branding shown at all)

**Partner co-branding toggle:**

- Each partner/affiliate has a setting in their portal profile: "Show me as referrer in customer portals" (default: ON)
- When toggled OFF, all attributed customers immediately stop seeing the co-branding card
- Attribution is still tracked internally (commissions still calculated, segmentation still works) — only the customer-facing display is suppressed
- Admin can also force-disable a partner's co-branding (with audit log + reason) for cases of inappropriate content
- Setting located at: Settings → Public Profile → Co-branding visibility

**Open question for Derek/George discussion:** is 30 days the right window, or should it be 14 / 60 / 90?

### 16.3 Internal segmentation (LOCKED)

Admin portal Customers page supports filters by attribution metadata:

- Filter customer list by referrer (e.g., "show all customers referred by Drysdale Trading Group")
- Filter by referrer tier or status
- Filter by attribution source (referral link / promo code / free seat / admin manual)
- Filter by attribution age (e.g., "referred customers in past 30 days")
- Cross-product slicing (referred to NinjaTrader vs TradingView)

### 16.4 Marketing automation depth (LOCKED — Option A at launch, with progression path)

**Ship with Option A: just segmentation, manual actions only at launch.**

- Admin filters customer list by attribution
- Admin can export the filtered list to CSV
- Admin manually composes emails or campaigns externally (Resend, Gmail, etc.)
- Admin can add internal tags/notes to customers via segment selection

**Documented progression path:**

- **v2:** Add Option B — segmentation + simple template-based campaigns (admin-managed templates, send-to-segment workflow, audit logging, suppression list management)
- **v3+:** Add Option C — buy a third-party tool like Customer.io, Klaviyo, or ActiveCampaign rather than building full automation in-house. Integrate with WavePoint's segmentation data via webhook/API push.

**Triggers to revisit:**

- Admin manual campaigning becomes a meaningful time burden (>4 hours/week)
- WavePoint crosses 1,000 active customers and segmentation use cases multiply
- Partner volume exceeds 50, making per-partner campaigns useful
- Specific marketing strategy needs lifecycle automation (welcome series, churn-prevention sequences, etc.)


---

## 17. Support tickets

> **Phase placement (v0.3):** **Phase 1a does NOT ship in-portal tickets.** Partners and affiliates email `support@wavepointanalytics.com` directly. The partner portal exposes:
> - **A prominent "Need help?" callout** on the Dashboard with the support email address
> - **A footer link on every page**: "Need help? support@wavepointanalytics.com"
> - **A separate footer link** for feature requests: "💡 Suggest a feature → feedback@wavepointanalytics.com"
>
> The full in-portal ticket system described in this section (§17.1–§17.14) — threaded UI, categories, ticket numbering, admin assignment queue, priority, internal SLAs, reopen window, email notifications — moves to **Phase 3**.
>
> **Operational requirement (Phase 1a):** `support@wavepointanalytics.com` must be actively monitored with a stated 1-business-day response SLA. `feedback@wavepointanalytics.com` is acknowledged within 1 week. Both are documented internally as part of the launch readiness checklist.
>
> When Phase 3 ships in-portal tickets, the historical email-based threads remain archived in Gmail; new conversations move to the portal. Phase 3 may add a "Feature Request" category that absorbs the `feedback@` channel.

### 17.1 Approach (Phase 3)

**In-portal tickets with email notifications.**

- Partners/affiliates create and reply to tickets entirely in the portal (threaded UI)
- Admins reply to tickets entirely in the admin portal
- Emails serve as notifications only — they alert admins/partners that something needs attention with a link back to the portal
- No email-to-ticket inbound parsing in v1 (avoids parsing fragility, threading conflicts, signature stripping issues)

This pattern matches modern helpdesks (Linear, GitHub Issues, Plain.com, etc.).

### 17.2 Customer-side tickets (LOCKED — DEFERRED)

- Customer (regular paying user) ticket system **deferred** to a later phase
- Existing customer portal "submit support ticket" button continues to send email to `support@wavepointanalytics.com` (current behavior preserved)
- Discord remains the primary customer support channel for v1
- This PRD scopes tickets to partners and affiliates only

### 17.3 Schema sketch

```sql
tickets
─────────
id
created_by_user_id (FK)
created_by_role (enum: partner_owner / partner_member / affiliate)
partner_org_id (FK, nullable for affiliates)
affiliate_id (FK, nullable for partners)
category (enum)
priority (enum: low / medium / high — default medium)
subject (text, max 200 chars)
status (enum: open / pending_user / pending_internal / resolved / closed)
assigned_to_admin_id (FK, nullable)
created_at, updated_at, last_activity_at, resolved_at, closed_at

ticket_messages
─────────────
id
ticket_id (FK)
author_user_id (FK)
author_role (enum: user / admin / system)
body (text, markdown supported)
created_at
```

**Note:** No `ticket_attachments` table or `is_internal_note` column in v1.

### 17.4 Categories (LOCKED)

**Partners — 9 categories:**

- General Help
- Indicator Issue
- Billing or Payout Dispute
- Refund or Clawback Dispute
- Content Request
- Team Management Help
- Marketing Asset Request
- Feature Request
- Other

**Affiliates — 7 categories** (drop "Content Request" and "Team Management Help"):

- General Help
- Indicator Issue
- Billing or Payout Dispute
- Refund or Clawback Dispute
- Marketing Asset Request
- Feature Request
- Other

### 17.5 Ticket numbering (LOCKED)

- Format: `WP-{YEAR}-{SEQUENTIAL}` (e.g., `WP-2026-0142`)
- Sequential resets per year
- Year prefix makes archival readable
- ID shown in portal, in any email subjects, and in admin views

### 17.6 Admin assignment (LOCKED)

- **Unassigned queue model**
- All admins see all unassigned tickets; first admin to claim works it
- Admin can manually reassign or unclaim
- No round-robin, no category-based routing in v1
- Works at 3-admin scale; revisit if admin team grows

### 17.7 Priority (LOCKED)

- Field defaults to `medium` on creation
- Admin can adjust priority (low / medium / high)
- Partners cannot self-set priority (avoids gaming)
- Priority changes audit-logged

### 17.8 SLA (LOCKED)

- **No SLA shown to user** — internal target only
- Internal targets (admin-facing): high = same day, medium = 1 business day, low = 3 business days
- Internal targets visible in admin queue view (overdue tickets flagged)

### 17.9 Internal notes — NOT in v1 (LOCKED)

- No internal notes capability in v1; all messages public to ticket participants
- Admin coordination happens out-of-band (Discord / email) for now
- Can revisit if admin team grows and coordination becomes harder

### 17.10 File attachments — NOT in v1 (LOCKED)

- No attachments in v1
- If partner needs to share files, message in ticket directs them to email `support@wavepointanalytics.com` with the ticket ID in subject
- v2: add R2-backed attachment support with virus scanning

### 17.11 Spam / rate limiting (LOCKED)

- Soft cap: 5 tickets per partner/affiliate per hour (visible warning, not blocked)
- Hard cap: 20 tickets per hour (blocked, error message)
- Audit log captures rate-limit hits

### 17.12 Reopen behavior (LOCKED)

- Partner can reopen a closed ticket within **30 days** of closure
- After 30 days, ticket is permanently closed; new issues require a new ticket
- Reopen creates a new event in the ticket thread; admin notified

### 17.13 Email notifications (LOCKED)

**Partner/Affiliate receives email when:**

- Their ticket is created (confirmation with ticket ID + portal link)
- An admin replies (notification with link to portal — no ticket content in email body to encourage portal-based reading)
- Status changes to resolved (with portal link to view resolution and reopen if needed)

**Admin receives email when:**

- Any new ticket is created (goes to all admins; whichever claims it gets subsequent updates)
- Partner/affiliate replies on a ticket the admin has claimed (only the assigned admin receives the reply notification)

**No emails for:**

- Status changes between intermediate states (open → pending_user, etc.) — visible in the portal
- Internal admin assignment changes (no impact on partner)

**Email format:**

- Sent via Resend
- From: `tickets@wavepointanalytics.com`
- Reply-to: `tickets@wavepointanalytics.com` (replies-to-email not parsed in v1; bounce-back response directs user to portal)
- Subject format: `[WP-2026-0142] [Category] — [Subject]`
- Body: short notification with action link, not full ticket content (encourages portal-based engagement, prevents inbox sprawl)

### 17.14 Implementation notes

- **Partner-side ticket UI:** list view (filter by status, sort by recency), detail/thread view, "Create Ticket" form with category dropdown + subject + body (markdown supported), reply input box at bottom of thread
- **Admin-side ticket UI:** queue view (all unassigned, all mine, all open globally), detail/thread view with status controls, reply input, claim/unclaim/reassign actions
- All ticket events recorded in audit log (creation, reply, status change, claim, reassign, reopen)
- Mobile-responsive at 768px+ baseline


---

## 18. Operational features

> **Phase placement (v0.3):** Phase placement varies by subsection:
> - **§18.1 Admin impersonation** — full UI moves to **Phase 3**. Phase 1a substitute: admin uses Supabase service-role direct queries for debugging partner-reported issues. No UI work; admins already have the access.
> - **§18.2 Suspension state** — moves to **Phase 2** (Admin Portal). Phase 1a does not need the full freeze workflow at 10-partner scale; admin can manually deactivate via DB if absolutely needed.
> - **§18.3 Marketing toolkit assets** — ships in **Phase 1a** (R2 storage, asset gallery widget, manual upload by admin)
> - **§18.4 Risk alerts** — moves to **Phase 2** (full Admin Portal feature)
> - **§18.5 Manual override toolkit** — light version in **Phase 1a** (issue refund via Stripe dashboard, reset machine ID via Supabase, override attribution via SQL — all manual, documented in `docs/Admin-Operations-Runbook.md` to be drafted in Phase 1a). Full UI in **Phase 2**.
> - **§18.6 Free Access Seats** — Phase 1a is **read-only display + admin manual grant**. Partner-Owner self-service grant/revoke moves to **Phase 3**.

### 18.1 Admin impersonation (Phase 3)

Admin has a "Switch user" capability in the admin portal for support purposes.

**Flow:**

- Admin selects a user from the Customers, Partners, or Affiliates table
- Clicks "View as this user" action
- System creates an impersonation session: time-limited (default 60 minutes), audit-logged with reason text required, original admin session preserved separately
- During impersonation: admin sees the user's portal exactly as the user would, including their data, permissions, and any role-specific UI

**Visible banner on every page during impersonation:**

- Banner text: "🔒 Viewing as [User Display Name] — Admin: [Admin Name]"
- Banner color: distinctive orange (matches admin portal accent for consistency)
- "Exit impersonation" button always visible in banner
- Banner CANNOT be dismissed

**Visibility:**

- Impersonated user does NOT see any visual indication on their session — admins are invisible to users (privacy: don't reveal admin investigation patterns)

**Expiration:**

- Impersonation expires automatically after 60 minutes
- Admin must re-confirm with reason to extend

**Hard restrictions during impersonation:**

- Admin CANNOT change user's password, email, payout method, or any sensitive setting while impersonating
- Admin CANNOT make purchases, send messages, or take destructive actions while impersonating
- These actions, if attempted, redirect admin back to their own session with a "complete this from your admin view" message

**Audit log:**

- All impersonation events written to audit log: admin_user_id, target_user_id, started_at, ended_at, reason, IP, user_agent

### 18.2 Suspension state (LOCKED)

Admins have a "Suspend (do not deactivate)" option for partner_orgs and affiliates. This is a freeze state distinct from full deactivation.

**Suspension semantics:**

- Attribution frozen — new conversions during suspension flag as `attribution_pending_review` and don't count toward commissions until reviewed
- Portal access frozen — partner/affiliate sees a holding-page message ("Your account is under review — please contact support") instead of dashboard
- Existing customer subscriptions UNAFFECTED — suspension of partner role doesn't kill user/customer subscription
- Promo codes deactivated for the duration (re-activatable on resume)
- Free seat assignments preserved but no new grants allowed
- Team members retain accounts; cannot perform partner-portal actions
- Statement emails paused
- Co-branding hidden in customer portals during suspension

**Resumption:**

- Admin can resume from suspension; all frozen state restored
- Pending-review attribution decisions made one-by-one

**Escalation:**

- If investigation results in deactivation, suspension transitions to deactivation with all the deactivation cleanup behaviors locked in §3.7
- Required reason text on suspend, resume, and any escalation
- All actions audit-logged

### 18.3 Marketing toolkit assets (LOCKED implicitly)

- Stored in Cloudflare R2
- Asset gallery widget displays all available assets to partners and affiliates (same toolkit per §4.2)
- Partner/affiliate can download via the gallery
- Admin uploads new assets via admin portal (asset metadata: name, description, category, file URL, upload date)
- Categories: Logos, Banners, Social Media (formatted for various platforms), Videos, Email Templates, Press Kit
- All access logged for content access audit trail

### 18.4 Risk alerts (LOCKED implicitly)

Admin Portal Risk Alerts page surfaces:

- **Chargebacks** — recent customer chargebacks affecting partner attributions
- **Account sharing** — multiple distinct IPs / devices on a single subscription
- **Machine-ID resets** — frequent resets indicating subscription sharing attempts
- **Coupon velocity** — abnormal promo code redemption rates (spam, abuse)
- **Promo code cycling** — partners deactivating and recreating codes frequently (>3 in 30 days, per §10.6)
- **Self-referral attempts** — flagged when self-referral block triggers (§3.4)

Each alert: date, type, affected entity, severity, status (new / investigating / resolved). Admin can claim, investigate, mark resolved with reason.

### 18.5 Manual override toolkit (LOCKED implicitly)

Admin actions available from the Manual Override Toolkit on the Admin Portal:

- Issue a refund (Stripe-driven, audit-logged)
- Reset a customer's machine ID (NinjaTrader/TradingView reset)
- Override an attribution (move customer from referrer A to referrer B with logged reason)
- Apply a manual clawback (reverse a commission with reason)
- Force-unlock a stuck subscription state
- Cancel/issue/refund a payout

All actions require reason text per §13.5 and write to audit log.

### 18.6 Free Access Seats (LOCKED implicitly)

- Partner Owner can grant a Free Access Seat to a specific email address
- Recipient receives an invitation; on acceptance, gets full indicator access without payment
- Partner sees their free seat usage: granted / accepted / revoked
- Quota enforced per partner_org (default 5, admin-configurable)
- Admin can also grant seats independently of partner quota (for special cases, audit-logged)
- Free seats can be revoked by partner or admin; recipient loses access immediately
- Optional auto-expiry per partner_org config (`free_seat_auto_expire_days`)
- Free seat redemption recorded in `referral_attributions` with `attribution_source = 'free_seat'`


---

## 19. Environments and deployment

### 19.1 Environment strategy (LOCKED)

**Two environments at launch: Preview → Production.**

- **Preview** (`portal-preview.wavepointanalytics.com`) behind Cloudflare Access — for mockup review and pre-launch QA
- **Production** (`partners.wavepointanalytics.com`, `admin.wavepointanalytics.com`, `app.wavepointanalytics.com`, `wavepointanalytics.com`) — real customers, real money

**Staging deferred** — accept the risk for v1, can add later as a config flip when scale justifies it.

**Documented trigger to add staging:** any of (a) preparing for SOC2 audit, (b) crossing 50+ partners, (c) onboarding an enterprise partner that requires it, (d) annual review identifies it as worth the time.

**Status page production:** `status.wavepointanalytics.com`.

### 19.2 Multi-environment readiness from Phase A (LOCKED)

Config and secrets externalization is a Phase A foundation requirement, even though only Preview exists initially.

**All environment-dependent values come from environment variables, never hardcoded:**

- Supabase URL/key
- Stripe key
- Rewardful API key
- Resend API key
- SignWell API key (Phase 3)
- R2 bucket name
- Kuma source URL
- D1 database binding

Single `env.config.ts` file in `apps/portal` reads from environment, exports typed config object. Cloudflare Pages environment variable management used for per-branch overrides.

**Adding staging later becomes:**

1. Provision new Supabase project
2. Configure new environment variables in Cloudflare Pages for `staging` branch
3. Add DNS entries for staging subdomains
4. Push to `staging` branch — done

No code refactor required. This pattern matches the broader unification roadmap (§7) — same principle applied to design tokens applied to environment config.

### 19.3 Cloudflare Pages branch deployments

- **`main` branch** → `portal-preview.wavepointanalytics.com` (Preview environment, Cloudflare Access enabled)
- **`production` branch** (post-launch) → production URLs (Cloudflare Access removed; Supabase auth takes over)

Storybook deploys from a separate branch or build target to `widgets-preview.wavepointanalytics.com`.

### 19.4 Production cutover plan (LOCKED — high level)

When Production environment goes live (post-Phase F):

- Production Supabase project provisioned with full schema migrations applied
- Real Stripe live keys, real Rewardful production campaign, real Resend domain (DKIM/SPF/DMARC verified), real SignWell templates
- Cloudflare Access removed from production URLs (real Supabase auth takes over)
- Real customer data migrated from existing user_db (`https://eqozbdekcawelfbpgrit.supabase.co`)
- DNS cutover from preview to production for the customer portal happens carefully (zero-downtime if possible)
- Run-book documenting the cutover written before launch day

Detailed cutover plan to be developed during Phase F polish.

### 19.5 Domain layout

| Environment | Marketing | Customer | Partner | Admin | Status |
|---|---|---|---|---|---|
| Preview | n/a | (combined under preview URL with role switcher) | `portal-preview.wavepointanalytics.com/partner` | `portal-preview.wavepointanalytics.com/admin` | n/a |
| Production | `wavepointanalytics.com` | `app.wavepointanalytics.com` | `partners.wavepointanalytics.com` | `admin.wavepointanalytics.com` | `status.wavepointanalytics.com` |


---

## 20. Forward-looking unification (out of scope for this PRD)

This section documents the long-term direction so the PRD's architectural decisions are intelligible to a reader who joins the project later.

### 20.1 The current state

WavePoint currently has three web properties on different stacks:

- **Marketing site** at `wavepointanalytics.com` — vanilla HTML/CSS/JS
- **Customer portal** at `app.wavepointanalytics.com` — vanilla HTML/CSS/JS
- **Partner + Admin portals** (this PRD) — React + Vite + Tailwind + shadcn/ui + ECharts

This is a deliberate choice: rebuilding the marketing site and customer portal at the same time as building the new portals would extend the timeline significantly without adding partner value.

### 20.2 The eventual goal

Unify all four web properties into a single codebase or monorepo with shared packages:

- Same React + Vite + TypeScript stack
- Same widget library (`packages/widgets`)
- Same design tokens (`packages/design-tokens`) — single edit propagates to all four properties
- Same auth provider (Supabase)
- Single Cloudflare Worker handles hostname dispatch to the right app

### 20.3 What this PRD does to enable that future

- **Widget library structure** (§6) deliberately avoids portal-specific imports inside widgets, uses dependency injection for things like Supabase client and auth context. Widgets work in any host that wires up the provider.
- **Design tokens** (§7) are structured to be extractable into a shared `@wavepoint/design-tokens` package.
- **Repo structure** (§8) uses pnpm workspaces — additional `apps/` can be added when marketing site and customer portal are migrated.
- **Environment config** (§19) is externalized so additional apps consume the same env model.

### 20.4 What triggers the unification project

- New brand changes become painful (multiple repos, hand-syncing colors and fonts)
- New shared features want to live across multiple properties (e.g., a single sign-in component)
- Performance / SEO needs of marketing site outgrow vanilla HTML
- Engineering capacity exists to do the migration

The unification is a multi-week project when it happens. It is **explicitly out of scope** for this PRD. This PRD's contribution is making that future migration as low-friction as possible.


---

## 21. Open items requiring board ratification

This section consolidates every PROVISIONAL or REVISIT item that needs founder ratification, conversation with Derek/George, or input from legal counsel before launch. It exists to make sure nothing falls between the cracks.

### 21.1 Items requiring Derek/George conversation

| # | Topic | Current PRD assumption | Why it's flagged |
|---|---|---|---|
| 1 | Refund clawback window | 30 days | Not yet finalized at board level; affects Refund Hold KPI and legal docs |
| 2 | Default partner commission rate | Per-contract, no system-wide default | Both rates pending Derek/George conversation |
| 3 | Affiliate flat commission rate | 15% | Both rates pending Derek/George conversation |
| 4 | Discount types at launch | % off first month + $ off first month | Founders haven't finalized which to enable |
| 5 | Customer co-branding window | 30 days post-signup, then auto-hide | Open question — could be 14 / 60 / 90 instead |
| 6 | DPA required for affiliates | Yes (conservative) | Locked conservatively; may relax after counsel review of masked-PII-only exposure |
| 7 | E-sign provider | SignWell (LOCKED in v0.3) | No longer pending |
| 8 | Email delivery service | Resend | Jason wants to think more; Postmark is the strong alternative |

### 21.2 Items requiring legal counsel review

| # | Topic | Status |
|---|---|---|
| 1 | Privacy Policy v0.1 (15 lawyer-review notes flagged in v0.1 doc) | Needs counsel review |
| 2 | Terms of Service v0.1 (25 lawyer-review notes) | Needs counsel review |
| 3 | Partner DPA Template v0.1 (24 lawyer-review notes) | Needs counsel review |
| 4 | Partner Agreement (commercial terms — separate from DPA) | Needs drafting |
| 5 | Affiliate Agreement (simpler doc) | Needs drafting |
| 6 | Whether masked-PII-only data exposure justifies skipping DPA for affiliates | Needs counsel opinion |
| 7 | Whether IP-stamped click-through is sufficient for partner agreements (vs requiring formal e-signature) | Needs counsel opinion |
| 8 | All terminology rename "coupon" → "promo" should be reflected in legal docs before publication | Needs diff and re-review during counsel pass |

### 21.3 REVISIT triggers (locked decisions designed to be re-examined)

| Decision | Trigger to revisit |
|---|---|
| Uptime Kuma at Yingson Labs | SOC2 prep / 50+ partners / enterprise partner asks / annual review |
| Audit log backup adds S3 Glacier | SOC2 prep / 100+ partners / enterprise partner contract requires / annual review |
| Skip staging environment | SOC2 prep / 50+ partners / enterprise partner requires |
| SignWell as e-sign provider | If volume exceeds free tier limits or enterprise compliance needs change |
| Manual ACH payouts | Partner volume exceeds ~30 |
| Marketing automation depth | Manual campaigning >4 hrs/week or 1,000+ active customers |
| Rewardful as commission engine | Volume exceeds Rewardful's pricing tier sweet spot |
| Customer-side ticket system | If Discord support becomes inadequate at scale |
| File attachments on tickets | When v2 feature work begins |

### 21.4 Implementation-time unknowns

These are operational details that will be filled in during build, not design:

- Exact reserved word lists (slugs and promo codes) — admin to populate before launch
- Exact synthetic data fixture (10 partner names, 50 customer names) — generated during Phase A, adjusted in review
- Exact public-friendly monitor names mapping in the status page — generated in Phase F, refined with feedback
- Exact email template copy (statements, ticket notifications, onboarding magic link) — drafted in Phase F
- Exact wording on legal documents (depends on counsel review)

### 21.5 Items explicitly out of scope for this PRD

Documented for clarity:

- Rewrite of marketing site
- Rewrite of customer portal (only nav refresh is in scope)
- Customer-side ticket system
- File attachments on tickets
- Marketing automation beyond manual segmentation
- Multi-currency display
- Smaller phone breakpoints (375px)
- SMS / phone alerts in monitoring
- After-payout statement cadence
- Self-service application form for partners/affiliates
- White-labeling of the partner portal for top-tier partners

---

---

## 22. Security and operational integrity

This section is new in v0.3 and represents the first structured security treatment for the portal project. The companion document `docs/Security-Overview.md` provides plain-language summaries suitable for sharing with partners or future enterprise customers; this section is the engineering-facing source of truth.

The principle: **proportionate to risk, not absent from risk.** WavePoint at 10 partners and ~5,000 customers does not need SOC2 or a CISO. It does need defensive coding hygiene from Day 1, operational discipline, and a documented baseline so the question "what's your security posture?" has a real answer.

### 22.1 Threat model (LOCKED)

**Assets being protected:**

- Customer subscription data (PII, payment metadata, usage patterns)
- Partner financial records (commission balances, payout method, payout history)
- Admin role privileges (full data access across the system)
- Brand and reputation (a breach at this scale would be operationally survivable but reputationally severe)

**Threat actors considered:**

- **Financially-motivated external attackers** seeking to redirect partner payouts, drain commission balances, or steal customer payment data
- **Malicious partners** attempting to inflate their commissions via self-referral, promo code abuse, fake conversions, or attribution manipulation
- **Account takeover** of partner Owner or Admin accounts via credential stuffing, phishing, or session hijacking
- **Insider threat from the admin role** — three founders share full data access; if any account is compromised, the system is compromised
- **Data exfiltration** via SQL injection, RLS policy gaps, or misconfigured API endpoints
- **Supply chain attacks** via compromised npm packages or compromised SaaS dependencies (Supabase, Stripe, Rewardful, R2, SignWell)

**Out of scope (explicit):**

- Nation-state actors
- Sustained DDoS attacks (Cloudflare baseline protection considered sufficient)
- Physical security of customer devices
- Customer's own credential hygiene (covered by ToS but not technically enforced beyond MFA recommendation)

### 22.2 Authentication and session security (LOCKED)

**MFA REQUIRED for admin accounts (Phase 1a).** This is the single highest-leverage security control for this product. Implementation:

- Supabase Auth supports MFA via TOTP; admin role policy enforces `auth.users.mfa_factor_count > 0` before granting admin role assignment
- Admin without MFA cannot complete login — they get the MFA enrollment flow on first login
- MFA cannot be removed from an admin account without revoking the admin role first
- Audit log captures MFA enrollment, MFA challenge success/failure, MFA reset requests

**MFA OPTIONAL but strongly encouraged for partner_owner accounts (Phase 1a).** A "Recommended" badge appears in partner Settings if MFA is not enabled. Phase 3 may make this required for partners above a certain commission threshold (REVISIT trigger).

**Session length (LOCKED):**

- Admin sessions: 4 hours (then re-auth required)
- Partner / Affiliate sessions: 7 days
- User sessions: 30 days
- Idle timeout: 30 minutes for admin, none for others
- "Remember me" not offered for admin accounts

**OAuth scope minimization:**

- Google OAuth requests only `openid email profile` — no broader Google scopes
- No OAuth refresh tokens stored anywhere outside Supabase Auth's managed storage

**Magic-link and password policy:**

- Magic-link expiry: 7 days for onboarding invites (per §15.1 Phase 3), 1 hour for password reset
- Password requirements: minimum 12 characters, no maximum, breached-password check via Have I Been Pwned API at signup and password change
- No password rotation requirement (NIST SP 800-63B guidance)

### 22.3 Authorization and least privilege (LOCKED)

**RLS as the primary authorization mechanism.** Widget code does not implement authorization itself; every query relies on Postgres RLS to filter rows.

- Every table containing user, partner, or financial data has RLS enabled
- Default RLS policies DENY; explicit grants for specific roles
- Service-role key NEVER exposed to client-side code; only used in Cloudflare Workers
- Anon key used by the React app has minimal grants (just enough to call RPC functions)
- Admin role check is `(SELECT role FROM user_role_assignments WHERE user_profile_id = auth.uid() AND role = 'admin' AND is_active)` — checked inside RLS policies, never trusted from client

**Pre-launch RLS review requirement (Phase 1a):**

- Every RLS policy reviewed by a second person (initially, the second person is Claude in a structured review session; longer-term, a security-focused human reviewer)
- Test cases written for each policy: positive (the right user can access) and negative (the wrong user cannot access)
- Test cases run as part of CI before any production deployment

**Cloudflare Worker authorization:**

- Worker routes that proxy to Supabase pass through user JWT; do not bypass RLS
- Worker routes that act with elevated privilege (e.g., admin actions) verify admin role server-side before acting

### 22.4 Secrets and credentials (LOCKED)

**Storage:**

- All production secrets stored in Cloudflare Workers secrets (`wrangler secret put`)
- Never committed to repo (`.gitignore` enforces; pre-commit hook scans for common secret patterns via `detect-secrets`)
- Per-environment secrets (Preview vs Production)
- Local development uses `.env.local` files (`.gitignore`d); developer-specific values

**Rotation cadence:**

- Quarterly rotation: Stripe webhook signing secret, Supabase service role key, Resend API key (Phase 3)
- Annual rotation: Supabase database password, Cloudflare API tokens, Rewardful API key
- On-demand rotation: any time a secret is suspected of being compromised, when a person with access leaves the company, or after any incident

**Access controls:**

- Cloudflare Workers secrets: only Jason and George can deploy (admin role on Cloudflare account)
- Supabase Studio access: only admin team members
- Stripe dashboard access: only admin team members, all on MFA
- Each secret access logged in the originating system (Cloudflare audit log, Supabase audit log, Stripe audit log)

### 22.5 Input validation and output encoding (LOCKED)

**Input validation (Phase 1a):**

- **Zod schemas at every API boundary** — every Worker endpoint validates its input via a typed schema before processing
- Schemas reject unknown fields by default (`.strict()`)
- All user-supplied strings have explicit length limits matching DB column constraints
- All user-supplied integers have explicit min/max bounds
- All user-supplied URLs validated for scheme (https only) and pattern matching expected domains
- All file uploads (Phase 3 marketing assets) checked for file type, size, and scanned via Cloudflare for malware

**Output encoding:**

- React's default JSX escaping is the primary defense; **no `dangerouslySetInnerHTML` anywhere in the codebase**
- ESLint rule `react/no-danger` enabled and set to `error`
- User-generated content (partner bios, public profiles) rendered through React's default escape, never as raw HTML
- If markdown rendering is needed (Phase 3 ticket bodies), use `react-markdown` with restricted allowed-tags allowlist; no raw HTML pass-through

**SQL injection:**

- All DB access via Supabase client library (parameterized queries)
- No raw SQL string concatenation anywhere in the application code
- RPC functions used for any complex queries; never built as runtime string assembly

### 22.6 Transport and at-rest encryption (LOCKED)

**In transit:**

- TLS 1.3 everywhere (Cloudflare default for all subdomains)
- HSTS header with `max-age=63072000; includeSubDomains; preload`
- Submitted to the Chrome HSTS preload list before public launch
- Cookies: `Secure`, `HttpOnly`, `SameSite=Lax` for session cookies; `SameSite=Strict` for any sensitive operation cookies
- Cookie domain scoped explicitly (no parent-domain leakage)

**At rest:**

- Supabase Postgres: AES-256 encryption at rest (Supabase default)
- Cloudflare R2: AES-256 encryption at rest (R2 default)
- Cloudflare D1 / KV: encryption at rest (Cloudflare default)
- Backups: encrypted by Supabase Pro; weekly Worker exports to R2 are encrypted via the same R2 default

**Data classification:**

- **Public:** marketing site content, partner public profiles (when consented)
- **Internal:** aggregated analytics, partner conversion counts
- **Confidential:** customer email addresses, partner commission balances, payout amounts
- **Restricted:** customer payment metadata (handled by Stripe — WavePoint never stores card data), partner banking details (handled by manual ACH process — stored in admin-only Supabase columns with explicit RLS lock)

### 22.7 Dependency security (LOCKED)

- **Dependabot enabled** on the GitHub repo from Day 1, with automatic PR creation for security updates
- **`npm audit` runs as a blocking step in CI** — high or critical vulnerabilities block the deploy
- **Renovate (or Dependabot) weekly run** for non-security patch updates, batched into a single PR per week
- **No direct npm dependencies** without team review — adding a dependency requires a brief PR comment justifying it
- **License compatibility check** for every dependency (avoid copyleft licenses incompatible with proprietary distribution)
- **Lockfile committed** (`pnpm-lock.yaml`); deploys use `--frozen-lockfile`

**Supply chain hardening:**

- Subresource Integrity (SRI) hashes for any external CDN resources (currently none planned, but enforced if added)
- npm scripts reviewed for `postinstall` shenanigans on new dependencies
- Annually: review the full dependency tree for unused packages, removed via PR

### 22.8 Operational security (LOCKED)

**Account hygiene:**

- Quarterly access review (calendar reminder): list every account with admin or service-role access; confirm each is still required
- Immediate revocation when a person leaves the company (procedure in `docs/Admin-Operations-Runbook.md`)
- Shared accounts forbidden — every login is to an individual account

**Logging and observability:**

- Cloudflare Workers logs: 7-day retention (default), critical events forwarded to Supabase `audit_log`
- Supabase logs: 7-day retention on free/pro tier
- Application errors: structured logging via `console.error`, captured by Cloudflare; sampled to a future error-tracking service (Sentry consideration in Phase 3)
- Audit log (per §13) is the long-term record for security-relevant events

**Anomaly detection (Phase 2):**

- Risk Alerts page surfaces patterns: chargebacks, account sharing, machine-ID resets, coupon velocity, self-referral attempts (already specified in §18.4)
- Cloudflare WAF rules for obviously malicious patterns (Phase 1a — Cloudflare default ruleset)
- Failed login spike detection (Phase 2)

### 22.9 Incident response (LOCKED — Phase 1a stub, Phase 3 expansion)

**Phase 1a deliverable:** 1-page incident response runbook (`docs/Incident-Response-Runbook.md`) with the following structure:

1. **Detect** — what triggers a response: unusual error rates, suspicious admin login patterns, partner reports of unauthorized charges, dependency vulnerability alerts, third-party breach notifications
2. **Triage** — first 15 minutes: who to call (founders), severity scale (P0 = data breach in progress, P1 = potential exposure, P2 = remediation needed but no exposure, P3 = informational)
3. **Contain** — immediate actions: rotate compromised secrets, revoke compromised sessions (`auth.users` `revoke_all_sessions`), disable affected feature flag if applicable, notify Cloudflare/Supabase support if their layer is involved
4. **Eradicate** — root cause: code patch, config change, dependency update, RLS policy fix
5. **Recover** — restore service: deploy fix, verify behavior, communicate with affected users if applicable
6. **Review** — post-incident retrospective: what went wrong, what worked, what changes (technical or process) prevent recurrence

**Notification obligations:**

- Personal data breach: 72-hour notification to UK ICO / EU supervisory authority per GDPR Article 33; affected individuals notified per Article 34 if high risk
- US state breach laws vary; ToS reviewer (counsel) confirms baseline approach
- Partner notification if their attributed customers' data is involved

**Phase 3 expansion:** add tabletop exercise cadence (annually), automated alerting for security-relevant events, formal post-incident review template, public security disclosure page.

### 22.10 Compliance commitments (LOCKED)

**GDPR Article 32 — Security of processing:**

- Pseudonymization where practical (referral attribution uses internal IDs, not user emails)
- Encryption in transit and at rest (per §22.6)
- Ability to ensure ongoing confidentiality, integrity, availability (RLS, audit log, backups)
- Ability to restore availability (Supabase Pro daily backups, R2 versioning)
- Regular testing of effectiveness (quarterly RLS review, annual dependency audit, incident response runbook drills)

**CCPA reasonable security:**

- Encryption per §22.6
- Access controls per §22.3
- Audit log per §13

**Subprocessors (per Partner DPA):**

- Supabase, Stripe, Rewardful, Cloudflare, Resend (Phase 3), SignWell, Web3Forms (during transition; replaced by Resend in Phase 3)
- Updated subprocessor list maintained in `docs/Subprocessors.md` (to be drafted in Phase 1a)

### 22.11 Periodic review (LOCKED)

**Cadence:**

- **Quarterly:** access review (who has admin? Still needed?), secret rotation, RLS spot-check
- **Annually:** policy review (this section), dependency tree audit, threat model refresh
- **Pre-major-release:** security checklist review against this section

**Calendar:**

- Reviews scheduled as recurring calendar events in admin team's shared calendar
- Review completion logged in `docs/Security-Review-Log.md` (drafted in Phase 1a, appended each cycle)

**Triggers for elevated review:**

- Any P0 or P1 incident
- New regulatory requirement (e.g., new state privacy law)
- Adding a new subprocessor or major dependency
- Expansion to a new market (e.g., adding payment processing in a new country)

### 22.12 Phase 1a security checklist

The following items must be complete before Phase 1a deploys to production. Each maps to a section above.

| # | Item | Section |
|---|---|---|
| 1 | CSP header configured via Cloudflare Worker | §22.6 |
| 2 | HSTS header with preload submission | §22.6 |
| 3 | Secure / HttpOnly / SameSite cookies | §22.6 |
| 4 | Zod validation at every Worker API boundary | §22.5 |
| 5 | No `dangerouslySetInnerHTML` (ESLint enforced) | §22.5 |
| 6 | Dependabot enabled with auto-PR | §22.7 |
| 7 | `npm audit` blocking step in CI | §22.7 |
| 8 | All secrets in Cloudflare Workers secrets | §22.4 |
| 9 | Pre-commit secret scanner (`detect-secrets` or equivalent) | §22.4 |
| 10 | RLS enabled on every user/partner/financial table | §22.3 |
| 11 | RLS policies reviewed by second person; test cases written | §22.3 |
| 12 | MFA required for all admin accounts | §22.2 |
| 13 | MFA recommended (badge) for partner Owners | §22.2 |
| 14 | Service role key never exposed to client | §22.3 |
| 15 | Incident response runbook drafted | §22.9 |
| 16 | Quarterly access review calendar event scheduled | §22.11 |
| 17 | Subprocessor list documented | §22.10 |
| 18 | Cloudflare WAF default rule set enabled | §22.8 |
| 19 | TLS 1.3 enforced on all subdomains | §22.6 |
| 20 | Pre-launch security review session conducted | §22.11 |

### 22.13 Phase 2 / Phase 3 security additions

**Phase 2:**

- Risk Alerts page (§18.4)
- Audit log searchable UI (§13.2)
- Failed login spike detection
- Formal a11y audit (which is also a security adjacency — accessibility issues are sometimes injection vectors)

**Phase 3:**

- Audit log DB triggers (defense in depth on top of app-level logging)
- Weekly R2 export for 7-year retention
- Sentry or equivalent error tracking
- Tabletop exercise cadence
- Public security disclosure page (`security.txt` per RFC 9116, security@ contact, optional bug bounty)
- SOC2 Type 1 prep (REVISIT trigger: enterprise partner inquiries, or 50+ partners)

### 22.14 What we're explicitly NOT doing in v1

For traceability:

- **No SOC2 audit** — the current scale doesn't justify the cost (~$15k–$50k for Type 1) or the operational lift. Documented as a future REVISIT.
- **No formal penetration test** — Phase 3 may add a third-party pen test before any enterprise partner; in the meantime, defensive coding hygiene + RLS review is the substitute.
- **No bug bounty program** — too noisy at current traffic; Phase 3 consideration.
- **No HSM-backed key management** — Cloudflare Workers secrets are sufficient at this scale; HSM is a SOC2-era concern.
- **No formal STRIDE threat modeling sessions** — the §22.1 threat model is the lightweight substitute. Adopt STRIDE if/when SOC2 prep begins.
- **No certificate pinning in the React app** — Cloudflare-managed TLS is the trust anchor; pinning adds operational fragility without proportionate benefit at current scale.
- **No Web Application Firewall custom rules beyond Cloudflare default** — Phase 2 may add custom rules for specific abuse patterns once observed.

The principle: **don't pretend to be more secure than we are, and don't be less secure than we should be.** The above items are documented absences, not blind spots.

### 22.15 Companion documents

- `docs/Security-Overview.md` — partner / customer-facing summary in plain language
- `docs/Incident-Response-Runbook.md` — operational runbook (Phase 1a deliverable)
- `docs/Admin-Operations-Runbook.md` — admin daily operations including security-relevant tasks (Phase 1a deliverable)
- `docs/Subprocessors.md` — public list of subprocessors with locations and purposes (Phase 1a deliverable)
- `docs/Security-Review-Log.md` — append-only log of quarterly / annual reviews (Phase 1a stub)


---



This document represents structured decision-making across 15 topics from v0.2, plus the Phase 1a / 1b / 2 / 3 reprioritization and security framework added in v0.3. It is the single source of truth for the partner & admin portal build. Future revisions should retain the LOCKED / PROVISIONAL / REVISIT / PHASE markings to make decision provenance traceable.

When this document is updated, the version number should be incremented (v0.3, v0.4, etc.) and a CHANGELOG entry added describing what changed and why.

