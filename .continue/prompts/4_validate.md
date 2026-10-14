---
name: validate
description: Final QA - compare raw vs clean, check all expected numbers, produce a PASS/FAIL table
invokable: true
---

ROLE: QA analyst. You verify; you do not clean. READ-ONLY on df_raw and df_clean.

SOURCE: PROJECT_CONTEXT.md (Expected and Validate lines of every step). If you cannot
access it, ask me to attach @PROJECT_CONTEXT.md.

PROTOCOL: Follow the workflow-protocol rule: PROPOSE the list of checks (no code), wait for
"APPROVED", write the code in 2 small batches, STOP after each.

CHECKS:
1. Raw integrity: reload the raw file into a temporary variable and confirm
   df_raw.equals(temp). Shape of df_raw still (56477, 19).
2. Row accounting: len(df_raw) - rows removed in change_log == len(df_clean).
3. Column accounting: list columns added/removed vs raw; originals still present.
4. Nulls: per column, raw vs clean, flag every difference and compare with the expected values.
5. Dtypes: raw vs clean, flag every difference.
6. Categoricals: SoldAsVacant values == {'Yes','No'}; LandUse distinct count == 37.
7. Text hygiene: zero repeated spaces, leading/trailing spaces or '_x000D_' in
   PropertyAddress, OwnerAddress, LandUse.
8. Duplicates: zero on the business key; UniqueID 11912 present, 12629 absent.
9. Flags: boolean dtype, row count unchanged.
10. Spot check: for each changed column, show 5 random changed rows side by side
    (raw vs clean, joined on UniqueID) so I can eyeball them.
11. Every other Expected number in PROJECT_CONTEXT.md.

OUTPUT: DataFrame `validation_report` with columns: check, expected, actual, status (PASS/FAIL).
Do NOT use assert here (I want to see ALL failures at once). Save
reports/validation_report.csv. End with one summary line: "X PASS / Y FAIL".
If anything FAILS, propose the cause as a hypothesis; do not fix it yourself.

Keep terminal commands short; follow my notebook rule (.py first, then nbformat).