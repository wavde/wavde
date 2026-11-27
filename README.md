# Tejas Wavde

Data scientist working on paid-media measurement, experimentation, causal inference, product analytics, and time-series forecasting.

The repositories below are the work I keep public: case studies on how I measure channels and features, a small Python package, and short essays on the methods. The focus is on calculations that have to be defended in writing, which means clean estimators, honest diagnostics, and memos that state their assumptions.

## Featured projects

- [**paid-media-playbook**](https://github.com/wavde/paid-media-playbook): seven case studies on paid-media measurement — DMA geo lift (synthetic control), multi-touch attribution (rule-based vs Markov vs Shapley), matched-market incrementality / conversion lift, media-mix modeling (MAP + Bayesian side-by-side), a web-attribution deep dive that accounts for cookie loss and iOS ATT, measurement when the ad platform's model controls targeting, creative, and pacing, and propensity vs uplift targeting on the public Criteo randomized uplift dataset. Synthetic data with recovered ground truth (plus one public randomized dataset), and decision memos.
- [**causal-inference-playbook**](https://github.com/wavde/causal-inference-playbook): worked case studies covering A/B with CUPED, synthetic control, difference-in-differences (including Callaway-Sant'Anna), propensity score matching with a LaLonde replication, sequential testing, switchback under SUTVA violation, observational estimates for an opt-in AI feature, and staggered rollout vs fixed holdout for measuring one. Each paired with a memo.
- [**demand-forecasting**](https://github.com/wavde/demand-forecasting): daily demand forecasting on UCI Online Retail II — seasonal-naive baseline, ETS / SARIMA / LightGBM, and a leakage-free rolling-origin backtest. SARIMA reaches MASE 0.74, 18.5% below baseline; a two-way fixed-effects difference-in-differences recovers a known +15% promo uplift. A zero-shot Chronos-Bolt comparison on the same backtest shows where a pretrained forecaster wins (short history) and where it doesn't. [Write-up →](https://wavde.github.io/demand-forecasting.html)
- [**experiment-toolkit**](https://github.com/wavde/experiment-toolkit): a small, typed, tested Python package on PyPI. Sample size and MDE, CUPED, mSPRT, ratio-metric delta method, staggered DiD, E-values, Rosenbaum bounds. Unreleased on main: PPI++ for A/B tests scored by an LLM judge, PPI++ with a pre-treatment covariate and closed-form label budgets, and a simulation of when offline LLM evals predict online A/B results.
- [**product-analytics-deepdive**](https://github.com/wavde/product-analytics-deepdive): SQL-first case studies. Funnels, retention cohorts, engagement segmentation, and a north-star metric memo.
- [**analytics-sandbox**](https://github.com/wavde/analytics-sandbox): 18 SQL interview-style problems with production framing and dialect notes.

## Writing

Short essays on methods that show up in the case studies:

- [A staggered rollout can't see novelty: holdout vs rollout for opt-in AI features](writing/staggered-rollout-vs-holdout.md)
- [A covariate stacks on the judge, but a better judge beats both: cutting the label budget of PPI++](writing/ppi-with-covariates-label-budget.md)
- [The targeting model already knows who would convert: propensity vs uplift on real randomized data](writing/propensity-vs-uplift-targeting-criteo.md)
- [An offline eval win is evidence about the eval set: offline LLM evals vs online A/B](writing/offline-evals-vs-online-ab.md)
- [Measuring marketing when the algorithm runs the campaign](writing/ai-mediated-marketing-measurement.md)
- [Opt-in AI features: when an observational impact estimate is usable](writing/opt-in-ai-adoption-observational-bias.md)
- [Pretrained forecasters vs SARIMA: when zero-shot wins, and when it doesn't](writing/pretrained-forecasters-vs-sarima.md)
- [LLM-as-judge is a noisy metric: what it does to A/B power and bias](writing/llm-judge-noise-in-ab-tests.md)
- [When boosting loses to SARIMA: baselines, MASE, and honest backtests](writing/when-boosting-loses-to-sarima.md)
- [Last-touch attribution systematically under-credits upper funnel](writing/last-touch-under-credits-upper-funnel.md)
- [Geo holdouts as paid-media A/B](writing/geo-holdouts-as-paid-media-ab.md)
- [MMM vs incrementality: what each answers and what neither does](writing/mmm-vs-incrementality.md)
- [When TWFE lies: staggered rollouts and what Callaway-Sant'Anna buys you](writing/twfe-lies-on-staggered-rollouts.md)
- [E-values: a one-number sensitivity check for observational memos](writing/e-values-for-observational-memos.md)
- [CUPED inside sequential testing: two variance reductions that compound](writing/cuped-inside-sequential-testing.md)
- [Finding (and fixing) a 17% Type-I error bug in an mSPRT implementation](writing/finding-and-fixing-msprt-bug.md)
- [LaLonde, revisited: what a replication tells you about PSM, IPW, and AIPW](writing/what-lalonde-taught-me.md)
- [Picking a north-star metric when every candidate has a tradeoff](writing/picking-a-north-star-metric.md)

Next up: [contribution plan](contribution-plan.md). Eight loops are done: LLM-as-judge metrics, pretrained forecasters, opt-in AI adoption, AI-mediated marketing measurement, offline evals vs online A/B, a public-data check of propensity vs uplift targeting, PPI++ with a pre-treatment covariate, and staggered rollout vs fixed holdout. Next up is judge drift during a test.

## Side projects

- [**games**](https://github.com/wavde/games): twelve classic and logic-puzzle browser games — vanilla HTML/CSS/JS, no build. Generators, solvers, daily-seed puzzles. [Play →](https://wavde.github.io/games/)

## Elsewhere

- Site: [wavde.github.io](https://wavde.github.io)
- GitHub: [@wavde](https://github.com/wavde)

## Usage note

Essays and profile content © 2026 Tejas Wavde. Please link back when quoting or referencing this work.
