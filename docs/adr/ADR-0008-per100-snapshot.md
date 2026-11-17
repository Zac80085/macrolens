# ADR-0008: Each entry stores a per-100 snapshot

| Field | Value |
|---|---|
| Status | Accepted |
| Date | 2026-11-17 |
| Deciders | Claude Code (designer) |

## Context

History must not change when CoFID is re-released, an Open Food Facts product is edited, or a favourite is changed or deleted (REQ-048, REQ-068). Requirements served: REQ-039, REQ-048, REQ-068, NFR-001.

## Options considered

| Option | Advantages | Disadvantages |
|---|---|---|
| A. Copy `per100`, `source`, `sourceId` and `flags` into each `LogEntry` at log time | Past totals never change; entries work offline and without the source | 6 numbers duplicated per entry (about 100 bytes) |
| B. Store only `source` and `sourceId`; look values up when displayed | Smaller records; fixes in a source reach history | History changes silently; USDA and OFF lookups need the network for every past day |

## Decision

We will store a per-100 snapshot in every entry and favourite item and never recompute a past entry from a live source.

## Consequences

| Type | Consequence |
|---|---|
| Positive | Day totals are stable and offline |
| Negative | A corrected source value does not reach entries logged before the correction |
| Follow-up | None |
