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

## Prioritized themes (proposed)

The order reflects AI relevance and how much existing work each theme can reuse.

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

Next up, once the first loop is done: theme 3, a pretrained-forecaster comparison on the existing demand-forecasting backtest.

## Log

| Loop | Status | Evidence | Essay |
|---|---|---|---|
| LLM-as-judge noise in A/B tests | Done | [experiment-toolkit#1](https://github.com/wavde/experiment-toolkit/pull/1), [results](https://github.com/wavde/experiment-toolkit/blob/main/docs/llm_judge_results.md) | [LLM-as-judge is a noisy metric](writing/llm-judge-noise-in-ab-tests.md) |
| Pretrained forecasters vs SARIMA | Done | [demand-forecasting#1](https://github.com/wavde/demand-forecasting/pull/1), [results](https://github.com/wavde/demand-forecasting/blob/main/docs/pretrained_results.md) | [Pretrained forecasters vs SARIMA](writing/pretrained-forecasters-vs-sarima.md) |

### Retro: LLM-as-judge noise in A/B tests

- **Held up:** the closed-form bias, variance, Type-I, and power matched the simulation in every scenario, and PPI++ with labels drawn within each arm kept Type-I near 5% under differential judge error while the naive test's false-positive rate climbed toward 1 with sample size.
- **Didn't:** the model is binary-only with a fixed judge (no drift), and at realistic label budgets the correction costs a lot of power (0.25 for PPI++ vs 0.59 for the naive metric at 5,000 units and 500 labels per arm), so the label budget is the binding constraint.
- **Next question:** theme 3. Does a pretrained time-series forecaster beat a tuned SARIMA on the existing demand-forecasting rolling-origin backtest?

### Retro: Pretrained forecasters vs SARIMA

- **Held up:** the comparison reused the existing rolling-origin backtest unchanged, reruns were byte-identical, and the answer was clear enough to act on: zero-shot Chronos-Bolt didn't beat the fixed-order SARIMA with full history (MASE 0.784 vs 0.741 on UK, 0.589 vs 0.584 on rest of world), but it did at 56 days of history on both series.
- **Didn't:** two series and six folds can't separate differences of a few percent, the UK gap comes mostly from one Christmas fold, pretraining overlap with the public dataset isn't audited, and only point forecasts were scored. Building the evidence also showed that `run.py` failed from a clean install because CI doesn't run the pipeline; that was fixed in the same PR.
- **Next question:** theme 2, causal questions about AI adoption. When are observational estimates of an AI feature's impact usable despite opt-in selection?
