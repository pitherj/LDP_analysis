# LDP Analysis

Statistical analysis of FAIR compliance scores for publications authored by Living Data Project (LDP) trainees versus matched controls, implementing the pre-registered inter-rater reliability and hypothesis-testing workflow.

---

## Background

The [Living Data Project (LDP)](https://www.ciee-icee.ca/ldp.html) is a Canada-wide graduate training program in open science, offered through the Canadian Institute of Ecology and Evolution (CIEE). This repository contains the analysis code for a quasi-experimental matched-groups study evaluating whether LDP training improved the FAIR compliance of graduates' published research data.

FAIR compliance (Findable, Accessible, Interoperable, Reusable) is assessed on a 0–4 scale by four independent raters using a standardised checklist. The primary outcome is the mean rater score per publication. Each LDP publication is matched within institution and publication year to a comparator publication from a non-LDP graduate student.

The full analysis plan is described in the [pre-registration on OSF](https://osf.io/uyqt4/files/xjmyg). It is also available [here](https://github.com/pitherj/LDP_pre-registration/blob/main/LDP_preregistration_OSF.md).  

The FAIR compliance checklist is available [here](https://github.com/pitherj/LDP_pre-registration/blob/main/FAIR-compliance-checklist.md).

**NOTE**: minor deviations from the pre-registration include (i) eliminating 3 pairs of papers because they were found to be ineligible (hence N = 19 pairs instead of 21); and (ii) using the permutation test rather than paired t-test as the primary method for the main analysis (but the latter is also reported).   

---

## Quick Start

```r
# 1. Clone the repository
# 2. Open LDP_analysis.Rproj in RStudio
# 3. Restore the R package library
renv::restore()
# 4. Open and render scripts/full_workflow.qmd
```

---

## Prerequisites

### Software

| Requirement | Version used | Notes |
|---|---|---|
| R | 4.5.2 | [cran.r-project.org](https://cran.r-project.org) |
| Quarto | ≥ 1.4 | [quarto.org](https://quarto.org) |
| RStudio | any recent | Recommended IDE; not required |

### R packages

R package dependencies are managed via `renv`. Restore the project library before first use:

**macOS / Linux**
```r
install.packages("renv")  # if not already installed
renv::restore()
```

**Windows**
```r
install.packages("renv")  # if not already installed
renv::restore()
```

> **macOS**: Packages with compiled code may require Xcode Command Line Tools (`xcode-select --install`).  
> **Windows**: Some packages with compiled code require [Rtools](https://cran.r-project.org/bin/windows/Rtools/) — match the Rtools version to your R version.

Key packages (all locked in `renv.lock`):

| Package | Purpose |
|---|---|
| `readr` | CSV import |
| `dplyr`, `tidyr`, `purrr` | Data wrangling |
| `irr` | Inter-rater reliability (ICC, Krippendorff's α, Fleiss' κ) |
| `effectsize` | Cohen's *d* |
| `ggplot2` | Visualisation |
| `here` | Portable file paths |
| `DiagrammeR` | PRISMA-style record-flow diagram (Graphviz) |
| `DiagrammeRsvg`, `rsvg` | SVG / PDF export of the flow diagram |

### Data

The four rater evaluation CSV files must be placed in `data/data_raw/` (see [Project Structure](#project-structure) below). The private pairing key (`rater_key_final_v2.csv`) must be placed in `data/data_raw/data_private/`. This folder is git-ignored; contact the project lead to obtain it.

---

## Project Structure

```
LDP_analysis/
├── data/
│   ├── data_raw/               # Raw rater evaluation files (one per rater)
│   │   ├── data_private/       # Git-ignored; contains identifiable pairing key
│   │   ├── rater_publications_final_JP.csv
│   │   ├── rater_publications_final_v2_DH.csv
│   │   ├── rater_publications_final_v2_JR_scored.csv
│   │   └── rater_publications_full-SE.csv
│   └── data_processed/         # Derived files written by full_workflow.qmd
│       ├── pairs_lookup.csv    # Non-sensitive pairing lookup (pub_id, pair_id, group)
│       └── rater_scores_wide.csv  # FAIR scores in wide format (one row per publication)
├── scripts/
│   ├── full_workflow.qmd       # Main Quarto document: PRISMA diagram + full analysis pipeline
│   ├── full_workflow.html      # Rendered analysis report (self-contained)
│   ├── prisma_flow.qmd         # Standalone PRISMA-style record-flow diagram
│   ├── prisma_flow.html        # Rendered standalone diagram (self-contained)
│   ├── prisma_flow.svg         # Vector export of the flow diagram
│   └── prisma_flow.pdf         # PDF export of the flow diagram
├── renv/                       # renv package library (git-ignored)
├── renv.lock                   # Locked package versions (committed)
├── LDP_analysis.Rproj          # RStudio project file
└── LICENSE                     # GNU GPL v2
```

---

## Analysis Workflow

All analysis is contained in `scripts/full_workflow.qmd`. The document is structured as follows:

1. **Summary of data screening** — PRISMA-style flow diagram tracing both publication pools from OpenAlex retrieval through all filtering, matching, and post-hoc correction steps to the final 19-pair rater set (rendered via `DiagrammeR`).
2. **Import private key** (hidden chunk) — reads `rater_key_final_v2.csv`, strips to `pub_id`, `pair_id`, `group`, writes `pairs_lookup.csv`, then removes the object from the environment.
3. **Import rater data** — reads all four rater CSVs; confirms 38 publications are common to all four raters.
4. **Reformat to wide** — aligns per-rater scores into a single wide tibble (`scores_wide`); writes `rater_scores_wide.csv`.
5. **Inter-rater reliability** — ICC (two-way mixed, absolute agreement, average of *k* = 4 raters); Krippendorff's α (interval scale); percent agreement.
6. **Primary analysis** — computes mean FAIR score per publication, joins pairing lookup, calculates paired differences (*D* = LDP − Comparator), runs a one-sided permutation test (seed = 20260329, *n* = 1000).
7. **Secondary analysis** — one-sided paired *t*-test; Cohen's *d* via `effectsize`.
8. **Visualisation** — paired plot (one line per matched pair).
9. **Post-hoc sensitivity analysis** — assessment of rater influence on results.

A standalone version of the PRISMA diagram (with extended documentation of data sources and filtering layer descriptions) is available in `scripts/prisma_flow.qmd`. Rendering that file also exports `prisma_flow.svg` and `prisma_flow.pdf` for use in manuscripts.

Render by opening `full_workflow.qmd` in RStudio and clicking Render, or from the terminal:

```bash
quarto render scripts/full_workflow.qmd
```

---

## Key Outputs

| Output | Location | Description |
|---|---|---|
| `full_workflow.html` | `scripts/` | Self-contained rendered analysis report (includes PRISMA diagram) |
| `prisma_flow.html` | `scripts/` | Standalone self-contained PRISMA record-flow diagram |
| `prisma_flow.svg` | `scripts/` | Vector (SVG) export of the PRISMA diagram |
| `prisma_flow.pdf` | `scripts/` | PDF export of the PRISMA diagram |
| `pairs_lookup.csv` | `data/data_processed/` | Non-sensitive pairing key |
| `rater_scores_wide.csv` | `data/data_processed/` | Wide-format FAIR scores (38 publications × 4 raters) |

---

## Documentation

| Document | Location | Description |
|---|---|---|
| Pre-registration | [OSF](https://osf.io/uyqt4/files/xjmyg) | Full analysis plan, hypotheses, and IRR decision rules |
| FAIR compliance checklist | `LDP_pre-registration/FAIR-compliance-checklist.md` | Rating instrument used by all raters |
| Data dictionary (raw) | `data/data_raw/DATA-DICTIONARY.md` | File-level and column-level schema for rater files |
| Data dictionary (processed) | `data/data_processed/DATA-DICTIONARY.md` | Schema for derived analysis files |

---

## How to Cite

[TODO: add citation once manuscript is published]

If referencing the pre-registration:

> Jason Pither, Diane Srivastava, David AGA Hunt, Sandra Emry, Jessica Reemeyer, and Mathew Vis-Dunbar. (2026). *Assessing Open Science Practices Among Graduates of the Living Data Project, a Canada-wide Graduate Training Program*. OSF Pre-registration. https://osf.io/uyqt4/files/xjmyg

---

## Authors

**Lead author and maintainer**

Jason Pither — [0000-0002-7490-6839](https://orcid.org/0000-0002-7490-6839)  
Department of Biology, University of British Columbia Okanagan  
Email: jason [dot] pither <at> ubc [dot] ca

**Co-investigators**

| Name | ORCID |
|---|---|
| Mathew Vis-Dunbar | [0000-0001-6541-9660](https://orcid.org/0000-0001-6541-9660) |
| Diane Srivastava | [0000-0003-4541-5595](https://orcid.org/0000-0003-4541-5595) |

**Raters** (independent FAIR compliance scoring)

| Name | ORCID | Rater ID in data |
|---|---|---|
| Jason Pither | [0000-0002-7490-6839](https://orcid.org/0000-0002-7490-6839) | JP |
| David Hunt | [0000-0002-7771-8569](https://orcid.org/0000-0002-7771-8569) | DH |
| Sandra Emry | [0000-0001-6882-2105](https://orcid.org/0000-0001-6882-2105) | SE |
| Jessica Reemeyer | — | JR |

---

## Ethics

This study received ethics approval from the UBC Behavioural Research Ethics Board (UBC BREB) on 2026-01-06.

---

## License

Code in this repository is licensed under the [GNU General Public License v2](LICENSE).

---

## Acknowledgments

- Funded through the Canadian Institute of Ecology and Evolution (CIEE) NSERC CREATE program
- Some coding assisted by [Claude Code](https://claude.ai/code) (Anthropic)
