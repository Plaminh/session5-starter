# CSC13008 IA#3 — VERDICTS.md

## Scope and evidence policy

This document evaluates the **original, untouched** Session 5 starter versions against `brief.md` and the nine checks in `README.md` (rows 1–8 plus `empty`). The predictions below were prepared with AI assistance from **static code inspection only**. They are hypotheses, not runtime findings; the student must review and commit them **before** running browser tests, running the type checker, or modifying source code.

- Files read: `brief.md`, `README.md`, `jsconfig.json`, `public/versions/a.js`, `public/versions/b.js`, `public/versions/c.js` (and the unchanged page/server contracts for context).
- No browser, server, or TypeScript tests were run to produce these predictions.
- `PASS`/`FAIL` in Section 1 means **predicted**, not confirmed.
- Section 2 and the final `keep`/`fix`/`reject` decisions must be completed from actual runs **after the prediction commit**.
- For every actual `FAIL`, capture `evidence/<version>-row<N>.png` showing the failing page and the relevant DevTools panel or editor/terminal, and identify the responsible source line(s).

## 1. Predictions BEFORE testing — commit this section first

### Version A — `public/versions/a.js`

| Row | Prediction | Reason from original source (not a test result) |
|---|---|---|
| 1 — normal cards | PASS | `getProducts()` fetches and parses `/api/products` (lines 12–16); `renderCards()` maps products in API order, fills template and prices, and replaces the list (26–35); `load()` clears the status for nonempty data (46–48). |
| 2 — stock total | **FAIL** | `products.forEach(async ...)` starts stock requests without awaiting them (52–55), but line 56 immediately renders the still-zero `total`; I expect **`0 đ` instead of `12.360.000 đ`**. Stock requests themselves are initiated concurrently. |
| 3 — literal scarf HTML | PASS | Product names are assigned with `textContent` at line 30, so `<i>limited</i>` should display literally. |
| 4 — slow | PASS | `Loading products…` is set before the awaited fetch (44–46), then the cards render and the status clears (47–48); the 5-second timeout (14) exceeds the server's 3-second delay. |
| 5 — error and Retry | PASS | A non-2xx response throws (15); `catch` clears cards, displays the exact error and shows Retry (57–60); Retry is registered (66) and is hidden when a new load starts (42). |
| 6 — hang timeout | PASS | `AbortSignal.timeout(5000)` is passed to the products fetch (14); timeout should enter `catch` and show the error and Retry (57–60) after about 5 seconds. |
| 7 — Console | PASS | Expected `/api/products` errors are handled by `catch` (57–60), without `console.error`. The detached stock promises are a potential issue, but the starter's valid stock endpoints should resolve during the specified tests; verify in Console. |
| 8 — `// @ts-check` | PASS | No obviously invalid browser option appears: `AbortSignal.timeout(5000)` is a valid API (14). With the starter's non-strict `jsconfig.json`, I predict no red type-check errors; verify in VS Code. |
| empty | PASS | Empty arrays render no cards (26–35), produce `No products yet.` (48), and skip the stock loop; line 56 formats the initial total `0` as `0 đ`. |

**Predicted overall outcome:** Version A likely needs a **targeted fix** for row 2. This is not a final verdict; confirm all nine rows first.

### Version B — `public/versions/b.js`

| Row | Prediction | Reason from original source (not a test result) |
|---|---|---|
| 1 — normal cards | PASS | `loadProducts()` fetches JSON (14–19); `products.map(createCard)` preserves order and applies template, IDs and formatted prices (28–33, 46–48). Nonempty results clear the status (48). |
| 2 — stock total | PASS | `Promise.all(products.map(...))` waits for stock fetches initiated in parallel (50–51); `reduce` calculates price × stock and prints the formatted result (52–53). |
| 3 — literal scarf HTML | PASS | `createCard()` writes the name via `textContent` (31), so `<i>limited</i>` should remain literal text. |
| 4 — slow | PASS | Loading is shown first (44); the products request is awaited (46), and the 3-second response should populate cards and clear status (47–48). The invalid timeout option is not expected to affect this 3-second case. |
| 5 — error and Retry | PASS | `response.ok` is checked (17); the error enters `catch` (54–56); Retry is wired (62), hidden when reloading (41), and should load cards when the mode returns to normal. |
| 6 — hang timeout | **FAIL** | `fetch('/api/products', { timeout: 5000 })` at line 16 uses an unsupported `RequestInit.timeout` property; it will not abort at five seconds. I expect **`Loading products…` still visible around 7 seconds**, until the server eventually responds after about 30 seconds. |
| 7 — Console | PASS | The products HTTP error and eventual hang response are handled by `try/catch` (45–58), without `console.error`; the ineffective timeout is a behavioural failure rather than necessarily an uncaught exception. Verify in Console. |
| 8 — `// @ts-check` | **FAIL** | Type-checking the file should flag the invalid `{ timeout: 5000 }` object property on `fetch` at line 16: `timeout` is not in `RequestInit`. The starter has DOM typings in `jsconfig.json`. |
| empty | PASS | `[]` yields zero cards (47), shows `No products yet.` (48), and `Promise.all([])` plus `reduce(..., 0)` produces `0 đ` (51–53). |

**Predicted overall outcome:** Version B likely needs a **targeted fix** for rows 6 and 8, both caused by line 16. This is not a final verdict; confirm all nine rows first.

### Version C — `public/versions/c.js`

| Row | Prediction | Reason from original source (not a test result) |
|---|---|---|
| 1 — normal cards | PASS | `fetchProducts()` returns parsed JSON (14–19); `render()` appends cards in iteration order with IDs and formatted prices (28–38); line 51 clears the normal status. |
| 2 — stock total | PASS | `Promise.all(products.map(...))` initiates stock fetches concurrently and waits for all of them (53–54), then sums price × stock (55–57). |
| 3 — literal scarf HTML | PASS | `ui.template` cards receive names via `textContent` (33), not `innerHTML`. |
| 4 — slow | PASS | Loading appears before the products request (47–49); the request's 5-second abort limit (16) allows a 3-second response, after which cards render and status clears (50–51). |
| 5 — error and Retry | PASS | Non-2xx responses throw (17), and `catch` clears cards, displays the error and unhides Retry (58–61); the Retry handler is attached (67), and it is hidden on a new attempt (45). |
| 6 — hang timeout | PASS | `AbortSignal.timeout(5000)` (16) should abort a hanging request around 5 seconds and enter the error/Retry `catch` (58–61). |
| 7 — Console | PASS | Expected failures are caught in `start()` (48–64), with no `console.error`; `Promise.all` is awaited so unexpected stock errors would also enter `catch`. Verify with Console. |
| 8 — `// @ts-check` | PASS | The browser APIs and `fetch` signal option appear valid (16); with non-strict `jsconfig.json`, I predict no red type-check errors. Verify in VS Code. |
| empty | PASS | `render([])` clears cards (28–38), status says `No products yet.` (51), and `Promise.all([])` plus an initial zero total prints `0 đ` (54–57). |

**Predicted overall outcome:** Version C appears a possible **keep** candidate, but a `keep` verdict requires all nine rows and the parallel request check to pass in actual tests.

### Additional brief constraint — parallel stock requests

This check is **in addition to** the 27 Done means results and matters for the 25-point final-behaviour rubric criterion.

| Version | Prediction from source | Evidence to collect later |
|---|---|---|
| A | Stock requests **start in parallel** (`forEach(async ...)`, lines 52–55), but the total is written before they complete (56). | Network waterfall should show concurrent starts; row 2 should reveal the premature display. |
| B | Stock requests start in parallel and are awaited together (`Promise.all`, line 51). | Verify concurrent `/api/stock/…` starts in Network or server logs. |
| C | Stock requests start in parallel and are awaited together (`Promise.all`, line 54). | Verify concurrent `/api/stock/…` starts in Network or server logs. |

## 2. Actual Test Results — after the prediction commit

**Testing date:** 2026-11-02. **Scope:** Original, unfixed A/B/C versions. The student confirms that all rows predicted PASS were tested and passed. Three predicted FAIL rows are supported by screenshots supplied in the conversation. The author of this update did **not** personally operate the student's browser or VS Code; PASS details below rest on the student's confirmation except where the screenshot itself supports them. No post-fix results are claimed here.

The **original predictions above remain unchanged**. Each entry below describes the reported observation, with a code reference to aid independent inspection; evidence screenshots are provided for every original FAIL. `KEEP (row)` means no change is needed for that row, not that a version with other failures is ready to ship.

| Version | Row | Result | Observation / basis | Related source lines | Row decision | Evidence | One precise sentence to assistant |
|---|---|---|---|---|---|---|---|
| A | 1 | **PASS** | Student reports: Under normal mode, eight product cards appeared in API order and the status line was empty. | `a.js:12–16, 26–35, 46–48` | KEEP (row) | Student report + `evidence/a-row2.png` (cards visible in captured view) | Row 1 passed the provided check; preserve this behavior when making any targeted fix to version A. |
| A | 2 | **FAIL** | Eight `/api/stock/…` requests finished with HTTP 200, but the page still displayed **`Stock value: 0 đ`** instead of `12.360.000 đ`; Network showed overlapping request starts. | `public/versions/a.js:51–56` (`forEach(async …)` is not awaited) | **FIX** | `evidence/a-row2.png` | Row 2 failed: all eight stock calls returned HTTP 200 but the total stayed `0 đ`; await their combined results before updating the value while preserving concurrent stock requests. |
| A | 3 | **PASS** | Student reports: The last name appeared literally as `Scarf <i>limited</i>`; it was not interpreted as italic markup. | `a.js:30` | KEEP (row) | Student report + `evidence/a-row2.png` (literal scarf name visible) | Row 3 passed the provided check; preserve this behavior when making any targeted fix to version A. |
| A | 4 | **PASS** | Student reports: Slow mode displayed `Loading products…` during the wait, then eight cards after approximately 3 seconds. | `a.js:14, 44–48` | KEEP (row) | Student-reported test; no screenshot required | Row 4 passed the provided check; preserve this behavior when making any targeted fix to version A. |
| A | 5 | **PASS** | Student reports: Error mode displayed `Could not load products.` and Retry; switching to normal and pressing Retry restored eight cards. | `a.js:15, 40–48, 57–67` | KEEP (row) | Student-reported test; no screenshot required | Row 5 passed the provided check; preserve this behavior when making any targeted fix to version A. |
| A | 6 | **PASS** | Student reports: Hang mode replaced Loading with `Could not load products.` and Retry around the five-second timeout. | `a.js:14, 57–62` | KEEP (row) | Student-reported test; no screenshot required | Row 6 passed the provided check; preserve this behavior when making any targeted fix to version A. |
| A | 7 | **PASS** | Student reports: No uncaught application errors were reported in Console during rows 1–6. | `a.js:45–63` | KEEP (row) | Student-reported test; no screenshot required | Row 7 passed the provided check; preserve this behavior when making any targeted fix to version A. |
| A | 8 | **PASS** | Student reports: With first-line `// @ts-check` in the full VS Code project, no TypeScript diagnostics were reported. | `a.js:4–8, 14` | KEEP (row) | Student-reported test; no screenshot required | Row 8 passed the provided check; preserve this behavior when making any targeted fix to version A. |
| A | empty | **PASS** | Student reports: Empty mode displayed `No products yet.`, zero cards and `0 đ`. | `a.js:26–35, 48, 51–56` | KEEP (row) | Student-reported test; no screenshot required | Row empty passed the provided check; preserve this behavior when making any targeted fix to version A. |
| B | 1 | **PASS** | Student reports: Under normal mode, eight product cards appeared in API order and the status line was empty. | `b.js:14–19, 28–33, 42–48` | KEEP (row) | Student-reported test; no screenshot required | Row 1 passed the provided check; preserve this behavior when making any targeted fix to version B. |
| B | 2 | **PASS** | Student reports: After the stock requests completed, the displayed stock value was `12.360.000 đ`. | `b.js:21–25, 50–53` | KEEP (row) | Student-reported test; no screenshot required | Row 2 passed the provided check; preserve this behavior when making any targeted fix to version B. |
| B | 3 | **PASS** | Student reports: The last name appeared literally as `Scarf <i>limited</i>`; it was not interpreted as italic markup. | `b.js:31` | KEEP (row) | Student-reported test; no screenshot required | Row 3 passed the provided check; preserve this behavior when making any targeted fix to version B. |
| B | 4 | **PASS** | Student reports: Slow mode displayed `Loading products…` during the wait, then eight cards after approximately 3 seconds. | `b.js:44–48` | KEEP (row) | Student-reported test; no screenshot required | Row 4 passed the provided check; preserve this behavior when making any targeted fix to version B. |
| B | 5 | **PASS** | Student reports: Error mode displayed `Could not load products.` and Retry; switching to normal and pressing Retry restored eight cards. | `b.js:17, 39–48, 54–63` | KEEP (row) | Student-reported test; no screenshot required | Row 5 passed the provided check; preserve this behavior when making any targeted fix to version B. |
| B | 6 | **FAIL** | With `hang` selected, after approximately seven seconds the page still showed **`Loading products…`**; the `/api/products` request was Pending, with no error or Retry. | `public/versions/b.js:16` (`{ timeout: 5000 }` is ignored by `fetch`) | **FIX** | `evidence/b-row6.png` | Row 6 failed: in hang mode `Loading products…` remained after seven seconds with `/api/products` Pending; use a real five-second abort signal while preserving concurrent stock requests and Retry. |
| B | 7 | **PASS** | Student reports: No uncaught application errors were reported in Console during rows 1–6. | `b.js:45–59` | KEEP (row) | Student-reported Console check; browser screenshot not supplied | Row 7 passed the provided check; preserve this behavior when making any targeted fix to version B. |
| B | 8 | **FAIL** | With `// @ts-check` on **line 1**, VS Code Problems reported **TS2353**: `timeout` does not exist in `RequestInit` (line 17 while the temporary comment was present). | `public/versions/b.js:16` in the original source (line 17 with temporary `// @ts-check`) | **FIX** | `evidence/b-row8.png` | Row 8 failed: VS Code reported TS2353 because `timeout` is not a `RequestInit` property; replace it with a supported signal and preserve parallel stock requests. |
| B | empty | **PASS** | Student reports: Empty mode displayed `No products yet.`, zero cards and `0 đ`. | `b.js:42, 47–53` | KEEP (row) | Student-reported test; no screenshot required | Row empty passed the provided check; preserve this behavior when making any targeted fix to version B. |
| C | 1 | **PASS** | Student reports: Under normal mode, eight product cards appeared in API order and the status line was empty. | `c.js:14–19, 28–38, 49–51` | KEEP (row) | Student-reported test; no screenshot required | Row 1 passed the provided check; preserve this behavior when making any targeted fix to version C. |
| C | 2 | **PASS** | Student reports: After the stock requests completed, the displayed stock value was `12.360.000 đ`. | `c.js:21–25, 53–57` | KEEP (row) | Student-reported test; no screenshot required | Row 2 passed the provided check; preserve this behavior when making any targeted fix to version C. |
| C | 3 | **PASS** | Student reports: The last name appeared literally as `Scarf <i>limited</i>`; it was not interpreted as italic markup. | `c.js:33` | KEEP (row) | Student-reported test; no screenshot required | Row 3 passed the provided check; preserve this behavior when making any targeted fix to version C. |
| C | 4 | **PASS** | Student reports: Slow mode displayed `Loading products…` during the wait, then eight cards after approximately 3 seconds. | `c.js:16, 47–51` | KEEP (row) | Student-reported test; no screenshot required | Row 4 passed the provided check; preserve this behavior when making any targeted fix to version C. |
| C | 5 | **PASS** | Student reports: Error mode displayed `Could not load products.` and Retry; switching to normal and pressing Retry restored eight cards. | `c.js:17, 43–51, 58–68` | KEEP (row) | Student-reported test; no screenshot required | Row 5 passed the provided check; preserve this behavior when making any targeted fix to version C. |
| C | 6 | **PASS** | Student reports: Hang mode replaced Loading with `Could not load products.` and Retry around the five-second timeout. | `c.js:16, 58–63` | KEEP (row) | Student-reported test; no screenshot required | Row 6 passed the provided check; preserve this behavior when making any targeted fix to version C. |
| C | 7 | **PASS** | Student reports: No uncaught application errors were reported in Console during rows 1–6. | `c.js:48–64` | KEEP (row) | Student-reported test; no screenshot required | Row 7 passed the provided check; preserve this behavior when making any targeted fix to version C. |
| C | 8 | **PASS** | Student reports: With first-line `// @ts-check` in the full VS Code project, no TypeScript diagnostics were reported. | `c.js:4–10, 16` | KEEP (row) | Student-reported test; no screenshot required | Row 8 passed the provided check; preserve this behavior when making any targeted fix to version C. |
| C | empty | **PASS** | Student reports: Empty mode displayed `No products yet.`, zero cards and `0 đ`. | `c.js:28–38, 51, 54–57` | KEEP (row) | Student-reported test; no screenshot required | Row empty passed the provided check; preserve this behavior when making any targeted fix to version C. |

### Additional brief constraint — concurrent stock requests

| Version | Pre-fix result | Evidence / source | Note |
|---|---|---|---|
| A | **PASS** | `evidence/a-row2.png`; `a.js:52–55` | The screenshot shows eight completed stock calls starting in an overlapping waterfall; total calculation nevertheless fails row 2. |
| B | **PASS — student-reported** | `b.js:50–53`; student's testing confirmation | No separate B Network screenshot was supplied to this update; verify again after the fix. |
| C | **PASS — student-reported** | `c.js:53–57`; student's testing confirmation | No separate C Network screenshot was supplied; keep C untouched if subsequent checks agree. |

**Screenshot provenance:** The three PNGs in `evidence/` are copies of the student's screenshots supplied during this conversation. `b-row6.png` visually shows a Pending products request and the loading state; the elapsed seven-second observation was reported by the student during testing. `b-row8.png` displays the first-line `// @ts-check` and TypeScript diagnostic. Return any temporary `// @ts-check` edit to the original source before committing documentation.

## 3. Final Verdicts — for the ORIGINAL versions (before fixes)

| Version | Failed rows | Verdict | Deciding evidence | One actionable sentence for the assistant |
|---|---|---|---|---|
| **A** | 2 | **FIX** | `evidence/a-row2.png`, `public/versions/a.js:51–56` | Row 2 failed: all eight stock requests returned 200 but the stock value stayed `0 đ`; await their results as a group and retain parallel stock requests and all passing UI states. |
| **B** | 6, 8 | **FIX** | `evidence/b-row6.png`, `evidence/b-row8.png`, `public/versions/b.js:16` | Rows 6 and 8 failed: `fetch` ignored `{ timeout: 5000 }` and `@ts-check` raised TS2353; use a supported five-second abort signal without changing concurrent stock aggregation, error states, or Retry. |
| **C** | None reported | **KEEP** | Student reports all nine rows PASS; source `c.js:16, 33, 54–57` | All nine checks were reported to pass; keep version C unchanged and preserve the concurrent stock requests. |

These decisions are for the **original starter versions**. A and B still need to be fixed and tested again under every condition; the table is **not** a claim that the final source is already compliant.

## 4. Prediction vs observation corrections

| Comparison | Observation |
|---|---|
| Predictions for A (row 2), B (rows 6 and 8), and every other row | All 27 initial predictions matched the student's reported outcomes. |
| Assistant-independent code review | The provisional A = FIX, B = FIX, C = KEEP recommendations matched these original-version findings. |
| Contradictory predictions / false positives | None reported from the standard nine-row test suite. |

## 5. Remaining verification for full marks

- Keep the three screenshot files under `evidence/` in the submitted repository.
- Verify the original prediction commit precedes these findings in Git history.
- Restore any temporary `// @ts-check` source edits used for row 8 evidence.
- After assistant-assisted fixes, rerun all 27 checks and the stock concurrency check. Commit **one version per fix** and keep the changes restricted to `public/versions/*.js`.
