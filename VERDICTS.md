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

## Phase 4 — Post-Fix Verification

**Documentation date:** 2026-11-02. **Scope:** Updated Version A only. All original predictions, test results, failure evidence, and A/B/C verdicts above remain unchanged.

**Original Version A:** **FIX** — README row 2 failed: the stock total remained `0 đ` despite eight successful concurrent stock requests. Original evidence remains `evidence/a-row2.png`; it is not post-fix evidence.

**Targeted correction:** Existing commit `7028acc` (`fix(a): await concurrent stock requests`) changes only `public/versions/a.js`. Current lines 51–53 use `await Promise.all(products.map((p) => getStock(p.id)))`, then reduce `price * stocks[i]` from zero and write the total through the existing `formatVnd` function. All stock requests start concurrently; calculation waits for their responses. Empty arrays retain `0 đ`, and stock failures enter the existing catch. Expected corrected normal total: **`12.360.000 đ`**.

**Updated Version A:** **PASS / VERIFIED — all 10 checks passed in Codex-executed post-fix testing on 2026-11-02.** Ready to finalize Version A. The historical original-version verdict remains **FIX (row 2)**. Version B still requires its targeted fixes; Phase 4 as a whole is not complete.

### Version A regression checklist

**Testing method and date:** On 2026-11-02, Codex automated installed Microsoft Edge `156.0.4314.58` in headless mode through the Chrome DevTools Protocol at `http://localhost:5050/?v=a`, with cache and extensions disabled. The existing server on port 5050 was reused after confirming its served A source exactly matched the workspace. Tests clicked the actual mode buttons, reloaded the page, and clicked Retry. DOM observations, browser network events, Resource Timing, and mutation timestamps supplied the observations below; no fetch/DOM mocks were used. Check 8 used VS Code's installed TypeScript `6.0.3` API with the existing `jsconfig.json` options plus `allowJs: true`, `checkJs: true`, and `noEmit: true`, restricted to A. No source comment or configuration was changed, and nothing was downloaded.

These are **Codex-executed results**, not student manual results. No additional student manual tests have been reported. All 10 checks completed; none remain pending. Evidence is the recorded browser/checker execution and measurements below; there was no confirmed application failure requiring a screenshot.

| Check | Actual observed behaviour | Result | Tool / evidence |
|---|---|---|---|
| 1. Normal cards | Exactly eight cards; IDs and names matched the actual products API in order: tshirt, hoodie, somi, cap, tote, socks, jacket, scarf. Prices were `180.000 đ`, `420.000 đ`, `350.000 đ`, `95.000 đ`, `120.000 đ`, `75.000 đ`, `890.000 đ`, `160.000 đ`. Final status was empty. | PASS | Edge DOM compared with the live API response |
| 2. Stock total | All eight stock requests returned HTTP 200. The displayed total changed from the placeholder to exactly `12.360.000 đ`; it did not remain at `0 đ`. | PASS | Edge DOM, network events, and mutation timestamps |
| 3. Literal scarf text | Last name was exactly `Scarf <i>limited</i>`; zero `i`/`em` descendants were created. Its DOM innerHTML was `Scarf &lt;i&gt;limited&lt;/i&gt;`. | PASS | Edge DOM text, markup, and descendant checks |
| 4. Slow loading | `Loading products…` appeared with zero cards; eight cards then appeared and status cleared. Products response arrived after **3008.219 ms**; the loading-to-cards interval was **3012.300 ms**. The total subsequently became `12.360.000 đ`. | PASS | CDP network timestamps and DOM mutation timestamps |
| 5. Error and Retry | Real HTTP 500 produced `Could not load products.`, visible Retry, and zero cards. Selecting normal and clicking Retry restored eight cards and `12.360.000 đ`; status cleared and Retry became hidden. | PASS | Edge mode-button/Retry clicks, DOM, and network events |
| 6. Five-second timeout | The pending products request was canceled after **5000.751 ms**, with `net::ERR_ABORTED` and `canceled: true`; the error message and Retry appeared with zero cards, without waiting thirty seconds. | PASS | CDP request/loadingFailed timestamps and DOM |
| 7. Console errors | Final extensions-disabled run recorded **0** runtime exceptions/unhandled rejections and **0** application console.error calls across the browser cases. Browser network diagnostics were the expected products HTTP 500 and missing `favicon.ico` HTTP 404; neither was an application exception. | PASS | CDP Runtime.exceptionThrown, Runtime.consoleAPICalled, and Log.entryAdded |
| 8. Type checking | Existing VS Code TypeScript **6.0.3** checker returned **0 diagnostics** for updated A with JavaScript checking enabled and the project compiler options. Source remained unchanged. | PASS | TypeScript API, checkJs/noEmit equivalent to the README opt-in; no download |
| 9. Empty response | Zero cards, exactly `No products yet.`, and `0 đ`; Retry hidden and no stock requests issued. | PASS | Edge DOM and network events |
| 10. Parallel stock requests | All eight stock requests started within **1.8 ms**; the first response began **226.6 ms** after the first start. All request lifetimes overlapped before any response. Last completion was **816.8 ms** after the first start; `12.360.000 đ` was displayed **2.2 ms** later. | PASS | Browser Resource Timing and DOM mutation timestamps; detailed timings below |

**Concurrent-request measurements:** Milliseconds relative to the first stock request start in the final normal-mode run. These are browser resource start times, including any browser connection scheduling, not a claim that eight sockets transmitted at the same instant.

| Stock ID | Request start (ms) | Response start (ms) | Response complete (ms) |
|---|---|---|---|
| tshirt | 0.0 | 545.0 | 546.3 |
| hoodie | 0.1 | 546.5 | 547.5 |
| somi | 0.5 | 528.0 | 529.8 |
| cap | 0.7 | 326.2 | 327.0 |
| tote | 0.9 | 526.5 | 527.8 |
| socks | 1.1 | 226.6 | 228.1 |
| jacket | 1.4 | 814.3 | 816.8 |
| scarf | 1.8 | 544.5 | 545.4 |

**Earlier validation and test limitations:** The implementation-time syntax and mocked Node checks remain historical preliminary checks; they are not the basis for the browser PASS results above. The initial browser script sometimes sampled the old page before reload completed; its synchronization was corrected. A subsequent run exposed `getAllPromptList` promise errors from `chrome-extension://difoiogjjojoaoomphldepapgpbgkhkb/content-all.js`, not application source. The final run repeated all browser cases with extensions disabled and recorded zero runtime exceptions. Diagnostic screenshots from the invalid pilot runs were discarded, not used as failure evidence. Original Phase 3 screenshots were untouched. A first checker invocation had a configuration-path discovery error; normalizing the Windows path produced the completed zero-diagnostic check. These tooling issues were not counted as application regressions.

**Regressions and final scope:** No application regression was found in the ten executed checks. The unrelated favicon 404 remains visible as a browser network diagnostic. No student manual testing is claimed. Validation covers this installed Edge/TypeScript environment, not every browser. The temporary browser was closed and the existing server was left running in normal mode. Version A is ready to finalize; Version B remains unfixed, so Phase 4 is not complete.

### Version B — Post-Fix Verification

**Testing date:** 2026-11-02. This subsection records the subsequent Version B fix and verification. Earlier references to B still needing repair describe the state when Version A was verified; this subsection supersedes that pending status without changing A's results or any original-version findings.

**Original Version B:** **FIX — rows 6 and 8 failed.** Original `public/versions/b.js:16` passed unsupported `{ timeout: 5000 }` to fetch. In hang mode, loading and the pending products request persisted beyond seven seconds (`evidence/b-row6.png`). With first-line `// @ts-check`, VS Code reported TS2353 because `timeout` is not a RequestInit property (`evidence/b-row8.png`; line 17 while the temporary comment was present). These original results and screenshots remain unchanged.

**Implemented correction:** Updated `public/versions/b.js:16` replaces only `{ timeout: 5000 }` with `{ signal: AbortSignal.timeout(5000) }`. The valid RequestInit signal aborts the products request after five seconds and routes its rejection through the existing catch at lines 54–56. The concurrent `Promise.all(products.map(...))` and indexed stock reduction at lines 51–53 are untouched, as are rendering, currency formatting, textContent, Retry, and empty-state handling. No fallback, dependency, other implementation edit, or commit was introduced.

**Environment and method:** Codex reused the running original server at `http://localhost:5050/?v=b` after confirming HTTP 200 and byte-for-byte equality between the served B source and the workspace. Installed Microsoft Edge **156.0.4314.58** ran headlessly through Chrome DevTools Protocol **1.3**, with cache and extensions disabled. Tests clicked the actual mode buttons, reloaded, and clicked Retry, using real endpoints and no mocked responses. DOM, network events, Resource Timing, and mutation timestamps supplied browser observations. VS Code's installed TypeScript **6.0.3** API checked B alone with `jsconfig.json` options plus `allowJs: true`, `checkJs: true`, and `noEmit: true`; no source comment or configuration change was needed. Node **v24.11.1** syntax checking and `git diff --check` also passed. No tool or dependency was installed.

| Check | Result | Actual observed behaviour | Method / evidence |
|---|---|---|---|
| 1. Normal cards | PASS | Exactly eight cards matched live API IDs, names, and order: tshirt, hoodie, somi, cap, tote, socks, jacket, scarf. Prices were `180.000 đ`, `420.000 đ`, `350.000 đ`, `95.000 đ`, `120.000 đ`, `75.000 đ`, `890.000 đ`, `160.000 đ`. Final status was empty. | Edge DOM compared with actual products API response |
| 2. Stock total | PASS | All eight stock requests returned HTTP 200; the final output was exactly `12.360.000 đ`. | Edge DOM, network events, and mutation timestamps |
| 3. Safe text rendering | PASS | Last name was exactly `Scarf <i>limited</i>`; zero i/em descendants existed, and innerHTML contained `Scarf &lt;i&gt;limited&lt;/i&gt;`. | Edge text/markup/descendant checks |
| 4. Slow loading | PASS | `Loading products…` appeared with zero cards. Products response arrived after **3016.536 ms**; eight cards appeared and status cleared after a **3017.400 ms** loading interval. The correct total followed. | CDP network and DOM mutation timestamps |
| 5. HTTP error and Retry | PASS | HTTP 500 displayed `Could not load products.`, visible Retry, and zero cards. Selecting normal and clicking Retry restored eight cards and `12.360.000 đ`; status cleared and Retry hid. | Real mode-button/Retry clicks, network status, and DOM |
| 6. Five-second timeout | PASS | The pending products request was canceled after **5005.099 ms**, with `net::ERR_ABORTED` and `canceled: true`. Loading changed to `Could not load products.`, Retry appeared, and zero cards remained; no thirty-second wait. | CDP request/loadingFailed timestamps and DOM |
| 7. Console | PASS | **0** uncaught runtime exceptions/unhandled rejections and **0** application console.error calls across browser scenarios. Browser-generated products HTTP 500 and missing favicon HTTP 404 diagnostics were recorded separately. | CDP Runtime.exceptionThrown, Runtime.consoleAPICalled, and Log.entryAdded |
| 8. Type checking | PASS | TypeScript **6.0.3** returned **0 diagnostics** for updated B with JavaScript checking enabled and project options. The former invalid timeout property is absent. | Existing VS Code TypeScript API; checkJs/noEmit equivalent to README opt-in |
| 9. Empty response | PASS | Zero cards, exactly `No products yet.`, and `0 đ`; Retry hidden and no stock requests. | Edge DOM and network events |
| 10. Parallel stock requests | PASS | All eight resource starts fell within **0.4 ms**, before the first response at **150.8 ms**. The last response completed at **719.7 ms**, and `12.360.000 đ` appeared **2.2 ms** later. Requests overlapped; concurrency was measured independently of the correct total. | Browser Resource Timing and DOM mutation timestamps |

**Stock timing evidence:** Milliseconds relative to the first stock request start in the final normal-mode run. Resource start times include browser connection scheduling; these measurements do not assert eight simultaneous socket transmissions.

| Stock ID | Request start (ms) | Response start (ms) | Response complete (ms) |
|---|---|---|---|
| tshirt | 0.0 | 150.8 | 151.1 |
| hoodie | 0.2 | 433.8 | 434.1 |
| somi | 0.2 | 451.3 | 451.6 |
| cap | 0.2 | 518.6 | 519.0 |
| tote | 0.3 | 585.3 | 585.7 |
| socks | 0.3 | 552.2 | 553.3 |
| jacket | 0.4 | 585.7 | 585.9 |
| scarf | 0.4 | 719.1 | 719.7 |

**Regressions, evidence, and limitations:** No application regression or new confirmed failure was found; no new failure screenshot was required or created. Original evidence and Version A's entire post-fix record remain unchanged. The favicon 404 is a browser network diagnostic, not a B application exception. These were Codex-executed tests; no new student manual results are claimed. All ten checks completed with no pending checks. Results cover this installed browser/checker environment. The temporary browser was closed and the existing server left running in normal mode.

**Updated Version B: PASS / VERIFIED — all 10 post-fix checks passed.** B is ready for the student's separate code-fix and documentation commits. The original B verdict remains **FIX (rows 6 and 8)**. Together with A's existing verification and C's unchanged original KEEP result, the targeted fixes and required post-fix checks for Phase 4 are complete in the recorded environment. No Phase 5 work was started.
