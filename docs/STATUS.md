# Macrolens build status

Read this first after any context reset. Phases and exit criteria: `docs/PLAN.md` Section 13.

## Next step

Start P2: write `docs/design.md` (playbook 2.2 HLD + LLD, design-level traceability matrix, diagrams with version and date, photo sequence with one alt branch per FM-001 to FM-005 and FM-011) and ADR-0001 to ADR-0009; retest `gemini-3.6-flash` before ADR-0007 locks; run the independent reviewer subagent; run the 3 gates; lock; commit.

## Phase checklist

| Phase | State |
|---|---|
| P0 Setup and spikes | Done 2026-11-17 |
| P1 Requirements analysis | Done 2026-11-17 (`docs/requirements.md` v1.0.0 locked) |
| P2 Design lock | In progress |
| P3 Scaffold | Not started |
| P4 Domain and data | Not started |
| P5 Services | Not started |
| P6 UI | Not started |
| P7 E2E and quality | Not started |
| P8 Release candidate | Not started |
| P9 UAT and release | Not started |

## Decisions and notes

| Date | ID | Decision or note |
|---|---|---|
| 2026-11-17 | P0 | `gh repo create` was refused by the session permission policy; Zac created `https://github.com/Zac80085/macrolens` and added `origin`; the agent pushed `main` |
| 2026-11-17 | P0 | OQ-001 Resolved: Gemini, USDA and OFF all readable from a `vite preview` origin in Chromium and WebKit. No proxy; ADR-0010 not needed |
| 2026-11-17 | P0 | TC-092 barcode is `5000157024671` (Heinz Beanz, 81 kcal per 100 g) |
| 2026-11-17 | P0 | SheetJS will be installed from the SheetJS CDN tarball 0.20.3, not npm 0.18.5 (high-severity advisory) |
| 2026-11-17 | P0 | The Gemini SDK retries 5 times by default; the client will set `httpOptions.retryOptions.attempts = 1` to keep the FM-001 budget |
| 2026-11-17 | P0 | B-03 cleared: Zac added `.env.local`; T0.3 run live. `gemini-3.6-flash` returned 503 on 4 of 4 calls; `gemini-3.5-flash-lite` returned a valid `AiAnalysis` with a reduced schema |
| 2026-11-17 | P1 | Zac's gap 1 (PRD-D2): 14 compound REQs split into REQ-027 to REQ-060 and REQ-066, REQ-067; originals retired (`docs/requirements.md` 5.7) |
| 2026-11-17 | P1 | Zac's gap 2 (PRD-D4): every REQ that takes input or uses a dependency has a failure-path AC naming an error code |
| 2026-11-17 | P1 | Zac's gap 3: objectives with metric/baseline/target (Section 2), actors and permissions (4), retention and classification (7.1), release criteria (11) added |
| 2026-11-17 | P1 | Zac's gap 4 (R-5): no section marked BLOCKED, because OQ-001 was Resolved in P0 before any design section was drafted |
| 2026-11-17 | P1 | Zac's gap 6: every NFR has a load condition column (Section 6) |
| 2026-11-17 | P2 (pending) | Zac's gap 5 (OD-D1): photo sequence diagram in `docs/design.md` gets one alt branch per FM-001 to FM-005 and FM-011 |
| 2026-11-17 | P2 (pending) | Zac's gap 7: every diagram in `docs/design.md` carries version and date |
| 2026-11-17 | P1 | Full REQ→HLD→LLD→test matrix moves to `docs/design.md` (playbook D-1); `docs/requirements.md` keeps objective→REQ→TC |
| 2026-11-17 | P1 | CoFID carbs stored unchanged (monosaccharide equivalents) for source fidelity; disclosed on About (REQ-061). Conversion by ÷1.05/÷1.10 rejected: it turns measured values into estimates. ADR-0004 records it |
| 2026-11-17 | P2 (pending) | ZXing WASM for `barcode-detector` must be self-hosted (offline, CSP `script-src 'wasm-unsafe-eval'`); design in ADR-0006 |
| 2026-11-17 | P0 | Accuracy findings P0-F1 to P0-F13 in `docs/spikes.md` go into `docs/requirements.md` (P1) and ADRs (P2) |

## P1 independent review: defect dispositions

Reviewer subagent (did not write the artifact) reported 19 defects and 4 advisories on draft 0.1.0. All fixed in 0.2.0 except where a reason is given.

| # | Catalogue ID | Disposition |
|---|---|---|
| 1 | D-1 | Fixed: references to `docs/design.md` LLDs replaced with PLAN Sections 7.3, 7.4, 10.3, 10.4 |
| 2 | R-2/R-4 | Fixed: ASM-005 states Mifflin-St Jeor, BMR floor, rounding to 10 kcal and whole grams |
| 3 | T-3 | Fixed: NFR-012 names `GEMINI_UNAVAILABLE` and `GEMINI_BAD_RESPONSE`; repair request runs only inside the 62,000 ms budget (skipped below 5,000 ms remaining) |
| 4 | PRD-D2 | Fixed: REQ-009 → REQ-076/077/078; REQ-033 → REQ-074/075; REQ-047 → REQ-069/070/071; REQ-050 → REQ-072/073; originals retired |
| 5 | PRD-D2 | Fixed: "Log items" disabled rule is REQ-079 |
| 6 | Consistency | Fixed: REQ-077 states "Check match" when no candidate scores ≥ 0.6 |
| 7 | PRD-D4 | Fixed: failure ACs on REQ-002, REQ-030, REQ-041, REQ-046; REQ-064 names exact codes |
| 8 | R-1 | Fixed: every derived REQ names its parent REQ and a reason |
| 9 | Consistency | Fixed: REQ-066 also fires when the Storage API is missing |
| 10 | F-2 | Fixed: DR-07 multiplier rounding and range; 9.2 rows added |
| 11 | F-2 | Fixed: DR-08 favourite naming from long item names; 80/81 rows added |
| 12 | F-2 | Fixed: DR-09 merge tie and kv records; stale search response row and REQ-031 AC-3 |
| 13 | PRD-D1 | Fixed: "first complete load" defined in REQ-057; AC per feature area; NFR-006 precache rule restored |
| 14 | Consistency | Fixed: REQ-001 covers sugar with and without a target |
| 15 | Consistency | Fixed: TC-061 and TC-070 amended in 12.1 |
| 16 | Consistency | Fixed: brand and the 3 USDA banana rows recorded in `docs/spikes.md` |
| 17 | Consistency | Fixed: REQ-061 states the `CARBS_INCLUDE_FIBRE` exception |
| 18 | Consistency | Fixed: 9.2 size row cites REQ-071 |
| 19 | Lost behaviour | Fixed: Gemini and USDA failure ACs added (REQ-076, REQ-032); REQ-080 Cancel at 3,000 ms; REQ-081 version and build date; REQ-082 key links; REQ-083 theme |
| L1 | Blueprint row 5 | Fixed: REQ-059, REQ-060, REQ-066 use MUST with priority S |
| L2 | Consistency | Fixed: change log shows 0.2.0 review fixes before 1.0.0 lock; this section records the review |
| A1 | Advisory | Kept: baselines are "none (no app)" because no tracker exists; first measurement becomes the reference |
| A2 | Advisory | Kept: retention "Unbounded" (Zac's own history) stated explicitly |
| A3 | Advisory | Kept: NFR-006 and NFR-011 are measured by CI checks, stated in Section 12 |
| A4 | Advisory | Kept: REQ-029 gestures are one trigger for one outcome; REQ-062 redaction is a constraint on the same list |

## Phase log

| Phase | Date | Gates | Result | Next step |
|---|---|---|---|---|
| P0 | 2026-11-17 | Traceability: n/a for spike notes. Syntax & Command: CORS spike run in 2 browsers, 3 of 3 APIs PASS; T0.3 live call not run (FAIL: no key, B-03). Precision: vague-term scan 1 match, inside a quotation from the CoFID user guide | Exit criteria met; T0.3 live call open | B-03, then T0.3, then P1 |
| P0 (T0.3) | 2026-11-17 | Syntax & Command: models list 200; image call valid on `gemini-3.5-flash-lite`; `gemini-3.6-flash` 503 (service-side, recorded as P0-F12); USDA key 200. Precision: scan re-run on `docs/spikes.md` | P0 done | P1 |
| P1 | 2026-11-17 | Traceability PASS: 65 REQ + 13 NFR rows in `docs/requirements.md` Section 12, 0 empty cells, 0 orphans, 18 retired IDs not reused. Syntax & Command PASS: 0 commands, 0 diagrams in the file; 6 placeholders declared; 5 referenced paths exist. Precision PASS: vague-term scan 0 matches (exit 1); every NFR has number, unit, window, load condition; every failure names a catalogue code | Locked v1.0.0; reviewer defects 19 of 19 fixed | P2 |
