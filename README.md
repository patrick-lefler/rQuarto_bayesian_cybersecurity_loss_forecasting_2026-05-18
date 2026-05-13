# Bayesian Cybersecurity Loss Forecasting

> A FAIR-aligned Monte Carlo model quantifying ransomware loss exposure for fintech firms — translating frequency and severity data from DBIR and IBM into a board-legible VaR distribution.

**Author:** Patrick Lefler
**Published:** 2026-05-13
**Rendered:**

---

## Introduction

Most fintech risk registers treat ransomware as a qualitative category. This model replaces the label with a loss curve — a full distribution of probable annual financial outcomes, expressed in dollars, with Value-at-Risk figures a CFO can act on.

---

## Overview

The model applies the Factor Analysis of Information Risk (FAIR) framework to a ransomware scenario at a small- to mid-market fintech firm. Loss Event Frequency is modeled as a Poisson random variable parameterized from Verizon's 2024 Data Breach Investigations Report. Loss Magnitude is modeled as a log-normal random variable with parameters back-solved from IBM's 2024 Cost of a Data Breach Report. Following FAIR's taxonomy, per-event loss is decomposed into primary loss (incident response, forensics, system restoration) and secondary loss (regulatory fines, litigation, breach notification, and lost business). A 100,000-trial Monte Carlo simulation aggregates these components into a full Annualized Loss Expectancy (ALE) distribution with VaR figures at the 95th and 99th percentiles.

---

## Key Findings

- **Secondary loss dominates at every percentile.** At the 95th percentile, secondary loss ($9.5M) is 1.7× primary loss ($5.7M). The gap widens further at the tail, reflecting the heavier regulatory and litigation exposure financial services firms carry relative to their direct response costs.

- **The VaR 99/mean ALE ratio is approximately 10.6×.** At $28.6M VaR 99 against a $2.7M mean, this distribution carries extreme tail leverage relative to credit or market risk benchmarks. Conventional operational risk reserve methodologies will systematically undercapitalize against a loss profile of this shape.

- **The median ALE of $0.0M is a governance warning, not a reassurance.** At λ = 0.30, 74.1% of simulated years produce no event. Boards that anchor on the median miscalibrate their risk posture in precisely the way that precedes control atrophy and budget compression during quiet periods.

---

## Tech Stack

- **Language:** R
- **Framework:** [Quarto](https://quarto.org/)
- **Primary Libraries:** tidyverse, fitdistrplus, mc2d, plotly, kableExtra, patchwork
- **Output:** Self-contained HTML report (`embed-resources: true`)

---

## Data Sources

| Source | Use | Year |
|--------|-----|------|
| [Verizon Data Breach Investigations Report](https://www.verizon.com/business/resources/reports/dbir/) | Ransomware incident frequency — financial sector | 2024 |
| [IBM Cost of a Data Breach Report](https://www.ibm.com/reports/data-breach) | Loss severity percentiles — financial services industry | 2024 |

No proprietary data, internal loss histories, or undocumented assumptions are used. All parameters are hardcoded with full source citations and auditable in the document's Data & Parameter Calibration section.

---

## Repository Structure

```
bayesian-cyber-loss/
├── data/               # Calibrated parameter inputs
├── scripts/            # Standalone R diagnostic scripts
├── output/             # Rendered HTML report
├── _brand.yml          # Brand configuration (Roboto, Space Mono, brand palette)
├── _quarto.yml         # Project-level Quarto configuration
├── INSTRUCTIONS.md     # Project standards, YAML constraints, writing rules
└── index.qmd           # Main Quarto entry point
```

---

## Model Structure

| FAIR Component | Distribution | Parameters |
|----------------|-------------|------------|
| Loss Event Frequency (LEF) | Poisson | λ = 0.30 (base case) |
| Primary Loss Magnitude | Log-normal | P₅₀ = $2.6M, P₉₀ = $6.5M |
| Secondary Loss Magnitude | Log-normal | P₅₀ = $3.5M, P₉₀ = $12.2M |
| Simulation trials | Monte Carlo | n = 100,000, seed = 42 |

A sensitivity sweep across λ ∈ {0.10, 0.30, 0.50} is included in the Results section to show how the ALE distribution responds to changes in the firm's control posture.

---

## License

This project is licensed under the MIT License. See the LICENSE file for details.

---

## Contact

Patrick Lefler
[LinkedIn](https://www.linkedin.com/in/patricklefler/) | [GitHub Pages](https://patrick-lefler.github.io) | [Substack](https://substack.com/@pflefler)
