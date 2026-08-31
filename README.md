# R Workflow Engine

One-file paste prompt. Not an installable R package. Not Airflow. Not a job scheduler.

You paste [`ENGINE.md`](ENGINE.md) into an AI. The model emits one complete tidyverse R script: thin CONFIG, stage functions, runner. Domain-agnostic. Generic `entity` / `class` / `groups` / `event_date` only.

## How to use

1. Copy [`ENGINE.md`](ENGINE.md).
2. Paste it into ChatGPT, Grok, or DeepSeek.
3. Attach a short owner brief, or let the generic in-memory demo run.
4. Get tidyverse R. Do not interview the model. The paste already says what to do.

## What it emits

Five owner jobs, one script:

| Job | What happens |
|-----|----------------|
| **Prep** | Read, join, tidy, fill the panel grid, cut the window |
| **Analyze** | Split by groups; one metric function; nested tibbles |
| **Expand** | Optional. Forecasts as extra columns on existing rows |
| **Assure** | Automatic quality gate. You see it only on fail |
| **Publish** | Same tibbles. No recomputation |

Publish mix is Excel and/or classic Shiny and/or shinydashboard.

You brief files, join key, date column, window, groups, forecasts yes/no, publish mix, and business recodes. Then keep/cut. The model names columns and picks charts.

## How it was made

Grok Bot coordinated Grok, ChatGPT, and DeepSeek for three passes. They reviewed, cross-critiqued, and merged. Grok Bot also wrote this README, the carousel PDF, the GitHub push, and the LinkedIn draft. I keep/cut at the end.

Last year: one AI, I was the critic. About 10–18 hours over a few days. That total is the whole job — 8–15 hours of intense critic attention, plus using that one AI to create the documents, plus logging into GitHub and LinkedIn to paste and upload. This year the other two models did the critic job, and the bot wrote the pack.

## Dialect

tidyverse pipes. Named lists. `map` over group names, never column positions.

My dialect, not a workplace clone. No former-employer residue.

## Repo contents

- [`ENGINE.md`](ENGINE.md) — the one paste
- this README
- [`docs/r-workflow-engine-carousel.pdf`](docs/r-workflow-engine-carousel.pdf) — LinkedIn document carousel

## License

MIT. Copyright 2026 Mark Ciganovic.
