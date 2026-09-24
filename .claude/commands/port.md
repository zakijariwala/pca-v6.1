---
description: Port one block of reference material into a public learn/ file
argument-hint: <block ID, e.g. B03>
disable-model-invocation: true
---

Port block `$ARGUMENTS` into this repo. If no block ID was given, ask for one and stop.

## 1. Gather
- Resolve STATE as in /start: the clone whose `origin` ends in `pca-v6.1-state`; never assume `../`.
- Read STATE/source/map.md and find the reference sections mapped to `$ARGUMENTS`.
- Read those sections of STATE/source/reference.md.
- Read the `$ARGUMENTS` section of `curriculum.md`. Its in-scope and out-of-scope lines bound the file.

## 2. Verify
Web search the official Google Cloud docs (cloud.google.com, docs.cloud.google.com) for every limit, quota, price, product name, and GA status the file states. Record each URL in `sources`. List every claim you can't verify under `unverified`, and tag it [UNVERIFIED] in the body. Set `last_verified` to today.

## 3. Write
Write `learn/$ARGUMENTS-<slug>.md` from `templates/learn.md`:
- One chunk per service or concept, in the order the block contract lists them.
- Rewrite in your own words, following the Writing rules in CLAUDE.md. Never copy source text, exam questions, or case study text.
- Strip personal content: names, emails, project IDs, billing IDs, scores, the learner's history.
- Put every check answer inside `<details>`.

If the file already exists, treat this as an update and show the diff against it.

## 4. Approve
Show the full diff and the `unverified` list. Wait for the learner's approval. On approval, commit with the message `port: $ARGUMENTS` and push. Without approval, commit nothing.
