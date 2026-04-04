# Changelog — coefconv

All notable changes to this package will be documented here.
Format follows [Keep a Changelog](https://keepachangelog.com/en/1.0.0/).

---

## [1.0.0] — 2026-04-04

### Added
- `coefconv`: computes 26 marginal effect types organized in 7 families
  for every predictor in the most recently estimated regression model
- Family 1: raw slope (β), fully standardized (β*), X-standardized,
  Y-standardized
- Family 2: point elasticity at means, X-semi-elasticity,
  Y-semi-elasticity
- Family 3: basis-point effect (β ÷ 10,000), percentage-point effect
  (β ÷ 100), per-1%-of-X effect (β × X̄ ÷ 100)
- Family 4: proportional marginal effect (β / Ȳ), % of mean-Y ME
- Family 5: squared standardized coefficient (β*²), product measure
  (β × r_XY), Pratt numerator (β* × r_XY), Pratt % of R-squared
- Family 6: discrete ΔY from moving X from its median to p10, p25,
  p75, p90 (user-expandable via quantiles() option)
- Family 7: growth-rate ΔX, IQR effect, full-range effect, ±1 SD,
  ±2 SD, and custom ΔX values via delta() option
- Pratt relative-importance summary table covering all predictors
- `r()` return scalars for every key effect type per predictor
- `saving()` option to export a wide results dataset (one row per
  predictor, one column per effect type) with variable labels
- `grate()`, `quantiles()`, `delta()`, `notable`, `format()` options
- Compatibility with regress, ivregress, areg, xtreg, and any
  estimator populating e(b), e(V), e(depvar), e(sample)
- Warning for non-linear models (logit, probit, tobit, etc.) where
  OLS slopes approximate marginal effects only
- `coefconv_plot`: three simultaneously named visualization graphs
- Graph 1 (ccv_std): standardized-slope forest plot with ±z × SE(β*)
  confidence intervals, Cohen (1988) benchmark lines, two-color coding
  for significant vs. non-significant predictors
- Graph 2 (ccv_pratt): Pratt R-squared decomposition horizontal bar
  chart, sorted by absolute contribution, suppressor variables in
  distinct color
- Graph 3 (ccv_eff_[var]): per-predictor discrete-effects ladder
  chart, nine scenarios sorted by |ΔY| ascending, original Y units
- `level()`, `scheme()`, `saving()`, `noSTD`, `noPRATT`, `noEFFects`
  options for coefconv_plot
- Full SMCL help files with interpretation examples using a consistent
  wage/education scenario for all 26 effects
- Twelve worked examples in coefconv_examples.do using built-in datasets
