# ADR-0006: Barcode decoding with the ZXing WASM ponyfill, self-hosted

| Field | Value |
|---|---|
| Status | Accepted |
| Date | 2026-11-17 |
| Deciders | Claude Code (designer) |

## Context

iOS Safari has no enabled `BarcodeDetector`. The primary client is an iPhone (ASM-001). `barcode-detector` 3.2.4 wraps `zxing-wasm` 3.1.7; by default it fetches `zxing_reader.wasm` from jsDelivr at run time, which is outside the CSP `connect-src` list (NFR-009) and unavailable offline (`docs/spikes.md` Phase 1 addendum). Requirements served: REQ-074, REQ-075, NFR-006, NFR-009.

## Options considered

| Option | Advantages | Disadvantages |
|---|---|---|
| A. `barcode-detector/ponyfill` with `prepareZXingModule({ overrides: { locateFile } })` pointing at a copy of `zxing_reader.wasm` served from the app origin; dynamic import when the Scan tab opens; WASM precached | Same-origin under CSP; offline after first complete load; initial JS unchanged | 410,992 bytes gzip of WASM in the precache; CSP needs `'wasm-unsafe-eval'` |
| B. Default jsDelivr WASM | No copy step | Breaks NFR-009 and offline scanning; depends on a CDN |
| C. Native `BarcodeDetector` only | No WASM | Not available on iOS Safari; fails ASM-001 |
| D. Typed barcode only | No camera code | Fails REQ-074 |

## Decision

We will use `barcode-detector/ponyfill`, copy `zxing_reader.wasm` into the build output at a versioned path, load it through `locateFile` only when the scanner starts, precache it in the service worker, and allow `'wasm-unsafe-eval'` in `script-src`.

## Consequences

| Type | Consequence |
|---|---|
| Positive | Scanning works on iOS Safari and offline after the first complete load |
| Negative | The first complete load downloads about 411 KB more |
| Negative | `'wasm-unsafe-eval'` widens `script-src` for WebAssembly compilation |
| Follow-up | Claude Code adds the `zxingWasm()` Vite plugin (LLD-09.1), which fails the build when the WASM file's SHA-256 differs from `ZXING_WASM_SHA256` = `33c8c50be17d0bb05454b4fd3ffb5b5ab9a8e8d1c0ebeb21a1adf4a89c1a8b59` (`docs/design.md` Section 6.3; Phase 3) |
