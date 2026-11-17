# ADR-0013: zod/mini and lazy chunks to meet the initial JavaScript budget

| Field | Value |
|---|---|
| Status | Accepted |
| Date | 2026-11-17 |
| Deciders | Claude Code (designer) |

## Context

NFR-006 caps initial JavaScript at 300 KB gzip; 1 KB = 1,024 bytes, so the cap is 307,200 bytes (`docs/design.md` Section 7). Measured gzip sizes (`docs/spikes.md` Phase 2 addendum): React + React DOM + React Router 101,853; Dexie 32,762; zod classic 92,626; `zod/mini` 13,297; `@google/genai` 78,510; `barcode-detector/ponyfill` 14,770. Requirements served: NFR-006, NFR-004.

## Options considered

| Option | Advantages | Disadvantages |
|---|---|---|
| A. `zod/mini` everywhere; `@google/genai` and `barcode-detector` loaded by dynamic `import()` on first use | Estimated initial JS 101,853 + 32,762 + 13,297 + 60,000 app code = 207,912 bytes, 32.3% headroom (99,288 bytes) | Functional `zod/mini` API is more verbose |
| B. zod classic; lazy Gemini and barcode | Familiar API | Estimated 287,241 bytes, 6.5% headroom (19,959 bytes); app growth breaks NFR-006 |
| C. Hand-written validators | Smallest | Loses the single schema definition (Rule D-3) and JSON Schema export |

## Decision

We will define all schemas once in `src/domain/schemas.ts` with `zod/mini`, and load `@google/genai` and `barcode-detector/ponyfill` with dynamic `import()` from their service modules.

## Consequences

| Type | Consequence |
|---|---|
| Positive | 32.3% headroom on the initial budget; offline features never load the Gemini SDK |
| Negative | The first photo analysis and first scan wait for a chunk download (78,510 and 14,770 bytes gzip) |
| Follow-up | Claude Code adds `scripts/check-size.ts`, which fails the build above 307,200 bytes gzip of initial JavaScript (Phase 3) |
