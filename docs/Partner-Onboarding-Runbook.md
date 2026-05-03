# Partner / Affiliate Onboarding Runbook (Phase 1a — Manual Process)

**Version:** 1.0 (initial stub)
**Owner:** Jason Johnson (COO)
**Audience:** WavePoint admin team (Jason, George, Derek, future hires)
**Last reviewed:** May 3, 2026

---

## Purpose

In Phase 1a, partner and affiliate onboarding is fully manual. This runbook is the authoritative checklist for taking a new partner or affiliate from "we've decided to work with them" to "they have working portal access and a signed agreement."

Phase 3 of the portal build replaces this manual flow with a wizard-driven self-service onboarding (PRD §15, Phase 3). Until that ships, every onboarding follows this document.

**Estimated time per onboarding:** ~20 minutes once you've done one or two.

---

## Pre-flight checklist

Before starting any onboarding, confirm:

- [ ] Decision to onboard has been made (Jason, George, Derek as applicable)
- [ ] Commission rate has been agreed (default affiliate 15%, custom for partners)
- [ ] Partner Agreement / Affiliate Agreement / DPA (as applicable) is ready to send
- [ ] You have the partner's preferred display name and slug request
- [ ] You have the partner's preferred contact email

If any of these are missing, stop and gather them before proceeding.

---

## Step 1 — Create the database row

**For partners:**

1. Open Supabase Studio at `https://eqozbdekcawelfbpgrit.supabase.co`
2. Navigate to Table Editor → `partner_orgs`
3. Insert a new row with:
   - `id`: auto-generated UUID
   - `org_name`: legal name of the partner organization
   - `display_name`: customer-facing display name
   - `referral_slug`: agreed slug (lowercase, alphanumeric with hyphens, 1-32 chars; check `docs/reserved-slugs.md` for forbidden values)
   - `commission_rate`: agreed rate (e.g., `0.20` for 20%)
   - `commission_structure`: enum (typically `'percentage_recurring'`)
   - `payout_method`: leave NULL until partner provides ACH details
   - `status`: `'pending_invitation'`
   - `is_active`: `false` (will flip to `true` after agreement signed and invite accepted)
   - `created_by`: your admin user UUID
   - `notes`: any context Derek/George/Jason should know
4. Note the new `partner_org_id` for next steps

**For affiliates:**

Same flow on the `affiliates` table; commission rate default 15% unless overridden.

---

## Step 2 — Reserve the slug

Slug reservation happens automatically as part of Step 1 (the `referral_slugs` table is FK-linked).

Verify:

1. Query `referral_slugs` table — your slug should appear with `entity_type` matching what you just created
2. Confirm `is_active = false` (matches the parent record)

If the slug appears taken on insert: pick a different one. The 1-year cooldown applies for slugs that were previously active (PRD §3.7, §15.4).

---

## Step 3 — Generate the QR code

In Phase 1a we generate QR codes manually using a free third-party tool.

1. Visit `https://www.qr-code-generator.com/` (or any equivalent free generator)
2. Enter URL: `https://wavepointanalytics.com/{slug}` (replace `{slug}` with the actual slug)
3. Choose: PNG output, error correction H, ~1024×1024 pixels
4. Optional: add the WavePoint logo overlay if the generator supports it (use the file from `branding-assets/wavepoint-logo-square.png` in the wavepoint repo)
5. Download the PNG
6. Upload to Cloudflare R2 at the path:
   - Partners: `partners/{partner_org_id}/qr.png`
   - Affiliates: `affiliates/{affiliate_id}/qr.png`
7. Verify the public R2 URL works in a browser

**Note:** SVG output is deferred to Phase 3 when the self-hosted Worker pipeline ships. If a partner specifically needs SVG before then, you can also generate SVG via the same tool and upload to `qr.svg` in the same R2 path.

---

## Step 4 — Send the e-signature request

Phase 1a uses out-of-band signing via SignWell.

1. Open SignWell (you should already have an account from existing WavePoint document workflows)
2. Send the appropriate documents:
   - **Partner:** Partner Agreement + Partner DPA
   - **Affiliate:** Affiliate Agreement + Partner DPA (DPA is provisional for affiliates per §4.2; subject to counsel review)
3. Recipient signs via SignWell's standard email flow
4. After signing completes:
   - Save the SignWell envelope ID
   - Update the database row:
     - `agreement_signed_at`: timestamp from SignWell
     - `agreement_version`: version of the agreement signed
     - `agreement_envelope_id`: SignWell envelope ID
     - `dpa_signed_at`: same fields for DPA

**Do not proceed to Step 5 until Step 4 is complete.** A partner without a signed agreement should not have portal access.

---

## Step 5 — Send the magic-link invitation

1. Open Supabase Studio → Authentication → Users
2. Click "Invite user"
3. Enter the partner's email address
4. Supabase sends a magic-link invitation with default expiry
5. Note: the invitation creates an `auth.users` row but does NOT yet have role assignments

---

## Step 6 — Wait for the partner to accept the invitation

The partner clicks the magic link, sets their password (if first time) or signs in (if pre-existing user).

When you see the user in `auth.users` with `last_sign_in_at` populated, proceed to Step 7.

---

## Step 7 — Assign the role

1. Find the new user in `auth.users` and copy their `id`
2. Find the corresponding `user_profiles` row (Supabase auto-creates one) and copy its `id` (will match `auth.users.id`)
3. Insert into `user_role_assignments`:
   - `user_profile_id`: the user's UUID
   - `role`: `'partner_owner'` (for partners) or `'affiliate'` (for affiliates)
   - `partner_org_id`: the org UUID from Step 1 (NULL for affiliates)
   - `affiliate_id`: the affiliate UUID from Step 1 (NULL for partners)
   - `is_active`: `true`
   - `granted_by`: your admin user UUID
   - `granted_at`: now
4. Update the parent `partner_orgs` / `affiliates` row:
   - `is_active`: `true`
   - `status`: `'active'`
5. Update the `referral_slugs` row:
   - `is_active`: `true`

---

## Step 8 — Send the welcome email manually

Send a welcome email via Gmail (your wavepointanalytics.com Workspace account). Suggested template:

```
Subject: Welcome to the WavePoint Partner Program, [Display Name]

Hi [Contact Name],

Welcome aboard. Your WavePoint partner portal is now active and ready to use.

→ Sign in: https://partners.wavepointanalytics.com
→ Your referral link: https://wavepointanalytics.com/{slug}
→ Your QR code: https://r2.wavepointanalytics.com/partners/{id}/qr.png
→ Your commission rate: [X]%

A few things worth knowing:

- Your dashboard shows your conversions, earnings, and recent referrals in real time
- Marketing assets (logos, banners, social media images) are available in your portal under "Marketing Toolkit"
- Need a promo code? Email support@wavepointanalytics.com and we'll set one up for you. (Self-service promo code creation is rolling out in a later phase.)
- Need help? support@wavepointanalytics.com — we typically respond within one business day
- Have an idea for the portal? feedback@wavepointanalytics.com

Looking forward to working together.

— The WavePoint Team
```

Personalize as appropriate; specifically include their commission rate and any agreed-upon launch arrangements.

---

## Step 9 — Verify portal access

Test the partner's setup yourself before considering onboarding complete:

1. Open an incognito window
2. Sign in to `https://partners.wavepointanalytics.com` using the partner's email + a temporary password they shared, OR use the admin "View as user" workaround for Phase 1a (direct Supabase SQL query to confirm RLS-filtered data is showing correctly)
3. Confirm:
   - Partner sees the dashboard with their org name in the header
   - Partner sees an empty (or appropriate) referrals table
   - Partner sees their referral link with the correct slug
   - Partner sees the QR code image you uploaded
   - Partner does NOT see any other partner's data
   - Partner sees the "Need help? support@wavepointanalytics.com" callout
4. Sign out

If anything is wrong, fix it before notifying the partner that they're live.

---

## Step 10 — Notify the partner that they're live

Send a follow-up message (Slack DM or email):

> "All set — your portal is fully live. The dashboard will populate with conversion data as referrals come in. Reach out any time."

Update your internal tracking:

1. Mark this onboarding complete in your tracking spreadsheet / Notion / wherever
2. Add a calendar event for 7 days out: "Check in with [Partner Name] — first-week onboarding follow-up"

---

## Special cases

### Pre-existing customer becoming a partner

If the email is already in `auth.users` (they're a paying customer):

- Skip Step 5 (magic-link invitation)
- In Step 7, just add the new role assignment
- The welcome email in Step 8 should acknowledge their existing relationship: "You'll see a new 'Partner' option in your top nav next time you sign in"

### Promoting an affiliate to partner

Per PRD §4.4:

1. Find the affiliate's row; do NOT delete it
2. Insert a new row in `partner_orgs` with `promoted_from_affiliate_id` = the affiliate's UUID
3. Move the slug reservation from the affiliate to the partner_org (update `entity_type` and `entity_id` in `referral_slugs`)
4. Move the QR code in R2 from `affiliates/{id}/qr.png` to `partners/{new_id}/qr.png`
5. Re-run e-sign for the new Partner Agreement (existing Affiliate Agreement does not transfer)
6. Mark the affiliate row `is_active = false` and `status = 'promoted_to_partner'`
7. Update role assignment from `'affiliate'` to `'partner_owner'`

### Multi-Owner partner_org

If onboarding a partner with multiple Owners from Day 1:

- Steps 1-7 work for the first Owner
- For each additional Owner: repeat Steps 5-7 with their email; in Step 7 assign `'partner_owner'` role to the SAME `partner_org_id`
- The org has multiple Owner role assignments; all have equivalent permissions per §3.3

### Adding a Member (not Owner) to existing partner_org

In Phase 1a, **admin sets all team member roles** (PRD §1.5 Phase 1a in-scope). Owner-managed team UI is Phase 3.

1. Confirm with the existing Owner that they want this member added
2. Steps 5-7 with the new email; assign `'partner_member'` role to the same `partner_org_id`
3. Notify the Owner that the member has been added

---

## What this runbook does NOT cover

These are out of scope for Phase 1a onboarding and are documented in PRD §15 / §18 for Phase 3:

- Self-service application form for prospective partners
- Magic-link invitation as part of an automated wizard flow
- Embedded SignWell signing inside the portal
- Welcome screen + setup summary screen on first login
- Automated onboarding notifications to admin team
- Slug reservation race-condition handling (manual coordination via Slack or shared sheet for now)

---

## Maintenance

- This runbook is reviewed and updated whenever the manual onboarding process changes
- When Phase 3 ships the wizard, this runbook becomes the **fallback procedure** for emergency manual onboarding
- Append errata to the bottom of this file as discovered, version-bump on material updates
