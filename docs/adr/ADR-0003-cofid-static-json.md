# ADR-0003: CoFID converted at build time and shipped as a static JSON file

| Field | Value |
|---|---|
| Status | Accepted |
| Date | 2026-11-17 |
| Deciders | Claude Code (designer) |

## Context

CoFID 2021 is the first-choice database (S-16). It is published only as a 4,629,542-byte XLSX on GOV.UK. Search must work offline (REQ-057) with p95 ≤ 100 ms (NFR-004), and the data must equal the source (NFR-002). Requirements served: REQ-031, REQ-057, NFR-002, NFR-004, NFR-006.

## Options considered

| Option | Advantages | Disadvantages |
|---|---|---|
| A. Node build script converts sheet `1.3 Proximates` to `public/data/cofid-2021.json`; a verify script checks parity in CI; the service worker precaches the file | 66,687 bytes gzip (measured); no XLSX parser in the app; parity checked on every CI run | The JSON must be regenerated and committed when CoFID changes |
| B. Parse the XLSX in the browser | Always the raw source | Ships an XLSX parser and a 4.6 MB file to the phone; breaks NFR-006 |
| C. Store CoFID rows in IndexedDB on first launch | Indexed queries | Duplicates the static file; adds a migration path for each CoFID release |

## Decision

We will convert CoFID with `scripts/build-cofid.ts` using rules CR-01 to CR-07 of `docs/requirements.md`, commit `public/data/cofid-2021.json`, verify it with `scripts/verify-cofid.ts` in CI, precache it with the service worker and search it in memory.

## Consequences

| Type | Consequence |
|---|---|
| Positive | Offline search after the first complete load; 89.1% headroom on the 600 KB (614,400-byte) gzip budget: 66,687 of 614,400 bytes used |
| Negative | 34 CoFID rows without kcal, protein, carbs or fat are not searchable (spices, stock cubes); USDA search covers them |
| Negative | A CoFID update needs a rebuild and a release; logged entries keep their snapshot (ADR-0008) |
| Follow-up | Claude Code adds `npm run data:cofid` and the CI parity check in Phase 3 and Phase 4 |
