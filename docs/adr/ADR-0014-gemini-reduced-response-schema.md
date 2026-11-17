# ADR-0014: Reduced JSON schema for Gemini output, full validation with zod

| Field | Value |
|---|---|
| Status | Accepted |
| Date | 2026-11-17 |
| Deciders | Claude Code (designer) |

## Context

The full `AiAnalysis` schema converted with `toJSONSchema` (length, item-count and range limits) was rejected by Gemini with 400 "The specified schema produces a constraint that has too many states for serving". With those keywords removed (types, enums and `required` kept; 1,066 bytes), `gemini-3.5-flash-lite` returned output that passed the full zod schema (P0-F10). Requirements served: REQ-076, REQ-010, NFR-013, FM-003, FM-018.

## Options considered

| Option | Advantages | Disadvantages |
|---|---|---|
| A. Send a reduced schema (drop `minLength`, `maxLength`, `minItems`, `maxItems`, `minimum`, `maximum`, `exclusiveMinimum`, `exclusiveMaximum`, `additionalProperties`, `$schema`) as `responseJsonSchema`; validate with the full zod `AiAnalysis` | Accepted by the API; structure enforced by Gemini, limits enforced by zod; repair request covers limit violations | Limits are checked only after the response, which can cost the repair request |
| B. Full schema | All limits sent to the model | Rejected with 400 (`GEMINI_REQUEST_REJECTED`) |
| C. No schema, JSON by prompt only | No schema limits | More malformed responses and repair requests, spending quota (NFR-013) |

## Decision

We will derive the reduced schema from the zod `AiAnalysis` at run time, on first call, with the memoised pure function `toGeminiSchema()` in `src/services/gemini/schema.ts`, send it as `config.responseJsonSchema` with `responseMimeType: 'application/json'`, and validate every response with the full zod schema.

## Consequences

| Type | Consequence |
|---|---|
| Positive | One schema definition serves both Gemini and validation (Rule D-3) |
| Negative | A response with 16 items or out-of-range numbers costs a repair request |
| Follow-up | Claude Code adds to TC-050 1 recorded response that violates a limit, to exercise the repair path, and a check that `toGeminiSchema()` keeps every property name of `AiAnalysis` (Phase 5) |
