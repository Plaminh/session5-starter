# CSC13008 IA#3 — Self-Assessment Report

## 1. Submission Information

- **Student name:** Phan Lê Anh Minh.
- **Student ID (MSSV):** 24127082.
- **Planned submission archive:** `24127082_98.zip` (after final review and commit).

- **Assignment:** CSC13008 Individual Assignment #3 — Three Answers to One Brief.

- **Submission preparation date:** 2026-11-02. No submission has been made.

- **Git branch:** `master`.

- **Audited implementation/documentation commit:** `4ec2fd5a559080d16bdea80d95441664f9d4fb2c` (`docs: complete CSR vs SSR analysis`).

- **Final submission commit:** Pending student review and commit of this report. The audited commit above does not contain this report.

- **Project:** Three browser catalogue implementations evaluated against one brief, with independent predictions, original failure evidence, separate targeted fixes to A and B, regression verification, and a CSR/SSR comparison.

- **Assessment basis:** The official IA#3 rubric and `Requirement.txt` specify the six criteria and a required evidence-based self-assessment. The official course report template was not accessible in the audited project workspace. The claimed score is provisional, not an instructor-awarded grade.

## 2. Rubric-Based Self-Assessment

| Criterion | Max | Claimed | Evidence | Justification |
|---|---:|---:|---|---|
| Verdicts and evidence | 30 | 30 | [VERDICTS.md](VERDICTS.md), original rows 1–8 and empty for A/B/C; [A row 2](evidence/a-row2.png), [B row 6](evidence/b-row6.png), [B row 8](evidence/b-row8.png); commits `84c852a`, `d30aa83` | All 27 original results have observations, source references, row decisions and preservation/fix instructions. Predictions precede the recorded tests. Historical A: FIX (2), B: FIX (6, 8), C: KEEP remain intact. Original passing results are identified as student-reported; failure screenshots were inspected. |
| Final correct behavior | 25 | 25 | Final A/B/C verification table below; `public/versions/a.js:51–53`, `b.js:16,51–53`, `c.js:16,54–57`; fixes `7028acc`, `d8d4b35` | All 27 final README checks and three separate concurrency checks passed in the audited environment. A awaits all stock results; B uses a valid five-second abort signal; C is unchanged. No tested application regression was found. |
| Assistant review | 15 | 15 | [ASSISTANT-REVIEW.md](ASSISTANT-REVIEW.md), sections 1–7; first review commit `6bd778c` | Recorded reviewer is ChatGPT (GPT Astra High). The supplied original prompt is retained, and the original response matches its first committed text. All 27 predictions are compared with original results; the three predicted failures match the recorded evidence. Static risks are not presented as tested failures. |
| CSR vs SSR | 10 | 10 | [CSR-SSR.md](CSR-SSR.md), questions 1–5 and limitations; commit `4ec2fd5`; `server.mjs:67–107,188–190`, `public/versions/c.js:33,54–57` | All five questions are answered using recorded browser/HTTP observations: source, escaping, disabled JavaScript, request counts and content availability, and a one-sentence choice. Local measurements are explicitly distinguished from general performance or crawler guarantees. |
| AI-LOG | 10 | 8 | [AI-LOG.md](AI-LOG.md), independent review, original testing, A fix/validation, B fix/validation, CSR/SSR entries | Required activities and Asked for / Kept / Changed / Rejected / By hand fields are present; manual student work is distinguished from Codex execution. Deduct 2 points because the later Codex entries identify the tool and test software but omit the exact assistant model. Historical model names cannot be inferred from browser or TypeScript versions. |
| Scope and commits | 10 | 10 | Git history table below; `git diff 8c9f2bb HEAD`; separate fixes `7028acc`, `d8d4b35`; protected-file blob comparisons | The required sequence is present. A's code commit changes only A, B's changes only B, C matches the starter, and all protected starter files match the root commit. No Git history was changed by this audit. |
| **Total** | **100** | **98** | Evidence above | **Provisional self-assessment: 98/100.** |

The two-point deduction is a conservative assessment of incomplete model attribution, not an invented official subcriterion. No deduction is invented for absent PASS screenshots: the provided instructions require screenshots for failures, and the original PASS provenance is disclosed.

## 3. Work Completed

### Original predictions, testing, and independent review

- All 27 original predictions and all 27 original result entries are present. Every result includes a source reference, observed behavior and an actionable preservation or correction instruction.

- The original prediction section matches its prediction commit, and the independent prompt/response text matches the first review commit. The prompt also matches the supplied original prompt, allowing for line-ending normalization and its Markdown fence.

- The review's 27 predictions agree with the student's recorded original outcomes: only A row 2 and B rows 6 and 8 failed. No incorrect standard-row prediction or missed standard-row failure is evidenced; none was invented.

- The original review explicitly separates static analysis from later student testing. Stock-only rejection risks remain marked as unexercised/outside the supplied standard scenarios.

- All three original PNG files are nonempty, valid images and were visually inspected:

  - `evidence/a-row2.png`: A displays `0 đ` despite eight completed HTTP 200 stock calls and overlapping request starts.

  - `evidence/b-row6.png`: B remains loading with the products request pending and no Retry after the expected timeout interval. The approximately seven-second observation is student-reported; the screenshot is not a precise stopwatch measurement.

  - `evidence/b-row8.png`: VS Code shows TS2353 for unsupported `RequestInit.timeout`, at line 17 with the temporary first-line checking comment (original source line 16).

- Historical findings and original evidence were not rewritten during this audit.

### Targeted fixes

- **A:** Commit `7028acc` replaces detached `forEach(async ...)` aggregation with awaited `Promise.all(products.map(...))`, then a reduction of matching prices and stock values. All requests still start concurrently; the total is written after completion. The initial reduction value preserves the empty total. Stock rejections now propagate into the surrounding error handler.

- **B:** Commit `d8d4b35` replaces unsupported `{ timeout: 5000 }` with `{ signal: AbortSignal.timeout(5000) }`. Existing concurrent stock aggregation is unchanged.

- **C:** No implementation change. Its Git blob equals the root starter's blob.

### Final regression verification executed on 2026-11-02

Codex executed the following checks against the existing server at `http://localhost:5050`, visiting `/?v=a`, `/?v=b`, and `/?v=c` sequentially. Installed Microsoft Edge **156.0.4314.58** ran headlessly through Chrome DevTools Protocol **1.3**, with extensions and cache disabled. Tests clicked real mode buttons, reloaded, and clicked Retry; endpoints and DOM were not mocked. Served A/B/C source matched the workspace.

| README row / additional check | A | B | C | Actual observation |
|---|---|---|---|---|
| 1. Normal cards/order/prices | PASS | PASS | PASS | Exactly eight cards match live API order, IDs, names and Vietnamese-formatted prices; final status empty. |
| 2. Stock total | PASS | PASS | PASS | Eight stock requests return HTTP 200 and the displayed total becomes `12.360.000 đ`. |
| 3. Literal scarf text | PASS | PASS | PASS | `Scarf \<i>limited\</i>` appears as literal text; zero italic descendants; angle brackets escaped in DOM markup. |
| 4. Slow loading | PASS | PASS | PASS | Loading appears with no cards, then eight cards appear after the approximately three-second response and status clears. |
| 5. HTTP 500 and Retry | PASS | PASS | PASS | Real HTTP 500 shows the required error and Retry with no cards; selecting normal and clicking Retry restores cards, total and cleared status. |
| 6. Five-second hang timeout | PASS | PASS | PASS | Pending products request is canceled near five seconds; error and Retry replace loading, with no cards. |
| 7. Application Console errors | PASS | PASS | PASS | Zero runtime exceptions/unhandled rejections and zero application `console.error` calls across the tested scenarios. |
| 8. JavaScript type checking | PASS | PASS | PASS | Installed TypeScript 6.0.3 reports zero diagnostics with existing project options and JavaScript checking enabled. |
| Additional empty case | PASS | PASS | PASS | Zero cards, `No products yet.`, `0 đ`, Retry hidden and no stock requests. |
| Separate concurrency check | PASS | PASS | PASS | All eight stock requests start before the first response; the total updates after the last response completes. |

This is **30/30 checks**: nine browser checks and one type check per version. Row 8 used VS Code's installed TypeScript API with the existing `jsconfig.json` options plus `allowJs: true`, `checkJs: true`, and `noEmit: true`; it did not involve manually editing `// @ts-check` or inspecting editor underlines.

Measured timings from these final runs:

| Measurement | A | B | C |
|---|---:|---:|---:|
| Slow products response (ms) | 3009.276 | 3036.703 | 3038.600 |
| Hang request cancellation (ms) | 5006.779 | 5009.511 | 5008.058 |
| Spread between first and last stock request starts (ms) | 3.0 | 1.2 | 3.3 |
| First stock response after first start (ms) | 202.2 | 183.9 | 270.6 |
| Last stock completion after first start (ms) | 634.0 | 557.7 | 927.4 |
| Total update after last stock completion (ms) | 13.8 | 5.0 | 4.4 |

The concurrency timings come from a separate normal-mode reload for each version. Normal row-2 total insertion occurred at approximately 882.1 ms (A), 1121.9 ms (B), and 966.1 ms (C) after navigation time origin; these are local observations, not a strict one-second service guarantee.

Browser-generated products HTTP 500 and missing `favicon.ico` HTTP 404 messages were observed separately from application errors. The favicon is absent from the untouched starter; no source change was made to suppress it. No new application failures were found, so no new failure screenshots were created.

A preliminary browser connection attempt occurred before Edge's debugging endpoint was ready and returned no test results. After Edge became available, the complete A run and subsequent B/C runs succeeded. That setup failure is not counted as an application result.

Temporary audit scripts and the browser profile were kept outside the project. No dependencies were installed. The temporary browser was closed, and the existing server was left in normal mode.

### CSR/SSR investigation

The existing Phase 5 record answers all five README questions and is consistent with the inspected implementation:

- CSR source contains an empty catalogue and total placeholder; SSR source contains eight complete cards and the calculated total.

- SSR escapes names through `escapeHtml`; CSR assigns names with `textContent`. Both preserve literal scarf text.

- With browser JavaScript disabled, CSR retains the shell but no product cards; SSR retains cards, prices and total.

- Recorded Phase 5 measurements show 14 CSR requests versus 2 SSR requests in the defined observation windows. The document explains the nine products/stock calls, mode/script requests, favicon variability and timing cutoff.

- The one-sentence recommendation is present. Source references are accurate, and local timing results are not presented as universal performance guarantees.

The CSR/SSR browser measurements were executed in Phase 5 and reviewed here; they were not rerun as a new Phase 6 experiment.

### Git history and scope audit

| Order | Commit | Recorded purpose | Scope verification |
|---|---|---|---|
| 1 | `8c9f2bb` | Starter | Root commit contains the original starter files. |
| 2 | `84c852a` | Predictions before testing | Adds original predictions in VERDICTS. |
| 3 | `6bd778c` | Independent assistant review | Records review and AI log before original results. |
| 4 | `d30aa83` | Original tests and evidence | Records results, verification and the three failure screenshots. |
| 5 | `7028acc` | A targeted fix | Changes only `public/versions/a.js`. |
| 6 | `98a8e83` | A verification documentation | Records A post-fix checks. |
| 7 | `d8d4b35` | B targeted fix | Changes only `public/versions/b.js`. |
| 8 | `2884a41` | B verification documentation | Records B post-fix checks. |
| 9 | `4ec2fd5` | CSR/SSR documentation | Records comparison and AI-log entry. |

Git confirms predictions precede recorded test results and each repaired version has a separate code-only fix commit. This establishes the recorded sequence, not independent proof of when every off-repository manual action occurred.

Only A and B differ among starter implementation files. Git blob comparisons confirm that C, `README.md`, `brief.md`, `server.mjs`, `jsconfig.json`, `public/index.html`, `public/panel.js`, and `public/style.css` match the root commit. No separate external starter archive/hash was available to authenticate the root commit beyond this repository.

### Files, encoding, and non-destructive checks

- All required root documentation/source files and `evidence/`, `public/`, and hidden `.git/` are present. Required files are nonempty.

- Original screenshot sizes: A row 2 **144,301 bytes**, B row 6 **101,359 bytes**, B row 8 **126,731 bytes**.

- Markdown files decode as UTF-8; inspected content contains no replacement characters or the specified mojibake patterns. Terminal display artifacts were not treated as file corruption.

- `node --check` passed for A, B, C and `server.mjs`.

- The installed TypeScript check passed for all three versions with zero diagnostics.

- `git diff --check` passed. The initial working tree and index were clean.

- `git fsck --full` exited successfully; it reported unreachable dangling objects, not repository corruption. No cleanup or history modification was performed.

- This audit creates only `SELF_ASSESSMENT_REPORT.md` in the project. Existing tracked files, evidence, index and HEAD remain unchanged.

## 4. What I Did Not Manage

- **Historical Codex model metadata is incomplete.** The later AI-LOG entries need confirmed model names if available; none was invented. This accounts for the two-point deduction.

- **Official report template was unavailable in the audited repository.** The six-category scoring framework was checked against the provided official IA#3 rubric and `Requirement.txt`.

- **Original PASS provenance is limited.** Original passing tests remain student-reported. This audit inspected original failure evidence and reran final implementations, without checking out and retesting the original broken versions.

- **Runtime coverage is bounded.** Final testing used one installed Edge version and local server conditions. It did not establish cross-browser support, real slow-phone performance, crawler indexing, or behavior under injected stock-only failures. Awaited stock errors entering the catch flow are supported by source inspection; no stock-failure injection is claimed.

- **CSR/SSR timings are a local sample.** The recorded Phase 5 observations are not a benchmark or universal speed guarantee.

- **Submission packaging remains pending.** This new report has not been reviewed or committed. No final ZIP was created and no assignment was submitted.

No failed final standard regression row remains unresolved in the tested environment. The items above distinguish documentation/packaging work and evidence limits from implementation failures.

## 5. Final Submission Checklist

- [x] `VERDICTS.md` preserves predictions, all 27 original results, original verdicts and post-fix verification.

- [x] `ASSISTANT-REVIEW.md` retains the original prompt/response and later claim checks.

- [x] `CSR-SSR.md` answers all five README questions with observations and limitations.

- [x] `AI-LOG.md` records the required phases and distinguishes student work from automated testing.

- [ ] Confirm the missing historical Codex model names if available; do not invent them.

- [x] `README.md`, `brief.md`, `server.mjs`, and `jsconfig.json` are present and unchanged.

- [x] `evidence/` contains the three correctly named, inspected original failure PNGs.

- [x] `public/` and full hidden `.git/` are present.

- [x] Required historical commit sequence and separate A/B fix commits are present.

- [x] C and all protected starter files match the root starter.

- [x] Final A/B/C README and concurrency checks passed; syntax/type checks completed.

- [x] `SELF_ASSESSMENT_REPORT.md` has been prepared for student review.

- [x] Student name and MSSV confirmed: Phan Lê Anh Minh — 24127082.

- [ ] Student reviews the report, resolves any factual corrections and commits the final documentation.

- [ ] Confirm the intended final commit and a clean working tree before packaging.

- [ ] Create `24127082_98.zip` only after the preceding steps (adjust the score suffix if the reviewed score changes).

- [ ] Validate one top-level `session5-starter/` directory containing the entire current submission, including full hidden `.git/`, all evidence and documentation, without including the ZIP itself.

Use a ZIP implementation that explicitly includes hidden files; do not rely on Windows Compress-Archive. Packaging must use this repository, not an older downloaded copy. No ZIP contents can be claimed validated before an archive exists.

**Review status:** Student identity is complete. The report is ready for final review and a student-created Git commit. **Packaging status:** Pending: this report must be added to the actual repository, reviewed, and committed before the complete ZIP (including `.git/`) is created and validated. No Git write command or submission action was performed by this edit.
