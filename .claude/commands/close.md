---
description: Close the session, write the log, update STATE
disable-model-invocation: true
---

Close the current session. Resolve STATE as in /start: the clone whose `origin` ends in `pca-v6.1-state`; never assume `../`. Everything below writes to STATE only. Nothing personal goes into this repo.

## 1. Write the log
- Name: `STATE/log/YYYY-MM-DD-<block>.md`, today's date. If that name exists, use `-2`, `-3`, and so on.
- Fill it from `templates/log.md`: mode, chunks covered, score and who graded it, each misconception as Said / True / Root cause / Retest, parking lot, next step.
- Never edit an existing log file.

## 2. Stage drills
Append every drill written this session to `STATE/staging/<block>.md`, in `templates/drill.md` format. Give each a staging ID `<block>-S###`, one higher than the last ID in that file, starting at S001. Append only; never edit existing entries.

## 3. Rewrite STATE.md
Rewrite `STATE/STATE.md` from `templates/state.md`, max 60 lines:
- Learner: keep, and refine from what this session showed.
- Current: block, "Last session" set to the new log filename, next step.
- Open weak spots: add new ones with root cause and log file; drop ones the learner passed on retest.
- Parking lot: keep open items, drop resolved ones.

## 4. Commit and push
Show `git -C STATE diff` and the new files. Commit on the current branch with the message `close: <block> <date>` and push with `git push -u origin <branch>`.

Remind the learner: merge this branch to main in both repos before the next /start, or /start will stop on a STATE mismatch.
