# Reserved Promo Codes — Seed List

**Version:** 1.0 (initial seed list)
**Owner:** Jason Johnson (COO)
**Last reviewed:** May 3, 2026
**Source of truth:** This file in Phase 1a, then admin-managed table in Phase 2

---

## Purpose

Promo codes that match any entry in this list cannot be created — by either admin or partner self-service (when Phase 3 enables self-service). Matching is **case-insensitive**.

The goal is to prevent codes that:

1. Could collide with marketing or system communications
2. Could imply false endorsement of competitors or regulators
3. Are offensive or otherwise embarrassing for the brand
4. Could trigger regulatory scrutiny on a futures-trading product
5. Are common placeholders that suggest a misconfigured system

---

## Phase 1a implementation

The list below is encoded as a TypeScript constant in the Worker / portal codebase:

```typescript
// packages/portal-shared/src/reserved-promo-codes.ts
export const RESERVED_PROMO_CODES = new Set([
  // (full list below)
]);
```

Code creation flow (admin-driven in Phase 1a):

1. Admin proposes a code string
2. System uppercases and trims
3. System checks: is this in `RESERVED_PROMO_CODES`?
4. If yes: reject with error `"This code is reserved and cannot be used."`
5. If no: proceed to uniqueness check (PRD §10.1)

---

## Phase 2 implementation

Phase 2 ships an admin UI to manage this list as a database table (`reserved_promo_codes`). Each entry has:

- `code` (uppercase, primary key)
- `category` (enum: `brand`, `system`, `competitor`, `regulatory`, `offensive`, `placeholder`)
- `added_by` (admin user UUID)
- `added_at` (timestamp)
- `notes` (free text)

The admin UI allows adding, removing, and bulk-importing entries. Removal is audit-logged.

---

## Categories and seed list

### 1. Brand and system (do not impersonate WavePoint)

```
WAVEPOINT
WAVEPOINTANALYTICS
WPA
WP
OFFICIAL
ADMIN
ADMINISTRATOR
ROOT
SUPPORT
HELP
STAFF
TEAM
WAVEPOINTTEAM
CEO
CTO
COO
FOUNDER
FOUNDERS
OWNER
SYSTEM
NOREPLY
DONOTREPLY
SECURITY
PRIVACY
LEGAL
COMPLIANCE
BILLING
ACCOUNTS
BANKING
PAYOUT
PAYOUTS
INVOICE
INVOICES
RECEIPT
RECEIPTS
```

### 2. Generic placeholders (suggest a misconfigured system)

```
TEST
TESTING
DEMO
EXAMPLE
SAMPLE
INVALID
NULL
NONE
TBD
PLACEHOLDER
DEFAULT
TEMP
TEMPORARY
TODO
FIXME
DELETEME
ASDF
QWERTY
123
1234
12345
123456
ABC
ABCDEF
```

### 3. Misleading "free" / incentive language

```
FREE
FREEACCESS
FREEMONTH
FREETRIAL
FREESUB
FREESUBSCRIPTION
FOREVER
FOREVERFREE
LIFETIME
UNLIMITED
ALLACCESS
VIP
ELITE
PREMIUM
GOLD
PLATINUM
DIAMOND
EXCLUSIVE
INSIDER
PRIVATE
SECRET
HIDDEN
```

### 4. Trademark / competitor / regulator (regulatory-risk for false endorsement)

```
NINJATRADER
NT
NT8
TRADINGVIEW
TV
THINKORSWIM
TOS
INTERACTIVEBROKERS
IBKR
TDAMERITRADE
TDA
SCHWAB
CHARLES
ETRADE
ROBINHOOD
WEBULL
TRADESTATION
SIERRA
SIERRACHART
MULTICHARTS
NINJASCRIPT
PINESCRIPT
CME
CMEGROUP
ICE
NYMEX
COMEX
CBOT
EUREX
NASDAQ
NYSE
CFTC
NFA
SEC
FINRA
FED
FEDERALRESERVE
TREASURY
BLOOMBERG
REUTERS
CNBC
FOXBUSINESS
WSJ
ZEROHEDGE
INVESTOPEDIA
```

### 5. Misleading / regulatory-risk performance claims

A futures-trading product is subject to heightened regulatory scrutiny around marketing claims. Codes that imply guaranteed profits or risk-free outcomes are forbidden.

```
GUARANTEED
GUARANTEE
PROMISED
RISKFREE
NORISK
ZERORISK
NOLOSS
NEVERLOSS
NEVERLOSE
ALWAYSWIN
ALWAYSPROFIT
PROFITABLE
PROFITS
PROFIT
WIN
WINS
WINNING
WIN100
WIN1000
WINEVERY
EVERYWIN
WINNER
WINNERS
DOUBLE
DOUBLEMONEY
TRIPLE
QUADRUPLE
TENX
10X
100X
1000X
MILLIONAIRE
MILLIONS
RICH
WEALTH
WEALTHY
INSIDER
INSIDERTIP
INSIDERINFO
SECRETSAUCE
HOLYGRAIL
EDGE
UNFAIRADVANTAGE
FRONTRUN
MANIPULATE
CONFIDENTIAL
LEAKED
EXCLUSIVE
PROVEN
SCIENTIFIC
ALGORITHM
AIPOWERED
AITRADING
QUANTITATIVE
HEDGEFUND
INSTITUTIONAL
PROFESSIONAL
EXPERT
MASTER
GURU
PRO
CHAMPION
LEGEND
LEGENDARY
```

### 6. Crypto / pump-and-dump adjacent terms

WavePoint is a futures-trading product, but futures and crypto audiences overlap and "moon" / "rocket" language has been associated with pump-and-dump schemes. Not allowed.

```
MOON
TOTHEMOON
ROCKET
LAMBO
HODL
DIAMOND
DIAMONDHANDS
APE
APING
BAGHOLDER
BUYTHEDIP
PUMP
PUMPIT
PUMPED
DUMP
DUMPED
DEGEN
DEGENS
WAGMI
NGMI
GMI
SHILL
SHILLED
RUGPULL
```

### 7. Profanity and offensive terms

Phase 1a hard-codes a starting list. Maintenance plan: seed from a maintained open-source list (e.g., LDNOOBW — `https://github.com/LDNOOBW/List-of-Dirty-Naughty-Obscene-and-Otherwise-Bad-Words`) covering English plus common variants in major languages.

The full profanity list is **not reproduced inline** to keep this document professional. Phase 1a implementation:

1. Pull the LDNOOBW English list at build time
2. Concatenate with WavePoint's brand-specific additions (see below)
3. Encode as part of the `RESERVED_PROMO_CODES` constant

**WavePoint-specific additions (terms not on standard lists but inappropriate for our brand):**

```
SCAM
SCAMMER
FRAUD
FRAUDULENT
PONZI
PYRAMID
WASH
WASHTRADE
INSIDERTRADE
FRONTRUNNER
MANIPULATION
MANIPULATOR
GAMBLE
GAMBLER
GAMBLING
ADDICT
ADDICTION
LOSER
LOSERS
SUCKER
SUCKERS
NEWB
NEWBIE
NOOB
NOOBS
RETAIL
RETAILTRADER
DUMBMONEY
SMARTMONEY
WHALE
SHARK
DEPRESSION
SUICIDE
KILLYOURSELF
KMS
KYS
```

### 8. Numeric / single-character / overly short patterns

- Codes shorter than 4 characters are blocked at the format-validation layer (PRD §10.3), so these are belt-and-suspenders:

```
A
B
C
1
12
123
0
00
000
NO
YES
OK
GO
HI
HEY
```

---

## What's NOT on this list (and why)

These were considered and intentionally excluded:

- **Specific numbers** (other than placeholder sequences): codes like `WINTER2026` or `DRYSDALE15` should be allowed for legitimate marketing
- **Common English words** that aren't in the categories above: codes like `NEWYEAR`, `SUMMER`, `BACKTOSCHOOL` should be partner-creatable
- **Partner display names**: a partner whose display name is "Drysdale Trading Group" should be able to use `DRYSDALE` or `DRYSDALE20` as their codes — uniqueness is enforced at code-creation time, but the words themselves are not reserved
- **Specific competitors not listed**: this list reflects the major US futures-and-trading-platform competitors as of May 2026. Updates as the competitive landscape evolves.

---

## Maintenance

**Review cadence:** annually, or:

- When a new competitor enters the market
- When a regulatory action highlights new risk language
- When a partner reports a code they wanted that is reserved (re-evaluate inclusion)
- When Phase 2 ships the admin UI (one-time migration to DB table)

**Maintainer:** Jason Johnson (COO) is the default maintainer. Updates require:

1. Changes proposed via PR or CHANGELOG entry
2. Review by at least one other founder for material additions
3. Update the version number at the top of this file

**When a code is removed from this list:**

The 12-month uniqueness cooldown (PRD §10.1) does not apply — newly-allowed codes can be used immediately.

**When a code is added to this list:**

Existing active codes matching the new entry are NOT auto-deactivated, but they are flagged in the admin portal for review. The admin team decides case-by-case whether to deactivate (with notice to the partner) or grandfather.

---

## Appendix: implementation note

This list will appear in the codebase in two places:

1. **Worker code:** `packages/portal-shared/src/reserved-promo-codes.ts` — used at code-creation time
2. **Admin UI:** in Phase 2, as a managed list backed by a `reserved_promo_codes` table

Both must stay in sync. The Phase 2 migration moves the source of truth from this Markdown file (and the TypeScript constant) into the database; this file then becomes documentation of the rationale and seed list, not the live source.
