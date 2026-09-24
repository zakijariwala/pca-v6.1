---
block: B22
title: Remediation (template)
pillars: [security, reliability, cost, performance, operational-excellence, sustainability]
exam_guide_refs: ["per weak spot"]
last_verified: 2026-09-24
sources:
  - learn/ files B00 to B20 and P01 to P08, which carry the per-fact sources
unverified: []
---
# B22 Remediation (template)

A method, not new content. Use it after each B21 run, or whenever the same kind of question keeps going wrong. With Claude Code, `/start B22` reads your open weak spots from STATE.md. Solo, keep your own list.

## Chunk 1: Find the root cause, not the topic
Problem it solves: "I'm weak on networking" is too broad to fix. A misconception is fixable.

Mental model: For each missed question write four lines.

| Line | Example |
|---|---|
| Said | "Read replica gives Cloud SQL HA." |
| True | "HA is a standby in a second zone with synchronous replication; replicas don't fail over on their own." |
| Root cause | "I treated any second copy as failover." |
| Retest | "A question where both a replica and HA appear as options." |

Group the root causes. Three misses often share one.

Exam signals: Misses cluster around distractors that sound right: bigger product, more features, familiar name.

Trap: Rereading a whole block because one question went wrong.

Pillar tie-in: Operational excellence (for your study process).

Check: Why record "Said" in your own words?
<details><summary>Answer</summary>It shows the exact wrong belief. The fix targets that belief, not the whole topic.</details>

## Chunk 2: Reteach from the misconception
Problem it solves: Rereading doesn't dislodge a wrong model. Contrast does.

Mental model:
1. Find the chunk in `learn/` that covers the fact. Read its Mental model and Trap.
2. Write the difference as a two-row table: what you thought, what's true, and the signal in a question that tells them apart.
3. Explain it out loud in under 60 seconds, as you would to a colleague.

Exam signals: The "signal" row in your table is what you'll look for on exam day.

Trap: Memorizing the answer to the one question you missed. The next question uses different words.

Pillar tie-in: Operational excellence.

Check: What goes in the third column of your contrast table?
<details><summary>Answer</summary>The keyword or requirement in a question stem that tells the two options apart.</details>

## Chunk 3: Write 8 retest questions
Problem it solves: Proof that the misconception is gone.

Mental model: For each weak spot, write 8 short scenarios in the `templates/drill.md` format. Vary the wording and context: different company, different numbers, the correct answer in different positions. Put the tempting wrong answer in at least 4 of them. Wait a day, then answer them cold. 7 of 8 correct closes the weak spot.

If you use Claude Code, drills written in a session go to staging in your state repo; `/promote` later moves reviewed ones to the public `drills/` files.

Exam signals: N/A; this builds your own question bank.

Trap: Writing questions whose wording gives the answer away.

Pillar tie-in: Operational excellence.

Check: What score closes a weak spot here?
<details><summary>Answer</summary>7 of 8 correct, answered cold at least a day after writing.</details>

## Chunk 4: Worked example
Problem it solves: See the method once, end to end.

Mental model:
- Missed: B21 Q12 (who read the patient records).
- Said: "Admin Activity logs record everything."
- True: Admin Activity records configuration changes and is always on. Data reads go to Data Access logs, off by default except BigQuery.
- Root cause: I assumed "audit log" means "all activity."
- Reteach: P05 chunk 4, the audit log table.
- Contrast table:

| | Admin Activity | Data Access |
|---|---|---|
| Records | Config changes | Reads of data or config, user data writes |
| Default | Always on | Off, except BigQuery |
| Question signal | "Who changed or deleted" | "Who read or accessed" |

- Retest ideas: who deleted a bucket; who read a BigQuery table; who changed a firewall rule; how to prove reads next quarter; which logs you can't disable; where Policy Denied logs fit; retention of `_Required`; Google staff access (Access Transparency, a distractor for all of them).

Check: In the example, what signal word points to Data Access logs?
<details><summary>Answer</summary>"Read" or "accessed" in the question.</details>
