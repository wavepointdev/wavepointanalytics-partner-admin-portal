# WavePoint Analytics — Security Overview

**Version:** 1.0
**Last reviewed:** May 3, 2026
**Owner:** Jason Johnson (COO)
**Audience:** Partners, prospective enterprise customers, internal team

---

## Purpose

This document is a plain-language summary of how WavePoint Analytics protects customer and partner data. It is the partner-facing companion to PRD §22 (Security and operational integrity), which contains the engineering-level detail.

If you have specific security questions not answered here, contact `security@wavepointanalytics.com`.

---

## Our principle

Security commitments must be proportionate to the actual risk. WavePoint Analytics is a small, focused company serving a few thousand customers and a small number of content-creator partners. We are not a bank, and we don't pretend to operate at SOC2 enterprise scale today.

What we *do* commit to: defensive coding hygiene from the start, operational discipline around access and secrets, and a documented baseline so the question "what's your security posture?" has a real, honest answer.

---

## What we protect

| Data category | Examples | How we protect it |
|---|---|---|
| Customer payment data | Credit card numbers, bank account numbers | We never see or store this — Stripe handles it directly with PCI-DSS compliance |
| Customer identity | Email, name, account preferences | Encrypted at rest and in transit; row-level security ensures only the customer (and authorized admins) can access their record |
| Partner financial records | Commission balances, payout amounts, payout method | Same as customer identity, plus stricter access controls — only admins and the specific partner can see |
| Authentication credentials | Passwords, OAuth tokens | We never store passwords directly — Supabase handles them with industry-standard hashing |
| Audit trails | Records of who did what, when | Append-only, cannot be modified or deleted, retained indefinitely |

---

## How we authenticate users

- **Customers** sign in with email + password or Google OAuth, with optional multi-factor authentication
- **Partners** sign in the same way; we strongly recommend MFA, indicated by a "Recommended" badge in your settings until enabled
- **Admins** are required to use MFA — no exceptions. The admin role cannot be granted to an account without MFA enrolled.
- Sessions expire after 4 hours for admins, 7 days for partners and affiliates, 30 days for customers
- Passwords must be at least 12 characters and are checked against the Have I Been Pwned breach database at signup and password change

---

## How we control access

We use **row-level security** at the database layer, not application-level checks. This means even if our application code had a bug, the database itself enforces who can see what.

- A partner can only see their own attributed customers, their own commissions, their own promo codes
- One partner cannot see another partner's data, ever
- Admins have broader access by role assignment; every admin action is logged
- Service-level credentials (the "master key" for the database) are never sent to your browser — they live only on our servers

---

## How we handle secrets and credentials

- All API keys, database credentials, and similar secrets are stored in Cloudflare Workers Secrets, encrypted at rest, accessible only to deployment automation
- We rotate secrets quarterly (high-sensitivity) or annually (lower-sensitivity), and immediately if compromise is suspected
- We use pre-commit scanning to prevent secrets from being committed to source control
- Only members of the WavePoint admin team can access production secrets

---

## How we encrypt data

**In transit:**

- All web traffic uses TLS 1.3 (the latest standard) via Cloudflare
- HSTS is enabled, meaning browsers will refuse to connect over plain HTTP
- We're submitted to Chrome's HSTS preload list, so even first-time visitors get HTTPS-only protection
- Cookies are marked Secure, HttpOnly, and SameSite-restricted

**At rest:**

- Database (Supabase): AES-256 encryption
- File storage (Cloudflare R2): AES-256 encryption
- Cache and metadata storage: encrypted by default

---

## How we manage dependencies

Modern software is built on hundreds of third-party libraries. A vulnerability in any one of them can affect us.

- **Dependabot** scans our dependencies daily and automatically opens pull requests for security updates
- **`npm audit`** runs as a blocking check before any deployment — high or critical vulnerabilities prevent the deploy
- **Annual dependency review** to remove unused packages and reduce attack surface
- Every new dependency requires explicit team review before being added

---

## How we monitor for problems

- **Audit log** captures every admin action with reason text required for sensitive actions
- **Cloudflare logs** capture web traffic anomalies; the Cloudflare WAF blocks obvious malicious patterns automatically
- **Risk Alerts** (rolling out in a later phase) surfaces patterns like unusual chargebacks, account sharing, or promo code abuse
- **Error tracking** captures application errors for diagnosis

---

## Our subprocessors

Subprocessors are third-party services that process data on our behalf. We use:

| Subprocessor | Purpose | Location |
|---|---|---|
| Supabase | Database, authentication | US |
| Stripe | Payment processing | US |
| Rewardful | Affiliate attribution and commission tracking | US |
| Cloudflare | Hosting, CDN, security | Global |
| Resend | Transactional email (rolling out in a later phase) | US |
| SignWell | E-signature for partner agreements | US |
| Web3Forms | Contact form submissions (transitional) | US |

The current authoritative list is maintained at `docs/Subprocessors.md` in our internal documentation. Material changes are communicated to partners under their DPA.

---

## What happens if something goes wrong

We maintain an **Incident Response Runbook** with documented procedures for:

1. **Detect** — recognizing an incident (error spikes, suspicious activity, breach notifications)
2. **Triage** — first 15 minutes, who to call, severity scale
3. **Contain** — immediate actions like rotating secrets, revoking sessions
4. **Eradicate** — root cause fix
5. **Recover** — restore service, verify behavior
6. **Review** — post-incident retrospective

If a personal data breach occurs, we notify affected supervisory authorities within 72 hours per GDPR Article 33, and affected individuals as required by Article 34. US state breach notification laws are followed per the locations of affected residents.

---

## What we're NOT doing (yet)

In the spirit of honesty, here's what's intentionally out of scope at our current scale:

- **No SOC2 audit** — the cost (~$15k–$50k) and operational lift aren't proportionate to our current customer base. We'll revisit when we have an enterprise partner asking for it, or when we cross 50+ partners.
- **No formal penetration test** — defensive coding hygiene plus database-level access control is the current substitute. We'll add a third-party pen test before onboarding any enterprise partner.
- **No bug bounty program** — too noisy at our traffic level. Future consideration.
- **No HSM-backed key management** — Cloudflare-managed secrets storage is sufficient at this scale.

We're transparent about these gaps because pretending they don't exist would be worse.

---

## Reviews and updates

This document is reviewed:

- **Annually** as part of our security policy review
- **After any P0 or P1 incident**
- **When we add a new subprocessor or major dependency**
- **When we expand to a new market or compliance regime**

The version number at the top of this document reflects the most recent material change.

---

## Questions or concerns

| Topic | Contact |
|---|---|
| Security questions or concerns | security@wavepointanalytics.com |
| Suspected vulnerability | security@wavepointanalytics.com |
| Privacy or data subject requests | privacy@wavepointanalytics.com |
| General support | support@wavepointanalytics.com |

---

*WavePoint Analytics LLC — © 2026*
