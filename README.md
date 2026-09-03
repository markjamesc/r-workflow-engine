# R Workflow Engine

One-file paste prompt for producing a complete tidyverse R analysis script. It is not an installable R package, Airflow replacement, or job scheduler.

Paste [`ENGINE.md`](ENGINE.md) into an AI together with a short owner brief. The model produces one R script with a thin configuration layer, named stage functions, a quality gate, and non-transforming publication outputs.

## Workflow

| Stage | Responsibility |
|---|---|
| **Prep** | Read, join, tidy, complete a panel when required, and apply the analysis window |
| **Analyze** | Split by named groups and apply one parameterized metric function |
| **Expand** | Optionally add forecasts as columns on existing rows |
| **Assure** | Apply the single automatic quality gate |
| **Publish** | Send the validated tibbles to Excel, Shiny, or both without recomputation |

```mermaid
flowchart LR
    A["Thin CONFIG"] --> B["Prep"]
    B --> C["Analyze"]
    C --> D["Optional expand"]
    D --> E["Assure gate"]
    E --> F["Excel or Shiny"]
```

## How to use

1. Copy [`ENGINE.md`](ENGINE.md).
2. Paste it into ChatGPT, Grok, or DeepSeek.
3. Attach an owner brief, or allow the generic in-memory demonstration to run.
4. Review the generated tidyverse R script and execute it against the intended data.

The owner brief supplies file paths, join keys, date column, analysis window, grouping variables, forecast choice, publication target, and business recodes. The engine chooses appropriate column names and charts while preserving the locked workflow structure.

## Implemented case study

The engine is applied in the [FulfillIQ](https://github.com/markjamesc/fulfilliq) portfolio project:

- [Generated and adapted R analysis](https://github.com/markjamesc/fulfilliq/blob/main/r/Stage_04_FulfillIQ_R_Analysis.R)
- [Executed Excel evidence workbook](https://github.com/markjamesc/fulfilliq/blob/main/results/2026-09-02/FulfillIQ_R_Evidence_2026-09-02.xlsx)
- [Decision evaluation](https://github.com/markjamesc/fulfilliq/blob/main/docs/Stage_05_Decision_Evaluation.md)

That implementation demonstrates the engine's named configuration, stage functions, explicit skips, assurance gate, and Excel publication path on a seller-performance analysis.

## Design dialect

- Tidyverse pipelines
- Named lists and explicit column roles
- `purrr::map()` over group names rather than column positions
- One reusable metric function
- A single quality gate before publication
- Publication from validated objects without hidden recomputation

This is my preferred analytical dialect, not a copy of a former employer's internal framework.

## Development method

Grok Bot coordinated Grok, ChatGPT, and DeepSeek through independent drafting, cross-critique, and controlled synthesis. Disagreements were resolved against the engine's locked requirements rather than by majority vote. The owner retained final keep/cut authority.

## Repository contents

- [`ENGINE.md`](ENGINE.md) — complete one-file prompt
- [`docs/r-workflow-engine-carousel.pdf`](docs/r-workflow-engine-carousel.pdf) — concise document carousel
- [`README.md`](README.md) — project overview and implemented example

## Limitations

- Generated code still requires human review and execution against the intended data.
- The repository is a prompt-based analytical specification, not an R package.
- The engine does not replace source-specific measurement design.

## License

MIT. Copyright 2026 Mark Ciganovic.

