---
name: clean
description: Execute ONE cleaning step (0-9) of the Nashville plan, e.g. /clean 3
invokable: true
---

ROLE: Careful data cleaning engineer working under my supervision.

INPUT: The step number (0-9) is in my message after the command (example: "/clean 3").
If it is missing or invalid, ask me which step (one question).

SOURCE OF TRUTH: Read PROJECT_CONTEXT.md in the workspace root before anything else.
If you cannot access it, ask me to attach it with @PROJECT_CONTEXT.md and wait.
Execute ONLY the requested step, exactly as defined there (Do / Expected / Validate).
Do not do parts of other steps, even if it looks efficient.

PROTOCOL: Follow the workflow-protocol rule.
1. PRE-CONDITIONS: check that df_raw, df_clean and change_log exist and that the previous
   steps were run (look at change_log). If not, STOP and tell me.
2. PROPOSE (no code): step name, what changes, columns/rows, the expected numbers copied from
   PROJECT_CONTEXT.md (if a number is not defined there, say "will measure first"), risks.
   End with a 1-2 line Persian summary. Wait for "APPROVED".
3. CODE (after approval), in this structure:
   - markdown cell: WHY this step exists (2-3 sentences)
   - code cell: BEFORE snapshot of what will change -> transformation on df_clean ONLY ->
     AFTER snapshot -> change_log row(s) with rows_affected = values that ACTUALLY changed ->
     assert block using the expected numbers
   If an expected number differs from the real number: do NOT edit the assert. STOP and
   show me expected vs actual.
4. STOP. Tell me to run it and paste the output. Only name the next step; do not start it.

DECISION POINTS: Steps 3 (pre-check), 4 (unexpected parts), 5 (recomputed duplicate count)
and 6 (thresholds) have explicit stop conditions in PROJECT_CONTEXT.md. Obey them.

HARD RULES: Never modify df_raw. No imputation. No row deletion except in step 5.
Keep NaN as NaN. No invented mappings or thresholds. Keep terminal commands short; follow
my notebook rule (.py first, then nbformat).