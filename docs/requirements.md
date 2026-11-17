# Macrolens: Requirements

## 1. Document control

| Field | Value |
|---|---|
| Document | `docs/requirements.md` |
| Version | 1.0.0 |
| Status | Locked (2026-11-17). Any change increments the version and re-verifies every downstream artifact (playbook Rule D-4) |
| Owner | Zac (product owner, sole user) |
| Author | Claude Code (requirements analyst, Phase 1) |
| Governing standard | `docs/ENGINEERING_QA_PLAYBOOK.md` v1.0.0, Section 2.1 |
| Inputs | `docs/PLAN.md` v1.0.0; `docs/spikes.md` (Phase 0 findings P0-F1 to P0-F13) |
| Supersedes | `docs/PLAN.md` Sections 2, 3, 8.1 and 15 where they differ (`CLAUDE.md`: the newer locked document wins) |

Change log:

| Version | Date | Change |
|---|---|---|
| 0.1.0 | 2026-11-17 | Draft from `docs/PLAN.md` v1.0.0 and Phase 0 findings |
| 0.2.0 | 2026-11-17 | 19 defects from the independent review fixed (list in `docs/STATUS.md`) |
| 1.0.0 | 2026-11-17 | Gates run (report in `docs/STATUS.md`); locked |

### 1.1 Changes from `docs/PLAN.md`

| Change | Reason |
|---|---|
| 18 compound requirements split into single-behaviour requirements with new IDs; the 18 original IDs are retired (Section 5.8) | Playbook PRD-D2: one MUST clause and one verifiable outcome per ID |
| Derived requirements added for behaviours that `docs/PLAN.md` Sections 8.2 and 11 name without an ID: REQ-061 to REQ-065, REQ-068, REQ-079 to REQ-083 | Playbook PRD-D5: every behaviour traces to a requirement |
| 2 error codes added: `OFF_RATE_LIMITED`, `GEMINI_REQUEST_REJECTED` | P0-F8, P0-F11 |
| NFR-012 now names its error codes and bounds the repair request inside the 62,000 ms cap; NFR-013 added (at most 3 Gemini requests per analysis) | Rule T-3; free-tier quota (ASM-003) |
| NFR-006: the service worker precaches the CoFID data file after first load; the app reads it into memory on first search | REQ-057 needs CoFID offline even before a first search |
| CoFID conversion rules made explicit (Section 7.3) | P0-F1 to P0-F7 |
| The design-level matrix (requirement → HLD → LLD → interface → test) is built in Phase 2; this document keeps objective → requirement → test | Playbook Rule D-1: a level-1 artifact references only level-0 and level-1 items |

## 2. Problem statement and objectives

Zac wants to count his calories and macronutrients every day on his iPhone (S-01, S-03). He has no tracker in use. The most important property is the accuracy of the nutrient values (S-04). Adding food has to need little effort: by photo with AI help (S-05), by description, search, barcode or manual entry (S-14), and by re-adding saved favourites (S-06). The app must not need the App Store (S-02) and keeps data on the phone (S-09).

Baselines: Zac uses no tracker, so baselines for app-specific metrics are recorded as "none (no app)"; the first measurement in Phase 8 or UAT becomes the reference.

| ID | Objective | Metric | Baseline | Target | Measurement window and point |
|---|---|---|---|---|---|
| OBJ-01 | Nutrient values are accurate to their source | (a) Share of database-sourced entries whose per-100 values equal the source record; (b) absolute difference between the app's kcal per 100 g and the pack label for scanned UK products | (a) none (no app); (b) none (no app) | (a) 100%; (b) ≤ 1 kcal per 100 g for 3 of 3 products | (a) Every CI run (TC-010, TC-041) and TC-093 after deploy; (b) UAT step U-06 |
| OBJ-02 | Zac keeps a day-to-day count | Days in the UAT window with 1 or more entries logged by Zac | 0 of 3 (no tracker in use) | 3 of 3 | UAT step U-11, 3 consecutive days |
| OBJ-03 | Adding food takes at most 4 taps and every value stays editable | Taps from Today to logged entries for a 2-item photo when the top matches are accepted (iOS shutter and "Use Photo" not counted) | none (no app) | ≤ 4 taps | TC-081 counts taps; UAT step U-04 |
| OBJ-04 | Repeated meals are re-added without re-entering detail | Taps from Today to log a saved meal favourite at multiplier 1.00 | none (no app) | ≤ 3 taps | TC-084 counts taps; UAT step U-08 |
| OBJ-05 | The app works as an installed phone app with data on the phone | Pass count of the install, offline and backup tests | none (no app) | TC-086 and TC-087 pass; UAT steps U-01, U-09 and U-10 pass | Each CI run; UAT |
| OBJ-06 | The build is verified at every level | Gate results, coverage and UAT feedback status | none (no app) | 3 of 3 gates PASS; NFR-011 coverage met; 100% of `docs/FEEDBACK.md` items have a status | Phase 8 and Phase 9 gate reports |

## 3. Scope

In scope (MVP):

| Item |
|---|
| Daily log of entries with totals against targets for kcal, protein, carbs, fat, fibre and sugar. |
| Adding food by photo, text description, database search, barcode, manual entry and favourites. |
| Editing and deleting entries, with undo for deletion. |
| Single-item and meal favourites with portion scaling. |
| Targets computed from a profile, with per-target override. |
| On-device storage, JSON backup export and import. |
| Installable PWA on iOS Safari, with offline use of local features. |
| Settings for API keys, Gemini model ID, About and Diagnostics. |

Out of scope (MVP):

| Item | Reason |
|---|---|
| Accounts, cloud sync and multiple users. | S-09: data stays on the phone. |
| A native iOS or Android app. | S-02. |
| A backend server or API proxy. | OQ-001 Resolved: all 3 APIs accept browser requests (`docs/spikes.md` T0.2). |
| Adding new items to an existing favourite. | Not in `docs/PLAN.md`; a new favourite can be saved instead (REQ-041, REQ-042). |
| Nutrition-label photo reading, charts, weight log, recipe builder, water tracking, copy-yesterday and cloud backup. | Not requested for the MVP; listed in `docs/PLAN.md` Section 16. |

## 4. Actors and permissions

| Actor | Type | Permitted operations |
|---|---|---|
| Zac | Person, sole user, device owner | All operations in Section 5: read, add, edit and delete entries and favourites; set profile, targets, keys and model; export and import backups; view About and Diagnostics |
| Gemini API | External system (Google) | Receives one photo or one description per analysis, the system instruction and the response schema, only after REQ-023 acceptance; receives the key and model ID for a key test. Receives no log entries, profile data or USDA key |
| USDA FoodData Central API | External system (USDA) | Receives a search query and the USDA key. Receives no other data |
| Open Food Facts API | External system (Open Food Facts) | Receives a barcode and the `X-User-Agent` identification. Receives no key and no other data |
| GitHub Pages | External system (GitHub) | Serves the static app files. Receives no user data |

No other actor exists: the app has no accounts, roles or sharing.

## 5. Functional requirements

Priority: **M** = Must (MVP), **S** = Should (MVP if the gate budget allows). Each requirement has one MUST clause and one outcome. "AC-n" labels its acceptance criteria; an AC that names an error code is a failure path (PRD-D4). Error codes are defined in Section 9.1. A derived requirement names its parent and the reason in the Source column (Rule R-1). "PLAN" means `docs/PLAN.md` v1.0.0.

### 5.1 Today and entries

| ID | Requirement | Pri | Source | Acceptance criteria |
|---|---|---|---|---|
| REQ-001 | The system MUST show, for the selected date, the total, the target and the remaining amount of each nutrient that has a target, and the total alone for a nutrient with no target. | M | S-03, S-13 | AC-1: Given entries totalling 1,200 kcal and a 2,430 kcal target, when Today opens, then it shows "1,200 / 2,430 kcal" and "1,230 left". AC-2: Given a protein total of 150 g and a target of 135 g, then protein remaining shows "−15 g" in the over-target style. AC-3: Given a sugar target of `null`, then sugar shows the total only; given a sugar target of 50 g and 20 g eaten, then sugar shows "20 / 50 g" and "30 left". AC-4: Given IndexedDB cannot open, then `STORAGE_UNAVAILABLE` is shown full screen. |
| REQ-002 | The system MUST list the selected date's entries in one list ordered by consumption time ascending, then by creation time ascending. | M | S-15 | AC-1: Given entries at 08:10, 13:05 and 07:30, then the order is 07:30, 08:10, 13:05, and each row shows time, name, quantity with unit, kcal, protein, carbs and fat. AC-2: Given 2 entries at 12:00, then the one created first is listed first. AC-3: Given 0 entries, then "No entries yet" is shown and every total is 0. AC-4: Given the read fails, then `STORAGE_UNAVAILABLE` is shown. |
| REQ-004 | The system MUST save edits to an entry's name, quantity, consumption time and per-100 values and recalculate the entry and the day totals. | M | S-05 | AC-1: Given an entry of 100 g at 200 kcal per 100 g, when the quantity changes to 150 g and is saved, then the entry shows 300 kcal and the day total rises by 100 kcal. AC-2: Given quantity 0, then Save is disabled with `VALIDATION_FAILED` "Enter 0.1 to 5,000" and the entry is unchanged. AC-3: Given the write fails, then `STORAGE_WRITE_FAILED` is shown with Retry and the form keeps its values. |
| REQ-027 | The system MUST let the user select the previous day, the next day or any date from a date picker as the selected date. | M | S-03; derived from REQ-001: a day-to-day count needs days other than today | AC-1: Given 2026-11-17 selected, when "previous day" is tapped, then the header shows 2026-11-16 and that day's entries. AC-2: When "next day" is tapped on 2026-11-16, then the header shows 2026-11-17. AC-3: When 2026-10-01 is picked, then the header shows 2026-10-01. AC-4: When the picker is dismissed without a choice, then the selected date is unchanged. |
| REQ-028 | The system MUST store an entry added while date D is selected with `date` = D and a consumption time on D that defaults to the current clock time. | M | S-03; derived from REQ-027: food added for a past day belongs to that day | AC-1: Given 2026-11-16 selected at 14:20 on 2026-11-17, when food is logged, then the entry has `date = 2026-11-16` and time 14:20. AC-2: Given the write fails, then `STORAGE_WRITE_FAILED` is shown and no entry is stored. |
| REQ-029 | The system MUST delete an entry only after the user confirms the deletion. | M | S-03; derived from REQ-004: correcting the log includes removing a wrong entry | AC-1: Given an entry, when the user swipes left or long-presses and taps "Delete" in the dialog, then the entry is removed and the totals fall by its values. AC-2: When "Cancel" is tapped, then the entry is unchanged. AC-3: Given the write fails, then `STORAGE_WRITE_FAILED` is shown and the entry stays. |
| REQ-030 | The system MUST restore a deleted entry with its original `id` when "Undo" is tapped within 5,000 ms of the deletion. | M | S-03; derived from REQ-029: a mistaken delete must be reversible | AC-1: Given a deleted entry, when "Undo" is tapped at 4,000 ms, then the entry returns with its original `id` and values. AC-2: At 5,000 ms the Undo control is removed and the deletion stays. AC-3: Given the restore write fails, then `STORAGE_WRITE_FAILED` is shown and Undo stays available until 5,000 ms. |
| REQ-013 | Every entry row and review row MUST show a source badge: "CoFID", "USDA", "Open Food Facts", "Manual" or "AI estimate". | M | S-04 | AC-1: Given one entry from each of the 5 sources, when Today opens, then each row shows the matching badge text. AC-2: Given one review item from each database source, then each review row shows the matching badge text. (Display only; no failure path.) |
| REQ-025 | The day totals MUST mark each nutrient whose total is incomplete with a "≥" prefix and the count of entries without a value for it. | M | S-04; derived from REQ-001: a total that silently skips unknown values reads as complete | AC-1: Given 2 entries, one with fibre 3 g and one with fibre `null`, then fibre shows "≥ 3 g" and "1 item missing fibre". AC-2: Given every entry has fibre, then no "≥" is shown. |
| REQ-067 | The Today screen MUST show the number of AI-estimated entries on the selected date when it is 1 or more. | M | S-04; derived from REQ-040: AI-estimated values are less accurate than database values | AC-1: Given 1 entry with source `ai_estimate`, then Today shows "Includes 1 AI-estimated item". AC-2: Given 0 such entries, then no notice is shown. |

### 5.2 AI analysis

| ID | Requirement | Pri | Source | Acceptance criteria |
|---|---|---|---|---|
| REQ-076 | The system MUST turn a photo into a review list of the foods Gemini identifies, each with a name and an estimated quantity. | M | S-05, S-08 | AC-1: Given the TC-090 photo of chicken and rice, when analysis completes, then the review lists a chicken item and a rice item, each with a name and an estimated quantity in g. AC-2: Given no Gemini key, then `NO_API_KEY` is shown with a link to Settings. AC-3: Given 0 items returned, then `NO_FOOD_DETECTED` is shown with "Retake" and "Describe instead". AC-4: Given a file that cannot be decoded, then `IMAGE_DECODE_FAILED` is shown. AC-5: Given 5xx responses or timeouts on 2 attempts, then `GEMINI_UNAVAILABLE` is shown with Retry and the photo stays in memory. AC-6: Given a response that fails validation after 1 repair request, then `GEMINI_BAD_RESPONSE` is shown with Retry and "Describe instead". AC-7: Given a key rejected during analysis, then `GEMINI_AUTH_FAILED` is shown with a link to Settings; given an unknown model, then `GEMINI_MODEL_NOT_FOUND` with a link to Settings. AC-8: Given a 400 response without `API_KEY_INVALID`, then `GEMINI_REQUEST_REJECTED` is shown with "Describe instead". |
| REQ-010 | The system MUST turn a typed food description of 3 to 500 characters into a review list of the foods Gemini identifies, each with a name and an estimated quantity. | M | S-14 | AC-1: Given "2 boiled eggs and 1 slice of wholemeal toast with butter", when submitted, then the review lists the eggs, the toast and the butter with estimated grams. AC-2: Given "ab" or a 501-character text, then `VALIDATION_FAILED` is shown and no request is sent. AC-3: Given a 429 response, then `GEMINI_RATE_LIMITED` is shown with the wait time and buttons for Search and Manual. |
| REQ-077 | The system MUST attach to each AI-identified item up to 3 database candidates and select the best one when its match score is 0.6 or more. | M | S-08, S-16; derived from REQ-039: nutrient values come from a database | AC-1: Given an item "rice, white, basmati, boiled" with a CoFID candidate scoring ≥ 0.6, then that candidate is selected with its badge and up to 2 more candidates are listed. AC-2: Given the best score is below 0.6, then no candidate is selected and the row shows "Check match". AC-3: Given the CoFID file cannot load, then every item shows "Check match" and `COFID_UNAVAILABLE` is shown once. AC-4: Given USDA fails for 1 item, then that item keeps its CoFID candidates. |
| REQ-023 | The system MUST show the free-tier data notice and receive "I understand" before it sends the first photo or description to Gemini. | M | S-07; derived from REQ-076 and REQ-010: Gemini's free tier may use submitted data (`docs/spikes.md` T0.3) | AC-1: Given no prior acceptance, when "Analyse" is tapped, then the notice "Gemini's free tier may use the photos and text you send to improve Google products" shows and 0 requests reach Gemini. AC-2: When "I understand" is tapped, then the analysis starts and later analyses skip the notice. AC-3: When "Cancel" is tapped, then 0 requests are sent and `AI_DISCLOSURE_REQUIRED` remains in effect. |
| REQ-080 | The system MUST show a Cancel control from 3,000 ms after a photo analysis, description analysis, barcode lookup or USDA search starts, and abort the request when it is tapped. | M | PLAN Section 8.2 (T-2 check); derived from NFR-012: a wait of up to 62,000 ms must be escapable | AC-1: Given a request still running at 3,000 ms, then "Cancel" is visible. AC-2: When "Cancel" is tapped, then the request is aborted and the previous screen returns with its input kept. |

### 5.3 Search, barcode and manual entry

| ID | Requirement | Pri | Source | Acceptance criteria |
|---|---|---|---|---|
| REQ-031 | The system MUST search the bundled CoFID foods by text and list matches with a "CoFID" badge. | M | S-14, S-16 | AC-1: Given "basmati", then CoFID foods with "basmati" in the name are listed with the "CoFID" badge, ranked by the scoring in PLAN Section 10.4. AC-2: Given "zucchini", then courgette foods are listed. AC-3: Given "ba" then "banana" typed, then the list shows the results for "banana" even when the "ba" results finish later. AC-4: Given a 1-character query, then `VALIDATION_FAILED` "Type at least 2 characters" is shown. AC-5: Given the CoFID file fails to load, then `COFID_UNAVAILABLE` is shown with Retry. |
| REQ-032 | The system MUST append USDA FoodData Central results, with a "USDA" badge, below the CoFID results when the user taps "Search USDA". | M | S-14, S-16 | AC-1: Given CoFID results for "basmati", when "Search USDA" is tapped, then USDA results follow the CoFID results. AC-2: Given no USDA key, then `NO_API_KEY` is shown with a link to Settings. AC-3: Given a 429 response, then `USDA_RATE_LIMITED` is shown and the CoFID results stay. AC-4: Given 5xx responses or timeouts on 2 attempts, then `USDA_UNAVAILABLE` is shown with Retry and the CoFID results stay. AC-5: Given a rejected key, then `USDA_AUTH_FAILED` is shown. |
| REQ-074 | The system MUST read EAN-13, EAN-8, UPC-A and UPC-E barcodes from the rear camera. | M | S-14 | AC-1: Given a valid EAN-13 in front of the camera, then the code is read and the lookup of REQ-034 starts. AC-2: Given camera permission is denied, then `CAMERA_PERMISSION_DENIED` is shown with the path "Settings > Safari > Camera" and the typed field of REQ-035. AC-3: Given no usable camera, then `CAMERA_UNAVAILABLE` is shown with the typed field. |
| REQ-035 | The system MUST accept a typed barcode as an alternative to the camera. | M | S-14 | AC-1: Given "5000157024671" typed and "Look up" tapped, then the same review as REQ-034 AC-1 opens. AC-2: Given a value that fails REQ-075, then `BARCODE_INVALID` is shown. |
| REQ-075 | The system MUST reject a barcode, scanned or typed, whose length is not 8, 12 or 13 digits or whose GTIN check digit is wrong, before any lookup. | M | S-14; derived from REQ-034: a misread code must not fetch the wrong product | AC-1: Given "5000157024672" (wrong check digit) or a 10-digit value, then `BARCODE_INVALID` is shown and 0 requests are sent. AC-2: Given valid EAN-13, EAN-8, UPC-A and UPC-E examples, then each is accepted. |
| REQ-034 | The system MUST look up a valid barcode in Open Food Facts and open the review screen with the product name, brand and per-100 values with an "Open Food Facts" badge. | M | S-14 | AC-1: Given `5000157024671`, then the review shows "Beanz in a rich tomato sauce", brand "H J Heinz Foods UK Ltd" and 81 kcal per 100 g. AC-2: Given a product not in Open Food Facts, then `OFF_PRODUCT_NOT_FOUND` is shown and manual entry opens with the barcode. AC-3: Given kcal and kJ both missing, or protein, carbs or fat missing, then `OFF_INCOMPLETE_NUTRITION` is shown and manual entry opens with the known values filled in. AC-4: Given a 429 response, then `OFF_RATE_LIMITED` is shown. AC-5: Given 5xx responses or timeouts on 2 attempts, then `OFF_UNAVAILABLE` is shown with Retry and "Enter manually". |
| REQ-008 | The system MUST create an entry from a name, a basis (per 100 g, per 100 ml, or per serving with a serving size), nutrient values and a quantity. | M | S-14 | AC-1: Given per-serving values for a 30 g serving at 120 kcal, when saved with quantity 45 g, then the entry stores 400 kcal per 100 g and shows 180 kcal. AC-2: Given the per-serving basis with no serving size, then `VALIDATION_FAILED` is shown. AC-3: Given kcal 901 per 100 g, or protein + carbs + fat + fibre above 105 g per 100 g, then `VALIDATION_FAILED` names the rule. AC-4: Given fibre left empty, then fibre is stored as `null`, not 0. |
| REQ-024 | The system MUST log foods whose source basis is per 100 ml in ml and all other foods in g. | M | S-04; derived from REQ-039: per-100 ml values multiplied by grams give wrong totals | AC-1: Given an Open Food Facts product with `product_quantity_unit = "ml"`, then the quantity field shows "ml". AC-2: Given the CoFID food "Wine, red" (group `QE`), then the unit is "ml". AC-3: Given an Open Food Facts product with no unit information, then the unit is "g". |

### 5.4 Review screen

| ID | Requirement | Pri | Source | Acceptance criteria |
|---|---|---|---|---|
| REQ-078 | The system MUST write nothing to the log from a review screen until the user taps "Log items". | M | S-05 ("show me back the information, that I can edit") | AC-1: Given a review with 2 items, when the user leaves the screen without tapping "Log items", then the log is unchanged. AC-2: When "Log items" is tapped, then all items are written in one transaction; given the write fails, then `STORAGE_WRITE_FAILED` is shown, 0 items are stored and the review is kept. |
| REQ-036 | The review and edit screens MUST make the quantity field, with its unit, the first editable field of every item. | M | S-11 | AC-1: Given a review item, when focus moves with the keyboard from the item heading, then the first input reached is the quantity field labelled "g" or "ml". (Layout only; input failures are REQ-037.) |
| REQ-037 | The quantity field MUST accept values from 0.1 to 5,000 with at most 1 decimal place. | M | S-11 | AC-1: Given 0.1 or 5,000, then the value is accepted. AC-2: Given empty, 0, 0.05, 212.25, 5,000.1 or text, then `VALIDATION_FAILED` "Enter 0.1 to 5,000" is shown and "Log items" (or Save) is disabled. |
| REQ-038 | The system MUST label an AI-estimated quantity "Estimated" and change the label to "Weighed" when the user edits it. | M | S-11 | AC-1: Given an AI-estimated 180 g, when the user types 212, then the label changes from "Estimated" to "Weighed" and the item's nutrients recalculate before saving. AC-2: Given an invalid value, then REQ-037 AC-2 applies and the label stays "Estimated". |
| REQ-039 | The system MUST save per-100 values equal to those of the selected database food. | M | S-04, S-08 | AC-1: Given an item with a selected CoFID match, when logged, then each saved per-100 field equals the CoFID value exactly. AC-2: Given a selected USDA or Open Food Facts match, then the saved per-100 values equal the values mapped by PLAN Sections 7.3 and 7.4 with the amendments in Section 12.1 (TC-020, TC-022). |
| REQ-079 | The system MUST keep "Log items" disabled while any review item has no selected food. | M | S-04; derived from REQ-039: an item with no food has no source values | AC-1: Given 1 of 2 items showing "Check match", then "Log items" is disabled. AC-2: When that item gets a selection (REQ-064) or "Use AI estimate" (REQ-040), then "Log items" is enabled. |
| REQ-040 | The system MUST use Gemini's own nutrient estimate for an item, flagged `AI_ESTIMATE`, only after the user taps "Use AI estimate" on that item. | M | S-04, S-08 | AC-1: When "Use AI estimate" is tapped on an item with no selection, then the item uses Gemini's per-100 values, source `ai_estimate`, flag `AI_ESTIMATE` and the "AI estimate" badge. AC-2: Given no "Use AI estimate" tap, then no Gemini nutrient value is saved. AC-3: Given Gemini's per-100 values break DR-02, then `VALIDATION_FAILED` is shown and the item stays unselected. |
| REQ-064 | The review screen MUST let the user replace an item's selected food with one of its candidates or with a result of a new search. | M | S-05; derived from REQ-077: a wrong match must be correctable | AC-1: Given an item with 3 candidates, when the second is picked, then the item shows that food's name, badge and recalculated nutrients. AC-2: When "Search" inside "Change match" is used and a result is picked, then that result becomes the selected food. AC-3: Given the search fails with `COFID_UNAVAILABLE`, `USDA_UNAVAILABLE`, `USDA_RATE_LIMITED` or `NETWORK_OFFLINE`, then that code is shown and the previous selection stays. |
| REQ-065 | The review screen MUST let the user remove an item before logging. | M | S-05; derived from REQ-076: Gemini can list a food that was not eaten | AC-1: Given 3 review items, when one is removed, then 2 items remain and the footer totals fall by its values. AC-2: Given the last item is removed, then "Log items" is disabled. |

### 5.5 Favourites

| ID | Requirement | Pri | Source | Acceptance criteria |
|---|---|---|---|---|
| REQ-041 | The system MUST save an entry or a review item as a single-item favourite when its star is tapped. | M | S-06, S-17 | AC-1: Given an entry, when the star is tapped, then a favourite of kind `item` with the same name, quantity and per-100 values exists. AC-2: Given a favourite named "Banana" exists, when another "Banana" is starred, then the new favourite is named "Banana (2)" (DR-08). AC-3: Given an item name of 120 characters, then the favourite name is its first 80 characters (DR-08). AC-4: Given the write fails, then `STORAGE_WRITE_FAILED` is shown and the star stays off. |
| REQ-042 | The system MUST save 2 to 30 selected entries as one named meal favourite. | M | S-06, S-17 | AC-1: Given 3 entries selected and "Save as meal" named "My porridge", then a favourite of kind `meal` with 3 items exists. AC-2: Given 1 entry selected, then "Save as meal" is disabled. AC-3: Given 31 entries selected, a blank name, an 81-character name or a name already used (case-insensitive), then `VALIDATION_FAILED` names the rule. |
| REQ-043 | The Favourites tab MUST list favourites by most recent use, then never-used favourites by name. | M | S-06, S-17 | AC-1: Given favourites used at 09:00 and 12:00 and one never used, then the order is the 12:00 one, the 09:00 one, then the never-used one. AC-2: Given 0 favourites, then "No favourites yet" is shown. AC-3: Given the read fails, then `STORAGE_UNAVAILABLE` is shown. |
| REQ-044 | The Favourites tab MUST filter favourites by a case-insensitive substring of the name. | M | S-17 | AC-1: Given "My porridge" and "Chicken salad", when "POR" is typed, then only "My porridge" is listed. AC-2: Given no match, then "No favourites match" is shown. |
| REQ-045 | The system MUST add a favourite to the selected date at the current time with each item quantity multiplied by a portion multiplier from 0.25 to 5.00 in steps of 0.25. | M | S-06, S-17 | AC-1: Given "My porridge" (oats 50 g, milk 200 ml, banana 100 g), when added with multiplier 1.5, then 3 entries with 75 g, 300 ml and 150 g are logged at the current time, sharing one `groupId`, and the favourite's `lastUsedAt` becomes the current time. AC-2: The stepper stops at 0.25 and 5.00. AC-3: Given 33.3 g at 0.25, then the entry is 8.3 g (DR-07). AC-4: Given a scaled quantity below 0.1 or above 5,000, then `VALIDATION_FAILED` names that item and nothing is logged. |
| REQ-046 | The add sheet MUST let the user edit each item's quantity before the favourite is added. | M | S-17 | AC-1: Given "My porridge" at multiplier 1.0, when milk is changed to 150 ml and added, then the milk entry is 150 ml and the favourite still holds 200 ml. AC-2: Given an edited value outside REQ-037, then `VALIDATION_FAILED` is shown and "Add" is disabled. |
| REQ-069 | The system MUST let the user rename a favourite. | M | S-17; derived from REQ-042: a saved name must be correctable | AC-1: Given "My porridge", when renamed "Oats breakfast", then the list shows "Oats breakfast". AC-2: Given a blank name, an 81-character name or a name already used, then `VALIDATION_FAILED` is shown and the favourite is unchanged. |
| REQ-070 | The system MUST let the user change the quantity of an item stored in a favourite. | M | S-17; derived from REQ-041: a saved portion must be correctable | AC-1: Given "My porridge" with oats 50 g, when oats is changed to 60 g and saved, then the favourite holds 60 g. AC-2: Given a value outside REQ-037, then `VALIDATION_FAILED` is shown and the favourite is unchanged. |
| REQ-071 | The system MUST let the user remove an item from a meal favourite, changing its kind to `item` when 1 item remains. | M | S-17; derived from REQ-042: a meal can hold an item that is no longer eaten | AC-1: Given a meal of 3 items, when 1 is removed, then the meal has 2 items. AC-2: Given a meal of 2 items, when 1 is removed, then the favourite has kind `item`. AC-3: Given a favourite of kind `item`, then its only item has no remove control (deletion is REQ-048). |
| REQ-048 | The system MUST delete a favourite without changing any entry already logged from it. | M | S-17; derived from REQ-068: logged history must not change | AC-1: Given an entry logged from favourite F, when F is deleted, then the entry and its values are unchanged. AC-2: Given the write fails, then `STORAGE_WRITE_FAILED` is shown and F stays. |

### 5.6 Targets and settings

| ID | Requirement | Pri | Source | Acceptance criteria |
|---|---|---|---|---|
| REQ-049 | The system MUST compute daily targets from sex, age, height, weight, activity level and goal with the formula of PLAN Section 10.3 and ASM-005. | M | S-12 | AC-1: Given male, 25 years, 178 cm, 75 kg, `moderate`, `lose_0_25`, then the targets are 2,430 kcal, 135 g protein, 68 g fat, 321 g carbs, 30 g fibre and no sugar target. AC-2: Given female, 60 years, 150 cm, 45 kg, `sedentary`, `lose_0_5`, then kcal is 930 (floored at BMR). AC-3: Given age 15, then `VALIDATION_FAILED` "Age must be 16 to 100" is shown. |
| REQ-072 | The system MUST let the user override each daily target. | M | S-12 | AC-1: Given a kcal override of 2,200, then Today uses 2,200 and Settings shows the computed value beside it. AC-2: Given a kcal override of 1,100, then it is saved and the warning "Below 1,200 kcal" is shown. AC-3: Given 799 kcal, then `VALIDATION_FAILED` "800 to 6,000" is shown and the target is unchanged. |
| REQ-073 | The system MUST reset an overridden target to its computed value when "Reset" is tapped. | M | S-12 | AC-1: Given a kcal override of 2,200 and a computed 2,430, when "Reset" is tapped, then Today uses 2,430 and `overridden.kcal` is `false`. AC-2: Given the write fails, then `STORAGE_WRITE_FAILED` is shown and the override stays. |
| REQ-051 | Settings MUST store a Gemini API key, a USDA API key and a Gemini model ID on the device. | M | S-07, S-10 | AC-1: Given saved keys and model, when the app restarts, then the values are present and each key is shown masked except its last 4 characters. AC-2: Given a key of 19 characters, then `VALIDATION_FAILED` is shown. AC-3: Given the model "gpt-4", then `VALIDATION_FAILED` is shown. |
| REQ-082 | Settings MUST link to the page where each API key is obtained. | M | PLAN Section 11; derived from REQ-051: a key cannot be stored before it is obtained | AC-1: When Settings opens, then a link to Google AI Studio's API key page and a link to the FoodData Central API key sign-up page are shown. (Links only; no failure path.) |
| REQ-052 | Settings MUST test the Gemini key and model with a "Test" button. | M | S-07, S-10 | AC-1: Given a valid key, then "Key works (model `<MODEL_ID>`)" is shown. AC-2: Given an invalid key, then `GEMINI_AUTH_FAILED` is shown. AC-3: Given an unknown model, then `GEMINI_MODEL_NOT_FOUND` is shown. AC-4: Given the network is off, then `NETWORK_OFFLINE` is shown. |
| REQ-053 | Settings MUST test the USDA key with a "Test" button. | M | S-10 | AC-1: Given a valid key, then "Key works" is shown. AC-2: Given an invalid key, then `USDA_AUTH_FAILED` is shown. AC-3: Given the network is off, then `NETWORK_OFFLINE` is shown. |
| REQ-022 | The About screen MUST show the data attributions for CoFID (Open Government Licence v3.0), Open Food Facts (ODbL 1.0) and USDA FoodData Central (CC0 1.0), each with a link. | M | S-08; derived from REQ-031, REQ-032 and REQ-034: the licences of these databases require attribution | AC-1: When About opens, then all 3 attribution texts and links are visible. (Display only; no failure path.) |
| REQ-061 | The About screen MUST state the carbohydrate convention of each source. | M | S-04; derived from NFR-003: carbs from different sources follow different conventions (P0-F4) | AC-1: When About opens, then it states: CoFID carbohydrate and sugars are monosaccharide equivalents and can read up to 10% above a UK pack label; USDA carbohydrate has fibre subtracted, except items flagged `CARBS_INCLUDE_FIBRE`; Open Food Facts values are as declared on the pack. |
| REQ-081 | The About screen MUST show the app version and build date. | M | PLAN Section 11; derived from REQ-063: Zac must be able to see which version runs during UAT | AC-1: When About opens, then it shows the version (for example "0.9.0") and the build date in `YYYY-MM-DD` form. |
| REQ-062 | Settings MUST show a Diagnostics list of the last 200 log events, newest first, with no API key or image data. | M | S-20; derived from REQ-076: AI and API failures reported in UAT need evidence from the phone | AC-1: Given 201 events, then 200 are listed and the oldest is dropped. AC-2: Given an event raised during a call that used a key, then the key's characters do not appear in the list. |
| REQ-083 | The app MUST use the light or dark theme that matches the iOS appearance setting. | M | PLAN Section 11 | AC-1: Given `prefers-color-scheme: dark`, then the dark theme is shown; given `light`, then the light theme. (Display only; no failure path.) |

### 5.7 Backup, storage and app shell

| ID | Requirement | Pri | Source | Acceptance criteria |
|---|---|---|---|---|
| REQ-054 | The system MUST export all entries, favourites, profile, targets and settings except API keys to one JSON file named `macrolens-backup-YYYY-MM-DD.json`. | M | S-09; derived from REQ-002: phone-only storage needs a copy outside the app | AC-1: Given 10 entries and 2 favourites, when "Export" is tapped, then the file holds 10 entries, 2 favourites and no `geminiApiKey` or `usdaApiKey`, and `lastBackupAt` becomes the current time. AC-2: Given IndexedDB cannot open, then `STORAGE_UNAVAILABLE` is shown. |
| REQ-055 | The system MUST import a valid backup in "replace" mode in one transaction, keeping the API keys already stored. | M | S-09; derived from REQ-054: an export is useful only if it can be restored | AC-1: Given an exported file, when imported in replace mode on an empty install, then every entry, favourite and setting except API keys is restored. AC-2: Given a file that fails validation, then `IMPORT_INVALID_FILE` names the first failing path and the database is unchanged. AC-3: Given the quota is exceeded during the write, then `STORAGE_QUOTA_EXCEEDED` is shown and the database is unchanged. |
| REQ-056 | The system MUST import a valid backup in "merge" mode by the rules of DR-09. | M | S-09; derived from REQ-054: a restore must not discard newer records on the phone | AC-1: Given entry E with `updatedAt` 100 on the device and 200 in the file, then E from the file is kept; given 300 on the device, then the device record is kept; given 200 on both, then the device record is kept. AC-2: Given an invalid file, then `IMPORT_INVALID_FILE` is shown and the database is unchanged. |
| REQ-020 | The app MUST be installable to the iPhone home screen and open in standalone mode with the name "Macrolens" and its icon. | M | S-02 | AC-1: Given the deployed URL in iOS Safari, when "Add to Home Screen" is used, then the app opens without Safari's address bar and shows the icon and the name "Macrolens". (Platform behaviour; the failure path is REQ-066.) |
| REQ-057 | Logging, editing, favourites, CoFID search and targets MUST work offline after the first complete load. | M | S-02; derived from REQ-020: an installed phone app is used without signal | "First complete load" means the first online launch after which the service worker has stored every precached file: the app shell, the CoFID data file and the barcode WASM file. AC-1: Given a first complete load and the network then disabled, when the user searches "apple" and logs a result, then the entry is saved. AC-2: Offline, editing an entry's quantity saves. AC-3: Offline, adding a favourite logs its entries. AC-4: Offline, changing the profile recomputes the targets. AC-5: Given the network is lost before the first complete load ends, then a CoFID search shows `COFID_UNAVAILABLE`. |
| REQ-058 | Photo analysis, description analysis, barcode lookup, USDA search and key tests MUST show `NETWORK_OFFLINE` without sending a request when the device is offline. | M | S-02; derived from REQ-020: these features need a remote service | AC-1: Given the network disabled, when a photo analysis starts, then `NETWORK_OFFLINE` is shown and 0 requests are sent. |
| REQ-059 | The system MUST request persistent storage on first launch. | S | S-09; derived from REQ-002: iOS can evict non-persistent site data, which here is the only copy | AC-1: Given a first launch, then `navigator.storage.persist()` is called once. AC-2: Given `navigator.storage.persist` is missing, then no error is shown and REQ-066 shows its banner. |
| REQ-066 | The system MUST show "Install to Home Screen and export a backup" when storage is not known to be persisted. | S | S-09; derived from REQ-059: a refused or unknown persistence request leaves data at risk (FM-016) | AC-1: Given `navigator.storage.persisted()` returns `false`, then the banner is shown on Today. AC-2: Given the Storage API is missing, then the banner is shown. AC-3: Given `true`, then no banner is shown. |
| REQ-060 | The system MUST show a backup reminder when 1 or more entries exist and the last export is more than 7 days old or has never happened. | S | S-09; derived from REQ-054: an export only protects data when it is recent | AC-1: Given `lastBackupAt` 8 days ago, then Today shows "Last backup 8 days ago — Export now". AC-2: Given 7 days or less, then no reminder is shown. AC-3: Given no export and 1 entry, then "No backup yet — Export now" is shown. |
| REQ-063 | The installed app MUST offer "Update available — Reload" when a new version has been deployed, and reload into it when tapped. | M | S-20; derived from REQ-020: UAT fixes must reach the installed app | AC-1: Given a new service worker is waiting, when the app opens, then the prompt is shown. AC-2: When "Reload" is tapped, then the new version runs and REQ-081 shows its version. |
| REQ-068 | A logged entry MUST keep its stored per-100 values when the source dataset, a favourite or a match is later changed. | M | S-04; derived from REQ-039: history must not drift when sources change | AC-1: Given an entry logged from CoFID, when the bundled CoFID data changes in a later release, then the entry's per-100 values are unchanged. |

### 5.8 Retired requirement IDs

These IDs are compound (PRD-D2). They are retired and never reused.

| Retired ID | Origin | Replaced by |
|---|---|---|
| REQ-003 | PLAN | REQ-027, REQ-028 |
| REQ-005 | PLAN | REQ-029, REQ-030 |
| REQ-006 | PLAN | REQ-031, REQ-032 |
| REQ-007 | PLAN | REQ-034, REQ-035, and REQ-033 (itself retired) |
| REQ-009 | PLAN | REQ-076, REQ-077, REQ-078 |
| REQ-011 | PLAN | REQ-036, REQ-037, REQ-038 |
| REQ-012 | PLAN | REQ-039, REQ-040, REQ-067, REQ-079 |
| REQ-014 | PLAN | REQ-041, REQ-042 |
| REQ-015 | PLAN | REQ-043, REQ-044, REQ-045, REQ-046 |
| REQ-016 | PLAN | REQ-048, and REQ-047 (itself retired) |
| REQ-017 | PLAN | REQ-049, and REQ-050 (itself retired) |
| REQ-018 | PLAN | REQ-051, REQ-052, REQ-053 |
| REQ-019 | PLAN | REQ-054, REQ-055, REQ-056 |
| REQ-021 | PLAN | REQ-057, REQ-058 |
| REQ-026 | PLAN | REQ-059, REQ-060, REQ-066 |
| REQ-033 | Draft 0.1.0 | REQ-074, REQ-075 |
| REQ-047 | Draft 0.1.0 | REQ-069, REQ-070, REQ-071 |
| REQ-050 | Draft 0.1.0 | REQ-072, REQ-073 |

`CLAUDE.md` cites REQ-012 for the "Use AI estimate" rule; that rule is now REQ-040.

## 6. Non-functional requirements

| ID | Requirement | Target and unit | Window or percentile | Load condition | Measurement point | Source |
|---|---|---|---|---|---|---|
| NFR-001 | Nutrient arithmetic: entry value = per-100 value × quantity ÷ 100 with no intermediate rounding; day totals sum unrounded values and round once for display | Display rounding: kcal to integer, grams to 1 decimal place, half away from zero; error 0 | All of 10,000 generated cases | 1 to 50 entries per generated day, quantities 0.1 to 5,000 | Unit and property tests of the nutrition module (TC-001, TC-002) | S-04 |
| NFR-002 | CoFID parity: every included row's 6 tracked fields in the bundled CoFID data file equal the source XLSX after the rules of Section 7.3 | Absolute difference ≤ 0.001; included rows = 2,853; excluded rows = 34 | 100% of included rows | Full dataset | `scripts/verify-cofid.ts` in CI (TC-010, TC-011) | S-04, S-16 |
| NFR-003 | Carbohydrate convention: stored carbs exclude fibre. USDA "carbohydrate by difference" (1005) has fibre (1079) subtracted; when 1079 is missing the item keeps 1005 and is flagged `CARBS_INCLUDE_FIBRE`. CoFID values are stored unchanged: they exclude fibre and are monosaccharide equivalents (REQ-061) | 100% of mapped USDA items follow the rule | All mapper test fixtures | n/a (pure function) | USDA mapper tests (TC-020) | S-04 |
| NFR-004 | CoFID search latency | ≤ 100 ms | p95 over 200 queries of 2 to 30 characters | 2,853 foods loaded; 1 query at a time | Playwright WebKit on the development Mac (TC-013) | S-02 |
| NFR-005 | Photo upload size after downscaling to a long edge of 1,024 px and JPEG quality 0.8 | ≤ 400 KB | Per photo | 1 photo of 4,032 × 3,024 px | Unit test with the TC-090 image (TC-030) | S-07 |
| NFR-006 | Download size | Initial JavaScript ≤ 300 KB gzip; CoFID data file ≤ 600 KB gzip, precached by the service worker after first load and read into memory on first search | Per production build | Production build | `scripts/check-size.ts` in CI | S-02 |
| NFR-007 | Accessibility | Every tap target ≥ 44 × 44 CSS px; 0 axe-core violations of impact `serious` or `critical` | Every screen | 390 × 844 CSS px viewport, light and dark themes | Playwright with `@axe-core/playwright` (TC-088) | S-02 |
| NFR-008 | Secrets: API keys exist only in IndexedDB on the device | 0 key occurrences in logs, exports, the repository and test snapshots | Every CI run | n/a | Export test (TC-062), Diagnostics test (TC-064), secret scan in CI | S-10 |
| NFR-009 | Content Security Policy `connect-src` | Exactly `'self'`, `https://generativelanguage.googleapis.com`, `https://api.nal.usda.gov`, `https://world.openfoodfacts.org` | Every build | n/a | E2E test reads the CSP meta tag (TC-089) | S-10 |
| NFR-010 | Supported clients | iOS Safari 18 or later in home-screen mode (primary); current Chrome for Android (secondary) | Every E2E run | 1 user | Playwright projects: WebKit with the newest iPhone descriptor, Chromium with the newest Pixel descriptor (TC-080 to TC-089) | S-02, S-21 |
| NFR-011 | Test coverage | ≥ 90% lines in `src/domain/**` and `src/services/**`; ≥ 80% lines overall | Every CI run | n/a | Vitest coverage thresholds | S-20 |
| NFR-012 | Photo or description analysis wait, from "Analyse" to the review screen or an error | ≤ 62,000 ms; at the cap the analysis ends with `GEMINI_UNAVAILABLE` when no response arrived, or `GEMINI_BAD_RESPONSE` when the last response failed validation. Budget: 2 attempts × 30,000 ms + 1,200 ms backoff = 61,200 ms; a repair request runs only inside the remaining budget and is skipped when less than 5,000 ms remains | Per analysis (maximum, not a percentile) | 1 analysis at a time | Unit test with fake timers (TC-032) | S-05 |
| NFR-013 | Gemini requests per analysis | ≤ 3 (2 attempts plus 1 repair request) | Per analysis | 1 analysis at a time | Gemini wrapper test counts requests (TC-031) | S-07; derived from ASM-003: the free-tier daily quota is shared by all analyses |

## 7. Data requirements

### 7.1 Entities

Field-level schemas (type, nullability, constraints, default) are those of PLAN Section 6, with the rules of Section 7.2.

| Entity | Mandatory attributes | Retention | Classification |
|---|---|---|---|
| Log entry | `id`, `date`, `consumedAt`, `name`, `quantity`, `unit`, `quantityOrigin`, `per100` (kcal, protein, carbs, fat required; fibre, sugar nullable), `source`, `flags`, `createdAt`, `updatedAt` | Unbounded: no automatic deletion; removed only when Zac deletes it | Personal (diet and health) |
| Favourite | `id`, `kind`, `name`, `items` (1 to 30), `useCount`, `createdAt`, `updatedAt` | Unbounded: removed only when Zac deletes it | Personal |
| Profile | `sex`, `ageYears`, `heightCm`, `weightKg`, `activity`, `goal` | Unbounded: replaced when Zac edits it | Personal (health) |
| Targets | `kcal`, `protein`, `carbs`, `fat`, `fibre`, `overridden` | Unbounded: replaced when recomputed or overridden | Personal |
| Settings | `geminiModel`, `schemaVersion` (keys, disclosure time and backup time nullable) | Unbounded: replaced when Zac edits it | Secret (`geminiApiKey`, `usdaApiKey`); Personal (other fields) |
| Photo | Image bytes in memory during one analysis | 0 days: never written to storage (ASM-010) | Personal |
| Diagnostics event | `name`, `at`, fields without secrets | In memory only; last 200 events; cleared on app restart | Internal |
| CoFID dataset | Per Section 7.3 | Replaced with each release | Public (Open Government Licence v3.0) |
| Backup file | Per PLAN Section 6.9 | Under Zac's control in Files | Personal |

"Unbounded" is chosen because the log is Zac's own history; the retention period in days is not limited.

### 7.2 Data rules

| ID | Rule | Source |
|---|---|---|
| DR-01 | Fibre and sugar may be `null` (unknown). The app never replaces `null` with 0. | S-04 |
| DR-02 | The cross-field rule `protein + carbs + fat + (fibre ?? 0) ≤ 105` per 100 g applies to values entered by the user (manual entry, edits of per-100 values) and to Gemini's per-100 estimate. It does not apply to rows copied from CoFID, USDA or Open Food Facts. | P0-F6: CoFID "Rosemary, dried" sums to 109.1 |
| DR-03 | A logged entry stores a copy of its per-100 values (REQ-068). | S-04 |
| DR-04 | `quantity` has at most 1 decimal place; per-100 values keep the precision of their source. | S-11, S-04 |
| DR-05 | `date` is the local calendar date of `consumedAt` in the device time zone (ASM-006). | S-03 |
| DR-06 | Favourite names are 1 to 80 characters and unique case-insensitively. | S-17 |
| DR-07 | A quantity computed by a multiplier is rounded half away from zero to 1 decimal place before validation; a result below 0.1 or above 5,000 is rejected. | DR-04, REQ-037 |
| DR-08 | A favourite created from a star is named after its item: trimmed, cut to 80 characters, and when the name is used, suffixed " (N)" with the lowest free N from 2, cutting the base so the total stays within 80 characters. | DR-06 |
| DR-09 | Merge import: entries and favourites are upserted by `id`; for an `id` on both sides the record with the larger `updatedAt` is kept, and the device record is kept when they are equal. Profile and targets from the file are used only when the device has none. Settings from the file are used only for fields that are `null` on the device. API keys are never read from a file. | REQ-056 |

### 7.3 CoFID conversion rules (input to NFR-002)

| ID | Rule | Evidence |
|---|---|---|
| CR-01 | Source: sheet `1.3 Proximates` of the CoFID 2021 XLSX with SHA-256 `436e9445ef2adb2a75f3d7edd51302de3adad25385f9795fc94ba58bd030e97d`; header rows 1 to 3; data from row 4. | `docs/spikes.md` T0.4 |
| CR-02 | Fields: kcal = `KCALS`, protein = `PROT`, fat = `FAT`, carbs = `CHO`, sugar = `TOTSUG`, fibre = `AOACFIB`. | ASM-009 |
| CR-03 | Cell values: a number or a numeric string is the value; `Tr` → 0; `N`, blank and `NA` → `null`. | ASM-009; user guide page 9 |
| CR-04 | A row is excluded when kcal, protein, carbs or fat is `null` after CR-03. Expected: 34 rows excluded, 2,853 included. | P0-F2 |
| CR-05 | `basis = ml` for rows whose food group starts with `Q` (alcoholic beverages; expected 31 rows); `basis = g` otherwise. | P0-F5; user guide page 9 |
| CR-06 | `sourceId` = food code. When a code repeats, the first row in sheet order keeps the code and each later row gets `<CODE>#<N>`, where `<N>` counts from 2 (expected: `13-669` "Aubergine, flesh and skin, roasted in rapeseed oil" and `13-669#2` "Watercress, raw"). | P0-F1 |
| CR-07 | Values are copied without unit or basis conversion; CoFID carbohydrate and sugars stay monosaccharide equivalents (REQ-061). | P0-F4 |

## 8. External interfaces and dependencies

| System | Direction of data flow | Owner | Availability commitment | Used by |
|---|---|---|---|---|
| Gemini API (`generativelanguage.googleapis.com`) | App → Gemini: photo or text; Gemini → app: JSON analysis | Google | None on the free tier; 503 "high demand" observed on 2026-11-17 (P0-F12) | REQ-076, REQ-010, REQ-052 |
| USDA FoodData Central API (`api.nal.usda.gov`) | App → USDA: query and key; USDA → app: foods | U.S. Department of Agriculture | None published; 3,600 requests per hour per key observed (P0-F13) | REQ-032, REQ-053, REQ-077 |
| Open Food Facts API (`world.openfoodfacts.org`) | App → OFF: barcode; OFF → app: product | Open Food Facts (non-profit association) | None published; 15 product reads per minute per IP | REQ-034 |
| GitHub Pages | GitHub → device: static app files | GitHub | None for free personal accounts | REQ-020, REQ-063 |
| CoFID 2021 XLSX (GOV.UK) | GOV.UK → build script, at build time only | UK Government (Office for Health Improvement and Disparities) | Static file, last updated 19 March 2021 | NFR-002, REQ-031 |

Data licences: CoFID Open Government Licence v3.0; Open Food Facts ODbL 1.0 (contents DbCL 1.0); USDA FoodData Central CC0 1.0 (REQ-022).

## 9. Error and edge-case behaviour

### 9.1 Error catalogue

This table is the single documentary definition of error codes (Rule D-3); `src/domain/errors.ts` implements exactly these codes. It carries PLAN Section 8.1 forward and adds `OFF_RATE_LIMITED` and `GEMINI_REQUEST_REJECTED`.

| Code | Upstream status | Retryable | Trigger | Message shown | Caller action |
|---|---|---|---|---|---|
| `VALIDATION_FAILED` | local | No | Input violates a rule in Section 7 or a schema | Field-level message naming the rule | Correct the field |
| `NO_API_KEY` | local | No | Required key is `null` | "Add your Gemini key in Settings" or "Add your USDA key in Settings" | Open Settings |
| `AI_DISCLOSURE_REQUIRED` | local | No | `aiDisclosureAcceptedAt` is `null` | Free-tier data notice (REQ-023) | Tap "I understand" |
| `NETWORK_OFFLINE` | local | Yes | `navigator.onLine === false`, or `fetch` rejects with `TypeError` | "You're offline" | Retry when online |
| `GEMINI_AUTH_FAILED` | 400 with `API_KEY_INVALID`, 401, 403 | No | Key rejected | "Gemini rejected the key" | Fix the key in Settings |
| `GEMINI_MODEL_NOT_FOUND` | 404 | No | Model ID unknown | "Model `<MODEL_ID>` not found" | Change the model in Settings |
| `GEMINI_REQUEST_REJECTED` | 400 without `API_KEY_INVALID` | No | Gemini refused the request, for example the response schema | "Gemini refused the request" | Describe instead, or use Search or Manual; event kept in Diagnostics |
| `GEMINI_RATE_LIMITED` | 429 | Yes | Free-tier quota reached | "Gemini limit reached. Try again in `<SECONDS>` s", or "after midnight Pacific time" for the daily quota | Wait, or use Search or Manual |
| `GEMINI_UNAVAILABLE` | 500 to 504, timeout | Yes | No valid response after 2 attempts or at the NFR-012 cap | "Gemini didn't respond" | Tap Retry |
| `GEMINI_BAD_RESPONSE` | 200 | Yes | Output fails `AiAnalysis` validation after 1 repair request, or the repair request is skipped under NFR-012 | "Couldn't read Gemini's answer" | Tap Retry or "Describe instead" |
| `NO_FOOD_DETECTED` | 200 | No | `items` is empty | "No food found in the photo" | Retake, or describe in words |
| `IMAGE_DECODE_FAILED` | local | No | The image cannot be decoded | "This image can't be read" | Choose another photo |
| `USDA_AUTH_FAILED` | 401, 403 | No | Key rejected | "USDA rejected the key" | Fix the key in Settings |
| `USDA_RATE_LIMITED` | 429 | Yes | Hourly limit reached | "USDA limit reached. Try again in 60 minutes" | Wait, or use CoFID |
| `USDA_UNAVAILABLE` | 500 to 504, timeout | Yes | Failed after 2 attempts | "USDA didn't respond" | Tap Retry |
| `BARCODE_INVALID` | local | No | Length is not 8, 12 or 13 digits, or the GTIN check digit fails | "That barcode isn't valid" | Rescan or type it |
| `OFF_PRODUCT_NOT_FOUND` | 200 with `status = 0`, or 404 | No | Product absent | "Product not in Open Food Facts" | Manual entry opens with the barcode |
| `OFF_INCOMPLETE_NUTRITION` | 200 | No | kcal and kJ missing, or protein, carbs or fat missing | "Some nutrition values are missing" | Complete manual entry |
| `OFF_RATE_LIMITED` | 429 | Yes | More than 15 product reads per minute from this IP | "Open Food Facts limit reached. Try again in 60 s" | Wait, or enter manually |
| `OFF_UNAVAILABLE` | 500 to 504, timeout | Yes | Failed after 2 attempts | "Open Food Facts didn't respond" | Tap Retry |
| `CAMERA_PERMISSION_DENIED` | local | No | `NotAllowedError` | "Camera access is off. Turn it on in Settings > Safari > Camera" | Enable the camera, or type the barcode |
| `CAMERA_UNAVAILABLE` | local | No | `NotFoundError`, `NotReadableError`, `OverconstrainedError` | "Camera can't start" | Type the barcode |
| `COFID_UNAVAILABLE` | local or 404 | Yes | CoFID data file fails to load or parse | "Food list didn't load" | Tap Retry when online |
| `STORAGE_UNAVAILABLE` | local | No | IndexedDB cannot open or read | "Storage is unavailable on this browser" | Leave private browsing |
| `STORAGE_QUOTA_EXCEEDED` | local | No | `QuotaExceededError` | "Phone storage is full" | Export, then free space |
| `STORAGE_WRITE_FAILED` | local | Yes | Any other transaction error | "Couldn't save" | Tap Retry |
| `IMPORT_INVALID_FILE` | local | No | Backup fails `format` or schema validation | "This isn't a valid Macrolens backup: `<FIRST_ERROR_PATH>`" | Choose another file |

### 9.2 Edge cases (playbook Rule F-2)

| Category | Condition | Observable behaviour |
|---|---|---|
| Size | Day with 0 entries | "No entries yet"; totals 0 (REQ-002) |
| Size | Day with 200 entries (ASM-013 upper bound) | All 200 listed in order; totals correct (TC-060) |
| Size | Meal favourite of 2, 30 and 31 items | 2 and 30 saved; 31 → `VALIDATION_FAILED` (REQ-042) |
| Size | Meal favourite reduced to 1 item | Kind becomes `item` (REQ-071) |
| Size | AI analysis with 0, 15 and 16 items | 0 → `NO_FOOD_DETECTED`; 15 listed; 16 → validation failure, repair request, then `GEMINI_BAD_RESPONSE` |
| Size | Search with 0 CoFID results | "No CoFID results" with "Search USDA" (REQ-032) |
| Size | Backup file of 0 bytes | `IMPORT_INVALID_FILE` |
| Value | Quantity empty, 0, negative, 0.05, 5,000.1, text | `VALIDATION_FAILED` (REQ-037) |
| Value | Quantity 0.1 and 5,000 | Accepted |
| Value | Scaled quantity 33.3 g × 0.25 and 0.1 g × 0.25 | 8.3 g; 0.025 g → `VALIDATION_FAILED` (DR-07) |
| Value | Per-100 kcal 900 and 901; protein −1 | 900 accepted; 901 and −1 → `VALIDATION_FAILED` |
| Value | Fibre or sugar unknown | Stored `null`; totals show "≥" (REQ-025, DR-01) |
| Value | Profile age 16 and 100; 15 and 101 | 16 and 100 accepted; 15 and 101 → `VALIDATION_FAILED` |
| Text | Name empty or whitespace only | `VALIDATION_FAILED` |
| Text | Entry name of 200 and 201 characters | 200 accepted; 201 → `VALIDATION_FAILED` |
| Text | Favourite name of 80 and 81 characters typed by the user | 80 accepted; 81 → `VALIDATION_FAILED` (DR-06) |
| Text | Starred item with a 120-character name | Favourite named with the first 80 characters (DR-08) |
| Text | Non-ASCII query or name, for example "Crème fraîche" | Stored as typed; search folds accents so "creme" matches |
| Text | Control characters in a name or description | Removed before validation |
| Text | Description of 3 and 500 characters; 2 and 501 | 3 and 500 accepted; 2 and 501 → `VALIDATION_FAILED` (REQ-010) |
| Time | Entry at 23:59 and at 00:01 local time | Dated on the local calendar day of each time (DR-05, TC-006) |
| Time | Daylight-saving change in Europe/London (last Sunday of March and October) | Entries keep the local date and time shown on the clock (TC-006) |
| Time | Device time zone changes after logging | Stored `date` is unchanged; `consumedAt` keeps its offset |
| Time | Clock skew between devices | Applies only to merge import: the larger `updatedAt` wins as recorded by each device (DR-09) |
| Time | Gemini retry delay in a 429 response | Shown in seconds (`GEMINI_RATE_LIMITED`) |
| Ordering | "Log items" tapped twice | Second tap ignored while saving; 1 set of entries written |
| Ordering | 2 entries with the same `consumedAt` | Ordered by `createdAt` (REQ-002) |
| Ordering | A response for an older search query arrives after a newer one | The older response is discarded (REQ-031 AC-3) |
| Ordering | Same record changed on device and in a backup file | Merge keeps the larger `updatedAt`; the device record on a tie (DR-09) |
| Resources | Gemini, USDA or OFF limit reached | `GEMINI_RATE_LIMITED`, `USDA_RATE_LIMITED`, `OFF_RATE_LIMITED` |
| Resources | Phone storage full | `STORAGE_QUOTA_EXCEEDED`; the form keeps its values |
| Resources | iOS evicts site data | REQ-066 banner beforehand; recovery by import (REQ-055) |
| Authorization | Key missing | `NO_API_KEY` |
| Authorization | Key invalid or revoked | `GEMINI_AUTH_FAILED` or `USDA_AUTH_FAILED` |
| Authorization | Camera permission denied | `CAMERA_PERMISSION_DENIED` |
| Authorization | Free-tier notice not accepted | `AI_DISCLOSURE_REQUIRED`; no request sent (REQ-023) |
| Partial failure | Logging 3 items and the write of item 3 fails | Transaction rolls back; 0 items stored; `STORAGE_WRITE_FAILED`; review kept (REQ-078) |
| Partial failure | Import fails during the write | Transaction rolls back; database unchanged |
| Partial failure | USDA fails during matching for 1 of 3 items | That item keeps its CoFID candidates; the other items are unaffected (REQ-077) |
| Partial failure | CoFID fails to load during matching | Every item shows "Check match"; `COFID_UNAVAILABLE` shown once (REQ-077) |

## 10. Assumptions and open questions

### 10.1 Assumption register

| ID | Statement | Reason | Impact if false | Validation method | Status |
|---|---|---|---|---|---|
| ASM-001 | The primary client is an iPhone 16 running iOS Safari 18 or later, installed to the home screen. | S-21 | Medium (barcode library and storage choices) | Zac confirms in UAT step U-01 | Open |
| ASM-002 | The default Gemini model is `gemini-3.6-flash` and the fallback is `gemini-3.5-flash-lite`; both are free-tier models that accept image input. | Both exist in the models list and are "Free of charge" (`docs/spikes.md` T0.3) | Low (setting change) | T0.3: models list PASS; image call PASS on the fallback; the default returned 503 on 4 of 4 calls. Retest before the Phase 2 model decision locks | Open |
| ASM-003 | The free-tier daily quota covers 30 Gemini requests per day (10 analyses at NFR-013's maximum of 3 requests each). | Personal use: about 6 meals per day plus retries | Low (wait for the Pacific-midnight reset) | Zac reads the quota in Google AI Studio during UAT step U-02 | Open |
| ASM-004 | Foods with a per-100 ml basis are logged in ml with no density conversion. | Labels state per 100 ml for drinks; CoFID uses per 100 ml only for alcoholic beverages | Low | Mapper tests (TC-011, TC-022) | Open |
| ASM-005 | Targets (PLAN Section 10.3): BMR by the Mifflin-St Jeor equation (10 × kg + 6.25 × cm − 5 × age, +5 for male, −161 for female); TDEE = BMR × activity factor 1.2, 1.375, 1.55, 1.725 or 1.9; kcal = max(BMR, TDEE + goal delta of −550, −275, 0 or +275), rounded to the nearest 10; protein 1.8 g per kg; fat 25% of kcal ÷ 9; carbs the remaining kcal ÷ 4, at least 0; protein, fat and carbs rounded to whole grams; fibre 30 g (UK SACN); no sugar target. | Common defaults; every target can be overridden (REQ-072) | Low | Zac reviews targets in UAT step U-03 | Open |
| ASM-006 | Day boundaries use the device's local time zone. | Personal use in Europe/London | Low | TC-006 | Open |
| ASM-007 | USDA energy is read from nutrient 1008, else 2048, else 2047; a food with none of them is excluded. | FDC Foundation foods can lack 1008 | Low | TC-020 with fixtures of each case; T0.3 saw 1008 on the first 3 of 83 "banana" results | Open |
| ASM-008 | USDA search uses data types Foundation, SR Legacy and Survey (FNDDS) only; UK branded products come from Open Food Facts. | USDA branded data covers US products | Low | Reviewed in Phase 1: no requirement needs USDA branded data | Validated |
| ASM-009 | CoFID value rules: `Tr` → 0; `N`, blank, `NA` → `null`; fibre uses the AOAC column. | CoFID 2021 user guide | Medium | T0.4 confirmed the markers and columns; AOAC fibre covers 1,548 of 2,887 rows (P0-F3) | Validated |
| ASM-010 | Photos are not stored after analysis. | Storage and privacy on the phone | Low | Zac confirms in UAT | Open |
| ASM-011 | The GitHub repository is public so that GitHub Pages serves it on a free account. | S-18 | Low | Repository created public on 2026-11-17; Phase 8 deploy | Open |
| ASM-012 | Open Food Facts lookups use `https://world.openfoodfacts.org/api/v2/product/<BARCODE>` with `fields`, `cc=gb` and `lc=en`. | OFF API documentation | Low | T0.2: 200 from Chromium and WebKit | Validated |
| ASM-013 | A day holds at most 200 entries. | Personal use; sizing for list rendering and tests | Low | UAT step U-11 | Open |
| ASM-014 | Open Food Facts accepts the `X-User-Agent` header as app identification for browser reads (its documentation names only `User-Agent`, which browsers do not let pages set). | `X-User-Agent` is in OFF's CORS allow-list; T0.2 request passed | Low (requests may be throttled) | No OFF 403 or block during Phase 8 and UAT | Open |
| ASM-015 | When the default Flash model returns 503, `gemini-3.5-flash-lite` is available. | T0.3: Flash-Lite answered while 3 Flash models returned 503 or 504 | Medium (photo analysis unavailable at busy times) | Retest in Phase 2 and during UAT | Open |

Placeholders: `<BARCODE>` is an 8-, 12- or 13-digit GTIN; `<MODEL_ID>` is a Gemini model ID matching `^gemini-[a-z0-9.\-]+$`; `<SECONDS>` is a whole number of seconds; `<FIRST_ERROR_PATH>` is a dotted JSON path such as `entries.3.quantity`; `<CODE>` is a CoFID food code; `<N>` is an integer from 2.

### 10.2 Open questions

| ID | Question | Impact | Status |
|---|---|---|---|
| OQ-001 | Do the Gemini, USDA and Open Food Facts APIs accept browser requests (CORS) with the headers this app sends? | High | Resolved 2026-11-17: yes for all 3 in Chromium and WebKit (`docs/spikes.md` T0.2). No proxy; ADR-0010 not written |

No assumption with impact High is open.

## 11. Release criteria

| ID | Criterion | Verified by |
|---|---|---|
| RC-01 | Every Must requirement in Section 5 has 1 or more passing tests from Section 12 | `scripts/check-trace.ts` and the test run |
| RC-02 | `npm run gates` exits 0 on `main` and the CI run on `main` is green | CI |
| RC-03 | The 3 playbook gates report PASS for `docs/requirements.md`, `docs/design.md` and the code, recorded in `docs/STATUS.md` | Gate report |
| RC-04 | The app loads from its GitHub Pages URL and TC-090 to TC-093 pass against live services | `npm run test:live` output in `docs/STATUS.md` |
| RC-05 | Every UAT step U-01 to U-11 in `docs/UAT.md` is marked Pass, or has a `docs/FEEDBACK.md` item with a status | `docs/UAT.md`, `docs/FEEDBACK.md` |
| RC-06 | Zac has written "accepted" in `docs/FEEDBACK.md` | `docs/FEEDBACK.md` |
| RC-07 | Tag `v1.0.0` exists on `main` and `docs/USER_GUIDE.md` exists | `git tag`, file check |

## 12. Traceability matrix (objective → requirement → test)

Test IDs and descriptions: PLAN Section 12, with the amendments in Section 12.1.

| Objective | Requirement | Tests |
|---|---|---|
| OBJ-01 | REQ-013 | TC-070, TC-071 |
| OBJ-01 | REQ-024 | TC-011, TC-022 |
| OBJ-01 | REQ-025 | TC-003, TC-070 |
| OBJ-01 | REQ-039 | TC-041, TC-081 |
| OBJ-01 | REQ-040 | TC-041, TC-081 |
| OBJ-01 | REQ-061 | TC-089 |
| OBJ-01 | REQ-067 | TC-070, TC-081 |
| OBJ-01 | REQ-068 | TC-060 |
| OBJ-01 | REQ-077 | TC-041, TC-081 |
| OBJ-01 | REQ-079 | TC-040, TC-071 |
| OBJ-01 | NFR-001 | TC-001, TC-002 |
| OBJ-01 | NFR-002 | TC-010, TC-011, TC-093 |
| OBJ-01 | NFR-003 | TC-020 |
| OBJ-02 | REQ-001 | TC-070, TC-080 |
| OBJ-02 | REQ-002 | TC-060, TC-080 |
| OBJ-02 | REQ-004 | TC-001, TC-008, TC-080 |
| OBJ-02 | REQ-027 | TC-080 |
| OBJ-02 | REQ-028 | TC-006, TC-080 |
| OBJ-02 | REQ-029 | TC-060, TC-080 |
| OBJ-02 | REQ-030 | TC-060, TC-080 |
| OBJ-02 | REQ-049 | TC-005, TC-085 |
| OBJ-02 | REQ-072 | TC-005, TC-085 |
| OBJ-02 | REQ-073 | TC-085 |
| OBJ-03 | REQ-008 | TC-004, TC-008 |
| OBJ-03 | REQ-010 | TC-031, TC-082 |
| OBJ-03 | REQ-023 | TC-033, TC-089 |
| OBJ-03 | REQ-031 | TC-012, TC-080 |
| OBJ-03 | REQ-032 | TC-021, TC-091 |
| OBJ-03 | REQ-034 | TC-022, TC-023, TC-083, TC-092 |
| OBJ-03 | REQ-035 | TC-083 |
| OBJ-03 | REQ-036 | TC-071 |
| OBJ-03 | REQ-037 | TC-008, TC-071 |
| OBJ-03 | REQ-038 | TC-071, TC-081 |
| OBJ-03 | REQ-064 | TC-081 |
| OBJ-03 | REQ-065 | TC-071 |
| OBJ-03 | REQ-074 | TC-083 |
| OBJ-03 | REQ-075 | TC-007, TC-083 |
| OBJ-03 | REQ-076 | TC-030, TC-031, TC-050, TC-081, TC-090 |
| OBJ-03 | REQ-078 | TC-040, TC-081 |
| OBJ-03 | REQ-080 | TC-040, TC-081 |
| OBJ-03 | NFR-005 | TC-030 |
| OBJ-03 | NFR-012 | TC-032 |
| OBJ-03 | NFR-013 | TC-031 |
| OBJ-04 | REQ-041 | TC-061, TC-084 |
| OBJ-04 | REQ-042 | TC-061, TC-084 |
| OBJ-04 | REQ-043 | TC-061 |
| OBJ-04 | REQ-044 | TC-061 |
| OBJ-04 | REQ-045 | TC-061, TC-084 |
| OBJ-04 | REQ-046 | TC-084 |
| OBJ-04 | REQ-048 | TC-061 |
| OBJ-04 | REQ-069 | TC-061 |
| OBJ-04 | REQ-070 | TC-061 |
| OBJ-04 | REQ-071 | TC-061 |
| OBJ-05 | REQ-020 | TC-087 |
| OBJ-05 | REQ-022 | TC-089 |
| OBJ-05 | REQ-051 | TC-072, TC-085 |
| OBJ-05 | REQ-052 | TC-072, TC-085 |
| OBJ-05 | REQ-053 | TC-072, TC-085 |
| OBJ-05 | REQ-054 | TC-062, TC-086 |
| OBJ-05 | REQ-055 | TC-062, TC-086 |
| OBJ-05 | REQ-056 | TC-062 |
| OBJ-05 | REQ-057 | TC-087 |
| OBJ-05 | REQ-058 | TC-087 |
| OBJ-05 | REQ-059 | TC-063 |
| OBJ-05 | REQ-060 | TC-063 |
| OBJ-05 | REQ-063 | TC-087 |
| OBJ-05 | REQ-066 | TC-063 |
| OBJ-05 | REQ-081 | TC-089 |
| OBJ-05 | REQ-082 | TC-072 |
| OBJ-05 | REQ-083 | TC-088 |
| OBJ-05 | NFR-004 | TC-013 |
| OBJ-05 | NFR-006 | `scripts/check-size.ts` (CI) |
| OBJ-05 | NFR-007 | TC-088 |
| OBJ-05 | NFR-008 | TC-062, TC-064 |
| OBJ-05 | NFR-009 | TC-089 |
| OBJ-05 | NFR-010 | TC-080 to TC-089 |
| OBJ-06 | REQ-062 | TC-064 |
| OBJ-06 | NFR-011 | Vitest coverage thresholds (CI) |

Check: 65 active REQ IDs and 13 NFR IDs, 78 rows, 0 empty cells. Every objective has 1 or more requirements. NFR-006 and NFR-011 are measured by CI checks rather than TC-numbered tests.

### 12.1 Test amendments to PLAN Section 12

| ID | Change |
|---|---|
| TC-005 | Covers REQ-049 and REQ-072 (override warning below 1,200 kcal, range 800 to 6,000) |
| TC-008 | Adds: the cross-field rule rejects a manual per-100 set summing to 105.1 g and accepts a database `FoodItem` summing to 109.1 g (DR-02); favourite names of 80 and 81 characters (DR-06) |
| TC-011 | Adds: numeric strings and numbers both parse (CR-03); the 34 excluded rows (CR-04); 31 `ml` rows (CR-05); `13-669` and `13-669#2` (CR-06) |
| TC-022 | Adds: `basis = ml` from `product_quantity_unit = "ml"`, and from `quantity` "330 ml", "33 cl" or "1 l" when `product_quantity_unit` is absent; `g` otherwise (P0-F9) |
| TC-023 | Adds: 429 → `OFF_RATE_LIMITED`, no retry |
| TC-031 | Adds: 400 without `API_KEY_INVALID` → `GEMINI_REQUEST_REJECTED`; request count ≤ 3 per analysis (NFR-013); the model used by attempt 2 follows the Phase 2 model decision |
| TC-032 | Adds: the repair request is skipped when less than 5,000 ms of the 62,000 ms budget remains, ending with `GEMINI_BAD_RESPONSE` |
| TC-040 | Adds: `cancel_tapped` in `ANALYSING` aborts the request (REQ-080); `log_tapped` in `REVIEW` is rejected while any item has no selection (REQ-079); nothing is written before `SAVING` (REQ-078) |
| TC-060 | Adds: 200 entries on one date list in order; restore after delete keeps the `id`; an entry's per-100 values are unchanged after the CoFID data used by search changes (REQ-068) |
| TC-061 | Adds: list order by `lastUsedAt` then name (REQ-043); case-insensitive filter (REQ-044); rename with uniqueness (REQ-069); item quantity change (REQ-070); removing an item from a 2-item meal gives kind `item` (REQ-071); starred names cut to 80 characters and suffixed (DR-08); multiplier rounding (DR-07) |
| TC-062 | Adds: merge tie on `updatedAt` keeps the device record; profile, targets and settings merge per DR-09 |
| TC-063 | Adds: `persist()` called once on first launch (REQ-059); banner when `persisted()` is `false` or the Storage API is missing (REQ-066); reminder text with no export and 1 entry (REQ-060) |
| TC-064 | New, unit level, covers REQ-062 and NFR-008: the log keeps the newest 200 events; key strings and image data are absent from stored events |
| TC-070 | Adds: Today entry rows show the badge text of each of the 5 sources (REQ-013); sugar with and without a target (REQ-001); "Includes 1 AI-estimated item" (REQ-067) |
| TC-071 | Adds: review rows show each database badge (REQ-013); removing a review row (REQ-065); quantity values 212.25 and 5,000.1 rejected (REQ-037); "Log items" disabled while a row shows "Check match" (REQ-079) |
| TC-072 | Adds: links to obtain each key (REQ-082) |
| TC-081 | Adds: tap count from Today to logged entries ≤ 4 (OBJ-03); "Cancel" visible at 3,000 ms with a delayed mocked response (REQ-080); "Use AI estimate" path (REQ-040) |
| TC-083 | Adds: invalid typed barcode sends 0 requests (REQ-075) |
| TC-084 | Adds: tap count from Today to a logged meal ≤ 3 (OBJ-04); edited item quantity in the add sheet (REQ-046) |
| TC-085 | Adds: "Reset" restores the computed target (REQ-073) |
| TC-087 | Adds: offline edit, favourite add and target recompute (REQ-057); update prompt when a new service worker is waiting (REQ-063) |
| TC-088 | Adds: light and dark themes follow `prefers-color-scheme` emulation (REQ-083) |
| TC-089 | Adds: About states the carbohydrate convention of each source (REQ-061) and shows version and build date (REQ-081) |

## Appendix A. Glossary additions

The glossary in PLAN Section 4 applies, with these additions.

| Term | Definition |
|---|---|
| Monosaccharide equivalent | Carbohydrate expressed as the weight of the monosaccharides it yields on hydrolysis; CoFID's convention. Starch in this form is about 1.10 × its own weight. |
| AOAC fibre | Total dietary fibre measured by the AOAC method, including resistant starch and lignin; the UK labelling method. |
| Trace (`Tr`) | CoFID marker for a nutrient present at a level too low to quantify; stored as 0. |
| Not determined (`N`) | CoFID marker for a nutrient present in significant quantities with no determined amount; stored as `null`. |
| Selected date | The date shown in the Today header; new entries are stored on it (REQ-028). |
| First complete load | Defined in REQ-057. |
| Match score | The CoFID search score of PLAN Section 10.4 applied to an AI item's search terms; 0 to 1. |
