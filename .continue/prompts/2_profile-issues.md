---
name: profile-issues
description: Inspect all columns and find every data quality issue; builds an issue register (read-only)
invokable: true
---

ROLE: Senior data scientist doing a data quality assessment before any cleaning.

PROTOCOL: Follow the workflow-protocol rule. First PROPOSE (no code): list which checks below
apply to THIS dataset (look at df_raw dtypes and column names) and which you will skip and why.
Wait for "APPROVED". Then deliver the code in 3 BATCHES, one small cell per check, and STOP
after each batch so I can run it and paste the output:
 Batch 1 = A, B, C   Batch 2 = D, E, F   Batch 3 = G, H, I, J

CHECKS (all on df_raw, read-only):
A. Missing values: count and % per column. Find columns that are missing in exactly the same
   rows (group columns with identical null masks) and report the number of rows where the
   whole group is missing vs only some of it.
B. Duplicates: fully identical rows; repeated ID-like columns; candidate business keys
   (PROPOSE the key columns, do not decide); groups that are identical except the ID column;
   groups that differ in other columns (show them transposed).
C. Text hygiene for every text column: leading/trailing spaces, repeated spaces, control
   characters and Excel artifacts (\r, \n, \t, literal '_x000D_'), mixed case, non-ASCII.
D. Categorical consistency: value_counts for columns with <= 60 distinct values. Find
   near-duplicate labels with a normalization key (lowercase, strip, collapse spaces, remove
   punctuation) and difflib similarity >= 0.85. List suspicious pairs with counts. Do NOT merge.
E. Boolean-like columns with mixed encodings (Y/N/Yes/No/1/0/True/False).
F. Numeric sanity: describe(); zeros; negatives; IQR outlier counts; max/median ratio;
   values impossible for the meaning of the column (e.g. 0 bedrooms, year in the future).
G. Dates: dtype, parseability, min/max, future dates, unexpected time components.
H. Cross-column consistency: parts vs total (A + B vs C), start <= end, city/state vs address.
   PROPOSE candidates first; report mismatch counts; do not assume a mismatch is an error.
I. ID sanity: gaps, range vs row count, non-numeric IDs.
J. Issue register: build DataFrame `issues` with columns: id, columns, issue_type,
   evidence, rows_affected, pct_rows, severity (High/Medium/Low), proposed_action,
   needs_client_decision (True/False). Fill it ONLY from numbers printed in A-I.
   Save reports/issue_register.csv. Then a markdown cell: the top 5 issues in plain English.

HARD RULES: READ-ONLY (no changes to any data). Every claim needs a number from the output.
Label hypotheses as hypotheses. Keep every cell small and every terminal command short;
follow my notebook rule (.py first, then nbformat).