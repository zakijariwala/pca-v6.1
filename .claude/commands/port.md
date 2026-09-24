---
description: Port one block into a public learn/ file and write its public drills
argument-hint: <block ID, e.g. B03> [drills]
disable-model-invocation: true
---

Port block `$0` into this repo. If no block ID was given, ask for one and stop.

If the second argument is `drills` (`/port B17 drills`), skip steps 1 to 3 and write drills only, from the existing `learn/$0-*.md`. Stop if that file doesn't exist.

## 1. Gather
- Resolve STATE as in /start: the clone whose `origin` ends in `pca-v6.1-state`; never assume `../`.
- Read STATE/source/map.md and find the reference sections mapped to `$0`.
- Read those sections of STATE/source/reference.md.
- Read the `$0` section of `curriculum.md`. Its in-scope and out-of-scope lines bound the file.

## 2. Verify
Search the official Google Cloud docs (cloud.google.com, docs.cloud.google.com) for every limit, quota, price, product name, and GA status the file states. Record each URL in `sources`. List every claim you can't verify under `unverified`, and tag it [UNVERIFIED] in the body. Set `last_verified` to today.

## 3. Write
Write `learn/$0-<slug>.md` from `templates/learn.md`:
- One chunk per service or concept, in the order the block contract lists them.
- Rewrite in your own words, following the Writing rules in CLAUDE.md. Never copy source text, exam questions, or case study text.
- Strip personal content: names, emails, project IDs, billing IDs, scores, the learner's history.
- Put every check answer inside `<details>`.

If the file already exists, treat this as an update and show the diff against it.

## 4. Write drills
Append 10 drills to `drills/$0.md` in `templates/drill.md` format. Create the file with a `# $0 drills` heading if it's missing.
- IDs: `$0-D###`, one higher than the highest ID in the file, starting at D001. Append only; never edit or delete existing entries.
- Cover the block contract's in-scope topics and exit criteria. For case study blocks, every drill refers to the case study and tests one of its requirements.
- Original scenarios only. Never reproduce real exam questions or copy case study text.
- Every fact must come from the learn file's verified `sources` or a fresh check of the official docs. Skip any drill that would rest on an `unverified` claim.
- Four options, one best answer. Vary the position of the correct letter. Explain why each wrong option fails.
- Set `Verified:` to today.

## 5. Approve
Show the full diff (learn file and drills) and the `unverified` list. Wait for the learner's approval. On approval, commit with the message `port: $0` (or `port: $0 drills`) and push. Without approval, commit nothing.
