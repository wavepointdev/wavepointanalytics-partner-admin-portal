# WavePoint Analytics — Partner Data Processing Agreement (DPA)

**Template Version:** 0.1 (First Draft)
**For use with:** WavePoint Analytics content-creator partners
**Pairs with:** Partner Agreement (commercial terms), WavePoint Terms of Service, WavePoint Privacy Policy

> ⚠️ **DRAFT STATUS — NOT FOR EXECUTION.** This document is an AI-assisted first draft intended to give counsel a starting scaffold. Every section must be reviewed and revised by a qualified attorney specializing in GDPR/data protection before being executed with any partner. See **§15 Lawyer Review Notes** at the end.

---

## Structural Note

**This DPA assumes WavePoint and the Partner are INDEPENDENT CONTROLLERS** of the personal data they share under the partner program — not a controller/processor relationship. The Partner decides how to use referral data for their own commission and marketing purposes; WavePoint does not direct that use. Counsel should confirm this is the correct characterization; if instead the Partner is deemed a Processor of WavePoint, the structure of this document must change significantly.

---

## 1. Parties

This Data Processing Agreement ("DPA") is entered into on the date signed below (the "Effective Date") between:

**WavePoint Analytics LLC** ("WavePoint"), a [state] limited liability company with its principal place of business at [Address — TBD]

and

**[Partner Legal Name]** ("Partner"), a [entity type / individual] with its principal place of business at [Partner Address].

Each a "Party" and together the "Parties."

This DPA is incorporated into and forms part of the **Partner Agreement** between the Parties dated [__________] (the "Partner Agreement"). In the event of conflict between this DPA and the Partner Agreement on data protection matters, this DPA controls.

---

## 2. Background and Purpose

The Parties are collaborating under the WavePoint partner program, under which the Partner refers potential customers to WavePoint's trading-indicator services and receives commissions on qualifying paid conversions.

To operate this program, the Parties share certain personal data relating to individuals who click a Partner's referral link, start a free trial, or become paying WavePoint subscribers ("Referred Individuals"). Both Parties determine independently the means and purposes of their respective use of this data and therefore act as **independent controllers** under applicable data protection laws.

This DPA sets out the Parties' respective responsibilities for handling that data in compliance with:

- The EU General Data Protection Regulation (2016/679) ("GDPR")
- The UK General Data Protection Regulation and Data Protection Act 2018 ("UK GDPR")
- The California Consumer Privacy Act / California Privacy Rights Act ("CCPA/CPRA")
- Any other applicable data protection laws (collectively, "Data Protection Laws")

---

## 3. Definitions

Capitalized terms used but not defined here have the meanings given in applicable Data Protection Laws. For clarity:

- **"Personal Data"** means any information relating to an identified or identifiable natural person.
- **"Processing"** means any operation performed on Personal Data, including collection, recording, storage, use, disclosure, and erasure.
- **"Data Subject"** means the Referred Individual to whom the Personal Data relates.
- **"Shared Personal Data"** means the specific categories of Personal Data described in Schedule A that the Parties share under this DPA.
- **"Security Incident"** means any actual or reasonably suspected unauthorized access to, disclosure of, loss of, alteration of, or destruction of Shared Personal Data.

---

## 4. Roles of the Parties

### 4.1 Independent Controllers

Each Party is an **independent Controller** of the Shared Personal Data it receives, to the extent it determines the purposes and means of processing for its own lawful business activities.

Neither Party is a Processor of the other. Neither Party processes the Shared Personal Data on behalf of or under the instructions of the other (except as specifically agreed in writing).

### 4.2 WavePoint's Role

WavePoint is the Controller of Personal Data for:

- All customer and trial user account records
- Subscription, billing, and payment records (processed via Stripe)
- Indicator access and usage records
- Support interactions

WavePoint makes a subset of this data available to the Partner (the "Shared Personal Data") strictly for the purposes described in §5.

### 4.3 Partner's Role

The Partner is the Controller of Personal Data for:

- Its own audience, subscribers, and community members
- Referral activity associated with its content
- Any data the Partner collects directly from Referred Individuals (e.g., email signups on the Partner's website)

The Partner becomes a Controller of the Shared Personal Data it receives from WavePoint at the point of transfer.

---

## 5. Permitted Purposes and Prohibited Uses

### 5.1 Permitted Purposes

The Partner may process the Shared Personal Data **only** for the following purposes:

1. **Commission tracking and reconciliation** — verifying conversions attributed to the Partner, reviewing commission calculations, and raising disputes
2. **Legitimate customer support** — responding to Referred Individuals who contact the Partner for help with the referral or related issues
3. **Relationship management** — limited, non-marketing communications with paid subscribers who have already engaged with the Partner's content, where such communication is consistent with the Referred Individual's reasonable expectations
4. **Internal audience analytics** — aggregated analysis of which content or campaigns produce conversions (the underlying data must remain in secure systems and not be re-shared)
5. **Compliance and record-keeping** — meeting the Partner's own legal, tax, and accounting obligations

### 5.2 Prohibited Uses

The Partner **must not**:

1. Sell, license, rent, or otherwise monetize the Shared Personal Data by transfer to any third party
2. Use the Shared Personal Data for **unsolicited marketing** to Referred Individuals who have not independently opted in to the Partner's marketing
3. Combine the Shared Personal Data with other data sources to enrich, re-identify, or build profiles of Referred Individuals beyond the Permitted Purposes
4. Use the Shared Personal Data to target Referred Individuals with advertising for products competitive with WavePoint
5. Disclose the Shared Personal Data to any third party except:
   - Service providers bound by equivalent confidentiality and data-protection obligations
   - As required by law, subject to §11
6. Retain the Shared Personal Data beyond the retention periods specified in §9
7. Transfer the Shared Personal Data outside the Partner's agreed processing locations (Schedule A) without first implementing appropriate safeguards under applicable Data Protection Laws
8. Re-share the Shared Personal Data with other partners, affiliates, or members of the Partner's network

### 5.3 No Creation of Joint Controllership

Nothing in this DPA creates a joint-controllership arrangement under GDPR Article 26 unless the Parties expressly agree otherwise in a separate written agreement.

---

## 6. Categories of Shared Personal Data

See **Schedule A** for the full description. In summary, the Partner receives:

- **Before a Referred Individual becomes a paid subscriber:** anonymized email (partial mask), event dates (click, trial start), plan tier interest, coarse geographic information (country, US state where known)
- **After a Referred Individual becomes a paid subscriber:** full email address, subscription tier, subscription lifecycle status (active, paused, canceled, refunded)

The Partner does NOT receive, at any time:
- NinjaTrader Machine IDs or TradingView usernames
- Payment information (card details, Stripe IDs, bank information)
- IP addresses or device fingerprints
- Indicator usage data or trading activity
- Any information about non-referred WavePoint customers

---

## 7. Lawful Basis for Sharing

### 7.1 WavePoint's Lawful Basis

WavePoint relies on the following lawful bases under GDPR Article 6 for sharing Personal Data with the Partner:

- **Legitimate interests** (Article 6(1)(f)) — operating a partner commission program is a legitimate business interest of WavePoint; the sharing is necessary for that purpose; Data Subjects' rights are safeguarded through the anonymization, opt-out, and use-limitation provisions of this DPA.
- **Contract performance** (Article 6(1)(b)) — for Referred Individuals who have subscribed, sharing attribution data is necessary to calculate partner commission and complete the commercial chain the Referred Individual triggered.

### 7.2 Partner's Lawful Basis

The Partner represents that it has identified an appropriate lawful basis for its processing of the Shared Personal Data under applicable Data Protection Laws, and will document this in its own records of processing activities.

### 7.3 Transparency to Data Subjects

WavePoint provides notice to Referred Individuals of this sharing in its Privacy Policy (§4.2) and at relevant points of collection. The Partner will not rely on WavePoint's notice to satisfy the Partner's own transparency obligations to Data Subjects where the Partner processes their data for new or different purposes.

---

## 8. Data Subject Rights

### 8.1 Forwarding of Requests

Each Party will **promptly (within 5 business days)** notify the other Party if it receives a Data Subject request (access, rectification, erasure, portability, objection, restriction) that relates to the other Party's processing.

### 8.2 Cooperation

The Parties will cooperate in good faith and without unreasonable delay to:

- Identify whether the request concerns the receiving Party's processing
- Provide any information reasonably necessary to fulfill the request
- Coordinate responses where both Parties process the same Data Subject's data

### 8.3 Opt-Out by Referred Individual

Referred Individuals may opt out of Partner visibility at any time through WavePoint's customer portal. Upon opt-out:

- WavePoint will cease sharing further data about the Data Subject with the Partner
- WavePoint will notify the Partner of the opt-out within 5 business days
- The Partner will, within 30 days of notification, delete or anonymize all Shared Personal Data it holds about the Data Subject from active systems (subject to legal retention obligations)

### 8.4 Costs

Each Party bears its own costs of responding to Data Subject requests, except where the request primarily concerns the other Party's processing and reasonable cooperation costs are documented.

---

## 9. Retention and Deletion

### 9.1 Retention Periods

The Partner will retain Shared Personal Data only for as long as necessary for the Permitted Purposes and in any case no longer than:

| Data Category | Maximum Retention |
|---|---|
| Pre-conversion activity (click, trial events) | 2 years after the event |
| Post-conversion subscriber data | Duration of the Referred Individual's active subscription + 2 years |
| Commission and financial records | 7 years from the end of the financial year (tax retention) |
| Records required by applicable law | The period required by that law |

### 9.2 Deletion on Partner Termination

Upon termination of the Partner Agreement or this DPA:

- The Partner will, within 60 days, delete or anonymize all Shared Personal Data in active systems
- The Partner may retain backups containing Shared Personal Data for up to 12 months if deletion from backups is technically infeasible, provided the backups remain encrypted and are not accessed for any purpose other than backup recovery
- Commission and financial records may be retained for the period required by tax and accounting law

Upon request, the Partner will provide written certification of completion of deletion.

### 9.3 Return vs. Deletion

At WavePoint's option, the Partner will return (rather than delete) any subset of the Shared Personal Data, in a commonly used electronic format, before deletion.

---

## 10. Security

### 10.1 Minimum Security Measures

Each Party will implement and maintain appropriate technical and organizational measures to protect Shared Personal Data against unauthorized or unlawful processing, accidental loss, destruction, damage, or disclosure. Minimum measures include:

- **Encryption in transit** (TLS 1.2+) for all Shared Personal Data transmission
- **Encryption at rest** for databases and storage systems holding Shared Personal Data
- **Access controls** limiting access to Shared Personal Data to personnel with a genuine need to access it for the Permitted Purposes
- **Unique authentication credentials** for each user with access (no shared logins)
- **Multi-factor authentication** on any account with access to more than aggregate/anonymized data
- **Audit logging** of access, modification, and export events
- **Secure deletion** of Shared Personal Data at end of retention
- **Employee training** on data-protection responsibilities
- **Confidentiality obligations** on all personnel, contractors, and processors with access

### 10.2 Proportionality

Security measures must be proportionate to the volume and sensitivity of Shared Personal Data processed and to the state of the art. Counsel should consider requiring ISO 27001 or SOC 2 certification for Partners processing significant volumes; for small content-creator partners, the measures above are typically adequate.

### 10.3 Sub-Processors

Where the Partner engages sub-processors (service providers, contractors) to help process Shared Personal Data, the Partner will:

- Impose equivalent data-protection obligations on the sub-processor in writing
- Remain responsible for the sub-processor's compliance
- Maintain a list of sub-processors available to WavePoint on request

---

## 11. Security Incidents

### 11.1 Notification

Upon becoming aware of a Security Incident affecting Shared Personal Data, the Partner will:

1. **Notify WavePoint without undue delay and in any event within 48 hours** of becoming aware
2. Provide the following information in the initial notification (to the extent known):
   - Nature of the incident
   - Categories and approximate number of Data Subjects affected
   - Categories and approximate number of records affected
   - Likely consequences
   - Measures taken or proposed to address the incident
3. Supplement the notification as additional information becomes available

### 11.2 Cooperation

The Partner will cooperate with WavePoint's investigation and response, including:

- Providing access to relevant logs and system information
- Assisting with communications to Data Subjects and supervisory authorities, where required
- Preserving evidence for regulatory and forensic purposes

### 11.3 Costs

Each Party bears its own costs of investigating and remediating a Security Incident affecting its own systems. Where a Party's breach of this DPA causes a Security Incident affecting the other Party, the breaching Party is responsible for reasonable remediation and notification costs.

### 11.4 Notification to Data Subjects and Authorities

Unless required by law to do so unilaterally, neither Party will notify affected Data Subjects or supervisory authorities about an incident involving Shared Personal Data without first coordinating with the other Party on the content and timing of the notification.

---

## 12. International Data Transfers

### 12.1 US Storage

Both Parties acknowledge that Shared Personal Data may be processed in the United States. For transfers from the UK, EU, or EEA to the United States:

- Each Party represents that it has implemented appropriate safeguards under applicable Data Protection Laws, including (where applicable) the **European Commission's Standard Contractual Clauses** (2021/914), the **UK International Data Transfer Agreement** or **UK Addendum**, or an adequacy decision
- The Parties will execute additional cross-border transfer instruments if required by regulators or changes in law

### 12.2 Partner Processing Location

The Partner represents that its processing locations are listed in Schedule A. The Partner will notify WavePoint at least 30 days in advance of any change in principal processing location.

---

## 13. Liability and Indemnification

### 13.1 General Liability

Each Party is liable for its own acts and omissions in processing Shared Personal Data. Neither Party is liable for the other Party's breach of Data Protection Laws, except to the extent the first Party contributed to the breach.

### 13.2 Indemnification by Partner

The Partner will indemnify and hold WavePoint harmless from and against any regulatory fines, Data Subject claims, and reasonable legal costs arising directly from the Partner's:

- Breach of this DPA
- Use of Shared Personal Data for prohibited purposes (§5.2)
- Security Incident caused by the Partner's negligence or willful act
- Failure to cooperate as required by §§8 and 11

### 13.3 Indemnification by WavePoint

WavePoint will indemnify and hold the Partner harmless from regulatory fines, Data Subject claims, and reasonable legal costs arising directly from WavePoint's breach of its obligations under this DPA.

### 13.4 Limitation

Each Party's aggregate liability under this DPA is subject to the liability caps and exclusions in the Partner Agreement, except that liability for (a) breach of §5.2 (Prohibited Uses), (b) a Security Incident caused by gross negligence or willful act, and (c) regulatory fines assessed against the other Party due to this Party's breach, is not so limited.

---

## 14. Miscellaneous

### 14.1 Term and Termination

This DPA takes effect on the Effective Date and continues until all Shared Personal Data has been deleted or returned to WavePoint in accordance with §9.

### 14.2 Audit Rights

Upon 30 days' written notice (or shorter in the event of a regulatory investigation or Security Incident), WavePoint may, no more than once per year, conduct or commission an audit of the Partner's processing of Shared Personal Data. The audit must be proportionate and respect confidential information. The Partner will cooperate in good faith, provide access to relevant records, and make knowledgeable personnel available. Each Party bears its own audit costs unless the audit reveals a material breach by the Partner, in which case the Partner will reimburse WavePoint's reasonable audit costs.

### 14.3 Changes in Law

If a change in Data Protection Laws requires modifications to this DPA, the Parties will negotiate the necessary modifications in good faith. If agreement is not reached within 60 days, either Party may terminate this DPA on 30 days' written notice.

### 14.4 Conflict

In the event of conflict between this DPA and any other agreement between the Parties on data-protection matters, this DPA controls.

### 14.5 Counterparts; Electronic Signature

This DPA may be executed in counterparts and signed electronically. Each counterpart, together, constitutes one instrument.

### 14.6 Governing Law

This DPA is governed by the same law specified in the Partner Agreement, except that the mandatory data-protection laws of a Data Subject's place of residence (where applicable) may also apply.

### 14.7 Notices

Notices under this DPA are sent to:

**To WavePoint:**
Email: [privacy@wavepointanalytics.com](mailto:privacy@wavepointanalytics.com)
Mailing address: [TBD]

**To Partner:**
Email: [Partner Privacy Contact Email]
Mailing address: [Partner Address]

Each Party must notify the other of any change to its data-protection contact within 30 days.

---

## 15. Lawyer Review Notes

> This section should be removed before execution. Items requiring legal judgment and input.

### 15.1 Structural decisions

1. **Controller-Controller vs. Controller-Processor** — this template treats both parties as independent Controllers. This is usually correct for referral/affiliate programs because the Partner decides for itself how to use the data for commission verification, audience analytics, and relationship management. If the Partner instead processes *solely on WavePoint's instructions* for commission purposes (which is unusual), the structure should shift to Controller-Processor and this document must be significantly rewritten. Counsel to confirm.
2. **Joint Controllers (GDPR Article 26)** — an alternative framing if the Parties truly co-decide purposes. The template explicitly disclaims this, but counsel should confirm based on actual business practice.

### 15.2 Critical pairings

3. **Pair with Partner Agreement** — this DPA references the "Partner Agreement" as the commercial contract. That document (separate deliverable) covers commission rates, payout cadence, FTC disclosure requirements for the partner's content, trademark use, exclusivity (if any), and termination. The two must be drafted consistently and cross-reference correctly.
4. **Pair with Privacy Policy §4.2** — the Privacy Policy describes the sharing the DPA governs. Any drift between the two creates risk. Cross-check carefully.
5. **Pair with WavePoint ToS §10** — the ToS points partners at this DPA. Make sure all three documents agree.

### 15.3 Transfer mechanism

6. **SCCs / UK IDTA attachment** — for transfers to the US from EU/UK, the EU Standard Contractual Clauses (2021/914) and/or UK International Data Transfer Agreement or UK Addendum should typically be attached as a separate schedule, not just referenced. Counsel to prepare and attach Schedule B (SCCs) if any partner is an EU/UK controller.
7. **Transfer Impact Assessment (TIA)** — post-Schrems II, transfers from EU to US generally require a documented TIA. Counsel to provide template or direction.

### 15.4 Commercial terms embedded in this DPA

8. **Retention periods (§9.1)** — 2 years for pre-conversion events, 2 years post-termination, 7 years for financial records. Confirm these align with state/country tax obligations for each Partner's jurisdiction.
9. **Deletion timeline on Partner termination (§9.2)** — 60 days is standard. Confirm acceptable operationally.
10. **Security Incident notification window (§11.1)** — 48 hours is aggressive but appropriate for the level of PII involved. GDPR requires 72 hours for notification to a supervisory authority, so 48 hours gives WavePoint 24 hours to prepare its own notification. Confirm.
11. **Audit cadence (§14.2)** — once per year is standard. For a small content-creator partner, this is rarely exercised in practice; for a larger enterprise partner it might be a real cost. Consider tiering based on partner size.
12. **Sub-processor rules (§10.3)** — requires equivalent contractual protections with Partner's sub-processors. Some small partners may not have formal agreements with their tools (e.g., Google Workspace); those tools have their own published DPAs which suffice. Language could be clarified.

### 15.5 Indemnification and liability

13. **Liability carve-outs (§13.4)** — indemnification for prohibited uses, willful/gross-negligent Security Incidents, and passed-through regulatory fines is standard. Confirm these carve-outs are compatible with the overall Partner Agreement liability cap.
14. **Insurance requirements** — consider requiring the Partner to carry cyber-liability insurance with a minimum coverage amount (typical: $1M) for partners processing significant data volumes. Not included in this template; counsel to advise based on partner tier.

### 15.6 Operational items for WavePoint

15. **Anonymization function (§6)** — the technical implementation of the email-masking rule (`j***@g***.com`) should be documented so we can prove to a regulator how it works. Spec lives in the Partner Portal Architecture doc; confirm engineering alignment.
16. **Opt-out propagation (§8.3)** — WavePoint must implement a technical mechanism that immediately removes opted-out Referred Individuals from the Partner's data feed. Spec: when a user opts out, flag on the user record → hourly batch updates include opt-out status → Partner Portal displays "Opted out" placeholder → 5-day alert to Partner that they must delete from their own systems.
17. **Partner onboarding checklist** — counsel should produce a short checklist WavePoint uses at partner onboarding to confirm the Partner has (a) a Privacy Policy, (b) documented processing locations, (c) security measures meeting §10.1, (d) signed this DPA, (e) contact points for §14.7.
18. **Termination of Partner with significant data holdings** — when a Partner is terminated, we must verify deletion. §14 requires certification; consider whether we want the right to validate via audit, and what "trust but verify" looks like in practice.

### 15.7 CCPA/CPRA-specific

19. **CCPA "sharing" vs. "sale"** — CCPA/CPRA treats transfers for "cross-context behavioral advertising" as "sharing" requiring opt-out. This DPA does not authorize such use (§5.2). Confirm the Privacy Policy and any marketing-site cookie banner reflect this.
20. **Service Provider vs. Contractor vs. Third Party status under CCPA** — this matters because it affects the notice and opt-out framework. As drafted (independent controllers with a Permitted Purpose / Prohibited Use regime), the Partner is closest to a "Third Party" under CCPA, which triggers more rigorous disclosure. Counsel to confirm characterization.

### 15.8 Jurisdiction-specific

21. **Quebec (Law 25) / Canada (PIPEDA)** — may require additional commitments if Canadian partners are onboarded.
22. **Brazil (LGPD)** — similar to GDPR; may need specific clauses if Brazilian partners are in scope.
23. **Australia (Privacy Act 1988)** — has Australian Privacy Principles that may apply; separate bridging language may be required.

### 15.9 Low-friction partner onboarding trade-off

24. **Practical friction** — this DPA is long and legalistic. Small content-creator partners will balk at a 15-page document. Counsel should advise on the minimum legally-required subset vs. the gold-standard document. One approach is a short "signature page" attached to a standardized DPA body — partners initial each schedule rather than negotiate line-by-line.

---

## Schedule A — Description of Data Sharing

### A.1 Subject Matter of Processing

Sharing of attribution data, commission-related data, and limited subscriber data relating to Referred Individuals who have interacted with the Partner's referral materials.

### A.2 Duration

For the duration of the Partner Agreement, plus the retention periods specified in §9.

### A.3 Nature and Purpose of Processing

- By WavePoint: providing referral attribution, commission calculation, and partner support
- By Partner: commission verification, legitimate customer support, relationship management, internal audience analytics

### A.4 Categories of Data Subjects

Individuals who click a Partner referral link, coupon code, or other Partner-attributed referral mechanism, and subsequently:

- Visit the WavePoint site, OR
- Create a WavePoint account / start a trial, OR
- Become a paying WavePoint subscriber

### A.5 Categories of Personal Data

**Pre-conversion (visit, signup, trial):**

- Masked email address (e.g., `j***@g***.com`)
- Partner referral identifier and campaign
- Event timestamps (click, signup, trial start)
- Plan tier interest (where declared)
- Country and, where applicable, US state (from Stripe Tax data)

**Post-conversion (paying subscriber):**

- Full email address
- Subscription tier
- Subscription lifecycle events (active, paused, canceled, refunded)
- Billing currency

**NOT shared at any time:**
Names (unless voluntarily shared by the subscriber), payment information, NinjaTrader Machine IDs, TradingView usernames, IP addresses, device fingerprints, usage data, trading activity, broker account information.

### A.6 Partner's Processing Locations

Primary location: **[Partner to specify — e.g., United States, California]**
Backup / disaster-recovery location: **[Partner to specify]**
Sub-processor locations: **[Partner to list, if any]**

### A.7 Technical Transfer Method

Shared Personal Data is made available to the Partner through:

1. The WavePoint Partner Portal web interface (`partners.wavepointanalytics.com`), accessed via authenticated login
2. The Rewardful partner dashboard, accessed via authenticated login
3. Data export (CSV) initiated by the Partner through the Partner Portal

No direct database access, API access beyond the published partner API, or bulk transfer is authorized without a separate written agreement.

### A.8 Retention

See §9.

---

## Schedule B — Standard Contractual Clauses *(if applicable)*

*[Reserved — attach EU SCCs (2021/914) Module One (Controller-to-Controller) and/or UK IDTA / UK Addendum here if the Partner is an EU/UK-based controller. Counsel to prepare.]*

---

## Signatures

**WavePoint Analytics LLC**

Name: ___________________________
Title: ___________________________
Date: ___________________________
Signature: ___________________________

**[Partner Legal Name]**

Name: ___________________________
Title: ___________________________
Date: ___________________________
Signature: ___________________________

---

*Template v0.1 — first draft, pending legal review. Do not execute without counsel approval.*
