# AI-LOG.md — CSC13008 IA#3

## 2026-11-02 — Independent Code Review

**Tool:** ChatGPT (GPT Astra High)

**Asked for:** Independently review the original versions A, B, and C against `brief.md` and `README.md` without reading `VERDICTS.md`, running tests, or modifying files. Identify potential failures, responsible code lines, and recommend keep/fix/reject verdicts.

**Kept:**
- **Version A — FIX:** Asynchronous `forEach` does not await stock responses before displaying the total (row 2).
- **Version B — FIX:** Unsupported `fetch` timeout option may cause failures in rows 6 and 8.
- **Version C — KEEP:** No defects identified through static inspection.
- Preserve concurrent stock requests across all versions.

**Changed:** No source code was modified. The review findings were documented in `ASSISTANT-REVIEW.md` for subsequent verification.

**Rejected:** None at this stage. No assistant recommendations have been rejected because the findings are pending runtime verification.

**By hand:** Browser testing, type-checking, screenshots, and verification of the assistant's claims are pending.

## 2026-11-02 — Original-Version Testing and Claim Verification

**Tool:** ChatGPT (GPT-6) for organizing results and updating documentation; the independent reviewer was ChatGPT (GPT Astra High).

**Asked for:** Update `VERDICTS.md`, `ASSISTANT-REVIEW.md`, and `AI-LOG.md` based on the original-version manual test results, keep the original predictions and independent response unchanged, and link each confirmed failure to screenshot evidence.

**Kept:** The student's reported outcomes match all 27 review predictions: Version A failed row 2; Version B failed rows 6 and 8; Version C and all other predicted-PASS rows passed. The recommendation remains **A — FIX, B — FIX, C — KEEP**. Version A's Network screenshot shows stock requests beginning concurrently; B and C concurrency were also reported as passing.

**Changed:** Completed the post-testing verdict and claim-verification tables and organized the submitted failure screenshots as `evidence/a-row2.png`, `evidence/b-row6.png`, and `evidence/b-row8.png`. The original predictions and independent assistant response were left intact. The B row 8 evidence was retaken with `// @ts-check` on line 1, as the README requires. No JavaScript implementation fix was made in this phase.

**Rejected:** No incorrect claim from the original GPT Astra High review was identified in the standard rubric checks. The original recommendations remain subject to regression checks after fixes; no rejection has been invented.

**By hand:** The student reports running the required original-version cases and confirming the predicted-PASS rows. The student supplied browser Network screenshots for A row 2 and B row 6, plus a VS Code Problems screenshot for B row 8 (`TS2353`). Other passing results are student-reported; separate screenshots are not required for PASS rows. Ensure the temporary `// @ts-check` edit is reverted before committing. Final post-fix regression testing has not yet occurred.

## 2026-11-02 — Phase 4 — Version A Targeted Fix

**Tool:** OpenAI Codex (VS Code)

**Asked for:** Fix Version A's confirmed README row 2 failure: all eight stock requests returned HTTP 200 concurrently, but the displayed total remained `0 đ` instead of `12.360.000 đ`. Exact excerpts from the student's fix request:

> TASK: Fix Version A only.
>
> Replace the incorrect asynchronous forEach
> aggregation with a properly awaited approach
> using Promise.all and products.map.
>
> Ensure all stock requests start concurrently,
> not sequentially.

The request also required calculating the sum after responses arrive, preserving Vietnamese formatting and the empty result, handling stock failures through the existing catch, preserving all passing behaviour, and making no other file or Git history changes.

**Kept:** Accepted `await Promise.all(products.map(...))` because it starts every stock request without waiting for earlier requests and awaits the combined results before calculating the total. `Promise.all` preserves input order, so each product's price matches its stock result even when responses finish out of order. The reduction starts at zero, preserving `0 đ` for an empty array. Awaited stock failures now reach the existing error/Retry flow. Sequential awaits were avoided because they would violate the brief's explicit parallel-request requirement.

**Changed:** The implementation replaced only the detached asynchronous aggregation in `public/versions/a.js`; current lines 51–53 await stock, sum `price * stock`, and display the existing formatted total. Existing commit `7028acc` (`fix(a): await concurrent stock requests`) confirms that the fix changed only this file. Product rendering, textContent assignments, loading states, product HTTP checks, Retry, and the five-second product timeout were not edited. This documentation entry does not create or alter that commit.

**Rejected:** None. No additional assistant suggestion was rejected; sequential requests were excluded by the task constraints.

**By hand:** The student supplied the original-version results recorded above, including A's row 2 failure and concurrent stock requests. No student post-fix browser, Console, Network, or VS Code results have been supplied for updated A; all ten post-fix checks remain PENDING VERIFICATION in `VERDICTS.md`. No new screenshots or manual outcomes are claimed.

**Assistant checks already executed during the fix:** `node --check public/versions/a.js` and `git diff --check` passed. An in-memory Node harness with mocked fetch and DOM passed concurrent-start, out-of-order completion, correct-total, card order/text/prices, empty-array, stock HTTP/network failure, and Retry checks. These were not browser tests or student manual results. No type checker ran because `tsc` was unavailable on PATH. The syntax and mocked application checks were not rerun during this documentation update.

### Post-fix execution update — 2026-11-02

This follow-up supersedes the pending validation status recorded above; the earlier request, implementation account, and preliminary checks remain historical records.

**Tool:** OpenAI Codex (VS Code), installed Microsoft Edge `156.0.4314.58` controlled through the Chrome DevTools Protocol, and VS Code's bundled TypeScript `6.0.3` API. No dependency was installed.

**Asked for:** Execute all ten updated-Version-A regression checks against the actual application, record real observations and timings, update only the permitted documentation/evidence, and preserve source files, original findings, and the Git index/history.

**Kept:** All ten checks passed in the final execution: eight ordered cards with correct names/prices; `12.360.000 đ`; literal scarf markup; slow loading; HTTP 500/Retry recovery; five-second abort; no application exceptions/error logs; zero type diagnostics; empty `0 đ`; and overlapping stock requests. Slow response: 3008.219 ms. Timeout abort: 5000.751 ms. Stock starts spanned 1.8 ms, before the first response at 226.6 ms; the total updated 2.2 ms after the last response completed. Full observations and individual stock timings are in the existing Phase 4 section of `VERDICTS.md`.

**Changed:** Updated the existing regression checklist from pending to executed PASS results and updated Version A to PASS / VERIFIED. Reused the running server on port 5050 after confirming served A matched the workspace. Browser tests used real mode-button/Retry clicks, DOM observations, and network instrumentation, without mocked responses. The installed TypeScript API used project options with allowJs/checkJs/noEmit for A only, without adding a source comment. No implementation file was changed. The already staged documentation and Git index were left intact; these follow-up edits are unstaged.

**Rejected:** No application fix was proposed or accepted during testing. Preliminary failures caused by sampling the old document before reload, browser-extension `getAllPromptList` errors, and an initial checker configuration-path error were rejected as evidence of application defects after identifying their causes. Reload synchronization was corrected, extensions were disabled, and the configuration path was normalized before final verification. Two pilot diagnostic screenshots were discarded; no original evidence was altered. Browser-generated products HTTP 500 and favicon HTTP 404 messages were distinguished from application errors.

**By hand:** No new student manual results were supplied, and none are claimed. Codex executed all nine browser checks and the type check; none of the ten requested checks remain unexecuted. No confirmed application failures or regressions were found. The temporary browser was closed; the existing server remains in normal mode. Version A is ready to finalize in the tested environment. Version B still needs its targeted fixes, so Phase 4 is not complete.

## 2026-11-02 — Phase 4 — Version B Targeted Fix

**Tool:** OpenAI Codex (VS Code). Browser verification used installed Microsoft Edge **156.0.4314.58** via Chrome DevTools Protocol **1.3**; type checking used VS Code's bundled TypeScript **6.0.3** API. Syntax checking used Node **v24.11.1**. No dependency was installed.

**Asked for:** Fix B's confirmed row 6 timeout and row 8 TS2353 failures by replacing the unsupported fetch timeout option with a valid five-second abort signal. Preserve concurrent stock fetching and all existing passing behaviour, execute all ten real post-fix checks, and append results without changing original findings, A's verification, other implementation files, or Git history.

**Kept:** Existing eight-card rendering and API order, product IDs and prices, textContent-based safe names, literal scarf text, `Promise.all` stock aggregation and indexed product/stock association, correct `12.360.000 đ` total, loading/error/Retry states, and empty `0 đ`. Real browser checks verified these behaviours and overlapping stock requests.

**Changed:** Only the fetch options at `public/versions/b.js:16`: `{ timeout: 5000 }` became `{ signal: AbortSignal.timeout(5000) }`, matching A/C conventions. Added the Version B subsection under `VERDICTS.md`'s existing Phase 4 section and this log entry. No other source changed, and no screenshots were created because no new failure was found. Existing evidence and A's documentation were preserved; no commit was created.

**Rejected:** The unsupported timeout property was removed. Sequential stock fetching, unnecessary fallback mechanisms, and unrelated refactoring were excluded by the request; no such implementation was introduced, and no additional rejected suggestion is invented.

**By hand:** No new student manual testing was reported or claimed. Codex executed nine browser checks against the actual server at `/?v=b` plus the existing TypeScript check: **10/10 PASS**, zero type diagnostics, no pending checks, and no application regressions. Slow response: **3016.536 ms**; product abort: **5005.099 ms**. All eight stock starts spanned **0.4 ms**, before the first response at **150.8 ms**; the total updated **2.2 ms** after the last completion. Browser exceptions/unhandled rejections and application console.error counts were zero; HTTP 500 and favicon 404 network diagnostics were distinguished. `node --check public/versions/b.js` and `git diff --check` passed. The running server was reused after confirming served B matched the workspace, and tests used real UI clicks and endpoints without mocks. The temporary browser was closed; the server remains in normal mode. Updated B is PASS / VERIFIED in this environment, ready for student review and separate commits. This completes the recorded Phase 4 fix/verification work; no Phase 5 work was started.

## 2026-11-02 — Phase 5 — CSR vs SSR Analysis

**Tool:** OpenAI Codex (VS Code), installed Microsoft Edge **156.0.4314.58** via Chrome DevTools Protocol **1.3**, Node **v24.11.1**, direct HTTP response inspection, and read-only source/Git inspection. No dependency was installed.

**Asked for:** Compare unchanged Version C at `/?v=c` with `/ssr`, execute the five README investigations, create `CSR-SSR.md`, and append this log without modifying application source, verdicts, existing evidence, or Git history.

**Kept:** Actual source views showed an empty CSR catalogue and `—` total, versus eight pre-rendered SSR cards and `12.360.000 đ`. SSR escaping and CSR textContent both preserved literal scarf text with no italic element. With page JavaScript disabled and pages reloaded, CSR retained only its shell/placeholder while SSR retained all product data. The CSR mode button did not change the server mode. With JavaScript restored and cache disabled, measured navigations issued **14 CSR requests versus 2 SSR requests**; CSR included two scripts, `/__mode`, one products request and eight stock requests, while SSR needed only its document and CSS. Product DOM availability was **130.9 ms CSR versus 22.8 ms SSR**; total DOM availability was **982.5 ms versus 22.8 ms**.

**Changed:** Created `CSR-SSR.md` with all five original questions, concrete answers, source excerpts, JavaScript-disabled observations, request counts/categories, timing definitions, implementation references, and limitations. Appended this entry only; all prior log entries and application files remain intact.

**Rejected:** Corrected potential assumptions rather than inventing rejected suggestions: CSR's empty template is not a rendered product; JavaScript-disabled CSR is not an API failure; live DOM differs from View Source; a fixed favicon count cannot be assumed; and crawler JavaScript capabilities vary. No favicon request occurred in the measured enabled-JavaScript windows, although favicon requests occurred in the separate disabled-JavaScript tests. Frame/layout observations were not represented as exact pixel-presentation timings. No universal SSR speed or SEO guarantee was claimed.

**By hand:** Codex executed the browser tests and measurements; no new student manual testing is claimed. All five requested investigations are complete, with no required manual check pending. Findings are limited to one local unthrottled sample per page; real slow-phone performance, crawler indexing, other browsers, and exact display latency were not measured. No screenshots were fabricated or created. JavaScript was restored, the temporary browser closed, and the existing server left in normal mode. Phase 5 documentation is complete; no Phase 6 work or Git write operation was performed.
