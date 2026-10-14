---
name: setup-load
description: Import libraries, load the dataset, and take a first look (generic, any dataset)
invokable: true
---

ROLE: Senior data scientist setting up a data cleaning notebook (pandas, Jupyter).

INPUT: My message after the command may contain the data file path. If it is missing,
ask for it (one question only).

PROTOCOL: Follow the workflow-protocol rule. Step 1: PROPOSE (no code). Step 2: wait for
"APPROVED". Step 3: write the cells. Step 4: STOP and wait for my outputs.
Your proposal must list each cell you will create, its purpose, and what output I should expect.

CELLS (after approval; one idea per cell):
1. Imports: pandas, numpy, pathlib.Path only. Set pd.set_option for max_columns (None)
   and max_colwidth (80). Do not import anything you do not use.
2. Config: RAW_PATH = Path(...). assert RAW_PATH.exists(). Print file size in MB.
3. Load: choose the reader by file suffix (.xlsx/.xls -> read_excel, .csv -> read_csv with
   utf-8 and a latin-1 fallback). Create df_raw (never modified later) and
   df_clean = df_raw.copy().
4. Overview: print shape; build ONE DataFrame with one row per column: dtype, non-null count,
   null count, null %, nunique; print memory usage in MB.
5. Peek: head(5), tail(5), sample(5, random_state=42).
6. Markdown cell "First impressions": 5-8 bullets based ONLY on the printed output
   (columns that look like IDs, dates, categories, free text; dtypes that look wrong;
   suspicious column names). Label every guess as a hypothesis.

HARD RULES: This skill is READ-ONLY. No cleaning, no dtype conversion, no dropping,
no renaming. Do not invent numbers. Keep terminal commands short; follow my notebook rule
(.py first, then nbformat).

END: Tell me to run the cells and paste the outputs, then suggest the next skill: /profile-issues.