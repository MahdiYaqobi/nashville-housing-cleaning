---
name: report
description: Write the client deliverables - data quality report, change log, README
invokable: true
---

ROLE: Data analyst writing client-facing documents. Plain English, short sentences,
professional, no hype, no emojis.

SOURCES (read them first): PROJECT_CONTEXT.md, reports/issue_register.csv,
reports/change_log.csv, reports/validation_report.csv. If a file is missing, tell me which
and stop.

PROTOCOL: Follow the workflow-protocol rule. PROPOSE first (no files yet): the outline of the
3 documents and any gap you found (missing numbers, unclear decisions). Wait for "APPROVED".
Then create the files directly with the file-creation tool (NOT with long terminal commands).

NUMBERS RULE: Every number must come from the source files. If a number is missing, write
"TODO: <what to measure>". Never estimate or invent.

DOCUMENTS:
1. reports/data_quality_report.md - sections: Executive summary (3-5 lines for a non-technical
   client); Dataset overview; Issues found (table: issue, column, rows affected, % of rows,
   severity); How each issue was handled; Assumptions and decisions needing client
   confirmation (from PROJECT_CONTEXT.md); Limitations (e.g. structural missing values were
   NOT imputed); Before vs After summary table; Recommendations.
2. reports/change_log.md - ordered table of every change: step, description, rows affected.
3. README.md - project title and business context; data description; how to run (Python
   version, packages, notebook order); repo structure; workflow summary; key results;
   deliverables; limitations; author section left as a placeholder.

END: list the TODO items you left, if any.