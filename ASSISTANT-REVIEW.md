# IA#3 — Independent Assistant Review

## 1. Review metadata and provenance

- **Assignment:** CSC13008 — IA#3, Three Answers to One Brief (Session 5).
- **Tool:** ChatGPT.
- **Exact model:** GPT Astra High.
- **Review date:** 2026-11-02.
- **Review method:** Independent static code review.
- **Runtime testing:** Not performed.
- **Provenance:** Reviewer metadata and the exact original prompt were verified by the student. The full original assistant reply supplied as `Pasted text.txt` is preserved below without editing its claims. Later checks are documented separately in Section 4; the original reply remains a source-only review.

## 2. Original prompt

```text
You are conducting an independent code review for
CSC13008 Individual Assignment #3:
Three Answers to One Brief.

This is a READ-ONLY review.

IMPORTANT:

- Do NOT read VERDICTS.md.
- Do NOT use any previous predictions or verdicts.
- Do NOT modify source code or documentation.
- Do NOT run Git write commands.
- Do NOT fix any issues yet.
- Do NOT claim runtime tests were performed.

Read these original starter files:

1. README.md
2. brief.md
3. public/versions/a.js
4. public/versions/b.js
5. public/versions/c.js
6. server.mjs
7. jsconfig.json

TASK 1 — Independent Review

Review each original version A, B, and C against
every requirement in brief.md.

For each version, examine:

- Product fetching and rendering
- Product order and formatted prices
- Stock value calculation
- Parallel stock requests
- Safe rendering using textContent
- Loading state
- HTTP error handling
- Retry behaviour
- Five-second timeout
- Empty response handling
- Console error handling
- JavaScript type-checking compatibility

TASK 2 — Rubric Row Predictions

For each version, evaluate README.md Done means
rows 1 through 8 and the additional empty case.

Provide a table containing:

- Version
- Row
- Predicted PASS or FAIL
- Evidence from the original code
- Exact responsible file and line numbers
- Explanation of the potential failure

Distinguish static findings from behaviours that
still require browser testing.

TASK 3 — Overall Evaluation

For each version:

- Recommend KEEP, FIX, or REJECT provisionally.
- Explain the recommendation.
- Identify the smallest necessary correction.
- State which behaviours must remain unchanged.

Pay special attention to preserving concurrent
stock requests.

TASK 4 — Review Limitations

Explicitly identify:

- Findings confirmed by reading the source
- Predictions requiring runtime verification
- Type-checking findings requiring VS Code or tsc
- Potential false positives or uncertainty

OUTPUT:
Return your complete review in English,
with separate sections for A, B, and C.

Do not create or modify any files.
Do not stage, commit, or push anything.
Do not modify Git history.
```

## 3. Original Assistant Response (Verbatim)

Provisional recommendations: **A — FIX; B — FIX; C — KEEP.** A calculates the displayed total too early. B does not implement the required timeout. C has no identified defect against the supplied brief.

This is a source-only review of the seven requested files. I did not read verdict files, modify files, run Git write commands, start the application, perform browser tests, or run a type checker.

All PASS/FAIL entries below are **predictions**, not recorded test results. Line numbers refer to the files as read.

## Review basis

The requirements come from [brief.md](D:/nam3/WEB/IA3/session5-starter/brief.md), lines 34–58; the rubric comes from [README.md](D:/nam3/WEB/IA3/session5-starter/README.md), lines 76–85.

The supplied [server.mjs](D:/nam3/WEB/IA3/session5-starter/server.mjs) establishes:

- Eight products in a fixed order, including `Scarf <i>limited</i>`: lines 19–28.
- Stock quantities: line 31. Multiplying these by the listed prices gives **12,360,000**, formatted as **`12.360.000 đ`**.
- Normal, empty, three-second slow, HTTP 500, and thirty-second hang responses: lines 166–172.
- Independently delayed stock responses, each scheduled for 100–600 ms: lines 176–183.

All three versions use plain browser JavaScript, invoke their loader on import, and install a Retry handler. Their product and stock requests default to GET and use same-origin URLs.

## Version A — FIX

Source: [public/versions/a.js](D:/nam3/WEB/IA3/session5-starter/public/versions/a.js)

### Independent findings

Product fetching and rendering follow the brief: A checks the HTTP status, clones the template, sets `data-id`, preserves API order, and formats prices correctly. API-derived visible text uses `textContent`. Loading, empty, product-error, and Retry states have appropriate code paths.

**The stock requests are concurrent, but their results are not awaited before displaying the total.** At lines 52–55, `forEach` starts asynchronous callbacks and ignores their returned promises. Line 56 therefore displays the initial `total`, which is zero. Later responses change the local variable without updating the output.

The same detached callbacks create a secondary risk: if a stock request rejects, its rejection does not reach the surrounding `try/catch`. This is distinct from the supplied normal stock responses, which succeed for all eight IDs.

### Rubric predictions

| Version | Row | Predicted | Original-code evidence and exact lines | Explanation / potential failure |
|---|---|---|---|---|
| A | 1 — Normal cards | PASS | `a.js:14–16`, `26–34`, `46–48`; formatter at `10` | Fetches products, maps them in API order, clones the template, assigns names/prices/IDs, and clears status. Stock-total failure does not invalidate this row. |
| A | 2 — Stock value | **FAIL** | `a.js:51–56` | Displays `0 đ` before any awaited stock result resumes. No later DOM update displays `12.360.000 đ`. This ordering defect is confirmed statically. |
| A | 3 — Literal scarf name | PASS | `a.js:30` | Assigns the name through `textContent`, so `<i>limited</i>` remains literal text. |
| A | 4 — Slow loading | PASS | `a.js:44–48`; timeout at `14` | Sets loading before awaiting products and clears it after rendering. The supplied three-second response is within the five-second timeout. |
| A | 5 — Error and Retry | PASS | `a.js:15`, `40–48`, `57–67` | Non-2xx responses throw; catch removes cards and shows the error and Retry. Retry hides the button and begins another load. The total remains wrong after successful Retry, but that is row 2. |
| A | 6 — Five-second timeout | PASS | `a.js:14`, `57–62` | Uses `AbortSignal.timeout(5000)` and handles rejection by showing the required error and Retry. Browser support and timing need verification. |
| A | 7 — Console | PASS | `a.js:45–63`; detached callbacks at `52–55` | Expected product failures are caught, with no explicit console logging. Supplied stock requests succeed. **A stock rejection would instead produce an unhandled promise rejection**, so this prediction is limited to the listed scenarios. |
| A | 8 — Type checking | PASS | `a.js:4–8`, `14`, `28–31`, `52–56`; `jsconfig.json:3–12` | DOM casts and the configured libraries should support the code in a current TypeScript checker. The asynchronous `forEach` bug is legal JavaScript and is not ordinarily a type error. Not checker-verified. |
| A | Empty | PASS | `a.js:27–34`, `48`, `51–56` | Empty cards replace the list; status becomes `No products yet.`; no stock requests run; the zero total is correct for this case. |

### Smallest necessary correction

Replace the detached asynchronous `forEach` at lines 51–56 with an **awaited `Promise.all` aggregation**, then calculate and display the total after all stock responses resolve. This also brings stock-request rejections into the existing catch path.

**Preserve concurrency:** construct all stock promises with `products.map(...)` before awaiting their collective result. Do not replace the loop with a `for...of` that awaits each stock request sequentially.

Keep unchanged:

- API product order, template cloning, `data-id`, and price formatting.
- `textContent` rendering.
- Product timeout, loading/error/empty messages, and Retry behaviour.
- Immediate card rendering before the stock total completes.

The defect is localized, so **FIX** is more proportionate than REJECT.

## Version B — FIX

Source: [public/versions/b.js](D:/nam3/WEB/IA3/session5-starter/public/versions/b.js)

### Independent findings

Product rendering, order, price formatting, safe text assignment, loading, empty results, HTTP error handling, and Retry are implemented appropriately.

Stock aggregation is correct: line 51 starts all stock requests through `map`, then awaits `Promise.all`. The resulting array preserves input order, so line 52 pairs each product with its corresponding stock even if requests finish out of order.

**The timeout is ineffective.** Browser `fetch` does not implement a `timeout` request option. Consequently, `{ timeout: 5000 }` at line 16 does not cancel the request after five seconds. It is also an unsupported property in the standard DOM `RequestInit` type.

### Rubric predictions

| Version | Row | Predicted | Original-code evidence and exact lines | Explanation / potential failure |
|---|---|---|---|---|
| B | 1 — Normal cards | PASS | `b.js:10–11`, `16–18`, `28–33`, `42–48` | Clears the catalogue, appends mapped cards in API order, sets IDs and formatted prices, and clears status. |
| B | 2 — Stock value | PASS | `b.js:21–25`, `51–53` | Awaits all concurrent stock requests, sums matching `price × stock` values, and formats the result. Actual completion within roughly one second remains environment-dependent. |
| B | 3 — Literal scarf name | PASS | `b.js:31` | Uses `textContent`; the name cannot create an italic element. |
| B | 4 — Slow loading | PASS | `b.js:44–48` | Loading remains while products are awaited and clears after rendering. The absent timeout does not prevent the supplied three-second response from succeeding. |
| B | 5 — Error and Retry | PASS | `b.js:17`, `39–48`, `54–63` | Checks non-2xx responses, starts with an empty catalogue, shows the error and Retry, and resets those states on another attempt. |
| B | 6 — Five-second timeout | **FAIL** | `b.js:16`, `44–46`; `server.mjs:171–172` | The unsupported option does not abort `fetch`. With the supplied hang response, loading is predicted to continue until the server returns HTTP 504 at about thirty seconds, rather than changing at five seconds. |
| B | 7 — Console | PASS | `b.js:45–59`, particularly `51` and `54–56` | Product and stock promise failures are awaited and caught without explicit logging. The unsupported option is normally ignored at runtime. A later browser-generated HTTP 504 message is a network diagnostic, not an uncaught application exception. |
| B | 8 — Type checking | **FAIL** | `b.js:16`; `jsconfig.json:5–12` | With `@ts-check`, the object literal is expected to fail excess-property checking because `timeout` is not a `RequestInit` member. Exact diagnostic wording requires VS Code or `tsc`. |
| B | Empty | PASS | `b.js:42`, `47–53` | Shows `No products yet.`. `Promise.all([])` completes, and the reduction’s initial value of zero produces `0 đ`. |

### Smallest necessary correction

Replace the unsupported timeout option at line 16 with a supported abort signal, such as `signal: AbortSignal.timeout(5000)` for the same browser target assumed by A and C. If older browsers must be supported, use an `AbortController` and a cleared timer instead.

Keep unchanged:

- **The concurrent `Promise.all(products.map(...))` stock requests.**
- Stock-to-product index matching and the reduction’s initial zero.
- Ordered template rendering, `data-id`, price formatting, and `textContent`.
- Loading, empty, HTTP-error, and Retry state transitions.

One localized change addresses both the timeout defect and the predicted type-checking failure, supporting **FIX**.

A secondary observation: if a stock request fails after cards render, B’s catch leaves those cards visible (`b.js:47`, `54–56`). The supplied rubric’s HTTP-error mode fails the **products** request before cards render, so this is not evidence of a row 5 failure. The brief does not clearly specify card visibility after a stock-only failure.

## Version C — KEEP

Source: [public/versions/c.js](D:/nam3/WEB/IA3/session5-starter/public/versions/c.js)

### Independent findings

C implements all identified brief requirements:

- Fetches products on import and checks HTTP success.
- Uses a five-second abort signal.
- Clones template cards, preserves API order, and sets `data-id`.
- Formats prices and totals using `vi-VN`.
- Assigns API-derived visible text using `textContent`.
- Handles loading, success, empty, error, and Retry states.
- Starts stock requests concurrently, awaits their completion, and calculates the total using matching indices.
- Catches awaited request failures without explicit console errors.

No necessary source correction was identified.

### Rubric predictions

| Version | Row | Predicted | Original-code evidence and exact lines | Explanation / potential failure |
|---|---|---|---|---|
| C | 1 — Normal cards | PASS | `c.js:12`, `16–18`, `28–37`, `49–51` | Iterates products in API order, clones and fills cards, replaces the catalogue, and clears status. |
| C | 2 — Stock value | PASS | `c.js:21–25`, `54–57` | Awaits concurrent stock requests, then sums each product’s price multiplied by its matching stock. Predicted total: `12.360.000 đ`. |
| C | 3 — Literal scarf name | PASS | `c.js:33` | Uses `textContent`, preserving the angle brackets as text. |
| C | 4 — Slow loading | PASS | `c.js:47–51`; timeout at `16` | Shows loading during the product request, then renders and clears status after the supplied three-second delay. |
| C | 5 — Error and Retry | PASS | `c.js:17`, `43–51`, `58–68` | Checks HTTP status, clears cards on failure, shows Retry, resets the in-flight guard, and starts a fresh attempt on click. |
| C | 6 — Five-second timeout | PASS | `c.js:16`, `58–63` | The abort signal rejects the pending request; catch shows the required error and Retry. Exact timing and browser support remain unverified. |
| C | 7 — Console | PASS | `c.js:48–64`, particularly `54` and `58–61` | Product and stock operations are awaited within the catch boundary. No explicit console logging appears. |
| C | 8 — Type checking | PASS | `c.js:4–10`, `16`, `31–34`; `jsconfig.json:3–12` | DOM casts and current DOM library definitions should support the code under the supplied non-strict configuration. Requires checker verification. |
| C | Empty | PASS | `c.js:29–37`, `51`, `54–57` | Replaces the catalogue with an empty fragment, shows `No products yet.`, and formats the unchanged zero total as `0 đ`. |

### Smallest necessary correction

**None identified.** KEEP is provisional until browser and type-checking verification.

Preserve the concurrent stock requests at line 54, indexed aggregation, ordered safe rendering, five-second product timeout, and existing loading/error/Retry/empty behaviour. A sequential stock-request rewrite would introduce a brief violation.

## Review limitations and uncertainty

**Confirmed by reading the source**

- A starts concurrent stock requests but writes the total before their completion; it never refreshes that output.
- A does not connect stock callback rejections to its outer catch.
- B supplies `timeout` rather than an abort signal.
- B and C await concurrent stock requests and associate results with products correctly.
- All three use `textContent` for names, prices, and totals, and explicitly check HTTP success.
- None of the version files calls `console.error`.
- UTF-8 rereading confirmed the intended `đ` and `…` characters; initial terminal decoding artifacts are not source defects.

**Predictions requiring browser verification**

- Actual card count, order, literal scarf text, displayed currency, and status transitions.
- Retry recovery, button visibility, duplicate-load guards, and observed timeout timing.
- Network waterfalls confirming concurrent stock requests.
- Actual Console output, particularly A’s behaviour if a stock request is deliberately made to fail.
- Availability of `AbortSignal.timeout` in the target browser. A and C’s timeout PASS predictions assume support.

**Type-checking findings requiring VS Code or `tsc`**

[jsconfig.json](D:/nam3/WEB/IA3/session5-starter/jsconfig.json), lines 3–12, enables ES2022 and DOM libraries, forces module detection, and sets `strict: false` and `checkJs: false`.

Row 8 therefore needs the requested `@ts-check` opt-in or an equivalent checker invocation. I did not add that comment or run a checker. A/C PASS and B FAIL are predictions under a current compatible TypeScript installation. Older library definitions could affect recognition of `AbortSignal.timeout`.

**Potential false positives and scope boundaries**

- A’s latent unhandled stock rejection does **not** establish a row 7 failure with the supplied successful stock fixtures.
- B’s later browser-generated 504 diagnostic should be distinguished from application logging, consistent with README lines 59–63.
- Nullable DOM queries and untyped parameters are not automatically row 8 failures under `strict: false`. The review assumes the page structure promised by the brief; `index.html` and `panel.js` were outside the requested file set.
- The explicit five-second failure scenario concerns `/api/products`. Missing individual stock timeouts are not treated as additional rubric failures.
- No claim is made that the workspace matches a historical starter commit; the review evaluates the supplied files as read.


## 4. Claim verification against README.md — original versions

**Verification date:** 2026-11-02. All 27 original-version row results were confirmed by the student in the conversation. A row 2, B row 6 and B row 8 were also demonstrated with the student's screenshots. The original assistant's predictions above are retained **verbatim** and should not be confused with test observations. Other PASS checks are **student-reported**, not independently observed in the student's browser by the author of this update.

| Version | Row | Assistant prediction | Actual result reported by student | Comparison | Evidence / note |
|---|---|---|---|---|---|
| A | 1 | PASS | PASS — student reports: Under normal mode, eight product cards appeared in API order and the status line was empty. | **CONFIRMED** | Student confirmation; screenshot not required for a PASS row (also visible in `evidence/a-row2.png`) |
| A | 2 | FAIL | Eight `/api/stock/…` requests finished with HTTP 200, but the page still displayed **`Stock value: 0 đ`** instead of `12.360.000 đ`; Network showed overlapping request starts. | **CONFIRMED** | `evidence/a-row2.png` |
| A | 3 | PASS | PASS — student reports: The last name appeared literally as `Scarf <i>limited</i>`; it was not interpreted as italic markup. | **CONFIRMED** | Student confirmation; screenshot not required for a PASS row (also visible in `evidence/a-row2.png`) |
| A | 4 | PASS | PASS — student reports: Slow mode displayed `Loading products…` during the wait, then eight cards after approximately 3 seconds. | **CONFIRMED** | Student confirmation; screenshot not required for a PASS row |
| A | 5 | PASS | PASS — student reports: Error mode displayed `Could not load products.` and Retry; switching to normal and pressing Retry restored eight cards. | **CONFIRMED** | Student confirmation; screenshot not required for a PASS row |
| A | 6 | PASS | PASS — student reports: Hang mode replaced Loading with `Could not load products.` and Retry around the five-second timeout. | **CONFIRMED** | Student confirmation; screenshot not required for a PASS row |
| A | 7 | PASS | PASS — student reports: No uncaught application errors were reported in Console during rows 1–6. | **CONFIRMED** | Student confirmation; screenshot not required for a PASS row |
| A | 8 | PASS | PASS — student reports: With first-line `// @ts-check` in the full VS Code project, no TypeScript diagnostics were reported. | **CONFIRMED** | Student confirmation; screenshot not required for a PASS row |
| A | empty | PASS | PASS — student reports: Empty mode displayed `No products yet.`, zero cards and `0 đ`. | **CONFIRMED** | Student confirmation; screenshot not required for a PASS row |
| B | 1 | PASS | PASS — student reports: Under normal mode, eight product cards appeared in API order and the status line was empty. | **CONFIRMED** | Student confirmation; screenshot not required for a PASS row |
| B | 2 | PASS | PASS — student reports: After the stock requests completed, the displayed stock value was `12.360.000 đ`. | **CONFIRMED** | Student confirmation; screenshot not required for a PASS row |
| B | 3 | PASS | PASS — student reports: The last name appeared literally as `Scarf <i>limited</i>`; it was not interpreted as italic markup. | **CONFIRMED** | Student confirmation; screenshot not required for a PASS row |
| B | 4 | PASS | PASS — student reports: Slow mode displayed `Loading products…` during the wait, then eight cards after approximately 3 seconds. | **CONFIRMED** | Student confirmation; screenshot not required for a PASS row |
| B | 5 | PASS | PASS — student reports: Error mode displayed `Could not load products.` and Retry; switching to normal and pressing Retry restored eight cards. | **CONFIRMED** | Student confirmation; screenshot not required for a PASS row |
| B | 6 | FAIL | With `hang` selected, after approximately seven seconds the page still showed **`Loading products…`**; the `/api/products` request was Pending, with no error or Retry. | **CONFIRMED** | `evidence/b-row6.png` |
| B | 7 | PASS | PASS — student reports: No uncaught application errors were reported in Console during rows 1–6. | **CONFIRMED** | Student confirmation; screenshot not required for a PASS row |
| B | 8 | FAIL | With `// @ts-check` on **line 1**, VS Code Problems reported **TS2353**: `timeout` does not exist in `RequestInit` (line 17 while the temporary comment was present). | **CONFIRMED** | `evidence/b-row8.png` |
| B | empty | PASS | PASS — student reports: Empty mode displayed `No products yet.`, zero cards and `0 đ`. | **CONFIRMED** | Student confirmation; screenshot not required for a PASS row |
| C | 1 | PASS | PASS — student reports: Under normal mode, eight product cards appeared in API order and the status line was empty. | **CONFIRMED** | Student confirmation; screenshot not required for a PASS row |
| C | 2 | PASS | PASS — student reports: After the stock requests completed, the displayed stock value was `12.360.000 đ`. | **CONFIRMED** | Student confirmation; screenshot not required for a PASS row |
| C | 3 | PASS | PASS — student reports: The last name appeared literally as `Scarf <i>limited</i>`; it was not interpreted as italic markup. | **CONFIRMED** | Student confirmation; screenshot not required for a PASS row |
| C | 4 | PASS | PASS — student reports: Slow mode displayed `Loading products…` during the wait, then eight cards after approximately 3 seconds. | **CONFIRMED** | Student confirmation; screenshot not required for a PASS row |
| C | 5 | PASS | PASS — student reports: Error mode displayed `Could not load products.` and Retry; switching to normal and pressing Retry restored eight cards. | **CONFIRMED** | Student confirmation; screenshot not required for a PASS row |
| C | 6 | PASS | PASS — student reports: Hang mode replaced Loading with `Could not load products.` and Retry around the five-second timeout. | **CONFIRMED** | Student confirmation; screenshot not required for a PASS row |
| C | 7 | PASS | PASS — student reports: No uncaught application errors were reported in Console during rows 1–6. | **CONFIRMED** | Student confirmation; screenshot not required for a PASS row |
| C | 8 | PASS | PASS — student reports: With first-line `// @ts-check` in the full VS Code project, no TypeScript diagnostics were reported. | **CONFIRMED** | Student confirmation; screenshot not required for a PASS row |
| C | empty | PASS | PASS — student reports: Empty mode displayed `No products yet.`, zero cards and `0 đ`. | **CONFIRMED** | Student confirmation; screenshot not required for a PASS row |

### Consistency checks

- **27/27 predictions agreed with the student's reported outcomes**: A row 2 FAIL; B rows 6 and 8 FAIL; all other rows PASS.
- **Failures missed by the independent assistant:** None reported in the standard nine-row tests.
- **False-positive failures claimed by the independent assistant:** None identified.
- The original review was **static only**; the claims above were checked later. Keeping the original unchanged is important to show what the assistant actually said before testing.

## 5. Additional claims and constraints

| Review claim | Verification | Status | Evidence / limitation |
|---|---|---|---|
| A's stock requests begin concurrently but total is displayed before awaiting them | Eight stock HTTP 200 responses; page still shows `0 đ`; overlapping Network waterfall | **CONFIRMED** | `evidence/a-row2.png`, `a.js:51–56` |
| A's detached stock callback could reject outside outer `try/catch` | Static reasoning; no failing stock request in standard fixtures | **STATIC RISK — NOT EXERCISED** | `a.js:52–55`; do not call standard row 7 a FAIL on this basis |
| B's unsupported `{ timeout: 5000 }` fails to abort `hang` | Loading remains while products request is Pending after student's reported 7-second interval | **CONFIRMED** | `evidence/b-row6.png`, `b.js:16` |
| B's `timeout` property fails type checking | VS Code diagnostic **TS2353**, `timeout` not in `RequestInit`, with first-line `// @ts-check` | **CONFIRMED** | `evidence/b-row8.png`, original `b.js:16` |
| B and C await stock requests concurrently | Student reports both passed the stock total and concurrency checks; matches `Promise.all` in source | **STUDENT-REPORTED PASS** | `b.js:50–53`, `c.js:53–57`; separate B/C Network screenshots not supplied |
| Version C needs no original-code fix | Student reports every original C test PASS, including `empty` and type checking | **CONFIRMED BY STUDENT REPORT** | C results in `VERDICTS.md`; no C failure screenshot |
| B may leave cards rendered after a stock-only rejection | Not part of the supplied standard HTTP product-error mode | **OUTSIDE TEST SCOPE** | Not counted as a row 5 failure |

## 6. What the assistant got wrong or missed

No incorrect PASS/FAIL prediction was identified in the standard nine-row checks, according to the student's testing. The model's identified failures — A row 2, B rows 6 and 8 — matched the submitted screenshots. The secondary stock-only failure observations were not treated as confirmed rubric failures. Do not invent a rejected claim simply to populate `AI-LOG.md`.

## 7. Final student verdicts — BEFORE repairs

| Version | Assistant's provisional verdict | Student verdict | Deciding rows | Evidence |
|---|---|---|---|---|
| A | FIX | **FIX** | 2 | `evidence/a-row2.png` |
| B | FIX | **FIX** | 6, 8 | `evidence/b-row6.png`, `evidence/b-row8.png` |
| C | KEEP | **KEEP** | All nine PASS (student-reported) | `VERDICTS.md`, original C source |

**Next step:** The original-version review is checked; however, A and B have not yet been repaired in these documents. After each fix, rerun every row and the concurrency check and document regressions, if any. This review does not establish post-fix compliance.
