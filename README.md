# Tejas Wavde

Data scientist working on paid-media measurement, experimentation, causal inference, product analytics, and time-series forecasting.

The repositories below are the work I keep public: case studies on how I measure channels and features, a small Python package, and short essays on the methods. The focus is on calculations that have to be defended in writing, which means clean estimators, honest diagnostics, and memos that state their assumptions.

## Featured projects

- [**paid-media-playbook**](https://github.com/wavde/paid-media-playbook): five case studies on paid-media measurement — DMA geo lift (synthetic control), multi-touch attribution (rule-based vs Markov vs Shapley), matched-market incrementality / conversion lift, media-mix modeling (MAP + Bayesian side-by-side), and a web-attribution deep dive that accounts for cookie loss and iOS ATT. Synthetic data, recovered ground truth, decision memos.
- [**causal-inference-playbook**](https://github.com/wavde/causal-inference-playbook): worked case studies covering A/B with CUPED, synthetic control, difference-in-differences (including Callaway-Sant'Anna), propensity score matching with a LaLonde replication, sequential testing, and switchback under SUTVA violation. Each paired with a memo.
- [**demand-forecasting**](https://github.com/wavde/demand-forecasting): daily demand forecasting on UCI Online Retail II — seasonal-naive baseline, ETS / SARIMA / LightGBM, and a leakage-free rolling-origin backtest. SARIMA reaches MASE 0.74, 18.5% below baseline; a two-way fixed-effects difference-in-differences recovers a known +15% promo uplift. [Write-up →](https://wavde.github.io/demand-forecasting.html)
- [**experiment-toolkit**](https://github.com/wavde/experiment-toolkit): a small, typed, tested Python package on PyPI. Sample size and MDE, CUPED, mSPRT, ratio-metric delta method, staggered DiD, E-values, Rosenbaum bounds.
- [**product-analytics-deepdive**](https://github.com/wavde/product-analytics-deepdive): SQL-first case studies. Funnels, retention cohorts, engagement segmentation, and a north-star metric memo.
- [**analytics-sandbox**](https://github.com/wavde/analytics-sandbox): 18 SQL interview-style problems with production framing and dialect notes.

## Writing

Short essays on methods that show up in the case studies:

- [Last-touch attribution systematically under-credits upper funnel](writing/last-touch-under-credits-upper-funnel.md)
- [Geo holdouts as paid-media A/B](writing/geo-holdouts-as-paid-media-ab.md)
- [MMM vs incrementality: what each answers and what neither does](writing/mmm-vs-incrementality.md)
- [When TWFE lies: staggered rollouts and what Callaway-Sant'Anna buys you](writing/twfe-lies-on-staggered-rollouts.md)
- [E-values: a one-number sensitivity check for observational memos](writing/e-values-for-observational-memos.md)
- [CUPED inside sequential testing: two variance reductions that compound](writing/cuped-inside-sequential-testing.md)
- [Finding (and fixing) a 17% Type-I error bug in an mSPRT implementation](writing/finding-and-fixing-msprt-bug.md)
- [LaLonde, revisited: what a replication tells you about PSM, IPW, and AIPW](writing/what-lalonde-taught-me.md)
- [Picking a north-star metric when every candidate has a tradeoff](writing/picking-a-north-star-metric.md)

## Side projects

- [**games**](https://github.com/wavde/games): twelve classic and logic-puzzle browser games — vanilla HTML/CSS/JS, no build. Generators, solvers, daily-seed puzzles. [Play →](https://wavde.github.io/games/)

## Elsewhere

- Site: [wavde.github.io](https://wavde.github.io)
- GitHub: [@wavde](https://github.com/wavde)
