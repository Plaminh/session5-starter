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
