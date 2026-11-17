# ADR-0004: Carbs stored as available carbohydrate, each source in its own published convention

| Field | Value |
|---|---|
| Status | Accepted |
| Date | 2026-11-17 |
| Deciders | Claude Code (designer); accuracy-first rule from Zac's kickoff |

## Context

UK labels declare carbohydrate as available carbohydrate by weight, excluding fibre. CoFID carbohydrate and sugars exclude fibre but are expressed as monosaccharide equivalents, which reads up to 10% higher for starch-rich foods (CoFID user guide pages 10, 11 and 16; P0-F4). USDA nutrient 1005 is carbohydrate by difference, which includes fibre. Requirements served: NFR-003, REQ-039, REQ-061.

## Options considered

| Option | Advantages | Disadvantages |
|---|---|---|
| A. USDA: subtract fibre (1079) from 1005, flag `CARBS_INCLUDE_FIBRE` when 1079 is missing. CoFID: store unchanged. About states each convention | Every stored value is a published or directly derived number; CoFID parity holds exactly (NFR-002, REQ-039) | CoFID carbs for starchy foods read about 5% to 10% above a pack label for the same food |
| B. Same USDA rule; convert CoFID to weight by total ÷ 1.05 (or starch ÷ 1.10, sugars ÷ 1.05) | Closer to label convention | The divisor depends on the starch and sugar split, which is `N` or blank for 1 or more of starch and sugars in rows whose number Phase 0 did not measure; turns measured values into estimates; breaks parity |
| C. Store USDA 1005 unchanged | No conversion code | Counts fibre as carbs for USDA items, unlike labels and CoFID |

## Decision

We will store USDA carbs as nutrient 1005 minus nutrient 1079 (flagging `CARBS_INCLUDE_FIBRE` when 1079 is missing), store CoFID and Open Food Facts carbs unchanged, and state each source's convention on the About screen (REQ-061).

## Consequences

| Type | Consequence |
|---|---|
| Positive | Every value traces to its source without an estimated factor |
| Negative | A day mixing CoFID and labelled foods mixes 2 carbohydrate conventions; the difference is at most 10% of the CoFID part |
| Negative | kcal is unaffected, but the carbs target (computed at 4 kcal/g) and CoFID carbs differ in convention |
| Follow-up | Zac compares a CoFID food with a pack label in UAT step U-07 and records any concern in `docs/FEEDBACK.md` |
