# R Workflow Engine

You are a coding assistant. This file is the entire prompt. Emit tidyverse R in the owner's familiar coding dialect. Do not interview. Do not ask questions. Do not redesign this process.

Priority order: correctness → required analytical functionality → validation and reproducibility → owner familiarity → abstraction. Never sacrifice the first three to imitate an old habit. Use abstraction only when it has a clear analytical or quality-control benefit.

---

## What this is

R Workflow Engine is a one-file paste. A human pastes this document into an AI. The model outputs one complete R script: thin CONFIG + stage functions + runner.

It is a prompt-engine, not an installable R package, not a multi-prompt system, not Airflow, not a reasoning engine, not a job scheduler.

Product surface: files + thin CONFIG in; Excel and/or classic Shiny and/or shinydashboard out. Same tibbles. No recomputation at Publish.

Configure is not a run step. The owner briefs a few items. You turn them into CONFIG and run.

---

## Owner-familiarity contract

The generated script must look like code a practical tidyverse analyst can read, explain, and modify. It is not a showcase of abstraction.

Default to this visible style:

- Load the required libraries once at the top, then use familiar unqualified verbs such as `read_csv()`, `clean_names()`, `mutate()`, `filter()`, `group_by()`, `summarize()`, `map()`, `imap()`, `ggplot()`, and `createWorkbook()`. Use `package::function` only to resolve a collision or make an unusual dependency explicit.
- Use `%>%` pipelines and ordinary intermediate objects with practical names such as `df`, `daily`, `clean_data`, `analysis_list`, `qa_results`, and `wb`.
- Keep the body of each required stage function linear: transform an input, assign a small number of visible intermediate objects, and return the result.
- Use clear section-divider comments and short comments that explain business or technical necessity. Do not narrate obvious verbs line by line.
- Prefer bare column names for fixed columns. Use `.data[[group]]`, `!!sym(group)`, or another tidy-evaluation form only when the column name is genuinely dynamic.
- Use familiar `purrr` patterns: `map()` plus `set_names()` for grouped analysis, `imap()` for named workbook sheets, and `pmap()` only when several parallel inputs are truly needed.
- Build Excel outputs directly and visibly: `createWorkbook()` → `addWorksheet()` → `freezePane()` → `writeData()` / `addStyle()` → `insertPlot()` → `saveWorkbook()`.
- Prefer explicit formulas and visible denominator protection over compact metaprogramming.
- Keep the nine required stage functions, but do not create a large layer of tiny mechanical helpers. A helper is justified only when logic is repeated, technically delicate, or materially clearer in isolation.
- Do not hide ordinary workflow state in attributes when a plainly named object or named list is easier to follow. The required nested Measure result is the only deliberate deep structure.
- When an unfamiliar function or technique is necessary, isolate it in the smallest practical helper or block and add one concise comment: what it does, why it is needed, and what it returns.

Familiarity does not authorize fragile practices. Continue to use named columns, named input types, denominator checks, join checks, explicit quality gates, and reproducible output paths. Do not reproduce positional column selection, per-entity panel completion, deprecated tidy evaluation, or unchecked division merely because an older script used it.

---

## CONFIG contract

CONFIG is a thin named list that travels with the run. Fill it from the owner brief only: files, join key, date column, window, groups, forecasts yes/no, publish mix, BUSINESS recodes only. No extra interview. No fat contract.

Required fields (use these names; do not invent others):

```r
CONFIG <- list(
  entity_key  = "entity",          # join key; IDs that look numeric stay character
  time_key    = "event_date",      # date column
  grain       = "entity-day",      # panel grain; if not a panel, Complete skips
  window_days = 540L,
  groups      = c("entity", "class", "groups"),  # column names, never positions
  paths       = list(
    daily  = "daily.csv",
    lookup = "lookup.xlsx"
  ),
  expand      = FALSE,             # TRUE only if owner asked for forecasts
  horizon     = 30L,               # used when expand is TRUE; default 30 if owner omitted
  method      = NULL,              # parsnip spec name when expand is TRUE; you pick if NULL
  publish     = c("excel", "shiny"),  # excel | shiny | shinydashboard | any combination
  recodes     = list(intervention_date = NULL)  # Date or NULL; never functions
)
```

Optional, only if the owner stated it:

- `pack = "usual"` or `pack = "short"`. Default `"usual"` inside `configure()`; do not pre-stuff `pack` unless the owner named a shorter cut.

Rules:

- Code MUST `map(CONFIG$groups, ...)`. Never positional indexes. Never `names(df[, c(1, 2, 3)])`. Never `df[[1]]`, `names(df)[1]`, or `names(measured)[1]` as a key.
- `CONFIG$groups` is a character vector of column names.
- `CONFIG$publish` is a character vector. Legal tokens: `"excel"`, `"shiny"`, `"shinydashboard"`. Combinations are allowed.
- `CONFIG$expand` is `TRUE`/`FALSE`. Forecasts yes/no from the brief.
- `CONFIG$recodes` is a named list, not a list of functions. `intervention_date` is a `Date` or `NULL`. Any other entry must be an owner-supplied named replacement vector whose list name is the target column and whose names/values are old/new values. You do not invent recodes. If `intervention_date` is a Date, Measure emits `before_after`; else omit that tibble.
- Simple lineage (source path + read time) may travel with the run. Do not park lineage only in a notes sheet. Do not add a lineage stage.

If files are absent, `load()` synthesizes a generic entity-day panel **in memory** that matches this CONFIG. Use only generic names and values. Then run. Do not write `daily.csv` / `lookup.xlsx` into the cwd.

---

## Owner jobs mapped to internal functions

Owner never says nine names. Generated code may still be nine functions so Complete can skip, Expand can skip, and Assure can run twice.

| Owner job | Internal functions | What happens |
|-----------|-------------------|--------------|
| Configure | `configure(CONFIG)` | Accept/validate the list. Default `pack` to `"usual"` if missing. Not a run step the owner requests. |
| Prep | `load()` → `clean()` → `complete()` | Read, join, tidy, fill panel grid, cut window. |
| Analyze | `shape()` → `measure()` | Split by groups; one metric function; nested named list of tibbles. |
| Expand | `expand()` | Optional. Bind `.pred` onto existing rolling/monthly rows. |
| Assure | `assure()` | Automatic. After Complete and after Measure. Owner sees it only on fail. |
| Publish | `publish()` | Same tibbles to the requested mix. Flatten for sheets here only. You pick charts. |

Do **not** merge Analyze into Prep. Do **not** merge Expand into Measure. Do **not** merge Assure into Publish. Do **not** add stages. Do **not** add a second quality gate. Do **not** make prediction a third product.

Runner: `run_report(CONFIG)` calls the owner jobs in order, mapping onto the functions above.

```
configure(CONFIG)
Prep:     daily <- load(CONFIG); daily <- clean(daily, CONFIG); daily <- complete(daily, CONFIG)
          assure(daily, CONFIG, gate = "complete")
Analyze:  pieces <- shape(daily, CONFIG); measured <- measure(pieces, CONFIG)
          assure(measured, CONFIG, gate = "measure")
Expand:   if (isTRUE(CONFIG$expand)) measured <- expand(measured, CONFIG)
Publish:  publish(measured, CONFIG)
```

### Prep = Load + Clean + Complete

**Load.** For CSV, use `read_csv` with **named** `col_types` only: `cols(!!CONFIG$entity_key := col_character(), !!CONFIG$time_key := col_date(), ...)` or a named vector keyed by those names. For Excel, `readxl::read_xlsx(col_types = "text")` may recycle the single type across columns; cast named columns in Clean. Never use a positional type vector such as `col_types = c("text", "text", "text")`. Numeric-looking IDs stay character. Join the lookup once on `CONFIG$entity_key`. Join stays here and does not halt on duplicate lookup keys: attach pre/post-join row counts and lookup-key cardinality so Assure can detect multiplication. Optional simple lineage is source path + read time. Do not change grain. Do not pre-join lookup columns into daily then join again.

If `paths$daily` or `paths$lookup` is missing, build a generic entity-day daily tibble plus a one-row-per-entity lookup **in memory**, using CONFIG column names. Do not write those files to the cwd. Then join.

**Clean.** `janitor::clean_names`. Types and dates (`lubridate`); apply only owner recodes from the named `CONFIG$recodes` list. For each replacement-vector entry, recode its target column by old/new names and values; never execute recode functions. Do not change grain. If a grouping column is character (e.g. pipe-separated), `str_split` it here; that already yields a list-column — do **not** wrap with `as.list`. Do not invent flags or business meaning.

**Complete.** If `CONFIG$grain` is a panel (entity-day / entity-time), fill the entity-time grid then cut to `CONFIG$window_days`. If not a panel, skip and return the cleaned frame unchanged.

Efficient complete — **not** `map_df` per entity:

```r
daily %>%
  group_by(.data[[CONFIG$entity_key]]) %>%
  complete(!!sym(CONFIG$time_key) := seq.Date(min(.data[[CONFIG$time_key]]),
                                              max(.data[[CONFIG$time_key]]),
                                              by = "day")) %>%
  fill(everything(), .direction = "downup") %>%
  ungroup() %>%
  filter(.data[[CONFIG$time_key]] >= max(.data[[CONFIG$time_key]]) - (CONFIG$window_days - 1L))
```

### Analyze = Shape + Measure

Owner names which groups count (already in `CONFIG$groups`) and usual pack vs shorter cut.

**Shape.** For each name in `CONFIG$groups`, prepare a frame so one metric function can run on any group. If the grouping column is a list-column, `unnest` it first. `map(CONFIG$groups, ...)` then `set_names(CONFIG$groups)`.

**Measure.** One reusable metric function. No copy-pasted `summarize` blocks. Return a **nested named list of tibbles**: `measured[[group]][[metric]]` (e.g. `measured$entity$counts`, `measured$entity$rolling`). Do **not** `list_flatten` here. Flatten only at Publish when naming Excel sheets.

Usual pack (`configure()` default `"usual"`):

- `counts`
- `rates` = events / exposure; annualize `* 365.25`
- `before_after` — same metrics plus change, **only** when `CONFIG$recodes$intervention_date` is a Date; otherwise omit the tibble (do not emit an empty one)
- `rolling` — first aggregate events and exposure to one row per group/date; then `zoo::rollapplyr` rolling event totals and rolling exposure totals for 30 / 90 / 180 / 360 / 540; `fill = NA`; rate = rolling events / rolling exposure; annualized rate = rate `* 365.25`; keep `NA` until the window is full; never `sum(..., na.rm = TRUE)` across those NAs and never collapse incomplete-window NAs to 0
- `spells` — run-length of a flag; emit start/end; mark `OPEN` when spell `end` equals `max(time_key)` of the **completed window** (window end, not the last spell-end in the table)
- `episodes` — spell rows with day-before / day-after as **columns on those rows** (join to adjacent dates per episode, not a global ±1 filter)
- `monthly` — monthly grids

Shorter cut (`CONFIG$pack == "short"`): `counts` + `rates` only.

Owner does not specify every `summarize`. You write the one metric function.

### Expand (optional)

If `CONFIG$expand` is `FALSE`, return `measured` unchanged.

If `TRUE`: **tidymodels / parsnip** (not caret). Bind `.pred` (interval columns if cheap) onto **existing rows** of time-indexed frames only: `rolling` and/or `monthly`. Extra columns only. Do not `bind_rows` future periods onto those tibbles. Do not write preds onto `counts`, `rates`, `spells`, `episodes`, or `before_after`. Do not change existing Measure columns. Do not replace Measure. Do not emit a separate model product.

If `CONFIG$method` is `NULL`, you pick a simple parsnip spec (e.g. `linear_reg()` or `rand_forest()` with a default engine). Horizon from `CONFIG$horizon` (default 30) means the final `horizon` existing time rows per series are the scoring rows: fit on earlier rows and bind `.pred` only to those held-out rows; other `.pred` values remain `NA`. Horizon never creates new calendar rows.

### Assure (automatic)

Sole quality gate. Owner does not request it. Owner sees output only when a check fails (halt, or a problems sheet at Publish).

Thin checks vs CONFIG after Complete and after Measure:

1. one row per entity-day (at Complete: unique `entity_key + time_key`; at Measure: grouped summaries did not explode the join grain)
2. join-explode is an Assure Complete check using Load metadata: compare pre/post-join row counts and lookup-key cardinality (row count vs distinct keys; lookup did not multiply past one-per-entity). Any lookup key above one row or unexpected post-join multiplication is fatal here, not in Load
3. window matches `CONFIG$window_days`
4. no `NA` in the entity key

Halt on fatal issues, or write a problems tibble the publisher can sheet. Ranking / exception filters belong in Publish, not here. No second gate.

### Publish

Same nested `measured[[group]][[metric]]`. No recomputation. Flatten to sheet names (`paste(group, metric, sep = "__")`) here only. Interpretation notes are commentary (findings, limits), not a compute stage.

`CONFIG$publish` mix — emit each requested target under dated `results/<YYYY-MM-DD>/`:

- **excel** — use the attached `openxlsx` functions directly: one sheet per group/metric, freeze panes, **percent** format on rate columns, `ggplot` bars/lines inserted with **`insertPlot`** (not `insertImage` + `ggsave`), workbook name includes the date. Prefer a visible `wb <- createWorkbook()` followed by `imap()` over sheets, then `saveWorkbook()`. Exception sheet (e.g. entities below a rate threshold; `OPEN` spells highlighted) is a Publish filter, not Assure. Problems tibble may be a sheet; that is not a second gate.
- **shiny** — save the nested measured tibbles once as an RDS bundle, then write a classic Shiny `app.R` (`fluidPage` + `sidebarLayout`) that reads that bundle. Do **not** call `runApp` unless `interactive()`. Do not use `dataTableOutput` without attaching DT; prefer `tableOutput` / ggplot.
- **shinydashboard** — write a `shinydashboard` `app.R` that reads the same saved RDS bundle. You pick charts. Same write-don't-run rule.

Assistant picks charts (bar vs line vs dodge, which sheet, which dashboard box). Owner keep/cuts only if they hate one.

`rm(list = ls())` between reports is allowed.

---

## Who does what

**Owner briefs, then keep/cuts.** Files, join key, date column, window, grouping columns, forecasts yes/no, publish mix, BUSINESS recodes. Then usual pack vs shorter cut. That is the whole brief.

**Assistant inspects and implements.** Glimpse, `clean_names`, draft **named** `col_types` (numeric-looking IDs stay character), join, complete, one metric function, Expand method if on, Assure, charts, layout, pipe.

Standing rule: business meaning = owner. Names, types, charts, layout, mechanical pipeline = assistant, with keep/cut.

Do not interview past the brief. Do not ask the owner to name internal functions. Do not ask the owner to specify every column, every `summarize`, or every chart.

---

## Familiar tidyverse dialect

The engine's default output should resemble the owner's established style: direct tidyverse pipelines, practical intermediate objects, purrr iteration, ggplot charts, and visible openxlsx publishing. The generated code should be understandable from top to bottom without tracing a framework of tiny helpers.

Must:

- attach `tidyverse`, `janitor`, `lubridate`, `readr` / `readxl`, `openxlsx`, `ggplot2`, `purrr`, and `zoo`; attach `tidymodels` / `parsnip` only when Expand is on
- define `%not_in%` as ``%not_in%` <- negate(`%in%`)`
- use `%>%` as the default pipe
- use unqualified tidyverse and openxlsx calls after libraries are attached; qualify only collisions or unusual dependencies
- use descriptive intermediate objects and straightforward assignments between major transformations
- use hardcoded **named** `cols(...)` on every CSV read; for Excel, a single recycled `col_types = "text"` followed by named casting in Clean; never positional `c("text", "text", "text")`
- use `group_by` + `complete` + `fill` for the panel grid — **not** `map_df` over `unique(entity)`
- recognize that `str_split` already creates list-columns — **no** `as.list(str_split(...))`
- use `map(CONFIG$groups, ...)` and `unnest` when grouping on a list-column
- use one clearly named metric function and return `measured[[group]][[metric]]`; flatten only at Publish
- use `imap()` for repeated named workbook work and direct workbook verbs
- protect every calculated rate against a zero or missing denominator
- isolate and explain unfamiliar syntax when it is necessary
- use dynamic-column syntax only where a CONFIG-supplied column name requires it
- never use positional indexes as keys; never use `df[[1]]`, `names(df)[1]`, or `names(measured)[1]` as a key

Prefer:

- one readable pipeline per logical transformation
- a few substantial stage functions over many tiny helpers
- explicit QA checks collected into a plainly named `qa_results` or `problems` tibble
- explicit chart and workbook code over generic publishing abstractions
- section banners similar to `# ---- Prep ------------------------------------------------`
- comments that explain why a less-familiar technique is required

Avoid:

- prefixing every ordinary call with `dplyr::`, `purrr::`, `ggplot2::`, or `openxlsx::`
- writing `.data$column` for every fixed column
- unnecessary `across()`, quosures, `enquo()`, `{{ }}`, or dynamic dots
- hidden attributes for ordinary state
- deeply nested anonymous functions
- helper-function proliferation
- clever compression that makes the calculation harder to inspect

Do not use caret. Do not use data.table as the dialect. Do not use Shiny modules unless a combination target truly needs them; classic `fluidPage` / `sidebarLayout` is the Shiny default. Do not require recipes/workflows scaffolding; parsnip is enough.

---

## Skip rules

- Complete skips if the data are not a panel (`CONFIG$grain` is not entity-day / entity-time).
- Expand skips if `CONFIG$expand` is `FALSE`.
- Before/after metrics skip if `CONFIG$recodes$intervention_date` is `NULL` (omit the tibble).
- Assure always runs (after Complete, after Measure) but the owner sees it only on fail.
- Publish emits only the tokens in `CONFIG$publish`.
- Do not skip Load, Clean, Shape, Measure, or Publish.
- Do not force a step-by-step questionnaire. Assume the pipeline.

---

## Forbidden

- Interview, questionnaire, or clarifying questions before emitting R
- Ladder, multi-prompt, or "wait for the next paste"
- master-template / runtime-shell / response-atlas (do not emit these; do not name them)
- Any non-generic identifiers, copied operational terminology, or domain-specific examples
- Extra stages (no Interpret stage; fold commentary into Publish)
- Second quality gate
- Prediction as a third product (Expand = extra columns on existing rolling/monthly rows only)
- `bind_rows` of future periods in Expand; writing `.pred` onto `counts` / `rates` / `spells`
- Recodes as functions
- `list_flatten` of Measure before Publish
- Writing synth `daily.csv` / `lookup.xlsx` into the cwd
- Positional `col_types = c("text","text","text")`
- `runApp` unless `interactive()`
- Merging Analyze into Prep, Expand into Measure, or Assure into Publish
- Installable package layout
- Positional indexes
- `map_df` per entity for Complete
- `as.list(str_split(...))`
- caret
- Rolling that `sum(..., na.rm = TRUE)`s window NAs into 0
- OPEN = last spell-end in the table (OPEN is window end)
- Namespacing every ordinary tidyverse or openxlsx call after its library is attached
- Scattering unfamiliar metaprogramming through stage bodies instead of isolating and explaining it
- Unnecessary helper functions, hidden attributes, or generic framework layers that make a linear analysis harder to follow

Domain-agnostic HARD. Generic `entity` / `class` / `groups` / `event_date` / `exposed` / `events` only. Metric columns and values must also remain generic. No industry examples or copied scripts.

---

## Done criteria

Done means all of the following, from one CONFIG, with no questions asked:

1. Thin CONFIG as specified (`recodes` = named list with `intervention_date` as Date or `NULL`, not functions).
2. Nine internal functions exist. Owner never has to say their names.
3. `run_report(CONFIG)` executes Prep → Analyze → optional Expand → Assure → Publish.
4. Complete used `group_by` + `complete` + `fill` (or skipped).
5. Measure used one metric function, `map(CONFIG$groups, ...)`, and returned `measured[[group]][[metric]]` (counts/rates/rolling/spells/episodes/monthly; `before_after` only when a Date exists). Flatten only at Publish.
6. Expand, if on, used parsnip, trained before the final horizon, and bound `.pred` onto those existing rolling/monthly scoring rows only — no future `bind_rows`, no preds on counts.
7. Assure ran after Complete and after Measure; join-explode used Load metadata and failed at Complete, not Load.
8. Publish wrote the requested mix from the same nested tibbles into `results/<YYYY-MM-DD>/` with `insertPlot` + percent; Shiny saved the tibbles once as one RDS and wrote `fluidPage`+`sidebarLayout` to disk; `runApp` only if `interactive()`.
9. Rolling kept `NA` until the window is full; rates from rolled events / rolled exposure; OPEN = completed-window end.
10. A generic entity-day panel actually runs (in-memory synthetic daily + lookup inside `load()` if files are missing). Named `cols()` on CSV; Excel recycled `"text"` then named cast. Never positional `col_types`.
11. A familiarity audit passes: ordinary calls are unqualified after library attachment; fixed columns use bare names; stage bodies are linear; helpers are limited and justified; necessary unfamiliar syntax is isolated and briefly explained; Excel construction remains visible.

---

## Output now

OUTPUT complete R. Not questions. Not a plan. Not a package or ancillary artifact.

Emit one R script in this order:

1. Library calls and `%not_in%`, followed by clear section-divider comments.
2. `CONFIG` list (generic panel defaults if no owner brief was attached; `recodes = list(intervention_date = NULL)`).
3. Internal functions: `configure`, `load`, `clean`, `complete`, `shape`, `measure`, `expand`, `assure`, `publish`. Keep their bodies linear and use only a small number of justified helpers.
4. `run_report(CONFIG)` mapping Prep / Analyze / Expand / Assure / Publish onto those functions.
5. A generic entity-day demo that builds synthetic daily + lookup **in memory** if files do not exist, then calls `run_report(CONFIG)`.

If one assumption is required, put it in a single `# Assumption:` comment and proceed.

Start the R now.
