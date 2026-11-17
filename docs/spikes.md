# Phase 0 spike notes

| Field | Value |
|---|---|
| Document | `docs/spikes.md` |
| Phase | P0 Setup and spikes (`docs/PLAN.md` Section 13) |
| Date | 2026-11-17 |
| Author | Claude Code |

This file records facts checked against live systems and current documentation on 2026-11-17. Spike scripts ran from a scratch directory outside the repository; the commands and their outputs are summarised here.

## T0.1 Tools and repository

| Check | Command | Result |
|---|---|---|
| Node.js | `node --version` | `v26.5.0` |
| npm | `npm --version` | `11.17.0` |
| Git | `git --version` | `git version 2.39.5 (Apple Git-154)` |
| GitHub CLI auth | `gh auth status` | Logged in as `Zac80085`, scopes `gist`, `read:org`, `repo`, `workflow` |
| Repository | Created by Zac (the agent's `gh repo create` call was refused by the session permission policy) | `https://github.com/Zac80085/macrolens`, public; `main` pushed at `5912bc0` |

## T0.2 CORS spike (OQ-001) and barcode for TC-092

Method: a static page built with Vite 8.3.8 and served by `vite preview` on `http://localhost:4173` sent one request to each API from Playwright 1.65.1 in Chromium and in WebKit 27.2. A request "passes" when the page can read the response status and body. Preflight headers were also read with `curl -X OPTIONS` and an `Origin` header.

| API | Request sent from the page | Chromium | WebKit | Preflight result |
|---|---|---|---|---|
| Gemini | `POST /v1beta/models/gemini-3.6-flash:generateContent` with headers `content-type`, `x-goog-api-key` (invalid test key), `x-goog-api-client` | Readable, status 400 `API_KEY_INVALID` | Readable, status 400 `API_KEY_INVALID` | 200; `access-control-allow-origin` echoes the origin; allows `content-type,x-goog-api-key,x-goog-api-client` |
| USDA FDC | `POST /fdc/v1/foods/search?api_key=DEMO_KEY` with JSON body from PLAN 7.3 | Readable, status 200 | Readable, status 200 | 204; `access-control-allow-origin: *`; allows `x-requested-with, Content-Type, Authorization` only |
| Open Food Facts | `GET /api/v2/product/5000157024671?...` with header `X-User-Agent` | Readable, status 200 | Readable, status 200 | `access-control-allow-origin: *`; allow-list includes `User-Agent` and `X-User-Agent` |

Result: **OQ-001 Resolved.** All 3 APIs accept browser requests from a static origin. No proxy is needed; ADR-0010 is not written and blocker B-05 is not raised.

Constraints found:

- USDA preflight allows only `x-requested-with`, `Content-Type` and `Authorization`. The USDA client MUST send no other custom header.
- A Gemini error response (400) carries CORS headers, so the app can read error bodies and map `API_KEY_INVALID` to `GEMINI_AUTH_FAILED`.
- The USDA `DEMO_KEY` returns `x-ratelimit-limit: 10`; the app requires a personal key (PLAN 7.3 states 1,000 requests per hour per key).

### Open Food Facts findings (ASM-012, ADR-0009 input)

| Item | Finding | Source |
|---|---|---|
| Identification | Docs: "always use a custom User-Agent to identify your app"; form `AppName/Version (ContactEmail)` | `https://openfoodfacts.github.io/openfoodfacts-server/api/` |
| Browser alternative | Docs give none for reads. The CORS allow-list accepts `X-User-Agent`, and the request with `X-User-Agent` passed in both browsers. | Preflight headers above |
| Read rate limit | "15 req/min/IP address for all read product queries" | Same page |
| Licence | Database ODbL 1.0; contents DbCL 1.0; images CC BY-SA 3.0 | Same page |
| Not found | HTTP 404 with body `{"status":0,"status_verbose":"product not found"}` (tested `5010029000580`) | Live request |
| `product_quantity_unit` | Returned only when requested together with `product_quantity` and `quantity` in `fields`; with the PLAN 7.4 field list it was absent for both products tested | Live requests |
| `nutrition_data_per` | `100g` for Coca-Cola Original 330 ml (`5449000000996`), so it does not indicate ml | Live request |
| ml detection | `product_quantity_unit = "ml"` and `quantity = "330 ml"` for both Coca-Cola barcodes tested | Live request |

Products checked (per 100 g or ml as returned):

| Barcode | Product | kcal | Protein | Carbs | Fat | Fibre | Sugar |
|---|---|---|---|---|---|---|---|
| `5000157024671` | Heinz Beanz in a rich tomato sauce, 415 g | 81 | 4.8 | 15.5 | 0.4 | 3.9 | 10.3 |
| `5000112637922` | Coca-Cola 330 ml can | 42.1212 | 0 | 10.6061 | 0 | absent | 10.6061 |

**TC-092 barcode: `5000157024671`** (Heinz Beanz; expects `energy-kcal_100g = 81`).

## T0.3 Gemini (ASM-002) and SDK option names

| Check | Finding | Source |
|---|---|---|
| `gemini-3.6-flash` exists | Yes, Stable; free tier input and output "Free of charge" | `https://aistudio.google.com/docs/models`, `/docs/pricing` (the `ai.google.dev` URLs redirect there) |
| `gemini-3.5-flash-lite` exists | Yes, Stable; free tier "Free of charge" | Same pages |
| Newer Flash model | `gemini-3.8-flash` (Stable, free tier) exists | Same pages |
| Free-tier data use | "Used to improve our products: Yes" for both models; confirms REQ-023 is needed | Pricing page |
| Image input | Not stated per Flash model on the models page; pricing for 3.5 Flash-Lite lists "text / image / video / audio" input | Pricing page |
| Free-tier RPD (ASM-003) | Not published on the rate-limits page; the page says limits are shown in Google AI Studio and RPD resets at midnight Pacific time | `https://aistudio.google.com/docs/rate-limits` |
| SDK version | `@google/genai` 3.4.0 | `npm view @google/genai version` |
| JSON schema option for `generateContent` | `config.responseMimeType: 'application/json'` plus `config.responseJsonSchema` (JSON Schema) or `config.responseSchema` (OpenAPI subset). Use `responseJsonSchema` with the output of zod `z.toJSONSchema`. | `dist/genai.d.ts` in 3.4.0 |
| Abort and timeout | `config.abortSignal: AbortSignal`; `httpOptions.timeout` (ms) | `dist/genai.d.ts` |
| SDK retries | `httpOptions.retryOptions.attempts` "If not specified, default to 5". The app MUST set `attempts: 1` so that `withPolicy` alone applies FM-001 (2 attempts) and the T-2 budget of 61,200 ms holds. | `dist/genai.d.ts` |
| Error type | `ApiError` with `status: number` | `dist/genai.d.ts` |
| Live models list and 1 image call | **Not run: no Gemini key** (`.env.local` missing). Blocker B-03 raised. | n/a |

## T0.4 CoFID 2021 XLSX (ASM-009)

| Item | Value |
|---|---|
| Source page | `https://www.gov.uk/government/publications/composition-of-foods-integrated-dataset-cofid` (last updated 19 March 2021) |
| File URL | `https://assets.publishing.service.gov.uk/media/60538b91e90e07527df82ae4/McCance_Widdowsons_Composition_of_Foods_Integrated_Dataset_2021..xlsx` |
| Local path | `data/raw/cofid-2021.xlsx` (gitignored) |
| Size | 4,629,542 bytes |
| SHA-256 | `436e9445ef2adb2a75f3d7edd51302de3adad25385f9795fc94ba58bd030e97d` |
| User guide | `https://assets.publishing.service.gov.uk/media/60538e66d3bf7f03249bac58/McCance_and_Widdowsons_Composition_of_Foods_integrated_dataset_2021.pdf` (37 pages) |
| Sheets | `List of tables`, `1.1 Notes`, `1.2 Factors`, `1.3 Proximates`, `1.4 Inorganics`, `1.5 Vitamins`, `1.6 Vitamin Fractions`, `1.7` to `1.12` fatty acid sheets, `1.13 Phytosterols`, `1.14 Organic Acids` |
| Header rows | Rows 1 to 3 (label, nutrient code, description); data from row 4 |
| Food rows in `1.3 Proximates` | 2,887 rows; 2,886 distinct food codes |

Columns used from `1.3 Proximates` (0-based index, header text, CoFID nutrient code):

| Field | Index | Header | Code |
|---|---|---|---|
| `code` | 0 | Food Code | n/a |
| `name` | 1 | Food Name | n/a |
| group | 3 | Group | n/a |
| `protein` | 9 | Protein (g) | `PROT` |
| `fat` | 10 | Fat (g) | `FAT` |
| `carbs` | 11 | Carbohydrate (g) | `CHO` |
| `kcal` | 12 | Energy (kcal) (kcal) | `KCALS` |
| `sugar` | 16 | Total sugars (g) | `TOTSUG` |
| `fibre` | 25 | AOAC fibre (g) | `AOACFIB` |

Value counts over the 2,887 rows:

| Field | Numeric | `Tr` | `N` | Blank |
|---|---|---|---|---|
| kcal | 2,849 | 5 | 32 | 1 |
| protein | 2,838 | 49 | 0 | 0 |
| fat | 2,789 | 98 | 0 | 0 |
| carbs | 2,765 | 88 | 32 | 2 |
| sugar | 2,634 | 155 | 96 | 2 |
| AOAC fibre | 1,473 | 75 | 471 | 868 |
| NSP fibre (index 24, `ENGFIB`) | 2,433 | 105 | 344 | 5 |

ASM-009 check: the user guide confirms `Tr` = trace and `N` = "present in significant quantities, but there is no reliable information on the amount". The `Tr` → 0 and `N`/blank → `null` rule matches the guide. Cells are numbers or strings; the converter MUST accept both.

### Findings that need a decision in Phase 1 or 2

| ID | Finding | Evidence | Proposed handling (accuracy first) |
|---|---|---|---|
| P0-F1 | Food code `13-669` is used by 2 different foods | Excel row 55 "Aubergine, flesh and skin, roasted in rapeseed oil"; row 2827 "Watercress, raw" | Keep both rows; give the second occurrence a distinct `sourceId` with a documented suffix rule; `verify-cofid.ts` checks that every `sourceId` is unique |
| P0-F2 | 33 rows have kcal `N` or blank (spices, stock cubes, lemon peel, 1 takeaway pork rib dish); all also have carbs `N` or blank. 1 more row, "Cheese, Brie, rind only" (`12-491`), has kcal 304 and blank carbs. 34 rows in total lack kcal, protein, carbs or fat | Row list from the spike script | Exclude the 34 rows where kcal, protein, carbs or fat is `N` or blank (all 4 are required by `Per100`); list the excluded codes in the build report; 2,853 rows remain |
| P0-F3 | AOAC fibre is numeric or `Tr` for 1,548 of 2,887 rows (53.6%); NSP covers 2,538 (87.9%). AOAC is the UK label method; NSP excludes resistant starch and lignin and reads lower | Counts above; guide page 11 | Keep ASM-009 (AOAC only, else `null`). Days with such foods show the REQ-025 "≥" marker instead of a mixed-method total |
| P0-F4 | CoFID carbohydrate, starch and sugars are "expressed as their monosaccharide equivalent"; UK labels declare carbohydrate by weight. The guide's conversion: total carbohydrate ÷ 1.05, starch ÷ 1.10 | Guide pages 10, 11 and 16 | Store CoFID values unchanged (NFR-002 parity, REQ-012 exact copy). Record in ADR-0004 that CoFID carbs can read up to 10% above a pack label for starchy foods; kcal is unaffected because CoFID uses 3.75 kcal/g for this basis |
| P0-F5 | Values are per 100 g except alcoholic beverages, which are per 100 ml; these are the 31 rows in food groups starting with `Q` | Notes sheet; guide page 9 | `basis = ml` for group prefix `Q`, else `g`; `verify-cofid.ts` checks the count |
| P0-F6 | "Rosemary, dried" (`13-882`) has protein 4.9 + carbs 46.4 + fat 15.2 + AOAC fibre 42.6 = 109.1, which fails the PLAN 6.2 cross-field rule (≤ 105) | Spike script | Apply the cross-field rule to user-entered values only (manual entry, AI estimate, edits), not to rows copied from a database |
| P0-F7 | Tea and instant coffee infusions have kcal `Tr` | Spike script | Stored as 0 under ASM-009 |
| P0-F8 | Open Food Facts limits product reads to 15 per minute per IP; the error catalogue has no code for an OFF 429 | OFF API page | Add a rate-limit error code for OFF in `docs/requirements.md` |
| P0-F9 | The PLAN 7.4 ml rule depends on `product_quantity_unit`, which is returned only when `product_quantity` and `quantity` are also requested | Live requests above | Add `quantity` and `product_quantity` to `fields`; `basis = ml` when `product_quantity_unit = "ml"`, else parse `quantity` for `ml`, `cl` or `l`, else `g` |

## Package versions seen on 2026-11-17 (`npm view <PACKAGE> version`)

| Package | Version |
|---|---|
| `react` | 19.3.1 |
| `vite` | 8.3.8 |
| `typescript` | 7.0.2 |
| `tailwindcss` | 4.3.5 |
| `react-router` | 8.5.1 |
| `dexie` | 4.4.7 |
| `zod` | 4.6.7 |
| `@google/genai` | 3.4.0 |
| `barcode-detector` | 3.2.4 |
| `vite-plugin-pwa` | 2.0.0 |
| `vitest` | 5.1.0 |
| `msw` | 3.0.6 |
| `playwright` | 1.65.1 |
| `xlsx` (npm registry) | 0.18.5, reported by `npm audit` as 1 high-severity vulnerability |
| `xlsx` (SheetJS CDN, `https://cdn.sheetjs.com/xlsx-latest/xlsx-latest.tgz`) | 0.20.3, tarball SHA-256 `8dc73fc3b00203e72d176e85b50938627c7b086e607c682e8d3c22c02bb99fe8` |

Decision for Phase 3: install SheetJS 0.20.3 from the versioned CDN tarball, not the npm registry, because the registry release has a known high-severity advisory and is used to parse an external file.
