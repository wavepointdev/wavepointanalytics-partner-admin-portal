# Reserved Referral Slugs — Seed List

**Version:** 1.0 (initial seed list)
**Owner:** Jason Johnson (COO)
**Last reviewed:** May 3, 2026
**Source of truth:** This file in Phase 1a, then admin-managed table in Phase 2

---

## Purpose

In v0.3, referral URLs use the format `wavepointanalytics.com/{slug}` (the `/r/` prefix from v0.2 was eliminated for cleaner branding). This means partner slugs share the namespace with marketing site routes.

Reserved slugs prevent partners from picking strings that:

1. Collide with current marketing site routes (`/about`, `/pricing`, etc.)
2. Collide with likely-future marketing routes (`/community`, `/events`, etc.)
3. Impersonate WavePoint brand or system communications
4. Collide with technical / infrastructure paths
5. Are operationally confusing (e.g., `redirect`, `link`)

---

## Phase 1a implementation

The list below is encoded as a TypeScript constant in the Cloudflare Worker that handles the `wavepointanalytics.com/{slug}` dispatch (PRD §10A.4):

```typescript
// workers/marketing-dispatch/src/reserved-slugs.ts
export const RESERVED_SLUGS = new Set([
  // (full list below)
]);
```

Slug-creation flow (admin-driven in Phase 1a, partner-self-service in Phase 3):

1. User proposes a slug
2. System lowercases and trims
3. System validates regex: `^[a-z0-9]([a-z0-9-]{0,30}[a-z0-9])?$`
4. System checks: is this in `RESERVED_SLUGS`?
5. If yes: reject with error `"This slug is reserved. Please pick a different one."`
6. If no: proceed to uniqueness check against `referral_slugs` table

---

## Phase 2 implementation

Phase 2 ships an admin UI to manage this list as a database table (`reserved_slugs`). Each entry has:

- `slug` (lowercase, primary key)
- `category` (enum: `marketing_route`, `system`, `brand`, `competitor`, `placeholder`, `regulatory_risk`)
- `added_by` (admin user UUID)
- `added_at` (timestamp)
- `notes` (free text — e.g., "added when /events page launched")

---

## Categories and seed list

### 1. Current marketing site routes

Routes that exist or are planned for the marketing site at `wavepointanalytics.com`. **Owner of marketing site routes must keep this list in sync** as new routes are added.

```
about
pricing
contact
contact-us
blog
news
press
careers
team
team-members
community
events
help
docs
documentation
docs-api
api
api-docs
guides
tutorial
tutorials
faq
faqs
features
roadmap
changelog
status
stats
analytics
case-studies
case-study
customers
testimonials
reviews
demo
free-trial
trial
download
downloads
get-started
getting-started
sign-up
signup
sign-in
signin
login
logout
register
account
profile
settings
preferences
billing
subscriptions
subscription
plans
plan
upgrade
downgrade
cancel
refund
refunds
support
contact-support
helpdesk
ticket
tickets
knowledge-base
kb
search
sitemap
robots
ads
ads-txt
security-txt
well-known
```

### 2. Legal and policy routes

```
terms
terms-of-service
tos
terms-and-conditions
privacy
privacy-policy
cookie-policy
cookies
dpa
data-processing
data-processing-agreement
acceptable-use
acceptable-use-policy
aup
legal
legal-notice
disclaimer
disclaimers
copyright
trademark
trademarks
licenses
licensing
gdpr
ccpa
imprint
impressum
disclosures
disclosure
risk-disclosure
risk-disclosures
```

### 3. Product routes

```
products
product
indicators
indicator
ninjatrader
ninjatrader-suite
nt
tradingview
tradingview-suite
tv
futures
options
markets
trading
strategies
strategy
toolkit
toolkits
suite
suites
```

### 4. Partner / affiliate / referral routes

These are reserved because of the multi-portal architecture:

```
partners
partner
partners-portal
partner-portal
partner-program
become-a-partner
apply
application
admin
admin-portal
affiliate
affiliates
affiliates-portal
affiliate-portal
affiliate-program
referral
referrals
refer
ref
r
share
invite
invitation
track
redirect
link
links
go
out
exit
landing
lp
landing-page
campaign
campaigns
promo
promos
promotion
promotions
deal
deals
offer
offers
discount
discounts
coupon
coupons
code
codes
```

### 5. Technical / infrastructure paths

```
www
mail
ftp
sftp
ssh
git
svn
api
api-v1
api-v2
api-v3
v1
v2
v3
graphql
rest
soap
websocket
ws
wss
webhook
webhooks
hooks
callback
callbacks
oauth
oauth2
sso
saml
auth
authenticate
authentication
authorize
authorization
verify
verification
token
tokens
session
sessions
csrf
xsrf
cors
csp
hsts
robots-txt
sitemap-xml
favicon
favicon-ico
manifest
manifest-json
service-worker
sw
sw-js
worker
workers
static
assets
public
private
internal
admin-api
internal-api
debug
test
testing
staging
preview
beta
alpha
canary
dev
development
prod
production
local
localhost
127-0-0-1
fonts
images
img
imgs
photos
videos
video
audio
media
files
documents
js
javascript
css
styles
stylesheet
stylesheets
branding-assets
branding
brand
logo
logos
icon
icons
graphics
.well-known
well-known
ping
health
healthz
readiness
liveness
metrics
status-page
statuspage
status-internal
```

### 6. Brand / system / impersonation prevention

```
wavepoint
wavepointanalytics
wp
wpa
official
staff
admin
administrator
root
system
sysadmin
support
help-desk
helpdesk
billing-team
security
privacy-team
legal-team
compliance
operations
ops
engineering
eng
marketing
sales
finance
accounting
payouts
banking
ceo
cto
coo
cfo
founder
founders
owner
owners
team
wavepointteam
employees
internal
corporate
hr
human-resources
recruiting
jobs
no-reply
noreply
do-not-reply
donotreply
mailer-daemon
postmaster
abuse
spam
notifications
notification
alert
alerts
```

### 7. Standard error pages and HTTP terms

```
error
errors
404
403
401
500
502
503
504
gone
not-found
forbidden
unauthorized
server-error
maintenance
under-construction
coming-soon
unavailable
offline
ratelimit
rate-limit
throttle
throttled
```

### 8. Common placeholders and reserved characters

```
null
undefined
none
empty
default
placeholder
example
sample
test
testing
demo
tbd
todo
fixme
deleteme
new
old
temp
temporary
draft
backup
archive
trash
deleted
removed
hidden
private
secret
confidential
restricted
internal
external
public
shared
copy
duplicate
original
unknown
unidentified
anonymous
guest
visitor
user
users
member
members
account
accounts
```

### 9. Competitors and regulators (false-endorsement risk)

Same regulatory-risk list as `docs/reserved-promo-codes.md` §4, lowercased for slug format:

```
ninjatrader
nt
nt8
tradingview
tv
thinkorswim
tos
interactivebrokers
ibkr
tdameritrade
tda
schwab
charles
etrade
robinhood
webull
tradestation
sierra
sierrachart
multicharts
ninjascript
pinescript
cme
cmegroup
ice
nymex
comex
cbot
eurex
nasdaq
nyse
cftc
nfa
sec
finra
fed
federalreserve
treasury
bloomberg
reuters
cnbc
foxbusiness
wsj
zerohedge
investopedia
```

### 10. Misleading performance terms (regulatory risk)

Same as `docs/reserved-promo-codes.md` §5, lowercased:

```
guaranteed
guarantee
promised
risk-free
riskfree
no-risk
norisk
zero-risk
zerorisk
no-loss
noloss
never-lose
neverlose
always-win
alwayswin
profit
profits
profitable
win
wins
winning
winner
winners
holy-grail
holygrail
secret-sauce
secretsauce
insider
insider-tip
secret
millions
millionaire
rich
wealth
unfair-advantage
unfairadvantage
edge
front-run
frontrun
manipulate
proven
scientific
algorithm
ai-powered
aipowered
ai-trading
aitrading
quantitative
hedge-fund
hedgefund
institutional
professional
expert
master
guru
pro
champion
legend
legendary
```

### 11. Crypto / pump-adjacent

Same as `docs/reserved-promo-codes.md` §6, lowercased:

```
moon
tothemoon
to-the-moon
rocket
lambo
hodl
diamond
diamond-hands
diamondhands
ape
aping
bag-holder
bagholder
buy-the-dip
buythedip
pump
pump-it
pumpit
pumped
dump
dumped
degen
degens
wagmi
ngmi
gmi
shill
shilled
rug-pull
rugpull
```

### 12. Profanity

Phase 1a uses the LDNOOBW open-source list (`https://github.com/LDNOOBW/List-of-Dirty-Naughty-Obscene-and-Otherwise-Bad-Words`) for profanity coverage, lowercased and joined with this file's WavePoint-specific additions (see `reserved-promo-codes.md` §7 for those). The profanity portion is not reproduced inline.

---

## Maintenance contract

**Whenever a new marketing site route is added at `wavepointanalytics.com/{newpath}`:**

1. Check this file — is `newpath` already in §1?
   - If yes: route can be added safely
   - If no: add it to §1, version-bump this file, deploy new Worker constant
2. Check the live `referral_slugs` table — does any active partner have `newpath` as their slug?
   - If yes: contact that partner to negotiate a slug change with 30-day notice; do not add the marketing route until resolved
   - If no: safe to proceed

**Whenever a new partner or affiliate is onboarded:**

The slug they pick is checked against this list at the database / Worker level. Conflict messages should be friendly and suggest alternatives.

**Annual review:**

- Add new competitors that have entered the market
- Remove obsolete entries (deprecated marketing routes, defunct competitors)
- Cross-check with `reserved-promo-codes.md` for consistency
- Bump version

---

## What's NOT on this list (and why)

- **Common English words that don't fall into the categories above** — partners should be able to pick `lighthouse`, `compass`, `journeyman`, etc., as long as they're not in the reserved list
- **Partner display name slugs** — `drysdale`, `mountainpeak`, etc. — are explicitly allowed unless they collide with this list
- **Numeric-only slugs** — these are blocked by the slug regex (must start with a letter), not by this list
- **Specific year / date patterns** — `2026`, `q1`, `winter`, etc., are not reserved (allowed)

---

## Appendix: implementation note

This list will exist in two locations during Phase 1a:

1. **Worker code:** `workers/marketing-dispatch/src/reserved-slugs.ts` — used by the URL dispatch logic
2. **Portal admin code:** `packages/portal-shared/src/reserved-slugs.ts` — used at slug-creation time in the admin tooling

Both must stay in sync. Phase 2 unifies them into a `reserved_slugs` database table consumed by both surfaces; this Markdown file then becomes documentation rather than the live source of truth.
