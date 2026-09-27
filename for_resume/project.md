---
schema: 2
slug: pca-v6-1-study-system
title: PCA v6.1 study system
tagline: Open course and tutor setup for the Google Cloud PCA exam
publish: true
reason: ""
status: active
kind: learning
size: medium
started: 2026-09
ended: null
role: "Owner — scoped the curriculum, designed the system, wrote and published it"
summary: Published an open, 32-block Google Cloud architect course with 310 original drills and 19 cost-capped labs, plus a tutor setup that keeps learner progress in a separate private repo.
problem: Exam prep for the Professional Cloud Architect exam tends to teach feature lists, drift out of date as products change, and give no safe, repeatable way to practise hands-on work.
stack: [Markdown, Claude Code, Google Cloud, Terraform, Cloud Shell]
categories: [cloud, reliability, ai]
links: { live: "", docs: "", demo: "" }
metrics:
  - value: "32"
    label: "curriculum blocks across 5 phases, each with a scope contract"
    evidence: "curriculum.md"
  - value: "310"
    label: "original exam-style drills, append-only with stable IDs"
    evidence: "drills/*.md"
  - value: "19"
    label: "optional hands-on labs, each with a cost ceiling and teardown"
    evidence: "labs/"
  - value: "5"
    label: "infrastructure CLIs the tutor is blocked from running"
    evidence: ".claude/settings.json"
highlights:
  recruiter:
    - Designed a complete study programme, from exam scope to case studies and final review, and published it openly under clear content and code licences.
    - Built cost control and safety into the labs, with budget alerts, cost ceilings and mandatory teardown, so practice cannot run up a surprise bill.
    - Separated public course material from private learner progress, so the course can be shared without exposing anyone's scores or history.
  engineer:
    - Each block has a contract in the curriculum (exam guide sections, scope, Well-Architected pillars, case-study tie-ins, lab candidates), and sessions stay inside the active block.
    - Drills carry stable IDs, pillar tags and a verified date; entries are never edited, and a correction arrives as a new drill.
    - Lessons record a last-verified date and list claims that could not be confirmed, and material older than 90 days is flagged as suspect.
    - The tutor configuration denies gcloud, gsutil, bq, kubectl and terraform, so the learner runs every command in Cloud Shell and pastes output back.
    - Four slash commands run the session loop (start, close, port, promote); the tutor stops if the private state file is behind the latest session log.
  story: ""
skills: [Curriculum design, Cloud architecture, Cost guardrails, Least-privilege tooling, Content governance, Open-source licensing]
ai_assisted: true
media: []
todo_owner:
  - "Why did you build a course rather than just study? A 1–3 sentence first-person story would fill highlights.story."
  - "The books repo records that you hold the Professional Cloud Architect certification. Did this system play a part, and do you want to say so?"
  - "Any learners, stars or forks beyond your own use?"
generated: { at: 2026-09-27, commit: edb65db }
---

## Overview

A self-paced course for the Google Cloud Professional Cloud Architect exam (guide v6.1, for exams from October 2025), built for production skills first and the exam second. It has 32 blocks of lessons, exam-style drills and optional labs in plain Markdown, with no build step. It can be used solo, or with an AI tutor configuration for Claude Code that runs guided sessions and keeps each learner's progress in a separate private repository.

## The problem

Cloud certification material often lists services and features rather than teaching how to choose between them. Product names, limits and prices change often, so static notes go stale without warning. Hands-on practice on a real cloud account carries a cost risk, and a tutor that can run infrastructure commands directly carries a safety risk.

## What I built

- A curriculum of 32 blocks in 5 phases: orientation, foundations, data, AI, security and operations, the 4 official case studies, and exam readiness. Teaching blocks alternate with practice blocks covering troubleshooting, Terraform, incident drills and a capstone.
- Lessons for every block, written in chunks with check questions and hidden answers.
- 310 original drills. Each names the Well-Architected pillars it tests, gives a scenario and 4 options, and explains why each wrong option fails.
- 19 optional labs for Cloud Shell, each with a cost ceiling, a budget alert and a teardown.
- Templates for lessons, drills, labs, session logs and learner state.
- A tutor configuration: teaching rules, 4 slash commands and a permissions file that blocks infrastructure commands.

## Architecture

- **Public repository:** curriculum, lessons, drills, labs, templates and tutor configuration, licensed CC BY-SA 4.0 for content and MIT for code.
- **Private state repository:** the learner's profile, weak spots, session logs and staged drills. Never mixed with public content.
- **Session loop:** start loads the state, block contract, recent logs and lesson; the tutor diagnoses, teaches one chunk at a time and runs drills; close writes a log, stages drills and rewrites the state file.
- **Publishing loop:** port drafts a lesson and 10 public drills with facts checked against official documentation; promote moves reviewed session drills into the public set.

## Key decisions

- **Trade-offs, not feature lists.** Services are compared on cost, operational burden, scale, latency and consistency, because the exam tests choices under constraints.
- **Append-only drills.** Stable IDs and no edits mean a learner's history and the public set never disagree; corrections are new drills.
- **Freshness is explicit.** Every lesson and drill carries a verified date, and anything that could not be confirmed is listed rather than stated as fact.
- **The learner runs the commands.** Denying infrastructure CLIs to the tutor removes the risk of an assistant changing a real cloud project, and makes the learner read real output.
- **Original scenarios only.** Real exam questions and case-study text are never reproduced; the repository links to Google's pages instead.
- **Free-trial realism.** The curriculum notes which topics stay theory on a trial account and which services burn credit fastest, and asks for daily teardown.

## Results

- All 32 lessons and drill files are published, with 310 drills and 19 labs.
- The structure was built in 7 phases and the content ported in batches over a concentrated build in September 2026.
- A private state repository and templates are in place for tracking progress.

## What's next

- Add labs for the remaining blocks that list lab candidates.
- Promote reviewed drills from learning sessions into the public set.
- Re-verify facts on a rolling 90-day basis as Google Cloud products change.
