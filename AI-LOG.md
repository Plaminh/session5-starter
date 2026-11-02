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
