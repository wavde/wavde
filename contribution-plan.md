# Contribution plan

A working plan for what I publish next. Everything under **Proposed** is not done yet. The only completed evidence is the work already linked from the [profile README](README.md).

## Principle

Fewer pieces, each one defensible. A contribution counts when it shows a decision made under uncertainty: a question, evidence with a known or checkable answer, a recommendation, and the conditions under which the recommendation would change. Volume isn't the goal.

## The loop

Run one loop at a time. Don't start the next one until the current one meets its finish criteria.

1. **Question.** Pick a decision a data science team actually has to make. Write it in one sentence.
2. **Evidence.** Build the smallest reproducible check that could change the answer: a simulation with known ground truth, a backtest, or a replication. Put it in the existing repo that fits ([experiment-toolkit](https://github.com/wavde/experiment-toolkit), [causal-inference-playbook](https://github.com/wavde/causal-inference-playbook), [demand-forecasting](https://github.com/wavde/demand-forecasting), [paid-media-playbook](https://github.com/wavde/paid-media-playbook), or [product-analytics-deepdive](https://github.com/wavde/product-analytics-deepdive)). Start a new repo only when none of them fits.
3. **Memo.** Write a short essay in [`writing/`](writing/) that leads with the recommendation and spells out assumptions, failure modes, and what would change my mind.
4. **Link.** Add the essay to both writing indexes and link it to the code.
5. **Retro.** Add two or three lines to the log below: what held up, what didn't, and what the next question is.

### Finish criteria (per loop)

- The evidence runs from a clean checkout with one documented command and a fixed seed.
- The result is checked against a known answer, or against a baseline that a skeptical reviewer would accept.
- The essay states the decision, the assumptions, at least one condition under which the conclusion breaks, and the limitations of synthetic or public data.
- The essay and code link to each other, and the essay is listed in `README.md` and `writing/README.md`.
- No claim goes further than the evidence. No performance numbers appear without the code that produced them.

### Cadence

One loop at a time, paced by the finish criteria rather than a calendar. Review this plan after every completed loop, or every six weeks if nothing has shipped, and cut scope if a loop has stalled.

## Review after loops 1–4

All four themes shipped, and each one met the finish criteria. No loop stalled, so nothing is cut.

**What held up across loops**

- Simulations where the ground truth is known by construction worked as evidence. Each loop showed the simulated value next to the true value, and the code that produced each number is linked.
- One command with a fixed seed and byte-identical reruns was achievable every time. Building the evidence in an existing repo kept each loop small.
- The decision rules share one theme: **randomize at the decision you're making.** That means human labels within each arm (loop 1), a holdout or staggered rollout instead of an opt-in comparison (loop 3), and a two-cell budget split or an audience-fixed creative test (loop 4).

**What didn't**

- Three of the four loops rest on synthetic data with one mechanism switched on at a time. Only loop 2 used public data. A skeptical reviewer's first question will be whether any of this shows up in real data.
- The corrections are expensive. PPI++ power was 0.25 at a realistic label budget (loop 1). The two-cell split needed 2,000,000 users (loop 4). The fake-launch placebo can't detect adoption timed to a post-launch rise in need (loop 3). Most of the cost problems are still unaddressed.
- Theme 1 is only partly covered. Offline-vs-online disagreement and guardrails were in scope but not built.
- CI in the repos runs the tests, not the evidence pipelines. In loop 2, `run.py` was broken from a clean install until the loop caught it.

**Changes for the next round**

- At least one loop uses public, randomized data instead of a simulation.
- Every evidence command gets a CI smoke run, a reduced-size run of the same pipeline, so a clean-install break is caught automatically.

## Next themes (proposed)

Ranked by AI relevance and by how directly each one answers a gap above.

1. **Offline evals vs online A/B for LLM features.** Decision: when can an offline eval win be trusted to predict, or replace, an online test of an LLM feature? Evidence: a simulation in experiment-toolkit where the offline eval set differs from live traffic (prompt mix, judge bias, and the effect varies by segment). It measures how often the offline winner loses online, and whether reweighting the eval set to the traffic mix fixes it. This closes the open part of theme 1.
2. **Public-data check of uplift vs propensity targeting.** Decision: on a real randomized ad dataset, does ranking users by conversion propensity rather than uplift waste spend the way loop 4's simulation says? Evidence: a replication in paid-media-playbook on the public Criteo uplift dataset, which comes from a randomized trial. This answers the synthetic-data gap.
3. **Cheaper corrections.** Decision: how far can sequential monitoring and variance reduction cut the cost of a valid answer? Candidates are combining PPI++ with CUPED-style covariates for the label budget, and staggered-rollout power versus a fixed holdout for opt-in features. It builds on experiment-toolkit's mSPRT and CUPED modules.

Next up: theme 1, offline evals vs online A/B.

## Completed themes

The order reflects AI relevance and how much existing work each theme can reuse. All four are done; see the log below.

1. **Measuring AI features with experiments.** Online evaluation of LLM-backed features: noisy or model-graded outcome metrics, offline-vs-online disagreement, and guardrails. Builds on experiment-toolkit and the CUPED and sequential-testing essays.
2. **Causal questions about AI adoption.** Selection into AI-feature usage, opt-in bias, and when observational impact estimates are usable. Builds on the DiD, PSM/AIPW, and E-value work in causal-inference-playbook.
3. **Honest baselines for AI forecasting.** When pretrained time-series models beat a tuned SARIMA under the same rolling-origin backtest, and when they don't. Builds on demand-forecasting and the "When boosting loses to SARIMA" essay.
4. **Measurement in AI-mediated marketing.** What incrementality and MMM can and can't say when targeting and creative are model-driven. Builds on paid-media-playbook.

## Recommended first contribution (done)

**"LLM-as-judge is a noisy metric: what it does to A/B power and bias."**

- **Decision:** Can a team ship on an A/B test whose primary metric is scored by an LLM judge, and how much extra sample or human labeling does that require?
- **Evidence:** A simulation in experiment-toolkit with no model API calls. True quality outcomes are generated, then passed through a judge with configurable error rates, including error that differs between arms. The simulation measures bias, variance, Type-I error, and power, and compares naive analysis with a correction that uses a small human-labeled calibration sample.
- **Why first:** It is AI-relevant, reuses the existing simulation and testing style (including the Type-I check from the mSPRT post), needs no new infrastructure, and is a judgment call that comes up in practice.
- **Done when:** It meets the finish criteria above, and the essay ends with a short decision rule for when a judge-scored metric is acceptable as the primary metric.

## Log

| Loop | Status | Evidence | Essay |
|---|---|---|---|
| LLM-as-judge noise in A/B tests | Done | [experiment-toolkit#1](https://github.com/wavde/experiment-toolkit/pull/1), [results](https://github.com/wavde/experiment-toolkit/blob/main/docs/llm_judge_results.md) | [LLM-as-judge is a noisy metric](writing/llm-judge-noise-in-ab-tests.md) |
| Pretrained forecasters vs SARIMA | Done | [demand-forecasting#1](https://github.com/wavde/demand-forecasting/pull/1), [results](https://github.com/wavde/demand-forecasting/blob/main/docs/pretrained_results.md) | [Pretrained forecasters vs SARIMA](writing/pretrained-forecasters-vs-sarima.md) |
| Opt-in AI adoption and observational bias | Done | [causal-inference-playbook#9](https://github.com/wavde/causal-inference-playbook/pull/9), [results](https://github.com/wavde/causal-inference-playbook/blob/main/case-studies/07-ai-feature-adoption/results.md) | [Opt-in AI features](writing/opt-in-ai-adoption-observational-bias.md) |
| Measurement in AI-mediated marketing | Done | [paid-media-playbook#1](https://github.com/wavde/paid-media-playbook/pull/1), [results](https://github.com/wavde/paid-media-playbook/blob/main/case-studies/06-ai-mediated-measurement/results.md) | [Measuring marketing when the algorithm runs the campaign](writing/ai-mediated-marketing-measurement.md) |

### Retro: LLM-as-judge noise in A/B tests

- **Held up:** the closed-form bias, variance, Type-I, and power matched the simulation in every scenario, and PPI++ with labels drawn within each arm kept Type-I near 5% under differential judge error while the naive test's false-positive rate climbed toward 1 with sample size.
- **Didn't:** the model is binary-only with a fixed judge (no drift), and at realistic label budgets the correction costs a lot of power (0.25 for PPI++ vs 0.59 for the naive metric at 5,000 units and 500 labels per arm), so the label budget is the binding constraint.
- **Next question:** theme 3. Does a pretrained time-series forecaster beat a tuned SARIMA on the existing demand-forecasting rolling-origin backtest?

### Retro: Pretrained forecasters vs SARIMA

- **Held up:** the comparison reused the existing rolling-origin backtest unchanged, reruns were byte-identical, and the answer was clear enough to act on: zero-shot Chronos-Bolt didn't beat SARIMA with full history, whether the order was fixed or re-selected by AIC per fold (UK MASE 0.784 vs 0.741 fixed and 0.748 tuned; rest of world 0.589 vs 0.584 for both), but it beat both at 56 days of history on both series. Per-fold tuning overfit on short UK history (1.048 at 56 days).
- **Didn't:** two series and six folds can't separate differences of a few percent, the UK gap comes mostly from one Christmas fold, pretraining overlap with the public dataset isn't audited, and only point forecasts were scored. Building the evidence also showed that `run.py` failed from a clean install because CI doesn't run the pipeline; that was fixed in the same PR.
- **Next question:** theme 2, causal questions about AI adoption. When are observational estimates of an AI feature's impact usable despite opt-in selection?

### Retro: Opt-in AI adoption and observational bias

- **Held up:** with known ground truth, the fake-launch placebo separated the cases it should. DiD was unbiased under a stable unobserved trait (bias -0.001, 94.8% coverage) while the placebo flagged AIPW with pre-periods in 100.0% of replications, and it flagged a pre-existing trend in 95.3%.
- **Didn't:** the placebo is blind to adoption timed to a post-launch rise in need. With a true effect of zero, the placebo-gated DiD acted on a spurious effect in 97.2% of replications, and the E-value didn't help (median 1.83). The design also uses linear trends, two pre-periods, fixed selection strengths and one unobserved channel at a time.
- **Next question:** theme 4, measurement in AI-mediated marketing. What can incrementality tests and MMM say when targeting and creative are model-driven?

### Retro: Measurement in AI-mediated marketing

- **Held up:** each tool was unbiased for the question it can answer. The ghost-ads holdout stayed within ±0.7% bias at every targeting sharpness, the two-cell budget split estimated marginal iROAS at 0.90 against a true 0.90 and never recommended an unprofitable scale-up, and the audience-fixed creative test declared no winner in 93% of replications when there was none.
- **Didn't:** the mechanisms are switched on one at a time with chosen effect sizes, the ghost-ads holdouts are assumed clean, the two-cell result needs 2,000,000 users per test, and the MMM is a best-case OLS with known transforms. Using the lift test as a prior was weaker than expected: +48% ROI bias at a spend–demand correlation of 0.6.
- **Next question:** all four themes are done, so the plan is due for its post-loop review before another loop starts. That review is in "Review after loops 1–4" above.
