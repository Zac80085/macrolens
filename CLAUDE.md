# CLAUDE.md: Macrolens

## Project summary

Macrolens is Zac's personal calorie and macro tracker: a PWA for his iPhone, hosted on GitHub Pages, with all data stored on the phone. Accuracy of nutrient values is the top requirement: values come from CoFID (UK), USDA FoodData Central and Open Food Facts; Gemini only identifies foods and estimates grams.

## Read first

1. `docs/STATUS.md`: current phase and next step. Read it at the start of every session and after any context reset.
2. `docs/PLAN.md`: the plan. Requirement, component, error and test IDs come from here.
3. `docs/ENGINEERING_QA_PLAYBOOK.md`: the quality rules every artifact follows.
4. `docs/requirements.md` and `docs/design.md` once they exist (Phase 1 and 2). When they differ from `PLAN.md`, the newer locked document wins.

## Technology stack

TypeScript (strict), React 19, Vite, Tailwind CSS, React Router (hash routing), Dexie (IndexedDB), zod, `@google/genai`, `barcode-detector` (ZXing WASM polyfill), `vite-plugin-pwa`, Vitest, Testing Library, MSW, fake-indexeddb, Playwright, `@axe-core/playwright`, SheetJS (`xlsx`, build scripts only). Package manager: npm with a committed `package-lock.json`.
<!-- Phase 3: replace this comment with the installed major version of each package and the Node.js version. -->

## Repository layout

| Path | Purpose |
|---|---|
| `src/domain/` | Pure logic and the single definition of schemas (`schemas.ts`) and error codes (`errors.ts`) |
| `src/data/` | Dexie database and repositories |
| `src/services/` | CoFID, USDA, OFF, barcode, image, Gemini, matching, backup, HTTP policy, log |
| `src/screens/`, `src/components/`, `src/app/` | UI |
| `scripts/` | CoFID build and verify, size check, traceability check, vague-term scan |
| `public/data/cofid-2021.json` | Generated CoFID data (committed); regenerate with `npm run data:cofid` |
| `data/raw/` | Downloaded CoFID XLSX (gitignored) |
| `tests/e2e/`, `tests/fixtures/` | Playwright specs and recorded API responses |
| `docs/` | Plan, requirements, design, ADRs, status, UAT, feedback |

## Commands

<!-- UNVERIFIED until Phase 3: run each command from a clean clone, then replace this block with the exact forms that exit 0. -->

```bash
npm ci                          # install
npm run dev                     # dev server
npm run build                   # production build to dist/
npm run preview                 # serve dist/ locally
npm test                        # all unit, integration and component tests
npx vitest run <TEST_FILE> -t "<TC_ID>"   # one test
npm run test:e2e                # Playwright, both projects
npm run test:live               # live API smoke tests TC-090 to TC-093; needs .env.local
npm run lint                    # ESLint
npm run format                  # Prettier write
npm run typecheck               # tsc --noEmit
npm run data:cofid              # rebuild and verify public/data/cofid-2021.json
npm run gates                   # lint, typecheck, coverage, e2e, size, trace, vague-scan, secret scan
```

## Conventions

- Every test name starts with its TC ID from `docs/PLAN.md` Section 12, for example `it('TC-001 scales nutrients for 185 g', ...)`.
- Service functions return `Result<T, AppError>` and do not throw for expected failures. Error codes come only from `src/domain/errors.ts`.
- Every remote call goes through `fetchWithPolicy` or `withPolicy` (`src/services/http/`) with the timeout and retry values in `docs/PLAN.md` Section 8.2.
- Nutrient maths: no rounding before display; use `scaleNutrients`, `sumNutrients` and `roundForDisplay` only.
- A logged entry stores a copy of its per-100 values. Never recompute a past entry from a live source.
- Carbs mean available carbohydrate (fibre excluded). Convert USDA carbs as in `docs/PLAN.md` Section 7.3.
- Fibre and sugar may be `null` (unknown). Never replace `null` with 0.
- Write tests before the implementation for `src/domain/` and `src/services/`.
- New names (fields, components, error codes, routes) are defined once and spelled identically everywhere.

## Workflow rules

- Branch: work on `main` for this solo project; one commit per completed task, message format `<type>(<scope>): <summary> [<IDs>]`, for example `feat(matching): select CoFID candidate above 0.6 [REQ-012, TC-041]`.
- A change is complete only when `npm run lint`, `npm run typecheck` and `npm test` exit 0.
- A phase is complete only when its exit criteria in `docs/PLAN.md` Section 13 pass and `docs/STATUS.md` has a new line.
- Record each decision with 2 or more options as an ADR in `docs/adr/ADR-nnnn-<slug>.md`.
- Ask Zac only for the blockers B-01 to B-07 in `docs/PLAN.md` Section 13. For anything else, choose, record the choice in `docs/STATUS.md`, and continue.

## Prohibitions

- Do not commit `.env.local`, API keys, or `data/raw/`. Never print a key in logs, test output or snapshots.
- Do not edit `docs/ENGINEERING_QA_PLAYBOOK.md` or `docs/PLAN.md`; record changes in `docs/requirements.md`, `docs/design.md` or an ADR.
- Do not edit `public/data/cofid-2021.json` by hand; regenerate it with `npm run data:cofid`.
- Do not let Gemini output become nutrient values except through the user's explicit "Use AI estimate" choice (REQ-012).
- Do not call live APIs from CI or from `npm test`; use the fixtures in `tests/fixtures/`.
- Do not force-push, rewrite published history, or delete the GitHub repository.

## References

- Error catalogue and failure modes: `docs/PLAN.md` Section 8
- Schemas: `docs/PLAN.md` Section 6
- Gemini prompt: `docs/PLAN.md` Section 10.6
- UAT and feedback process: `docs/PLAN.md` Section 14
