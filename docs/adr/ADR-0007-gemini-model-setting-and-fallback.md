# ADR-0007: Gemini model ID is a setting; default gemini-3.6-flash with gemini-3.5-flash-lite on the second attempt

| Field | Value |
|---|---|
| Status | Accepted |
| Date | 2026-11-17 |
| Deciders | Claude Code (designer) |

## Context

Zac uses the Gemini free tier (S-07). Model IDs change over time, so the ID is a setting (REQ-051). Phase 0 and Phase 2 probes with Zac's key got 503 "high demand" or 504 on 11 of 11 Flash calls (7 to `gemini-3.6-flash`, 2 to `gemini-3.8-flash`, 1 to `gemini-3.7-flash`, 1 to `gemini-3.5-flash`), while `gemini-3.5-flash-lite` answered 3 of 3, including a valid image analysis in 9,205 ms (`docs/spikes.md`). The `*-latest` aliases return 404. FM-001 allows 2 attempts within the 62,000 ms cap (NFR-012). Requirements served: REQ-051, REQ-052, REQ-076, REQ-010, NFR-012, NFR-013, ASM-002, ASM-015.

## Options considered

| Option | Advantages | Disadvantages |
|---|---|---|
| A. Default `gemini-3.6-flash`; attempt 2 uses `gemini-3.5-flash-lite` when attempt 1 ends in 503, 504 or timeout | Uses the ASM-002 default (the Flash tier) whenever it answers; still answers when Flash is overloaded; no extra request beyond FM-001 | A failed Flash attempt adds up to 30,000 ms (11,012 ms observed) before the fallback |
| B. Default `gemini-3.5-flash-lite`, no fallback | Answered 3 of 3 probes, in 1,579 ms (text) and 9,205 ms (image) | Flash-Lite is the lower Gemini tier, and its identification and gram accuracy against Flash is unmeasured (TC-094 measures it); contradicts ASM-002's default |
| C. Default `gemini-3.6-flash`, retry the same model | No model switch logic | Observed to fail on every attempt with this key |

## Decision

We will store the model ID in `Settings.geminiModel` with default `gemini-3.6-flash`, and when attempt 1 ends in 503, 504 or a timeout, attempt 2 will use the constant `GEMINI_FALLBACK_MODEL = 'gemini-3.5-flash-lite'` (unless the setting already equals it). The model that produced each analysis is recorded in the `ai.request.succeeded` log event.

## Consequences

| Type | Consequence |
|---|---|
| Positive | Photo analysis keeps working while Flash is overloaded |
| Negative | Analyses can come from 2 different models, so gram estimates differ in quality between days |
| Negative | When Flash is overloaded every analysis spends 1 request on a 503 before the fallback |
| Follow-up | Option B is cheaper on the evidence so far (0 of 11 Flash calls answered). A is kept because nutrient accuracy is the top requirement and its cost is bounded: ≤ 1 extra request (within NFR-013) and ≤ 30,000 ms (within NFR-012). Zac reports in UAT whether analyses take too long; Claude Code reviews `ai.request.failed` counts in Diagnostics during UAT and changes the default if Flash still fails |
