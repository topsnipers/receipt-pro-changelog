# Changelog

All notable changes to Receipt Pro are documented here.

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
- Early Stop — 4 consecutive empty periods auto-breaks scan
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
