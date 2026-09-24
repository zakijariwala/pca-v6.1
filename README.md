# PCA v6.1 study system

A self-paced course for the Google Cloud Professional Cloud Architect exam, built for production skills first and the exam second. 32 blocks: lessons, practice drills, and optional hands-on labs.

- **Scope:** Professional Cloud Architect exam guide v6.1, for exams taken on or after October 30, 2025, and its four case studies.
- **Format:** plain Markdown. Answers hide inside collapsible `<details>` blocks. Nothing needs a build step or a script.

## Two ways to use it

### Solo
1. Open [curriculum.md](curriculum.md). Work the blocks in order.
2. For each block, read `learn/<block>-*.md`. Answer each check question before you expand the answer.
3. Run the drills in `drills/<block>.md`. Write down your choice, then expand the answer and read why each wrong option fails.
4. Do a lab from `labs/` only in a sandbox project with a budget alert set. Run the teardown every time.
5. Track your own progress. [templates/log.md](templates/log.md) and [templates/state.md](templates/state.md) give a format.

### With Claude Code
The repo ships a tutor setup: [CLAUDE.md](CLAUDE.md) sets the teaching rules, and `.claude/commands/` holds four slash commands.

| Command | What it does |
|---|---|
| `/start <block>` | Loads your state and the block, then runs a tutoring session |
| `/close` | Writes a session log, stages new drills, updates your state |
| `/port <block>` | Drafts a `learn/` file with facts checked against official docs |
| `/promote <block>` | Moves reviewed drills into the public `drills/` file |

Progress lives in a second, private repo that you create: `STATE.md`, `log/`, `staging/`. Open both repos in the same Claude Code session. Without the private repo, `/start` runs with a fresh state and saves nothing.

Claude never runs `gcloud`, `gsutil`, `bq`, `kubectl`, or `terraform`. [.claude/settings.json](.claude/settings.json) denies them. You run lab commands in Cloud Shell and paste the output back.

## Layout

| Path | Holds |
|---|---|
| `curriculum.md` | One contract per block: exam guide refs, scope, pillars, case study tie-ins, lab candidates |
| `learn/` | Lessons, one file per block, in chunks |
| `drills/` | Exam-style scenarios, one file per block, append-only |
| `labs/` | Optional Cloud Shell labs with cost ceilings and teardown |
| `templates/` | Formats for lessons, drills, labs, logs, and state |
| `CLAUDE.md`, `.claude/` | Tutor rules, slash commands, permission denies |

### Using the drills
Each drill has an ID like `B03-D007`, the Well-Architected Framework pillars it tests, a scenario, and four options. The answer block names the best option, explains why each other option fails, and carries a `Verified:` date. Drills are original. IDs never change and entries are never edited; a correction arrives as a new drill.

## Freshness
Google Cloud renames, retires, and reprices products often. Every lesson records `last_verified` and lists claims it couldn't confirm under `unverified`; every drill carries a `Verified:` date. Treat anything older than 90 days as suspect, and check limits, quotas, prices, and product names against [Google Cloud documentation](https://cloud.google.com/docs) before you rely on them.

## Disclaimer
- This project has no affiliation with Google and Google doesn't endorse it. Google Cloud and related product names are trademarks of Google LLC.
- All scenarios and questions are original. None come from the real exam.
- The case studies belong to Google. Read them on the [exam guide page](https://cloud.google.com/learn/certification/guides/professional-cloud-architect). This repo links to them and never copies them.

## License
- Content (Markdown lessons, drills, labs, curriculum): [CC BY-SA 4.0](LICENSE).
- Code (`.claude/` config and commands, and shell snippets in labs): [MIT](LICENSE-CODE).
