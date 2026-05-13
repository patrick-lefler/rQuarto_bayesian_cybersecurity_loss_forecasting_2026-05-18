# Bayesian Cybersecurity Loss Forecasting — Project Instructions

## 1. Project Overview

- **Short Description:** A Quarto-based risk quantification report that applies Bayesian inference and Monte Carlo simulation to estimate the probable annual financial loss exposure from a ransomware incident at a small- to mid-market fintech firm.

- **Description:** This project builds a reproducible, FAIR-aligned loss forecasting model for a ransomware scenario targeting a small- to mid-market financial technology firm. It uses publicly available frequency data from the Verizon Data Breach Investigations Report (DBIR) and severity data from the IBM Cost of a Data Breach Report to calibrate a Poisson frequency model and a log-normal severity model. A Monte Carlo simulation aggregates these into a distribution of annualized loss exposure, decomposed into primary and secondary loss components. The output is designed to translate technical risk estimates into board-legible financial terms — specifically a Value-at-Risk (VaR) figure directly comparable to financial market risk metrics.

- **Abstract:** To be added by author after confirmed clean build. Target: 150–250 words. Should cover: (1) the problem — qualitative risk labels fail to support budget decisions or board-level risk comparison; (2) the method — FAIR taxonomy, Poisson frequency model, log-normal severity model, primary/secondary loss decomposition, Monte Carlo simulation; (3) the data — Verizon DBIR for incident frequency, IBM Cost of a Data Breach for severity percentiles; (4) the output — annualized loss exposure distribution, 95th and 99th percentile VaR, primary vs. secondary breakdown; (5) the value — a financially quantified risk statement a board can act on.

- **Target Audience:** Board members, risk committee chairs, CISOs, and CFOs at small- to mid-market fintech firms. Assumes financial numeracy; does not assume statistical background.

- **Output Format:** HTML (self-contained)

- **Render Command:** `quarto render index.qmd`

- **Expected Output Path:** `index.html`

- **Data Sources:**
  - Verizon Data Breach Investigations Report (DBIR) — incident frequency by industry; used to parameterize the Poisson lambda for ransomware events in the financial services sector. Data extracted manually from the current report year and hardcoded as calibrated parameters.
  - IBM Cost of a Data Breach Report — severity percentiles (median and 90th percentile) by industry and breach type; used to parameterize the log-normal severity distribution. Data extracted manually and hardcoded as calibrated parameters.
  - No external API calls or live data dependencies. All parameters are hardcoded with full source citations.

- **Project Status:** [ ] In Progress / [ ] Complete / [ ] Archived

---

## 2. Directory Structure

```
bayesian-cyber-loss/
├── data/               # Parameter tables and any extracted source data
├── scripts/            # Standalone R diagnostic scripts (outside Quarto)
├── output/             # Rendered HTML output
├── _brand.yml          # Brand configuration
├── _quarto.yml         # Project-level Quarto configuration
├── INSTRUCTIONS.md     # This file
└── index.qmd           # Main Quarto entry point
```

---

## 3. Document Sections — Required & Ordered

| # | Section | Purpose | Constraints |
|---|---------|---------|-------------|
| 3.1 | Setup | Libraries, brand colors, `theme_brand()` definition | No prose; code only |
| 3.2 | Introduction | The ransomware problem for fintech firms; why qualitative risk labels fail; FAIR as the governing framework | 300–400 words; no charts |
| 3.3 | The FAIR Model | Non-technical explanation of LEF × LM; primary vs. secondary loss; where Poisson, log-normal, and Monte Carlo fit; how this project extends FAIR at the tail | 350–450 words; one summary diagram or table showing the model structure |
| 3.4 | Data & Parameter Calibration | Verizon DBIR frequency inputs; IBM severity inputs; mapping from source data to model parameters; auditable calibration table | 250–350 words; one calibration table |
| 3.5 | The Loss Model | Frequency model (Poisson); severity model (log-normal); primary/secondary split; Monte Carlo simulation engine | Code-forward; prose kept to transition paragraphs; EVT decision deferred to build |
| 3.6 | Results | ALE distribution (histogram + exceedance curve); VaR at 95th and 99th percentiles; primary vs. secondary breakdown | Minimum two interactive charts; one summary table |
| 3.7 | Insights & Conclusion | 2–3 non-obvious insights drawn from model output; conclusion paragraph | 300–400 words (insights) + 150–200 words (conclusion); no bullets |
| 3.8 | Session Information | `sessioninfo::session_info()` | `echo: false` |

---

## 4. YAML Header

```yaml
---
title: "Bayesian Cybersecurity Loss Forecasting"
subtitle: "A FAIR-Aligned Ransomware Risk Quantification Model for Fintech Firms"
author: "Patrick Lefler"
abstract: |
  [Abstract to be added by author after confirmed clean build — 150–250 words.
  See Section 1 abstract guidance for required content.]
date: "[explicit date string YYYY-MM-DD to be added by author]"
format:
  html:
    code-fold: true
    code-copy: true
    code-overflow: wrap
    code-tools: false
    code-summary: "Display code"
    df-print: kable
    embed-math: true
    embed-resources: true
    fig-align: center
    fig-height: 6
    fig-width: 10
    highlight-style: arrow
    lightbox: true
    linkcolor: "#0166CC"
    number-sections: false
    page-layout: full
    smooth-scroll: true
    theme: sandstone
    toc: true
    toc-depth: 3
    toc-location: right
    toc-title: "Contents"
execute:
  echo: true
  warning: false
  message: false
html-math-method: mathjax
knitr:
  opts_chunk:
    comment: "#>"
---
```

---

## 5. Brand & Theme Configuration

Confirm `_brand.yml` is present in project root before rendering.

**Theme brand variables:**

```yaml
color:
  palette:
    grey: "#E5E5E5"
    dark-grey: "#222222"
    date-grey: "#6E6E73"
    white: "#FFFFFF"
    off-white: "#FEFEFA"
    white-smoke: "#F5F5F5"
    link-blue: "#0166CC"
    sandstone: "#EAE5DD"
  foreground: dark-grey
  background: white
typography:
  fonts:
    - family: Roboto
      source: google
      weight: [300, 400, 500]
      style: normal
    - family: Space Mono
      source: google
  base:
    family: Roboto
    weight: 400
  headings:
    family: Roboto
    color: dark-grey
    weight: 500
  monospace: Space Mono
```

**Brand color variables (defined in Setup chunk):**

```r
brand_primary   <- "#1A1A2E"
brand_secondary <- "#16213E"
brand_accent    <- "#0F3460"
brand_highlight <- "#E94560"
brand_surface   <- "#F5F5F5"
brand_text      <- "#1A1A2E"

brand_palette <- c(
  primary   = brand_primary,
  secondary = brand_secondary,
  accent    = brand_accent,
  highlight = brand_highlight
)
```

---

## 6. Visualization Rules

- Default stack: `ggplot2` → `ggplotly()` → `plotly` direct (in that priority order)
- `scale_fill_manual()` / `scale_color_manual()` applied throughout using `brand_palette`
- **Required charts for this project:**
  - ALE distribution histogram (Section 3.6) — simulated annual loss outcomes, colored by primary vs. secondary loss; wrapped in `ggplotly()`
  - Exceedance probability curve (Section 3.6) — P(Annual Loss > x) vs. loss amount; VaR thresholds marked at 95th and 99th percentile
  - Primary vs. secondary loss breakdown (Section 3.6) — stacked bar or side-by-side comparison showing decomposition of expected loss
- Chart captions carry independent narrative weight — do not restate axis labels; add context the chart alone cannot convey

---

## 7. Table Rules

- Default: `kable` + `kableExtra` → `gt` → `DT::datatable()`
- **Required tables for this project:**
  - Parameter calibration table (Section 3.4) — source, parameter name, value, and citation for every model input
  - VaR summary table (Section 3.6) — ALE at mean, 50th, 75th, 95th, and 99th percentiles, split by primary and secondary loss
- Default `kable` setup:

```r
kable(
  data,
  format    = "html",
  digits    = 3,
  caption   = "Table N: [Description]",
  col.names = c("Col 1", "Col 2", "Col 3")
) |>
  kable_styling(
    bootstrap_options = c("striped", "hover", "condensed"),
    full_width        = TRUE,
    position          = "left",
    font_size         = 13
  )
```

- `DT::datatable()` only when explicitly requested

---

## 8. R Libraries

```r
# --- Default libraries (all projects) ---
library(kableExtra)   # Table formatting
library(knitr)        # Document rendering
library(plotly)       # Interactive chart wrapping
library(scales)       # Axis and label formatting
library(sessioninfo)  # Session provenance
library(tidyverse)    # Data manipulation and ggplot2

# --- Project-specific libraries ---
library(fitdistrplus) # Maximum likelihood fitting for log-normal severity parameters
library(mc2d)         # Two-dimensional Monte Carlo simulation support
library(evd)          # Extreme value distributions (GEV, GPD); loaded for EVT extension if adopted
library(actuar)       # Heavy-tailed distributions and loss modeling utilities
library(patchwork)    # Chart composition for multi-panel result figures
```

**Notes:**
- `mc2d` provides the primary Monte Carlo infrastructure; a manual simulation loop is an acceptable fallback if `mc2d` introduces rendering complexity.
- `evd` is loaded speculatively for the EVT decision in Section 3.5. If EVT is not adopted during the build, remove it from the Setup chunk and this list.
- `actuar` provides the Poisson-inverse Gaussian and other compound distributions that may be useful for sensitivity analysis.

---

## 9. Writing Standards

All Claude-generated written content — including body prose, captions, abstract, README, and LinkedIn post — conforms to the following standards without prompting.

- **Voice:** Third person, direct, precise.
- **Grammar:** Vary sentence length deliberately. Avoid the flat, homogeneous rhythm of AI-generated prose.
- **Structure:** Lead with the non-obvious insight, not the methodology. One idea per paragraph. No bullet-point dumps masquerading as analysis. Lists for genuinely enumerable items only. Minimize em-dashes.

**Banned vocabulary:** delve, leverage (as verb), harness, unlock, seamlessly, robust (outside statistical context), transformative, elevate, navigate (as metaphor), landscape (as metaphor), ecosystem (as metaphor), paradigm, game-changer, cutting-edge, state-of-the-art, empower, innovative (unless citing a specific novel technique), holistic, synergy, streamline, deep dive (as noun), unpack (as metaphor), it's worth noting, it is important to note, in today's rapidly evolving world, in conclusion (never opens a closing paragraph).

**Domain terminology — use consistently:**
- FAIR, not "FAIR framework" or "the FAIR method" on second reference — just FAIR
- Loss Event Frequency (LEF) — spell out on first use, abbreviate thereafter
- Loss Magnitude (LM) — spell out on first use, abbreviate thereafter
- Annualized Loss Expectancy (ALE) — spell out on first use, abbreviate thereafter
- Primary loss / secondary loss — lowercase, no hyphen
- Value-at-Risk (VaR) — spell out on first use; use VaR thereafter
- Monte Carlo simulation — capitalize Monte Carlo, lowercase simulation
- Ransomware — one word, no hyphen
- Fintech — lowercase unless starting a sentence

**Number formatting:**
- Percentages: one decimal place in body text (99.2%, not 99%)
- Monetary values: $X,XXX format for thousands; $X.XM format for millions
- Statistical intervals: bracket notation [lower, upper]
- Loss percentiles: stated as "95th percentile ALE" not "ALE at 95%"

**Figures and tables:** Captions add information not visible in the chart or table itself. Assume numeracy; do not over-explain the math.

---

## 10. Deliverables Checklist

- [ ] `index.qmd` — primary rendered document
- [ ] `_brand.yml` — confirmed in project root
- [ ] `_quarto.yml` — confirmed in project root
- [ ] `README.md` — complete
- [ ] Abstract — embedded in YAML, 150–250 words, added by author post-build
- [ ] Date — explicit `YYYY-MM-DD` string added by author post-build
- [ ] Output HTML — confirmed self-contained (`embed-resources: true`)
- [ ] LinkedIn post — if requested

---

## 11. README.md File

When requested, the README.md shall include:

```
# Bayesian Cybersecurity Loss Forecasting

> A FAIR-aligned Monte Carlo loss model quantifying ransomware risk exposure for fintech firms in financial terms a board can act on.

Author: Patrick Lefler
Published: [date based on rendered output]
Rendered: [leave blank]

## Introduction
> A Poisson-log-normal Monte Carlo simulation calibrated to Verizon DBIR and IBM Cost of a Data Breach data, producing a VaR-style annualized loss distribution for a ransomware scenario at a small- to mid-market fintech firm.

## Overview
[3–4 sentences covering: FAIR taxonomy as the conceptual structure; Poisson frequency + log-normal severity as the distributional choices; primary/secondary loss decomposition; Monte Carlo as the aggregation engine; VaR as the board-facing output metric.]

## Tech Stack
- Language: R
- Framework: Quarto (https://quarto.org/)
- Primary Libraries: tidyverse, fitdistrplus, mc2d, plotly, kableExtra
- Output: Self-contained HTML report

## Repository Structure
├── data/               # Calibrated parameter inputs
├── scripts/            # Standalone diagnostic R scripts
├── output/             # Rendered HTML
├── _brand.yml          # Brand configuration
├── _quarto.yml         # Project configuration
└── index.qmd           # Main Quarto entry point

## Key Findings
[2–3 key findings — to be completed after confirmed clean build]

## License
This project is licensed under the MIT License. See the LICENSE file for details.

## Contact
Patrick Lefler | https://www.linkedin.com/in/patricklefler/ | patrick-lefler.github.io | https://substack.com/@pflefler
```

---

## 12. Open Issues & Decisions Log

- [2026-05-13] — EVT adoption decision deferred to Section 3.5 (The Loss Model) build. If the log-normal tail fit is adequate for the scenario's severity range, EVT will be noted as a documented extension rather than implemented. If adopted, Peaks-Over-Threshold (GPD) is the preferred approach over a global heavy-tailed fit.
- [2026-05-13] — `mc2d` vs. manual Monte Carlo loop: preference is `mc2d` for conciseness, but a manual loop using `replicate()` or `purrr::map_dbl()` is the fallback if `mc2d` introduces rendering or dependency complexity.
- [2026-05-13] — Primary/secondary loss split ratio: the IBM report provides aggregate severity figures; the primary/secondary decomposition will require a documented assumption about the proportion of secondary loss (regulatory fines, litigation, reputational costs). This assumption should be stated explicitly in Section 3.4 with a sensitivity note.
- [2026-05-13] — Abstract and date fields: left blank in YAML pending confirmed clean build. Author completes both fields before final publication.

---

## 13. Change Log

- [2026-05-13] — INSTRUCTIONS.md created. Project scoped to ransomware scenario, small- to mid-market fintech firm. Data sources confirmed as Verizon DBIR and IBM Cost of a Data Breach Report. Section structure finalized. EVT decision deferred to build phase.
