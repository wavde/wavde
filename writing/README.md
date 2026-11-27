# Writing

Short technical posts on methods used in the case studies, grouped by theme. Start with [Which number supports which decision](which-number-supports-which-decision.md).

## Measuring AI features

- [Which number supports which decision: measuring AI features](which-number-supports-which-decision.md)
- [Five classic data science case studies, re-read in 2026](classic-case-studies-reread-2026.md)
- [LLM-as-judge is a noisy metric: what it does to A/B power and bias](llm-judge-noise-in-ab-tests.md)
- [A covariate stacks on the judge, but a better judge beats both: cutting the label budget of PPI++](ppi-with-covariates-label-budget.md)
- [Upgrading the judge mid-test is safe only if your labels know about it](judge-drift-mid-test.md)
- [When the judge drifts silently, random labels protect the readout and monitors only tell you where to look](silent-judge-drift.md)
- [An offline eval win is evidence about the eval set: offline LLM evals vs online A/B](offline-evals-vs-online-ab.md)
- [Opt-in AI features: when an observational impact estimate is usable](opt-in-ai-adoption-observational-bias.md)
- [A staggered rollout can't see novelty: holdout vs rollout for opt-in AI features](staggered-rollout-vs-holdout.md)

## Marketing measurement and targeting

- [Measuring marketing when the algorithm runs the campaign](ai-mediated-marketing-measurement.md)
- [The targeting model already knows who would convert: propensity vs uplift on real randomized data](propensity-vs-uplift-targeting-criteo.md)
- [It didn't replicate: propensity vs uplift targeting on a second randomized dataset](uplift-targeting-replication-hillstrom.md)
- [MMM vs incrementality: what each answers and what neither does](mmm-vs-incrementality.md)
- [Geo holdouts as paid-media A/B](geo-holdouts-as-paid-media-ab.md)
- [Last-touch attribution systematically under-credits upper funnel](last-touch-under-credits-upper-funnel.md)

## Experimentation and causal inference

- [CUPED inside sequential testing: two variance reductions that compound](cuped-inside-sequential-testing.md)
- [Finding (and fixing) a 17% Type-I error bug in an mSPRT implementation](finding-and-fixing-msprt-bug.md)
- [When TWFE lies: staggered rollouts and what Callaway-Sant'Anna buys you](twfe-lies-on-staggered-rollouts.md)
- [LaLonde, revisited: what a replication tells you about PSM, IPW, and AIPW](what-lalonde-taught-me.md)
- [E-values: a one-number sensitivity check for observational memos](e-values-for-observational-memos.md)
- [Picking a north-star metric when every candidate has a tradeoff](picking-a-north-star-metric.md)

## Forecasting

- [Pretrained forecasters vs SARIMA: when zero-shot wins, and when it doesn't](pretrained-forecasters-vs-sarima.md)
- [When boosting loses to SARIMA: baselines, MASE, and honest backtests](when-boosting-loses-to-sarima.md)
