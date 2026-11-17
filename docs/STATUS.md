# Macrolens build status

Read this first after any context reset. Phases and exit criteria: `docs/PLAN.md` Section 13.

## Next step

Wait for blocker B-03: Zac creates `.env.local` holding `GEMINI_API_KEY` and `USDA_API_KEY`. Then finish T0.3 (call the Gemini models list endpoint, send 1 image request to `gemini-3.6-flash` with `responseJsonSchema`, read the free-tier RPD in Google AI Studio if shown), record the results in `docs/spikes.md`, commit, and start P1.

## Phase checklist

| Phase | State |
|---|---|
| P0 Setup and spikes | Exit criteria met (OQ-001 Resolved; `docs/spikes.md` written). T0.3 live Gemini call open, blocked by B-03 |
| P1 Requirements analysis | Not started |
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
| 2026-11-17 | P0 | Accuracy findings P0-F1 to P0-F9 in `docs/spikes.md` go into `docs/requirements.md` (P1) and ADRs (P2) |

## Phase log

| Phase | Date | Gates | Result | Next step |
|---|---|---|---|---|
| P0 | 2026-11-17 | Traceability: n/a for spike notes. Syntax & Command: CORS spike run in 2 browsers, 3 of 3 APIs PASS; T0.3 live call not run (FAIL: no key, B-03). Precision: vague-term scan 1 match, inside a quotation from the CoFID user guide | Exit criteria met; T0.3 live call open | B-03, then T0.3, then P1 |
