# Changelog

All notable changes to Seed Gift Forge are documented here. Versioning follows [SemVer](https://semver.org).

## [1.0.1] — 2026-10-19 · “Confirmed”
Security & correctness update after an external audit. **Upgrade recommended.**
SHA-256: `1eef9946b04894a73c7860dc7d0b3dd48fbe1d28d1981737b7f24b3f402ae5c0`

### Fixed
- Export could use a stale derived wallet if Seed/Passphrase changed just before export (#1)
- Watch-only QR descriptor now preserves the wallet's exact derivation steps, with address round-trip check (#2)
- UR multipath (external/internal pair) keypath parsed per BCR-2020-007 — previously produced wrong addresses (#3)
- Wallet verification now compares account key, fingerprint, script, network and addresses #1–#3; partial checks no longer report MATCH (#4)
- Unreadable imported wallet data now blocks export (#5)
- Self-test failure now locks export for the whole session (#6)
- Strict descriptor parsing: wildcard only at the end, integer non-hardened indexes, no invented paths (#7)
- Coin/HEX entropy in SHA-256 mode must be whole bytes — no zero-padding collisions (#8)
- BIP32 master key validity check; card entropy now uses draw-with-replacement

### Fixed (audit round 2)
- Import edits/QR uploads invalidate wallet & verification immediately; request revisioning prevents stale async results (R2-1)
- zpub QR restricted to standard-path SegWit wallets with address round-trip check (R2-2)
- Cross-check result is stateful and bound to its inputs; NOT MATCH blocks export (R2-3)
- Verification compares change addresses (BIP389) as well as receive (R2-4)
- Canvas fallback no longer drops ImageData pixels (JPEG QR) (R2-5)
- Failed QR image decode blocks export; added "clear import" button (R2-6)
- Multisig Ypub/Zpub rejected at parse time (R2-7)
- UR multipath pair indexes fully validated (R2-8)

### Fixed (audit round 3)
- Cross-check freezes its inputs before awaiting and discards results if data changed mid-check; no stale MATCH (R3-1)
- Export is re-entrancy safe (`exporting` state, snapshot inside try/finally) (R3-2)
- PNG tRNS transparency for grayscale/RGB; canvas fallback whenever the internal decoder finds no QR; image-load timeout (R3-3)
- Imported xpub requires a script type before export

### Changed
- PDF password restricted to printable ASCII
- In-app self-test expanded from 10 to 13 checks
- Verification mismatch/partial now blocks export (was a warning)
- Notices for missing UR parent fingerprint, assumed xpub paths, filtered entropy characters, multi-frame QR
- Seed source order: Manual (default) → Real Entropy → System CSPRNG; no seed is auto-generated on page load

## [1.0.0] — 2026-10-18 · “First Block”
Initial public release. *(superseded by 1.0.1)*
SHA-256: `3fc8620c2796cae5ae6c797b24d21a7dad4666e8db79e3a9758899e44832e94e`
- One-stop seed workflow (CSPRNG / Real Entropy / manual), BIP39 passphrase, BIP84 derivation
- Import & verify via Output Descriptor, UR, BBQr, x/y/zpub
- 7 gift designs, A4 tri-fold with printed fold guides, vector signatures
- AES-256 (PDF R6) encrypted export with embedded Thai fonts
