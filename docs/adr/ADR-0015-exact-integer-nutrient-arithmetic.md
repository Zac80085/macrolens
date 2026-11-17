# ADR-0015: Exact nutrient arithmetic in integer micro-units, with per-100 values stored to 3 decimals

| Field | Value |
|---|---|
| Status | Accepted |
| Date | 2026-11-17 |
| Deciders | Claude Code (designer) |

## Context

NFR-001 requires display values with error 0: entry value = per-100 value × quantity ÷ 100, day totals summed unrounded and rounded once, half away from zero. With IEEE 754 doubles, summing n values has a worst-case error of (n − 1) × 2^−53 × Σ|x|: about 2.0 × 10^−7 kcal for 200 entries of 45,000 kcal. Any non-zero error can flip a display digit when the exact total is a half value such as 2.5 kcal or 0.15 g (the Phase 2 confirmation review, defect N1). CoFID sheet `1.3 Proximates` has 65,445 numeric cells: 6,884 with 0 decimals, 37,725 with 1, 20,828 with 2 and 4 with 3. The other 4 cells have 16 decimals, and each is a binary artefact of a value with 2 or 3 decimals, for example 1.7999999999999998 for 1.8 (measured 2026-11-17 with SheetJS 0.20.3). Quantities have 1 decimal place (PLAN 6.4). Open Food Facts, USDA and kJ ÷ 4.184 values can have more decimals. Requirements served: NFR-001, REQ-001, REQ-004, REQ-025.

## Options considered

| Option | Advantages | Disadvantages |
|---|---|---|
| A. Store per-100 values to 3 decimals (`quantisePer100`, half away from zero); compute each entry as the integer `round(per100 × 1,000) × round(quantity × 10)` = value × 10^6; sum integers; round for display with integer arithmetic | Exact: the largest day sum, 9 × 10^12, is below 2^53, so error 0 for any number of entries up to 200 | Values from OFF, USDA, kJ conversion and serving conversion change by at most 0.0005 per 100 g or ml before storage |
| B. Doubles with compensated (Neumaier) summation and `toPrecision(15)` normalisation before rounding | No change to stored values | Error is bounded at about 2 × 10^−9, not 0; the half-boundary result depends on exact entry values having at most 6 decimals, which OFF and kJ values do not guarantee |
| C. Decimal library (for example `decimal.js`) | Exact decimal arithmetic on any input | Adds a dependency to the initial bundle (size not measured; NFR-006 has 99,288 bytes of headroom); every nutrient value becomes an object |

## Decision

We will store every per-100 value in `FoodItem`, `LogEntry` and `FavouriteItem` with at most 3 decimal places (`StoredPer100`, `quantisePer100` in `src/domain/nutrition.ts`), and compute entry values, day totals and display rounding in integer micro-units, as in `docs/design.md` Section 7 (NFR-001) and LLD-03.1.

## Consequences

| Type | Consequence |
|---|---|
| Positive | Displayed totals equal the exact decimal result of the stored values, with no tolerance (NFR-001 "error 0") |
| Positive | For CoFID, quantisation only removes the 4 binary artefacts, so NFR-002 parity is unchanged |
| Negative | A per-100 value with more than 3 decimals from OFF, USDA or a kJ or serving conversion is changed by up to 0.0005 before it is stored; at 5,000 g this changes an entry by up to 0.025 kcal or g |
| Negative | `Nutrients` and `NutrientTotals` hold integers in micro-units, so every display path must call `roundForDisplay` |
| Follow-up | Claude Code writes TC-001, TC-002 and TC-008 with the vectors in `docs/design.md` Section 11.22 before the implementation (Phase 4) |
