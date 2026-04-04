# coefconv — Comprehensive Marginal Effects for Stata

**Version 1.0.0 · April 2026 · Stata 14+**

---

## Overview

`coefconv` computes 26 marginal effect types for every predictor in the most
recently estimated regression model, organized into 7 interpretation families.
A companion program `coefconv_plot` visualizes the results with three named graphs.

All inputs are drawn automatically from Stata's `e()` results and restricted to
the estimation sample — no data preparation is required.

---

## Installation

```stata
ssc install coefconv, replace
```

`ssc install` places all files in Stata's `PLUS` directory.
This matters: `PLUS` is managed by `ado update`, so whenever a bug fix
or new version is released on SSC, running `ado update` will
automatically download it.

> **Do not copy files manually into `PERSONAL`.**
> The `PERSONAL` directory precedes `PLUS` on Stata's adopath.
> Any files placed there will occlude the SSC copy, meaning
> `ado update` can never deliver a bug fix — users would be
> silently stuck on the old version even after updating.

Verify installation:

```stata
which coefconv
help coefconv
```

---

## Quick Start

```stata
sysuse auto, clear
regress price mpg weight foreign

* Full marginal effects table
coefconv

* Visualization (three named graphs)
coefconv_plot
```

---

## Programs

### `coefconv`

Computes 26 effect types per predictor and prints a structured output table
followed by a Pratt relative-importance summary across all predictors.

**Syntax**

```stata
coefconv [, grate(#) quantiles(numlist) delta(numlist)
          saving(filename[, replace]) notable format(fmt)]
```

**Options**

| Option | Default | Description |
|---|---|---|
| `grate(#)` | `0.01` | Growth rate for default ΔX = grate × X̄ |
| `quantiles(numlist)` | `10 25 50 75 90` | Percentiles for Family 6 quantile displacements |
| `delta(numlist)` | — | Custom ΔX values applied to every predictor |
| `saving(file[, replace])` | — | Save wide results dataset (one row per predictor) |
| `notable` | — | Suppress all display; useful for programmatic access |
| `format(fmt)` | `%12.6f` | Stata number format for output |

**Returned scalars** (accessible via `r()` after running)

```stata
r(N)                  // observations
r(r2)                 // R-squared
r(ymean)              // mean of Y
r(ysd)                // SD of Y
r(pratt_tot)          // sum of Pratt numerators (= R-squared)
r(b_varname)          // raw slope per predictor
r(bstd_varname)       // standardized slope per predictor
r(elas_varname)       // elasticity at means per predictor
r(ysemi_varname)      // Y-semi-elasticity per predictor
r(pratt_pct_varname)  // Pratt % of R-squared per predictor
```

---

### `coefconv_plot`

Produces three simultaneously open named graphs assessing statistical and
practical meaningfulness of each coefficient.

**Syntax**

```stata
coefconv_plot [, grate(#) level(#) scheme(str)
               saving(stub[, replace])
               noSTD noPRATT noEFFects]
```

**Options**

| Option | Default | Description |
|---|---|---|
| `grate(#)` | `0.01` | Growth rate passed to `coefconv` internally |
| `level(#)` | `95` | Confidence level for forest plot CIs |
| `scheme(str)` | — | Stata graph scheme |
| `saving(stub[, replace])` | — | Saves `stub_std.gph`, `stub_pratt.gph`, `stub_eff_X.gph` |
| `noSTD` | — | Skip the standardized slopes forest plot |
| `noPRATT` | — | Skip the Pratt importance bar chart |
| `noEFFects` | — | Skip the per-variable discrete effects plots |

**Graph names** (remain open simultaneously in Stata's Graph window)

| Graph | Name | Bring to front |
|---|---|---|
| Standardized slopes forest | `ccv_std` | `graph display ccv_std` |
| Pratt importance | `ccv_pratt` | `graph display ccv_pratt` |
| Discrete effects for `weight` | `ccv_eff_weight` | `graph display ccv_eff_weight` |

---

## Effect Families

| Family | Effects included |
|---|---|
| **1 — Raw & Standardized** | β, β\* (fully std.), X-std., Y-std. |
| **2 — Elasticity** | Point elasticity at means, X-semi-elas., Y-semi-elas. |
| **3 — Basis-Point & Pct-Point** | β ÷ 10,000; β ÷ 100; β × X̄ ÷ 100 |
| **4 — Relative & Proportional** | β / Ȳ; (β / Ȳ) × 100 |
| **5 — Variance & Importance** | β\*², β × r; Pratt numerator; Pratt % of R² |
| **6 — Quantile Displacements** | ΔY from median to each percentile (default: p10, p25, p75, p90) |
| **7 — Discrete Change** | Growth-rate ΔX; IQR; full range; ±1 SD; ±2 SD; custom Δ |

---

## Examples

```stata
* ── Basic use ──────────────────────────────────────────────────────────────
sysuse auto, clear
regress price mpg weight foreign
coefconv

* ── Change growth rate and add extreme quantiles ────────────────────────────
coefconv, grate(0.05) quantiles(5 95)

* ── Custom delta-X scenarios (e.g. +500 and +2000 units) ───────────────────
coefconv, delta(500 2000)

* ── Save wide results for downstream analysis ───────────────────────────────
coefconv, saving(coefconv_results, replace)

* ── Access returned scalars programmatically ────────────────────────────────
coefconv, notable
display "Elasticity of mpg:    " r(elas_mpg)
display "Pratt% of weight:     " r(pratt_pct_weight)
display "Std. slope of foreign:" r(bstd_foreign)

* ── Visualization ───────────────────────────────────────────────────────────
coefconv_plot
coefconv_plot, noeffects                         // summary graphs only
coefconv_plot, level(90) grate(0.10)             // 90% CI, 10% growth rate
coefconv_plot, saving(auto_plots, replace)       // save all graphs
graph display ccv_std                          // bring forest plot to front

* ── After other estimators ──────────────────────────────────────────────────
ivregress 2sls price (mpg = gear_ratio) weight foreign
coefconv, grate(0.03)

sysuse nlsw88, clear
areg wage age hours tenure, absorb(industry)
coefconv_plot
```

---

## Compatibility

`coefconv` and `coefconv_plot` work with any estimator that populates:

- `e(b)` — coefficient vector
- `e(V)` — variance-covariance matrix
- `e(depvar)` — dependent variable name
- `e(sample)` — estimation sample marker

This covers `regress`, `ivregress`, `areg`, `xtreg`, `xtgls`, `reg3`, and
most other linear estimators. A warning is displayed if a non-linear estimator
(e.g. `logit`, `probit`) is detected, since OLS slopes are not marginal effects
in those models.

Factor variables (e.g. `i.region`) are skipped in Families 2–7 since they
cannot be summarized by a single mean and SD. Their raw β appears in Family 1.

---

## Interpretation Guide

See `help coefconv` for a complete interpretation guide with numerical examples
for all 26 effect types, using a consistent wage/education scenario throughout.

A coefficient is **meaningful** on two independent dimensions:

- **Statistical** — the CI does not cross zero (Graph 1 forest plot)
- **Practical** — β\* clears Cohen's threshold, Pratt% is substantial,
  and ΔY under the IQR scenario is large relative to Ȳ or field benchmarks

The most meaningful predictors satisfy all criteria simultaneously.

---

## Known Limitations

- Results depend on the estimation sample defined by `e(sample)`. Ensure
  the correct estimation has been run before calling `coefconv`.
- Elasticities and proportional effects are undefined when Ȳ = 0.
- Standardized slopes require σX > 0 and σY > 0.
- `coefconv_plot` requires `coefconv` to be installed in the same adopath.

---

## Citation

If you use `coefconv` in published research, please cite:

> Arshed, N. (2026). *coefconv: Comprehensive Marginal Effects for Stata.*
> Statistical Software Components [number], Boston College Department of Economics.
> Available from https://ideas.repec.org/c/boc/bocode/[number].html

---

## Version History

| Version | Date | Notes |
|---|---|---|
| 1.0.0 | April 2026 | Initial release |

---

## Author

**Dr Noman Arshed**  
Senior Lecturer, Department of Business Analytics  
Sunway Business School, Sunway University  
nouman.arshed@gmail.com

Bug reports and suggestions welcome.
