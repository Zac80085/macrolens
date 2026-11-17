# Macrolens build status

Read this first after any context reset. Phases and exit criteria: `docs/PLAN.md` Section 13.

## Next step

Start P1: write `docs/requirements.md` v1.0 (playbook 2.1 blueprint, 12 sections), folding in findings P0-F1 to P0-F13 from `docs/spikes.md`; run the independent reviewer subagent; run the 3 gates; lock; commit. Before P2 locks ADR-0007, retest `gemini-3.6-flash` (it returned 503 on 2026-11-17).

## Phase checklist

| Phase | State |
|---|---|
| P0 Setup and spikes | Done 2026-11-17 |
| P1 Requirements analysis | In progress |
| P2 Design lock | Not started |
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
| 2026-11-17 | P0 | Accuracy findings P0-F1 to P0-F13 in `docs/spikes.md` go into `docs/requirements.md` (P1) and ADRs (P2) |

## Phase log

| Phase | Date | Gates | Result | Next step |
|---|---|---|---|---|
| P0 | 2026-11-17 | Traceability: n/a for spike notes. Syntax & Command: CORS spike run in 2 browsers, 3 of 3 APIs PASS; T0.3 live call not run (FAIL: no key, B-03). Precision: vague-term scan 1 match, inside a quotation from the CoFID user guide | Exit criteria met; T0.3 live call open | B-03, then T0.3, then P1 |
| P0 (T0.3) | 2026-11-17 | Syntax & Command: models list 200; image call valid on `gemini-3.5-flash-lite`; `gemini-3.6-flash` 503 (service-side, recorded as P0-F12); USDA key 200. Precision: scan re-run on `docs/spikes.md` | P0 done | P1 |
