# ADR-0012: CoFID fibre from the AOAC column only

| Field | Value |
|---|---|
| Status | Accepted |
| Date | 2026-11-17 |
| Deciders | Claude Code (designer); accuracy-first rule |

## Context

UK labels declare AOAC fibre. CoFID has AOAC fibre for 1,548 of 2,887 rows (53.6%) and NSP (Englyst) fibre for 2,538 (87.9%). NSP excludes resistant starch and lignin and reads lower than AOAC (P0-F3). Requirements served: REQ-025, DR-01, ASM-009.

## Options considered

| Option | Advantages | Disadvantages |
|---|---|---|
| A. AOAC only; `null` when AOAC is `N` or blank | One method throughout, matching labels; unknown values are visible as "≥" (REQ-025) | 46.4% of CoFID foods have unknown fibre |
| B. AOAC, else NSP | Fewer unknowns | Mixes 2 methods in one total; NSP understates fibre with no visible marker |
| C. AOAC, else NSP × a conversion factor | Fewer unknowns | No published single factor; adds an estimate |

## Decision

We will take CoFID fibre from `AOACFIB` only and store `null` when it is not determined.

## Consequences

| Type | Consequence |
|---|---|
| Positive | Fibre totals never mix methods silently |
| Negative | Fibre totals show the "≥" marker whenever a logged CoFID food has no AOAC fibre value (`null`) |
| Follow-up | Zac notes in UAT whether the marker is acceptable; a change would be a new ADR |
