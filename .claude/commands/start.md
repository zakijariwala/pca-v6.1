---
description: Start a study session for one block
argument-hint: <block ID, e.g. B03>
disable-model-invocation: true
---

Start a session for block `$ARGUMENTS`. If no block ID was given, ask for one and stop.

## 1. Resolve the state repo
Find the clone of `pca-v6.1-state` on disk: a git repo whose `origin` remote ends in `pca-v6.1-state`. Search the filesystem; never assume `../`. Call its path STATE and report it.

If no state repo exists, tell the learner. Run the session with a fresh STATE.md held in the conversation only. Write nothing personal to this repo.

## 2. Check freshness of STATE
If STATE/STATE.md exists:
- Find the newest log: the file most recently added under STATE/log/, per `git -C STATE log --diff-filter=A --name-only --format= -- log/ | head -1`.
- Compare it with the "Last session" line in STATE.md.
- If they differ, stop. Tell the learner the previous session's branches are unmerged, name the branches if you can see them, and ask them to merge both repos to main and start a new session.

If STATE/STATE.md is missing, create it from `templates/state.md`. Ask the learner 3 profile questions, one per message:
1. Background: what infrastructure, cloud, and coding work have you done?
2. Strengths and gaps: what feels solid, and what feels shaky?
3. Analogy base: which systems do you know well enough to compare against (for example Linux admin, on-prem networking, AWS)?
Fill the Learner section from the answers.

## 3. Load context
Read, and report any file that is missing:
- STATE/STATE.md
- The `$ARGUMENTS` section of `curriculum.md`
- The last two files in STATE/log/ whose names contain `$ARGUMENTS`
- `learn/$ARGUMENTS*.md`

Never reconstruct past sessions from guesswork.

## 4. Warn on stale content
In the learn file front matter:
- If `last_verified` is more than 90 days before today, warn and name the date.
- If `unverified` is non-empty, list its entries and verify them before you teach them.

## 5. Run the session
Follow the session flow in CLAUDE.md. Track, for /close: chunks covered, check-question results, misconceptions (Said / True / Root cause / Retest), parking lot items, and every drill you write.
