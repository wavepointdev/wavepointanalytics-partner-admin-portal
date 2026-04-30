# Session Handoff — Partner & Admin Portal Project

**Created:** 2026-04-19
**For:** The next Claude session continuing this work
**Repo:** `wavepointanalytics-partner-admin-portal` (GitHub: `github.com/wavepointdev/wavepointanalytics-partner-admin-portal`, Gitea mirror: `gitea.yingson.com/wavepointdev/wavepointanalytics-partner-admin-portal`)
**Project status:** Planning + mockup phase complete; awaiting board review and legal engagement before implementation

---

## TL;DR for the Next Session

Jason (COO of WavePoint Analytics, a 3-person futures-trading-indicator SaaS startup) and I produced a complete planning package for two new portals: a multi-tenant **Partner Portal** for B2B content-creator partners, and an **Admin Portal** for the founders. Outputs include two interactive HTML mockups and four legal/architectural documents. Nothing has been built or deployed. The next session is most likely to be one of: (a) board feedback iterating on the mockups or decisions, (b) drafting the Partner Agreement (commercial terms doc that pairs with the DPA), (c) building secondary screen mockups, or (d) starting actual implementation.

**If Jason starts the next session by referencing "the portal project," "the partner portal," "the admin portal," or "the board briefing" — read this entire document before responding.**

---

## Project Context

### What WavePoint is

- **Product:** Premium trading indicators for futures traders, distributed via NinjaTrader (initial) with TradingView coming in Phase 2
- **Stage:** Pre-launch — no shipping product or portal yet
- **Team:** 3 founders — George (CTO, 70% equity), Jason (COO, 15%, the user), Derek (15%)
- **Existing infrastructure:** Marketing site at `wavepointanalytics.com`, customer portal at `app.wavepointanalytics.com` (with the `tvUsername||''` JS bug Jason has been chasing — see his memory)
- **Tech stack:** Cloudflare Pages + Workers + DNS, Supabase (PostgreSQL + Auth + RLS), Stripe (billing + tax), Rewardful (planned, for commissions)

### Where this project fits

This is the **third** WavePoint repo, alongside the marketing site and customer portal. All three share the same design language, color tokens, and typography. This repo will house two new portals at `partners.wavepointanalytics.com` and `admin.wavepointanalytics.com`, both deployed from a single codebase via Cloudflare Worker hostname dispatch.

### Target scale (design constraint)

- ~5,000 end-user customers
- ~10 B2B content-creator partners
- Up to ~3 team members per partner
- 3 founders + future hire roles for admin

---

## What Was Decided (and is settled — don't re-litigate)

If Jason or another founder revisits any of these, ask whether the decision is being changed or just discussed; don't assume the door is open.

| Decision | Resolution |
|---|---|
| Buy vs. build for commission engine | **Buy: Rewardful** (~$49–$149/mo). Custom-build everything else around it. |
| Deployment topology | **Subdomains** on **single codebase**, **Cloudflare Worker** dispatches by host |
| Charting library | **ECharts** (not Chart.js — premium look) |
| Platform priority | **NinjaTrader first**, TradingView later. Data model supports both from Day 1. |
| Refresh cadence | **Hourly** (no real-time infrastructure) |
| Geographic scope | **Worldwide with GDPR/EU built in from Day 1** |
| Partner archetype | **Content creators / educators only** (not brokers, not strategic resellers) |
| Partner team roles | **Owner + Team Member** (two roles) |
| Commission structure | **Flat % of net revenue, paid conversions only**, with future support for finder's fees |
| Anonymization rule | **Mask emails pre-paid (`j***@g***.com`), reveal post-paid**; users can opt out of partner visibility |
| Partner payout cadence | **Monthly** |
| Coupons & free seats | **Per-partner admin-configurable guardrails** (defaults: 25% max discount, 50 uses, 90-day expiry, 3 concurrent codes; 5 free seats default) |
| Custom partner economics | **Yes, per partner** — admin can override commission %, offer alternative structures (e.g., 2 months free), set per-product rates, custom guardrails |
| Audit log | **Append-only via RLS**, even admins can't edit/delete |

---

## What's Open (and likely to come back as topics)

Tracked in detail in `docs/Partner-Portal-Architecture.md` §6. Most consequential ones:

### High priority — board to decide

1. **Refund clawback window** (30 / 45 / 60 days) — affects Privacy Policy, ToS §4.6, Partner Agreement, mockups. Must be consistent everywhere.
2. **Default commission rate** (assumed 30%) — sets expectations for all partner deals
3. **Default free seat quota** — currently 5, possibly should scale with partner tier
4. **Free seat auto-expiry** — permanent until revoked, or auto-expire after N days?
5. **Coupon max discount ceiling** — currently 25%
6. **Anonymization for refunded users** — keep masked, or reveal since the partner takes a clawback?
7. **Future admin role tiers** — what does "Support Agent" / "Analyst" / "CS Manager" see?
8. **Lifetime access ceiling** — company-wide cap?

### High priority — legal

9. **Engage privacy/SaaS counsel** for review of all four legal documents as a package
10. **CFTC / NFA Commodity Trading Advisor registration question** — flagged in ToS §19.2 item 8. This is the biggest regulatory risk. Should be addressed *before* significant build investment.
11. **EU / UK Article 27 representative** required (~$100–$400/year service)
12. **Partner DPA execution timeline** — cannot onboard partners without signed DPA

### Medium priority

13. Risk alert thresholds (account sharing, coupon velocity)
14. Founder alert routing (who gets paged for what)
15. Discord integration for system alerts
16. Chargeback handling SLA ownership

---

## Artifacts Produced (where they are, what they're for)

All artifacts live in `/mnt/user-data/outputs/` from the previous session. Jason has them. They have not yet been committed to the repo as of the end of last session.

| Filename (working) | Purpose | Repo Destination |
|---|---|---|
| `README.md` | Repo orientation | `/README.md` |
| `CHANGELOG.md` | Project change history | `/CHANGELOG.md` |
| `WavePoint-Board-Briefing-Portal-Architecture.md` | Board briefing — single source of truth | `/docs/Partner-Portal-Architecture.md` *(renamed by Jason)* |
| `WavePoint-Privacy-Policy-v0.1.md` | First-draft Privacy Policy (US + GDPR + UK + CCPA) | `/docs/Privacy-Policy-v0.1.md` |
| `WavePoint-Terms-of-Service-v0.1.md` | First-draft Terms of Service | `/docs/Terms-of-Service-v0.1.md` |
| `WavePoint-Partner-DPA-Template-v0.1.md` | First-draft Partner DPA (Controller-to-Controller structure) | `/docs/Partner-DPA-Template-v0.1.md` |
| `partner-portal-dashboard-mockup.html` | Owner-view dashboard mockup, fully interactive | `/mockups/partner-portal-dashboard-mockup.html` |
| `admin-portal-dashboard-mockup.html` | Founder-view dashboard mockup, fully interactive | `/mockups/admin-portal-dashboard-mockup.html` |
| (this file) | Session handoff | `/docs/SESSION-HANDOFF.md` or transient note |

### Key things to know about the artifacts

- **The two HTML mockups have annotated "Mockup notes" panels** (bottom-right floating button). These document every design decision and list open questions. Other founders are expected to use these to self-orient.
- **All four legal documents have lawyer-review-note sections at the end** (numbered items). These are the checklist for counsel and should be removed before publication.
- **The mockups use Manrope (display) + Inter (body), `#0b1326` surface, `#4edea3` teal, `#adc6ff` primary, glass cards** — consistent with the marketing site and customer portal.
- **Mockups demonstrate working interactions:** coupon creation modal, free seat grant modal, copy-to-clipboard on referral link, scroll-to-section, notes panel toggle. ECharts visualizations render with realistic sample data.

---

## Pending Work Items (likely candidates for next session)

In rough order of likelihood:

### A. Board feedback iteration (most likely)
Jason will share the package with George and Derek. They'll come back with reactions. Possible outcomes:
- KPI ordering changes on either dashboard
- Adding/removing specific features
- Resolving the open questions in §6 of the Architecture Brief
- Adjusting visual elements
- Requesting the Team Member view of the Partner Portal as a separate mockup

**If this is the path:** Read the relevant document section first, ask Jason for the specific feedback, then iterate. Don't redesign from scratch.

### B. Partner Agreement draft
The DPA is the *data-protection* document. The Partner Agreement is the *commercial terms* document — commission rates, payout cadence, FTC disclosure requirements, trademark usage rules, exclusivity, termination. The DPA references it (`§1` and `§2` of the DPA). It's the next legal document to draft.

**If this is the path:** Use the same structure as the ToS and DPA — first draft with `[TBD]` placeholders for commercial values, plain-language summary at top, lawyer-review notes at the bottom. Cross-reference correctly with the DPA.

### C. Admin Portal Customer Detail / Partner Detail mockups
The Admin Portal dashboard is the overview. The detail pages (where the manual override toolkit lives, and where per-partner custom commission/coupon/seat configuration happens) have been documented but not mocked. **The Partner Detail page is the more interesting one** — it's where Jason configures all the per-partner customizations.

**If this is the path:** Match the visual style of the existing admin portal mockup. Reuse the orange admin distinction. Include the manual override toolkit prominently. The customization controls (commission rate, alternative structures, coupon ceilings, seat quota, auto-expiry policy) are the primary content.

### D. Partner Portal Team Member view
Documented in the Owner mockup's notes panel — Team Member sees: no commission $ anywhere, no Upcoming Payout card, dollar KPIs replaced with conversion counts, no Team nav, settings read-only.

**If this is the path:** Branch from the Owner mockup, strip the financial UI, demonstrate the role-based RLS visually.

### E. Cookie consent banner spec
Required before EU/UK launch. Touches the marketing site, customer portal, and these new portals. Could be a self-hosted open-source component or a service like CookieYes/Osano.

### F. Implementation kickoff
If the board approves direction and decisions, the build itself starts. Phase 1 plan in Architecture Brief §8: 4–6 weeks for foundation. First implementation steps would be Supabase schema + migrations, Cloudflare Worker hostname routing, and base layout/navigation shared across both portals.

### G. CLAUDE.md update
Jason mentioned wanting to add a note in `CLAUDE.md` (the Yingson Labs / WavePoint AI session context file) that **WavePoint repos mirror GitHub → Gitea** (opposite direction from home lab repos where Gitea is source of truth). The new `wavepointanalytics-partner-admin-portal` repo now makes three WavePoint repos following this pattern.

---

## How Jason Likes to Work (preferences observed across sessions)

- **Prefers concrete drafts over blank-page exercises.** Give him something to react to. Even a flawed first draft is more useful than a list of questions.
- **Wants rationale embedded.** When you make a design choice, briefly explain why so he can defend it to George and Derek (or push back if he disagrees).
- **Asks for board-discussion items / decisions.** Whenever you produce something with implicit assumptions or design choices, surface those as a list for him to bring up with co-founders.
- **Wants to know when you disagree with someone else's plan.** He brought a Gemini-authored brief into the first session and explicitly asked Claude to push back on the parts that didn't fit. Be direct, not deferential.
- **Values "ultra-premium" feel.** Mentioned this multiple times. Restraint reads premium — small fonts, generous whitespace, minimal chart chrome, no aggressive gradients. Look at what's already in the mockups.
- **Cost-conscious.** Free tier of any service unless there's a clear business case. SaaS over self-hosting only when build cost > buy cost AND complexity is genuinely lower.
- **Prefers junior-engineer-level documentation.** Detailed enough that someone unfamiliar can execute. Especially for runbook steps.
- **Wants future-flexibility built in.** "Build the foundation here so it's much easier to think about considerations now and build accordingly than make painful changes later."
- **Good at telling you to stop boiling the ocean.** "We are a very small 3-person org so lets not boil the ocean here."
- **Will rename and reorganize files.** Don't be precious about working filenames. He renamed `WavePoint-Board-Briefing-Portal-Architecture.md` to `Partner-Portal-Architecture.md` mid-session.
- **Excited builder energy.** Multiple "this looks pretty sweet" / "I'm really excited" reactions. Match the enthusiasm without being sycophantic.

---

## Things to Avoid

- **Don't redesign the look-and-feel.** Manrope + Inter, `#0b1326` / `#4edea3` / `#adc6ff`, glass cards. Already aligned with the existing marketing site and customer portal. Diverging creates inconsistency Jason will catch.
- **Don't propose Stripe Connect, Airbyte, Xero direct integration, or "niche heatmaps."** All considered and explicitly rejected. If they come up again, point at the Architecture Brief §1 / "Changed (vs. original Gemini brief)" section in the CHANGELOG.
- **Don't forget the futures-only product scope.** Original Gemini brief listed "Stocks, Options, Crypto" — none of those apply. Indicators are for futures.
- **Don't add real-time / websocket infrastructure.** Hourly refresh is the agreed cadence. Adding realtime adds cost and complexity for little partner-facing benefit.
- **Don't propose adding a leaderboard between partners in the Partner Portal.** Jason explicitly rejected this. Internal admin leaderboard is fine.
- **Don't reproduce the lawyer-review notes verbatim into the public versions of the legal documents.** They're working scaffolding for counsel.
- **Don't confuse the two Gitea sync directions.** WavePoint repos: GitHub → Gitea (pull mirror, GitHub is source of truth). Yingson Labs repos: Gitea → GitHub (push mirror, Gitea is source of truth). This is documented in his memory and was explicitly flagged.

---

## Useful References to Pull Up Quickly

If the next session starts with a question, here's where the answer probably lives:

| Question category | Look at |
|---|---|
| "What was decided about X?" | `docs/Partner-Portal-Architecture.md` §1 (confirmed decisions) |
| "What are the open questions?" | `docs/Partner-Portal-Architecture.md` §6 |
| "How does the partner portal work?" | `mockups/partner-portal-dashboard-mockup.html` notes panel |
| "How does the admin portal work?" | `mockups/admin-portal-dashboard-mockup.html` notes panel |
| "What does the partner see vs. not see?" | DPA §6, Privacy Policy §4.2, Architecture Brief §2.3 |
| "How are coupons / free seats configured?" | Architecture Brief §2.4, §2.5; admin partner-detail page (not yet mocked) |
| "What's the legal posture?" | Privacy Policy, ToS, DPA — read all three; their lawyer notes are the gap analysis |
| "What's the cost?" | Architecture Brief §7 |
| "What's Phase 1 vs. Phase 2?" | Architecture Brief §8 |
| "What did the previous session do?" | `CHANGELOG.md` |
| "How do I sync GitHub to Gitea?" | Memory + last response of previous session (PAT-based mirror, 3-step Gitea form) |

---

## Open with Jason like this (suggested)

If Jason opens with something general like "let's keep going on the portal project," respond with something like:

> Welcome back. Quick pulse-check — should we pick up with: (a) board feedback on the mockups, (b) drafting the Partner Agreement, (c) building the Admin Customer/Partner Detail mockups, (d) something else? Quick orientation: the package we wrapped last session is committed-or-ready in the repo and CHANGELOG shows the state. Ready when you are.

If he opens with something specific, just dive in — but read the relevant artifact section first, don't redo work that's been done.

---

*End of handoff. Good luck, future Claude.*
