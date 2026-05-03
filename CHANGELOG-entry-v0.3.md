# CHANGELOG entry — append to existing /CHANGELOG.md

The following entry should be added to the top of the existing `CHANGELOG.md` in the `wavepointanalytics-partner-admin-portal` repo, above the v0.2.0 entry.

---

## [0.3.0] — 2026-05-03

PRD reprioritization session. No code changes; this version delivers a restructured PRD that compresses Partner Portal time-to-market by deferring approximately 40% of v0.2's Phase 1 scope to Phase 3, while preserving every locked decision from v0.2 except where explicitly noted.

### Added — PRD and companion documents

- **`docs/Partner-Admin-Portal-PRD-v0.3.md`** — supersedes v0.2. Adds:
  - **§1.5 Build phases and timelines** — Phase 1a (Partner Portal MVP, ~3-4 weeks) → Phase 1b (Affiliate Portal, ~1-2 weeks) → Phase 2 (Admin Portal, ~3-4 weeks) → Phase 3 (Partner/Affiliate Upgrades, ~3-4 weeks across independently-shippable sub-items) → Future (out of scope for this PRD)
  - **⚠️ Launch Dependencies callout** in §1 — 12-item table of non-engineering blockers (CFTC/NFA registration question, counsel review, Partner Agreement drafting, Article 27 representative procurement, Resend domain verification, etc.) that must be resolved in parallel with engineering or launch is blocked
  - **§22 Security and operational integrity** — full new section: threat model, authentication policy (MFA required for admins), authorization (RLS as primary mechanism), secrets management, input validation, encryption, dependency security, operational security, incident response stub, compliance commitments, periodic review cadence, Phase 1a security checklist (20 items), explicit list of what we're NOT doing
  - **§10A Referral URL dispatch** — new section locking the URL format change from `wavepointanalytics.com/r/{slug}` to `wavepointanalytics.com/{slug}`. Implementation via Cloudflare Worker dispatch with KV-cached slug lookup and reserved slug list as namespace isolation against marketing routes
  - Phase placement notes added at the top of §6, §7, §10, §11, §12, §13, §14, §15, §16, §17, §18 — clearly marking which subsections ship in Phase 1a vs deferred phases
- **`docs/Security-Overview.md`** — new partner-facing companion document. Plain-language summary of authentication, authorization, encryption, dependency management, monitoring, subprocessors, incident response, and explicit acknowledgment of what's NOT done at current scale (no SOC2 yet, no pen test yet, etc.)
- **`docs/Partner-Onboarding-Runbook.md`** — new operational document. 10-step manual onboarding process for Phase 1a (database row creation, slug reservation, manual QR code generation via qr-code-generator.com, SignWell signing, magic-link invitation, role assignment, welcome email, verification, notification). Replaces the deferred Phase 3 wizard. Estimated ~20 minutes per onboarding.
- **`docs/reserved-promo-codes.md`** — new seed list. ~250 reserved code strings across 8 categories: brand/system, placeholders, "free" language, competitors/regulators, misleading performance claims, crypto-adjacent, profanity (LDNOOBW + WavePoint additions), short patterns. Hard-coded as a TypeScript constant in Phase 1a; Phase 2 migrates to admin-managed DB table.
- **`docs/reserved-slugs.md`** — new seed list. Reserved URL slugs to prevent collision with marketing routes, brand impersonation, technical paths, etc. Includes a **maintenance contract** for the marketing site team: whenever a new route is added, check this file and the live slug table.

### Decisions changed vs v0.2

**Phase reprioritization (16 items moved):**

- **Promo codes** — partner self-service creation/deactivation moves to Phase 3. Phase 1a: read-only list with stats; admin creates codes on partner request.
- **Free Access Seats** — partner self-service grant/revoke moves to Phase 3. Phase 1a: read-only display; admin grants manually.
- **QR codes** — self-hosted Worker generation pipeline moves to Phase 3. Phase 1a: admin manually generates one PNG per partner via qr-code-generator.com and uploads to R2.
- **Status page** — public `status.wavepointanalytics.com` moves to Phase 3. Phase 1a: internal Uptime Kuma at `uptime.yingson.com` only; partners notified via Discord/email if something breaks.
- **Customer-facing co-branding** — `ReferredByCard` widget moves to Phase 3. Phase 1a: data fields exist (schema forward-compatible) but no customer-facing display.
- **Support tickets** — full §17 in-portal ticket system moves to Phase 3. Phase 1a: prominent `support@wavepointanalytics.com` callouts in dashboard and footer; 1-business-day response SLA documented internally.
- **Admin impersonation full UI** — moves to Phase 3. Phase 1a: admins use Supabase service-role direct queries for debugging.
- **Onboarding wizard + magic-link invite** — moves to Phase 3. Phase 1a: fully manual per `docs/Partner-Onboarding-Runbook.md`.
- **Resend email integration** — moves to Phase 3 (statement automation, ticket emails, magic-link delivery all defer with it).
- **Light mode** — moves to Phase 3. Phase 1a: dark mode only. Light-mode token values are still defined for forward-compatibility.
- **Partner Owner role-management UI** — moves to Phase 3. Phase 1a supports multi-user partner teams but admin sets all team member roles.
- **Statements automated cron + opt-in toggle** — moves to Phase 3. Phase 1a: admin clicks "Generate statement" button per partner per month, attaches PDF to Gmail manually.
- **Storybook** — moves to Phase 2. Phase 1a: widgets reviewed inline within app via `/dev/widgets` sandbox route.
- **Formal WCAG 2.1 AA audit** — moves to Phase 2. Phase 1a builds *to* WCAG patterns but doesn't run formal axe-core or screen reader audits.
- **Risk Alerts page** — moves to Phase 2.
- **Audit log enhancements** — DB triggers, weekly R2 export, full searchable UI move to Phase 3. Phase 1a: app-level logging, append-only RLS, basic admin table view.
- **User opt-out of partner visibility self-service toggle** — moves to Phase 3. Phase 1a: data field exists; opt-out handled manually via privacy@ email.
- **Notification preferences UI** — minimized in Phase 1a (single statement-related toggle); full preferences page moves to Phase 3.

**Other material changes:**

- **E-signature provider locked to SignWell** (replaces v0.2's provisional BoldSign default). Rationale: WavePoint is already using SignWell for other documents, so no new vendor onboarding or compliance review needed.
- **Referral URL format changed** to `wavepointanalytics.com/{slug}` (eliminates the `/r/` prefix). Implementation via Cloudflare Worker dispatch with reserved slug list per new §10A.
- **Reserved promo code seed list** added as a Phase 1a deliverable per `docs/reserved-promo-codes.md`.
- **`feedback@wavepointanalytics.com`** added as a separate inbox for feature requests (kept distinct from `support@` to prevent feature requests drowning out actual support). Implemented as a footer link + dashboard one-liner; folds into Phase 3 in-portal feature request submission.

**Decisions explicitly UNCHANGED from v0.2:**

- Roles model (admin / partner_owner / partner_member / affiliate / user)
- Partner vs Affiliate feature differential matrix
- Two-table data architecture (`partner_orgs` + `affiliates` with shared `referral_slugs`)
- Frontend stack (React 18 + Vite + Tailwind + shadcn/ui + ECharts + TanStack)
- TypeScript throughout
- Design tokens single-source-of-truth pattern
- Synthetic data layer for development
- Self-referral block, attribution conflict resolution (promo code overrides cookie), email anonymization rules — all kept in Phase 1a (these are core attribution math and privacy commitments, not features)
- Manual ACH payouts, 30-day clawback, $25 minimum payout, USD-only display
- Partner DPA requirement (still under counsel review)
- Statement contents and PDF format
- Support ticket schema (deferred to Phase 3 but design preserved)

### Architectural notes

The Phase 1a security baseline (§22.12) is the most operationally significant addition in v0.3. Key items:

- **MFA REQUIRED for admin accounts** — single highest-leverage control
- **CSP, HSTS, secure cookies** via Cloudflare Worker
- **Zod validation at every API boundary**
- **Dependabot + `npm audit` blocking in CI**
- **RLS reviewed by second person before production** (initially via structured Claude review session)
- **`docs/Incident-Response-Runbook.md`** to be drafted in Phase 1a (1-page stub minimum)
- **Quarterly access review calendar event** scheduled before launch
- **Subprocessor list** documented at `docs/Subprocessors.md`

### Pending / Provisional / Revisit

The PRD's §21 still consolidates open items. Notable updates to that list in v0.3:

- ~~E-sign provider~~ (LOCKED to SignWell — removed from pending)
- Affiliate DPA requirement (still provisional pending counsel review)
- Refund clawback window (still 30 days, provisional)
- Commission rates (still provisional pending Derek/George ratification)
- Customer co-branding window length (when Phase 3 ships — still 30 days, open question)
- Email delivery service (Resend default, no longer urgent since email integration deferred to Phase 3 — Jason has more time to evaluate Postmark or alternatives)

### Notes

- This PRD is the canonical reference for Phase 1a onwards.
- Future updates increment the version number (v0.4, v0.5, etc.) and add a CHANGELOG entry.
- The PRD does not replace the existing v0.1 legal documents (Privacy Policy, ToS, Partner DPA), which still need counsel review per their flagged notes.
- The launch dependencies callout in §1 makes clear that engineering completion of Phase 1a does NOT equal launch readiness — counsel review, CFTC question, Article 27 representative, and signed Partner Agreements with the first ~10 partners are all blocking.
