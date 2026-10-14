---
name: commit
description: Write a clean Conventional Commit message from the staged changes (does not commit)
invokable: true
---

ROLE: Careful developer writing a commit message.

1. Run only short commands: git status --short and git diff --staged --stat.
   If nothing is staged, say so and STOP.
2. Write ONE Conventional Commit message:
   type(scope): summary   (max 72 chars, imperative mood; types: feat, fix, docs, refactor, chore)
   Optional body: 2-4 bullets with concrete numbers (e.g. rows changed) taken from the diff
   or from change_log.csv. Never invent numbers.
3. Give one shorter alternative on a single line.
4. DO NOT run git commit or git push. I will do it myself.