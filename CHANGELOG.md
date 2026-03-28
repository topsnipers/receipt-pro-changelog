# Changelog

All notable changes to Receipt Pro are documented here.

## [2.0.4] — 2026-03-27

### Added
- **Multi-country support**: Canada (costco.ca) fully supported — country auto-detection, CAD currency, HST/GST/PST tax breakdown in Excel exports
- **Dynamic tax columns**: Canadian receipts with tax breakdown automatically add HST/GST/PST columns in Excel and CSV exports

### Fixed
- **Share Trial modal text**: UI displayed "30 Days" but actual trial was 14 days — corrected to "MAX Unlocked for 14 Days!"
- **Early stop threshold**: Increased from 4 → 9 consecutive empty periods — fixes Canadian members with sparse purchase history being missed by aggressive early stop

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
- **Critical**: Executive `E`-prefixed item lines without leading tab now captured by primary regex (e.g. `E 1476106 PISTACHIOTRF`)
- **High**: Return receipt qty/amount correctly captured — negative values are valid data (`unit !== 0` replaces `unit > 0`)
- **High**: Qty/unitPrice derivation moved after GraphQL enrichment to prevent premature single-product assignment
- **Medium**: Coupon/adjustment lines excluded from "missing qty" detection — yellow notices now only for genuine data gaps
- **Medium**: Single-SKU return receipts with consistent data no longer show false "bulk receipt" warnings

### Changed
- Detailed Scan uses 3-layer item capture: modal text → GraphQL enrichment → GraphQL supplement
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
- **Critical**: Date range scan no longer deletes history outside scan window (scraper.js)
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
- Online Order scraping and export (MAX)
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
- Warehouse Scan: item-level detail extraction
- CSV export
- Chrome Web Store submission

---

© 2026 Ulan Hada Corporation. All rights reserved.
