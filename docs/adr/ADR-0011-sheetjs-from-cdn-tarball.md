# ADR-0011: SheetJS installed from the SheetJS CDN tarball

| Field | Value |
|---|---|
| Status | Accepted |
| Date | 2026-11-17 |
| Deciders | Claude Code (designer) |

## Context

`scripts/build-cofid.ts` parses the CoFID XLSX (HLD-10). The npm registry's `xlsx` is 0.18.5, which `npm audit` reports as 1 high-severity vulnerability; SheetJS publishes 0.20.3 only at `https://cdn.sheetjs.com/xlsx-0.20.3/xlsx-0.20.3.tgz`; that versioned tarball and the `xlsx-latest` tarball measured in Phase 0 have the same SHA-256, `8dc73fc3b00203e72d176e85b50938627c7b086e607c682e8d3c22c02bb99fe8` (2,409,319 bytes, both hashed 2026-11-17; `docs/spikes.md`). `gitleaks` and `npm audit --audit-level=high` run in CI (`docs/design.md` Section 6.4, NFR-008). Requirements served: NFR-002.

## Options considered

| Option | Advantages | Disadvantages |
|---|---|---|
| A. `npm install https://cdn.sheetjs.com/xlsx-0.20.3/xlsx-0.20.3.tgz` as a dev dependency, pinned with its integrity hash in `package-lock.json` | Current release with the fix; build-time only | Depends on the SheetJS CDN for installs |
| B. npm `xlsx` 0.18.5 | Standard registry install | Known high-severity advisory; `npm audit` fails |
| C. Another XLSX parser (for example `exceljs`) | Registry install | Different cell typing; parity rules would need re-validation |

## Decision

We will install SheetJS 0.20.3 from the versioned SheetJS CDN tarball as a dev dependency used only by `scripts/`.

## Consequences

| Type | Consequence |
|---|---|
| Positive | No known advisory in the build toolchain |
| Negative | `npm ci` needs `cdn.sheetjs.com` reachable |
| Follow-up | Claude Code installs the versioned URL, checks its SHA-256 against the value above, and lets `package-lock.json` hold the integrity (Phase 3) |
