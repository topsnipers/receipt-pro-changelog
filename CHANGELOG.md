# Changelog

All notable changes to Receipt Pro are documented here.

## [2.1.2] — 2026-04-08

### Added
- **Server-side trial gating**: New `/api/start-trial` endpoint stores `trial:{device_id}` in Redis with 14-day TTL. Prevents trial abuse via reinstall — same device_id blocked from re-claiming
- **Device ID generation**: `CostcoPro.getDeviceId()` generates persistent UUID via `crypto.randomUUID()`, stored in `chrome.storage.local`
- **Trial check in verify-key**: When no license key provided, `/api/verify-key` checks for active server-side trial and returns `tier: "trial"` with expiration

### Changed
- **grantShareTrial() server-first**: Share Card trial now calls `/api/start-trial` before granting locally. Server rejection (`already_used`) immediately blocks. Network failure falls back to local 14-day grant (intentional offline tolerance)
- Applied to: chrome-ext-v2 (Chrome Web Store build)

### Security (Codex Review Fixes)
- **Critical: Async executor anti-pattern** — `new Promise(async (resolve) =>` in `grantShareTrial()` replaced with proper `async function`. Prevents silently swallowed rejections
- **Critical: Webhook idempotency timing** — Dedup key now written AFTER successful processing (was before). Failed processing returns 500 so Stripe retries instead of silently dropping events
- **Critical: XSS in success page** — `esc()` HTML escaper applied to `data.key`, `data.email`, and `err.message` in `success.js` innerHTML
- **High: getDeviceId() error handling** — Added `chrome.runtime.lastError` checks to prevent silent storage failures
- **High: device_id length validation** — Capped at 128 characters in `start-trial.js` and `verify-key.js` to prevent oversized Redis keys
- **High: Trial lookup rate limiting** — `/api/verify-key` trial path (no key) now shares rate limiter, prevents unlimited Redis lookups
- **High: checkout-success hardening** — Added rate limiting (10/min per IP), `session_id` format validation (`cs_` prefix, 256 char max)
- **Medium: Email send deduplication** — `email_sent` flag in Stripe session metadata prevents duplicate emails on page refresh

## [2.1.1] — 2026-04-08

### Added
- **Stripe webhook security**: Full payment lifecycle coverage — `charge.refunded`, `charge.dispute.created`, `customer.subscription.updated` handlers added alongside existing `checkout.session.completed` and `customer.subscription.deleted`
- **Webhook idempotency**: `webhook-event:{event.id}` deduplication key in Redis (24h TTL) prevents duplicate event processing on Stripe retries
- **Webhook audit logging**: All webhook events logged to `webhook-events:{date}` Redis lists with 90-day TTL
- **License key email delivery**: Resend integration sends branded email with license key on purchase. Covers both first-time generation and idempotent retry (user refresh). reply-to set to support@myreceiptpro.com
- **Success page spam folder hint**: "Don't see it? Check your spam or junk folder." shown after purchase

### Fixed
- **Critical: Success page CSP block**: Inline `<script>` in success.html was blocked by Content-Security-Policy `script-src 'self'`. Extracted to external `/js/success.js`. All new purchases were stuck on "Verifying your purchase..." spinner
- **Critical: Stripe Payment Link redirect**: `after_completion` was set to `hosted_confirmation` (Stripe's own page) instead of redirect to `myreceiptpro.com/success?session_id={CHECKOUT_SESSION_ID}`. Users never reached the key delivery page
- **Critical: Email not sent (Vercel process kill)**: `sendKeyEmail()` was fire-and-forget (`.catch(() => {})`), Vercel killed the serverless function before the fetch completed. Changed to `await sendKeyEmail()` to ensure delivery before response
- **ADMIN_SECRET trailing `\n`**: Vercel env var contained literal `\n`, causing all admin-lookup API calls to return 401 Unauthorized
- **Success page false email claim**: "A copy has been sent to email" removed — no email sending existed. Replaced with accurate "Purchased with email. Save your key."
- **CSP `/_vercel/` invalid path**: Updated to `https://va.vercel-scripts.com` for Vercel Analytics/Speed Insights scripts

### Changed
- **Device limit**: MAX_DEVICES reduced from 3 to 2 per license key
- **Refund policy**: Any refund (full or partial) immediately revokes license key — no prorated support
- **Dispute policy**: Chargeback (`charge.dispute.created`) immediately revokes with dispute reason logged
- **Subscription status tracking**: `unpaid` and `incomplete_expired` statuses trigger revoke; `past_due` and `cancel_at_period_end` logged for audit only
- **Email template**: Branded card layout — blue header, dark blue key block, structured sections (How to activate / What you can do with Max / Notes), Ulan Hada Corp copyright footer

### Security
- **Live Stripe webhook endpoint created**: Production endpoint with 5 subscribed events, webhook signing secret synced to Vercel env
- **Resend domain verified**: myreceiptpro.com DKIM record added to GoDaddy DNS, verified by Resend
- **RESEND_API_KEY**: Added to Vercel production environment
- Applied to: myreceiptpro.com (Vercel), chrome-ext-v2 (Chrome Web Store build)

## [2.0.5] — 2026-04-07

### Added
- **What's New banner**: One-time notification after major version upgrades, auto-dismissed, localized in 5 languages (EN/ZH/JA/KO/ES)
- **Online scan completion panel**: Online order scan now shows full completion overlay with stats and export button (matches warehouse scan experience)
- **Extensible receipt type system** — `RECEIPT_TYPES` white-list replaces hardcoded regex. Adding a new receipt type requires one array entry. Each record carries a `type` field (`warehouse`, `gas`, `pharmacy`)
- **Gas Station modal parser** — parses fuel type, gallons, price-per-gallon from gas receipt modal. Uses GraphQL for product name and item number, modal text for quantity and unit price

### Fixed
- **Critical: Cross-year period date range**: Date-range scans crossing year boundaries (e.g., Nov 2023 – Jan 2024) returned 0 receipts. Root cause: `periodToDateRange()` treated the year label as start year for "November - January" periods, but Costco uses it as end year. Fix: when start month > end month, subtract 1 from start year
- **Critical: Gas Station receipts not parsed** — `RECEIPT_REGEX` only matched `In-Warehouse`, silently dropping all Gas Station fuel purchases. Now matches both `In-Warehouse` and `Gas Station` receipt types
- **Coupon unitPrice**: Coupon/discount lines showed `$0.00` unit price in Excel exports. GraphQL returns `itemUnitPriceAmount: 0` for all coupons. Now derives `unitPrice = |amount| / |qty|` automatically
- **Bug: waitForUpdate() timeout always returned true** — now correctly returns `false` on timeout
- **Double punctuation**: Card descriptions showed `details..` due to HTML hardcoded period conflicting with i18n text

### Changed
- Scan phase labels: "Reviewing your receipts" / "Building your report" (was "Processing" / "Generating" — avoids scraping-adjacent terminology)
- Privacy note: "all processing happens locally on your device" (was "nothing is sent anywhere")
- MAX card text: "Understand your 2% cashback" (was "CPA-ready export" — aligns with Executive rewards positioning)
- Share Card: highest-year bar color changed from red to Costco blue
- Excel qty format: integers display cleanly (`1`), gas station gallons show decimals (`19.37`)
- Per-page receipt miss diagnostics with real-time detection
- Applied to: chrome-ext-v2, chrome-ext-v3, safari-extension, safari-xcode

## [3.0.1] — 2026-04-05

### Changed
- **Full compliance terminology cleanup**: Removed all scraper/crawler/bot terminology from codebase per Costco Terms of Use review
  - File renames: `scraper.js` → `receiptParser.js`, `online-scraper.js` → `onlineParser.js`
  - Message constants: `START_SCRAPE` → `START_PARSE`, `SCRAPE_COMPLETE` → `PARSE_COMPLETE`, etc.
  - High-risk terms cleaned: `capture` → `read`, `intercept` → `monitor`, `monkey-patch` → `observe`
- **Trust messaging added** (5 languages): "Only analyzing data already visible in your account — nothing is sent anywhere."
- **Privacy policy**: New "What Data We Access" section
- Applied to: chrome-ext-v2, chrome-ext-v3, safari-extension, safari-xcode, website

## [3.0.0] — 2026-04-04

### Added
- **Apple × Monarch UI redesign** (Safari version): Complete visual overhaul combining Apple's native macOS feel with Monarch Money's data dashboard density
  - SF Pro system font stack, `#f2f2f7` grouped background, single Apple Blue accent
  - Frosted glass navigation bar with backdrop blur
  - Bento 12-column KPI grid with large bold numbers and tabular-nums alignment
  - Pill-style scan phase badges (green=done, blue=active, gray=pending)
  - iOS-style footer row: Activate · Privacy · Terms · Support · Switch Member
- **Auto Share Card** for MAX users: Automatically shows Share Card when new scan data detected (`lastScanAt` > `lastShareAutoShownAt`), 1.5s delay, timestamp prevents repeat popups
- **Dynamic View Report button**: Free → "View Report", MAX → "View Report & Export to Excel"
- **Per-page scan diagnostics**: Logs page-by-page receipt count breakdown when receipts are missed (e.g., `Page 1: parsed 10 | Page 2: parsed 9 (TIMEOUT)`)

### Fixed
- **Critical: Refund receipts not parsed** — 3 separate issues caused refund/return receipts to be silently dropped:
  - `RECEIPT_REGEX` couldn't match `-$269.93` (leading minus before `$`) — fixed regex group
  - `RECEIPT_REGEX` only matched `Total`, not `Refunded Total` — added `(?:Refunded\s+)?` prefix
  - `parseReceipts()` used `parseFloat()` instead of existing `parseAmount()` — trailing minus `$25.00-` returned NaN and was silently skipped
  - Now handles all 4 formats: `$X`, `-$X`, `$X-`, `Refunded Total\n-$X`

### Changed
- Header: "Receipt Pro" left + 🌐 "MyReceiptPro.com" link right (replaces emoji icon + LOCAL/PRIVATE chips)
- Footer links: Privacy → website section, Support → support page (was mailto:)
- "MAX Activated" shown in green in copyright line (not standalone row)
- Scan phase labels shortened: "Preparing", "Scanning", "Processing", "Generating report"
- Applied to: chrome-ext-v3, safari-extension, safari-xcode

## [2.0.3-fix] — 2026-04-02

### Fixed
- **Chrome Web Store rejection (Blue Argon)**: Removed old design/demo HTML files that referenced `https://cdn.tailwindcss.com` via `<script>` tag — violated Manifest V3 remotely hosted code policy
  - Deleted: `popup/code.html`, `Loading/code.html`, `results/results-v2-design.html` (Tailwind CDN mockups)
  - Deleted: `popup/DESIGN.md`, `popup/screen.png`, `popup/preview.html`, `popup/popup-mock.html`, `popup/popup-mock-popup.js`, `results/results-mock.html`, `results/share-card-v2-design.html`, `Loading/` directory, `logo/` directory

### Changed
- Build process upgraded: explicit 17-file whitelist zip (227KB) replaces full-directory zip — prevents non-runtime files from entering release package
- Verified zero remote code references: no `cdn.tailwindcss.com`, `eval()`, or `new Function()` in release package

## [2.0.4] — 2026-04-02

### Fixed
- **Costco quarterly period parser**: Costco rotated quarterly periods from Jan/Apr/Jul/Oct to Feb/May/Aug/Nov on April 1st. Replaced all hardcoded month names with adaptive parser that dynamically handles any `YYYY MonthName - MonthName` format. Future quarterly rotations will not break the parser
- **Build script terser collision**: Fixed variable name collision in minified builds — files sharing global scope via `<script>` or `importScripts` no longer use `toplevel` mangle
- **Safari App Store compliance**: Replaced Stripe external payment link with myreceiptpro.com redirect (Guideline 3.1.1)

### Changed
- **Share Card redesign**: Updated text across all 5 languages (EN/ZH/JA/KO/ES)
  - "Weekly Average" → "Monthly Average" (calculation: days ÷ 30.44)
  - "Your activity" → "You have shopped across"
  - New CTA: "Helps you see if Executive is worth the upgrade — and where your 2% rewards come from."
  - Footer simplified to 2-line layout: blue CTA + gray copyright line
  - Pro promo: "Receipt Pro — all in one place. Now available for Safari and Chrome."
- **Trial modal text**: "MAX Unlocked for 14 Days!" → "MAX Version Unlocked for 2 Weeks!"
- **Product nicknames**: Cleared built-in nickname data — Nicknames sheet now exports as blank template for users to customize

## [2.0.3-hotfix] — 2026-03-27

### Added
- **Multi-country support**: Canada (costco.ca) fully supported — country auto-detection, CAD currency, HST/GST/PST tax breakdown in Excel exports
- **Dynamic tax columns**: Canadian receipts with tax breakdown automatically add HST/GST/PST columns in Excel and CSV exports

### Fixed
- **Share Trial modal text**: UI displayed "30 Days" but actual trial was 14 days — corrected to match 14-day trial duration
- **Early stop threshold**: Increased from 4 → 9 consecutive empty periods — fixes Canadian members with sparse purchase history

### Changed
- Project directory renamed from `Costco Contacts` to `ReceiptPro`

## [2.0.3] — 2026-03-25

### Added
- **PrivacyInfo.xcprivacy**: Privacy manifest for Safari extension (App Store requirement for iOS 17+ / macOS 14.4+)
- **Vercel Web Analytics**: Traffic analytics enabled on myreceiptpro.com (privacy-friendly, no cookies)
- **Vercel Speed Insights**: Core Web Vitals monitoring (LCP, CLS, INP) enabled on myreceiptpro.com
- **Terms of Service link**: Added to Safari extension popup footer (Privacy · Terms · Support)

### Changed
- Safari extension: Stripe external payment link removed — redirects to key activation (App Store Guideline 3.1.1 compliance)
- Safari manifest description: "Analyze your warehouse and online purchases, track store visits, and export receipts to Excel. All data stays local." (trademark compliance)
- Website: "No tracking or analytics" → "No personal data tracking"
- Website: Updated Share Card image, Result Dashboard and Extension Popup screenshots

### Fixed
- Safari extension: Results upgrade modal now directs to myreceiptpro.com instead of external Stripe payment link

## [2.0.1] — 2026-03-23

### Added
- **Excel VLOOKUP formulas**: Purchase Details (Description) and Summary (Product) now link to Item# Nicknames sheet
  - Custom Name filled → shows "English/Chinese"; empty → shows original English name
  - No extra columns — formulas replace static text in existing cells
- **Nicknames pre-fill**: Custom Name column pre-filled from product_names.json (Chinese translations)
- **Share Card 3-line CTA**: "Helps members see if upgrading makes sense, and understand their 2% Executive rewards. All in Receipt Pro — now available as an extension for Safari and Chrome."

### Changed
- Popup text: Quick Scan → "Check your store visits and total spending"; Executive Scan → "Export your warehouse and online order details"
- Messaging shift: "verify" → "understand" throughout extension copy
- All 5 languages updated for Share Card CTA (en/zh/ja/ko/es)

## [2.0.0] — 2026-03-23

### Fixed
- **Critical**: Anchor-based modal matching replaces index-based matching — fixes N→N-1 receipt offset where each receipt's items belonged to the previous receipt in export
- **Critical**: GraphQL items now supplement modal text when regex misses products — fixes 85+ receipts where main items were lost (only coupons/deposits shown)
- **Critical**: Executive `E`-prefixed item lines without leading tab now matched by primary regex (e.g. `E 1476106 PISTACHIOTRF`)
- **High**: Return receipt qty/amount correctly parsed — negative values are valid data (`unit !== 0` replaces `unit > 0`)
- **High**: Qty/unitPrice derivation moved after GraphQL enrichment to prevent premature single-product assignment
- **Medium**: Coupon/adjustment lines excluded from "missing qty" detection — yellow notices now only for genuine data gaps
- **Medium**: Single-SKU return receipts with consistent data no longer show false "bulk receipt" warnings

### Changed
- Detailed Scan uses 3-layer item reading: modal text → GraphQL enrichment → GraphQL supplement
- Button-to-record matching uses total+store anchor instead of array index
- All 4 codebases synced: chrome-ext-v2-draft, chrome-ext, safari-extension, safari-xcode

## [1.7.1] — 2026-03-22

### Added
- Server-side license verification (Upstash Redis + Vercel API)
- Device binding with composite fingerprint (SHA-256, max 3 devices per key)
- HMAC-SHA256 signed tokens for offline license validation
- Token auto-refresh every 7 days via service worker alarm (12h check interval)
- 3-day offline grace period with 3-tier degradation (Active → Grace → Degraded)
- Rate limiting on license APIs (10/min verify, 20/min refresh)
- Audit logging for all license events (90-day retention in Redis)
- Admin API: key lookup, audit viewer, device reset (`/api/admin-lookup`)
- Key revocation API with Stripe webhook auto-revoke on subscription cancel
- Dual signing key support for secret rotation without service disruption

### Fixed
- **Critical**: Date range scan no longer deletes history outside scan window (receiptParser.js)
- **Critical**: Online order range scan now merges by orderNumber instead of full replacement
- **High**: License refresh failure no longer permanently downgrades paying users
- **High**: Service worker validates sender tab ID before saving scan results
- **High**: All popup storage writes routed through service worker serialization
- **High**: Online order activeMemberId always follows most recent scan
- **High**: Dedup key expanded to include receiptNumber, itemsSold, payment
- **Medium**: parseUsDate() now accepts both M/D/YYYY and MM/DD/YYYY formats
- **Medium**: deactivate() clears all 8 related storage keys (no orphaned state)
- **Medium**: "Switch Member" confirm dialog accurately describes data deletion scope
- **Medium**: Server requests have 10s timeout + resp.ok validation + error categorization

### Changed
- License activation requires server verification (local-only keys no longer accepted)
- Error messages rewritten as user-friendly product language
- Key input placeholder changed from format hint to "Enter your license key"
- Trademark text updated: "Trademark pending" → "Receipt Pro™ is a trademark of Ulan Hada Corporation"
- Safari debug flag set to false for production
- Mobile sharecard image now visible on phones (was hidden below 1024px)
- Install section includes official Google/Apple help links

### Security
- Copyright registered with U.S. Copyright Office (eCO submission complete)
- Server-side HMAC prevents client-side key forgery
- Token tampering detected via crypto.timingSafeEqual
- Rate limiting prevents brute-force key enumeration
- Audit trail enables piracy detection and customer support

## [1.7.0] — 2026-03-22

### Added
- Safari Web Extension support (macOS + iOS universal)
- Online Order parsing and export (MAX)
- Tax-Exempt Excel template export (MAX)
- Share Card — annual spending summary with QR code, shareable as PNG
- Multi-language support (EN/ZH/JA/KO/ES) for UI and Share Card
- Date range picker replaces year chips for flexible scan filtering
- Scan mode memory — last selected mode restored on popup open
- Stop Scan — graceful interruption with partial data save
- Early Stop — consecutive empty periods auto-breaks scan
- Data gap detection notice for members with history beyond Costco retention
- Export/Import JSON backup for dataset portability
- 5-column action grid on Results page (Warehouse Report, Accounting, Share, Online, Tax-Exempt)
- Product nickname sheet in Excel exports with pre-filled Chinese names

### Changed
- Pricing: 2-tier model (Free / MAX $9.97/year) via Stripe
- Branding: all user-facing "Ultra" renamed to "MAX"
- Chrome icon updated to official Google Chrome logo
- Hero section redesigned with Share Card preview
- Website fully redesigned (Manrope + Inter, deep blue gradient)

### Fixed
- Executive `E` prefix regex: receipts like `E  1182051  ALMOND ROCA` now parsed correctly
- Negative amount regex: refund receipts with trailing minus (`25.00-`) parsed as -25.00
- DEPOSIT qty: `Math.abs(amount) / 25` handles both purchase and refund
- By Product dedup: same receipt, same itemNum+amount counted only once
- Safari clipboard limitation: download works, clipboard blocked by async policy

### Security
- Copyright headers added to all source files
- IP protection: Terms of Service Section 5 (Intellectual Property) + Section 6 (DMCA)
- Code obfuscation build pipeline (terser)
- Brand attribution in all extension UI pages

## [1.3.0] — 2026-03-13

### Added
- MemberId-based data architecture (12-digit Costco card number)
- Multi-account support (MAX: 2 datasets, Free: 1)
- Incremental scan — skips already-scanned periods
- Phase A/B scan architecture — identity resolution before dataset selection
- Styled XLSX export Template A (Warehouse Insight Report, 3 sheets)
- Styled XLSX export Template B (Accounting Export, 6 sheets)
- 8MB storage limit enforcement

### Changed
- Storage migrated from flat `costcoData` to `datasetsByMemberId`
- React `__reactProps$` onChange trigger for reliable period switching

## [1.0.0] — 2026-02-28

### Added
- Initial release
- Quick Scan (free): receipt-level spending analysis
- Warehouse Scan: item-level detail reading
- CSV export
- Chrome Web Store submission

---

© 2026 Ulan Hada Corporation. All rights reserved.
