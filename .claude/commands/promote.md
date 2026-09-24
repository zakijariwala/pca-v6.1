---
description: Promote staged drills into the public drills/ file
argument-hint: <block ID, e.g. B03>
disable-model-invocation: true
---

Promote drills for block `$ARGUMENTS`. If no block ID was given, ask for one and stop.

## 1. Pick
Resolve STATE as in /start: the clone whose `origin` ends in `pca-v6.1-state`; never assume `../`. List the drills in `STATE/staging/$ARGUMENTS.md` that have no `promoted:` line: staging ID, pillars, and the scenario's first line. Ask the learner which IDs to promote. Wait.

## 2. Clean and verify
For each picked drill:
- Strip personal context: the learner's employer, projects, names, real IDs, and any reference to their answers or mistakes.
- Web search the official Google Cloud docs to re-verify every fact in the scenario, answer, and explanations. Fix what changed. If a fact can't be verified, stop and tell the learner; don't promote that drill.
- Set `Verified:` to today.

## 3. Append
- Assign public IDs `$ARGUMENTS-D###`, one higher than the highest ID in `drills/$ARGUMENTS.md`, starting at D001.
- Append the drills to `drills/$ARGUMENTS.md` in `templates/drill.md` format. Create the file with a `# $ARGUMENTS drills` heading if it's missing. Append only; never edit existing entries.
- In `STATE/staging/$ARGUMENTS.md`, add a line `promoted: <public ID>` directly under each source drill. This is the one permitted change to existing staging entries.

## 4. Commit and push
Show both diffs. Commit this repo with `promote: $ARGUMENTS <public IDs>` and STATE with `promote: $ARGUMENTS <staging IDs>`. Push both with `git push -u origin <branch>`.
