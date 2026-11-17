# ADR-0002: Nutrient values come from databases; Gemini identifies foods and estimates grams

| Field | Value |
|---|---|
| Status | Accepted |
| Date | 2026-11-17 |
| Deciders | Zac (S-08), recorded by Claude Code |

## Context

Accuracy of nutrient values is the top requirement (S-04). Zac chose that nutrient numbers come from a food database and that Gemini names the food and estimates grams (S-08). Requirements served: REQ-039, REQ-040, REQ-076, REQ-077, REQ-079.

## Options considered

| Option | Advantages | Disadvantages |
|---|---|---|
| A. Gemini identifies foods and grams; values come from CoFID, USDA or Open Food Facts; Gemini's own per-100 estimate is used only when the user taps "Use AI estimate" | Values trace to a published dataset; the same food always gets the same values; history is auditable by source badge | Needs a matching step that can pick the wrong food; some items have no good match |
| B. Gemini returns nutrients directly | 1 step; every item has values | Values vary between calls and are not traceable to a source; contradicts S-08 |

## Decision

We will take nutrient values only from the selected database food and use Gemini's per-100 estimate only after an explicit "Use AI estimate" tap, flagged `AI_ESTIMATE`.

## Consequences

| Type | Consequence |
|---|---|
| Positive | Database-sourced entries equal their source exactly (OBJ-01, TC-041) |
| Negative | Items whose best match scores below 0.6 need a user action before logging (REQ-079), adding taps |
| Negative | Wrong matches are possible; the user must check the candidate shown (REQ-064) |
| Follow-up | Claude Code reports the top-1 match rate with the optional TC-094 evaluation in Phase 8 |
