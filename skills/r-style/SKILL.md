---
name: r-style
description: Jeremiah's R style. Use whenever writing, editing, or reviewing R code (.R, .Rmd, .qmd).
---

# R style

Style for research data pipelines. Derived from `cces_candidates`. Defaults below apply unless the surrounding file clearly does otherwise — match the file you're in.

## 1. Core principles

- **Readability over cleverness.** A script should read top-to-bottom like a recipe. A reader should be able to stop at any line and know what the data looks like.
- **Flat beats nested.** No wrapper functions, no `map()` over a list of configs, no config objects, when four lines do the job.
- **Don't functionalize until it repeats.** Write it inline first. Extract a helper only when the same operation is used in 3+ places or across files.
- **Surgical edits.** When fixing something, change that thing. Don't reformat, rename, or "improve" surrounding code.

## 2. Pipes and syntax

- Native pipe `|>` only. Never `%>%`.
- No `.` placeholder. When a function takes data in a non-first argument, use a lambda:

```r
# good
js_fct |>
  filter(office != "P") |>
  (\(d) xtabs(~ party_formal + office, d))()

# or just break the chain — often clearer
js_sub <- filter(js_fct, office != "P")
xtabs(~ party_formal + office, js_sub)

# bad
js_fct |> filter(office != "P") %>% xtabs(~ party_formal + office, .)
```

- No magrittr, ever — not `%>%`, not `%<>%`, not `%$%`. The package should not be loaded or depended on.
- **Every `|>` ends its line, by default.** The next verb starts the next line, indented two spaces.

```r
# good
pres_24 <- pres_24_init |>
  left_join(pres_24_ptys, by = "candidate") |>
  mutate(party_detailed = coalesce(party_detailed.x, party_detailed.y))

# bad
pres_24 <- pres_24_init |> left_join(pres_24_ptys, by = "candidate") |> mutate(...)
```

  Always break when the pipe builds a named intermediate (`x <- ...`) or is a real
  stage of the pipeline — anywhere a reader might want to stop and inspect the data.

  The one exception: **a run of short, parallel, terminal calls**, where one-per-line
  makes the block scan like a table and the varying argument is the thing you're
  checking. Breaking these would triple the line count and hide the differences.

```r
# fine — the block is the unit; you read down the column that varies
js_fct |> filter(office == "G", party %in% c("D", "R")) |> write_numbers("gov")
js_fct |> filter(office == "H", party %in% c("D", "R")) |> write_numbers("house")
js_fct |> filter(office == "S", party %in% c("D", "R")) |> write_numbers("sen")
```

  Qualifying means all of: the call is terminal (writes, prints, counts — nothing
  downstream reads it), it fits the line comfortably, it's two verbs at most, and it
  sits in a block of siblings. One isolated same-line pipe is not an exception, it's
  just a skipped break.

- **Assignment is always `<-`.** Never `=` for assignment, never `->`. `=` is only for naming arguments inside a call.
- Lambdas are `\(x)`, never `function(x)` in `across()`/`map()`, never `~ .x`.
- `dplyr::` / `tidylog::` prefixes only when disambiguating, or when you want the row/value count printed as a check that an edit actually hit something.

## 3. Script layout

Numbered scripts, each a stage of one pipeline, driven by a master script that just `source()`s them in order:

```r
source("01_stack-sources.R")
source("02_party-recode.R")
source("03_manual-fixes.R")
```

Within a script:

1. `library()` calls (tidyverse first, then the rest), then `source()` of helper files.
2. Read inputs.
3. Transform, in stages.
4. Write outputs.

Each stage reads a named intermediate from disk and writes the next one. Paths are relative to the project root — never `setwd()`.

```r
library(tidyverse)
library(haven)
source("00c_party-recode-functions.R")

js0 <- read_rds("data/intermediate/prelim/candidates_stacked.rds")
...
write_rds(js4, "data/intermediate/prelim/candidates_party-recoded.rds")
```

## 4. Section headings

Only the R/RStudio convention:

```r
# Read data ----
## Party corrections ----
```

`# Name ----` for top-level, `## Name ----` for nested. Lowercase or sentence case; keep one pattern per file.

**Never** ASCII-art dividers — no `# ===== SECTION =====`, no `# ─────────`, no `####### block #######`. (Some older files in this project carry `# SECTION =====`; don't propagate it.)

## 5. Naming intermediates

Name each stage of a transformation rather than building one enormous pipe. Two acceptable patterns:

```r
# suffix chain, when the steps are conceptually distinct
pres_raw <- get_dataframe_by_name(...)

pres_fmt <- pres_raw |>
  transmute(...)

pres_sel <- pres_fmt |>
  filter(candidatevotes >= 10)

# numbered chain, for a run of same-kind edits
js1 <- js0 |>
  mutate(...)

js2 <- js1 |>
  mutate(...)

# reassignment, for a long run of independent one-off fixes
jsdat <- jsdat |>
  tidylog::mutate(...)
```

Suffixes: `_raw` (as read), `_fmt` (renamed/recoded), `_sel` (filtered), `_fct` (factor version). Short object names (`js`, `cand`, `pres`) are fine — the script is the context.

## 6. Avoid row-wise loops

Don't loop over rows to append. Use a set operation:

```r
# good
jsdat <- jsdat |>
  bind_rows(anti_join(additions, jsdat, by = c("state", "year", "office", "dist", "name_snyder")))

# avoid
for (i in seq_len(nrow(additions))) {
  row <- additions[i, ]
  exists <- jsdat |> filter(state == row$state, ...) |> nrow() > 0
  if (!exists) jsdat <- jsdat |> add_row(!!!row)
}
```

This loop pattern exists in older scripts here. Don't copy it into new code; when editing near one, leaving it alone is fine — replacing it is an improvement, not a requirement.

If you do need an index loop, `seq_len(nrow(x))`, never `1:nrow(x)`.

## 7. Functions

Helpers live in a `00*_*-functions.R` file and are `source()`d. They exist only because they're reused.

- No roxygen blocks on internal helpers. A one-line `#` comment if the name isn't enough.
- Short, positional-first signatures with sensible defaults: `fmt_xtab(xtbl, name, long = FALSE, rowname = "year")`.
- A helper does one thing, usually a pipe in and a value out.

```r
wri_xtab <- function(tbl, name, dir = "guide/Tables/cands") {
  tbl |>
    str_remove("\\\\centering\\n") |>
    write_lines(glue("{dir}/{name}.tex"))
}
```

- Use `glue()` / `fs::path()` for constructed paths, not `paste0()`.

## 8. Comments and output

- Comment *why*, never *what*. `# Loeffler was appointed incumbent, Warnock is a challenger` earns its place; `# filter the data` does not.
- Lowercase, short, above the block they describe.
- No decorative comments, no banner blocks, no `# END` markers.
- **Comments describe the code as it is. They do not refer to earlier versions of it.** No `# previously used a loop here`, no `# switched from name to id because names broke`, no `# fixed: old join dropped rows`. A reader sees only the current script; a comparison to a draft they never saw tells them nothing, and it goes stale as the draft is forgotten. If a comment is needed, give the positive reason the current code is right:

```r
# good — the reason stands on its own
# district_id is stable across renames, so it is the join key
jsdat <- jsdat |>
  left_join(dist_xwalk, by = "district_id")

# bad — only makes sense if you saw the old draft
# switched from district name to id, names broke on 2018 renames
jsdat <- jsdat |>
  left_join(dist_xwalk, by = "district_id")
```

  Words that flag the wrong framing: `previously`, `old`, `used to`, `switched`, `replaced`, `fixed`, `no longer`, `instead of`. When one shows up, rewrite the comment to say what is true now and why. Where a change really needs a record, that record belongs in the commit message, not in the script.

- Write comments in plain Simplified Technical English (ASD-STE100), applied loosely — about 75%: one idea per comment, short sentences, active voice, present tense, the same word for the same thing throughout. Domain terms (`fixed effects`, `crosswalk`, `tibble`) stay as they are; the approved-word list is a guide, not a gate.
- Console output: single-line summaries. No multi-line `cat()` progress reports, no emoji, no "✓ Done!". If a step's effect needs to be visible, `tidylog::` already prints it.
- Inline sanity checks are welcome and should stay in the script: a bare `xtabs(~ year + party, pres_sel)`, a `count()`, a `stopifnot()`.

## 9. I/O

- `read_rds()`/`write_rds()` for intermediates, `read_csv()`/`write_csv()` for small human-readable ones, `haven::read_dta()`/`write_dta()` for Stata sources and releases.
- `show_col_types = FALSE` on `read_csv()` in pipelines.
- Write the output at the bottom of the script, unpiped and obvious.

## 10. What not to do

- Don't add `tryCatch()`, logging frameworks, config lists, or `if (interactive())` guards to a cleaning script.
- Don't convert a working flat script into functions or a package.
- Don't introduce `data.table`, base `$<-` subassignment, or `apply()` where a dplyr verb reads better.
- Don't reorder or reformat code you weren't asked to touch.
- Don't add defensive checks for conditions that can't occur given the upstream script.
