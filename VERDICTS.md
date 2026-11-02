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

## 2. Actual Test Results — COMPLETE ONLY AFTER THE PREDICTION COMMIT

**Status: NOT TESTED.** The entries below are deliberately blank. Do not turn static predictions into claimed observations. Run `normal`, `slow`, `error`, `hang`, `empty`, Console and `// @ts-check` as specified by `README.md` for **every original version**.

For each row, replace `NOT RUN` with `PASS` or `FAIL`, record **exactly what appeared**, and link to `evidence/<version>-row<N>.png` for every actual FAIL. Record the code line responsible for every FAIL. Write one actionable sentence to your assistant naming the **row**, the **observed evidence**, and the **constraint to preserve**.

| Version | Row | Result | Observed behaviour / evidence | Responsible line(s) if FAIL | Screenshot if FAIL | One precise sentence for assistant if FAIL |
|---|---|---|---|---|---|---|
| A | 1 | NOT RUN | — | — | — | — |
| A | 2 | NOT RUN | — | — | — | — |
| A | 3 | NOT RUN | — | — | — | — |
| A | 4 | NOT RUN | — | — | — | — |
| A | 5 | NOT RUN | — | — | — | — |
| A | 6 | NOT RUN | — | — | — | — |
| A | 7 | NOT RUN | — | — | — | — |
| A | 8 | NOT RUN | — | — | — | — |
| A | empty | NOT RUN | — | — | — | — |
| B | 1 | NOT RUN | — | — | — | — |
| B | 2 | NOT RUN | — | — | — | — |
| B | 3 | NOT RUN | — | — | — | — |
| B | 4 | NOT RUN | — | — | — | — |
| B | 5 | NOT RUN | — | — | — | — |
| B | 6 | NOT RUN | — | — | — | — |
| B | 7 | NOT RUN | — | — | — | — |
| B | 8 | NOT RUN | — | — | — | — |
| B | empty | NOT RUN | — | — | — | — |
| C | 1 | NOT RUN | — | — | — | — |
| C | 2 | NOT RUN | — | — | — | — |
| C | 3 | NOT RUN | — | — | — | — |
| C | 4 | NOT RUN | — | — | — | — |
| C | 5 | NOT RUN | — | — | — | — |
| C | 6 | NOT RUN | — | — | — | — |
| C | 7 | NOT RUN | — | — | — | — |
| C | 8 | NOT RUN | — | — | — | — |
| C | empty | NOT RUN | — | — | — | — |

## 3. Final Verdicts — COMPLETE AFTER TESTING ORIGINALS

| Version | Failed rows (observed) | Final verdict (`keep` / `fix` / `reject`) | Deciding row(s) and evidence | One-sentence instruction to assistant |
|---|---|---|---|---|
| A | NOT RUN | PENDING | — | — |
| B | NOT RUN | PENDING | — | — |
| C | NOT RUN | PENDING | — | — |

A `keep` decision means the original version passes every required test and stock requests are parallel. A `fix` decision requires an exact failing row and a targeted change; a `reject` decision still requires an acceptable replacement at the end. After each fix, rerun **all** rows and the parallel check. Commit each fixed version separately, changing only its corresponding file under `public/versions/`.

## 4. Prediction vs observation corrections — COMPLETE AFTER TESTING

Record any prediction contradicted by a real run; incorrect predictions are acceptable when clearly corrected with evidence.

| Version | Row | Prediction | Actual observation | What I learned |
|---|---|---|---|---|
| — | — | — | NOT TESTED | — |
