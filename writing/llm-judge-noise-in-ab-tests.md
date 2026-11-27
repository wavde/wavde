# LLM-as-judge is a noisy metric: what it does to A/B power and bias

*~8 min read. Companion code is [`llm_judge`](https://github.com/wavde/experiment-toolkit/blob/main/src/experiment_toolkit/llm_judge.py) in experiment-toolkit, with [tests](https://github.com/wavde/experiment-toolkit/blob/main/tests/test_llm_judge.py), a [one-command simulation](https://github.com/wavde/experiment-toolkit/blob/main/scripts/llm_judge_sim.py), and the [generated results](https://github.com/wavde/experiment-toolkit/blob/main/docs/llm_judge_results.md).*

## The recommendation

Don't ship on a naive judge-scored primary metric. If an LLM judge scores the primary metric, draw a random sample of human labels **within each arm** and analyse with PPI++, a per-arm control variate that uses the judge score as the covariate. Size the label budget from the judge–human correlation, not from the number of judged units. Accept the naive judge metric as primary only when the treatment cannot change what the judge sees. Otherwise keep it as a secondary or diagnostic metric.

The reason, from a simulation with known ground truth: if the judge is 5 points more lenient on the treatment arm and the true effect is zero, the naive test rejects **15% of the time at 1,000 units per arm, 49% at 5,000, and 99% at 20,000**. More traffic makes it worse. PPI++ with 500 human labels per arm stays between 3.6% and 5.8% at every sample size.

## The decision

A team ships an LLM-backed feature: a new prompt, a new model, a new retrieval step. Nobody can hand-label every response, so the primary metric becomes "share of responses an LLM judge rates as good." Can the team ship on that metric, and how much extra sample or human labeling does it need?

## The setup

The simulation makes no model API calls. Everything is known by construction.

- **True quality** per unit is binary (good or not), with a control rate of 0.60. The true effect is +0.03, or zero under H0.
- **The judge** is a confusion matrix per arm: sensitivity `P(judge says good | good)` and specificity `P(judge says bad | bad)`. Every unit is judged.
- **Human labels** go to a simple random subsample within each arm and are treated as the truth.

Three estimators are compared: the naive difference in mean judge score, the difference in mean human label on the labeled subsample, and PPI++ (Angelopoulos, Bates, Fannjiang, Jordan, Zrnic 2023; Angelopoulos, Duchi, Zrnic 2023). Per arm, PPI++ is

```
theta_hat = mean_lab(Y) - lam * (mean_lab(S) - mean_unlab(S))
```

where `Y` is the human label, `S` the judge score, and `lam` is tuned to minimise variance. If that looks like CUPED with the judge score as the covariate, it is (Chaganty, Mussmann, Liang 2018 make the same point for NLG evaluation). It is unbiased for any judge, good or bad, because the labeled units are a random sample of the arm. Every simulated number in the results file is printed next to its closed-form value, and the two agree throughout. Seed `20261126`; `python scripts/llm_judge_sim.py` reproduces all of it.

![LLM-as-judge simulation](https://raw.githubusercontent.com/wavde/experiment-toolkit/main/docs/llm_judge.png)

## When the judge errs the same way in both arms

This is the benign case. The naive estimate shrinks toward zero by Youden's `J = sensitivity + specificity - 1` (Bross 1954; Rogan & Gladen 1978), but under H0 there is nothing to shrink, so Type-I error holds: 0.044 observed at 5,000 units per arm with sensitivity = specificity = 0.85. With a true effect of +0.030, the naive estimate averages +0.021, and its 95% CI covers the true effect only 84% of the time. The test is valid; the effect size it reports is not.

The price is sample. Two-sided alpha 0.05, 80% power, true effect +0.03 on 0.60:

| Judge accuracy (sens = spec) | Youden J | Sample multiplier | Per-arm n |
|---:|---:|---:|---:|
| 1.00 | 1.00 | 1.00x | 4,126 |
| 0.95 | 0.90 | 1.25x | 5,149 |
| 0.90 | 0.80 | 1.59x | 6,579 |
| 0.85 | 0.70 | 2.10x | 8,665 |
| 0.80 | 0.60 | 2.88x | 11,878 |
| 0.70 | 0.40 | 6.55x | 27,019 |

The multiplier is roughly `1/J²`, a little more in practice because a noisier score sits closer to a coin flip and has more variance. A judge that agrees with humans 85% of the time, which sounds respectable, doubles the experiment.

## When the treatment changes how the judge errs

This is the case that should worry a team, and it is plausible. A treatment that changes length, tone, or formatting can change how the judge scores it, independent of quality. LLM judges have documented preferences for things like verbosity and their own model family's outputs (Zheng et al. 2023).

In the simulation the judge is 0.85 sensitive and 0.85 specific on control, but only 0.80 specific on treatment: it waves through 5 points more of the bad treatment responses. With a true effect of zero, the naive estimate converges to `0.05 × 0.40 = +0.020`, and that bias doesn't shrink with sample size. The standard error does. So the false-positive rate climbs with traffic:

| Units per arm | Naive Type-I | PPI++ Type-I (500 labels/arm) |
|---:|---:|---:|
| 1,000 | 0.154 | 0.048 |
| 2,000 | 0.276 | 0.058 |
| 5,000 | 0.492 | 0.054 |
| 10,000 | 0.822 | 0.046 |
| 20,000 | 0.988 | 0.036 |
| 50,000 | 1.000 | 0.050 |

When there is a real +0.030 effect, the naive metric reports +0.039. In both cases it is wrong with a tight confidence interval.

The tempting fix doesn't work. Calibrating the judge once, on an offline validation set or on labels from the control arm only, applies the same correction to both arms. An identical correction cancels in the difference, and you're back to the naive estimate. The calibration has to happen within each arm, because the error being corrected is the difference between arms.

## The correction, and what it honestly costs

PPI++ is unbiased in every scenario. Under the lenient judge its 95% CI covers the truth 95.7% of the time at 5,000 units per arm. With a perfect judge it loses nothing: power is 0.857 against 0.853 for the naive metric.

With a noisy judge, being correct costs power. At 5,000 judged units and 500 labels per arm, with sensitivity = specificity = 0.85 and the error identical in both arms (the one case where the naive metric is valid), power is:

- naive judge: 0.59
- PPI++: 0.25
- human labels only: 0.18

The label budget, not the judged traffic, is the binding constraint. Holding judged units at 20,000 per arm, the labels needed for 80% power are:

| Judge accuracy | Corr(judge, human) | Human labels only | PPI++ labels | Saving |
|---:|---:|---:|---:|---:|
| 0.95 | 0.90 | 4,126 | 982 | 76% |
| 0.90 | 0.79 | 4,126 | 1,767 | 57% |
| 0.85 | 0.69 | 4,126 | 2,397 | 42% |
| 0.80 | 0.59 | 4,126 | 2,901 | 30% |
| 0.70 | 0.39 | 4,126 | 3,610 | 13% |

The saving sits a little below corr² (0.81 against 76%; 0.15 against 13%), which is the CUPED `1 - rho²` logic with a finite pool of judged units. A good judge makes labeling about four times cheaper. A mediocre one barely helps. At 0.85 accuracy, a +0.03 effect needs about 2,400 labels per arm, or about 12% of 20,000 judged units.

## Assumptions and limitations

All of this is synthetic, which is what makes the ground truth checkable. It is also where it can mislead.

- **Binary quality.** Real judges often return graded scores or pairwise preferences. PPI++ generalises to those, but these numbers don't.
- **A fixed judge.** No drift during the experiment, no prompt or model version change mid-test.
- **A known confusion matrix.** In practice you estimate it from the same human labels, and it can vary across query types.
- **Human labels as ground truth.** Raters disagree with each other. PPI++ is unbiased for whatever the human labeling process measures, not for "true" quality.
- **Normal approximations.** All tests and intervals are Wald. Coverage held at these sample sizes; it is not guaranteed at a few dozen labels.
- **Independent units, labels random within each arm.** No clustering by user or conversation.

## When the conclusion breaks

- **Labels that aren't random within the arm.** If raters label only user-flagged responses, only the control arm, or can see which arm a response came from, PPI++ inherits the bias it was supposed to remove.
- **Rater error that differs by arm.** If treatment responses are recognisable and raters treat them differently, the differential-error problem has moved from the judge to the humans. Nothing in this setup detects that.
- **The "judge can't see the change" exception is wrong.** A latency-only change that triggers timeouts, truncation, or a fallback model does change what the judge sees.
- **An underpowered PPI++ read as "no effect."** At 500 labels per arm, the correct analysis misses a real +0.03 effect most of the time. A null result is a statement about the label budget.

## What would change my mind

- In-arm audits across many launches showing that the judge's confusion matrix doesn't move with treatment. Then the naive metric plus the `1/J²` inflation becomes a reasonable default, with periodic audits instead of every-test labeling.
- Real experiments where PPI++ coverage fails, for example with conversation-level clustering, graded scores, or judge drift.
- Judges whose correlation with humans on the product's own traffic is 0.90 or higher. Then the labeling cost drops to about 1,000 per arm, and the recommendation becomes cheap to follow.

## The decision rule

1. **A judge-scored metric is acceptable as primary only with human labels drawn at random within each arm**, analysed with PPI++ (or human labels alone). Estimate the judge–human correlation from those in-arm labels and size the label budget from it.
2. **A naive judge metric is acceptable as primary only if the treatment cannot change what the judge sees**, such as a pure serving or latency change with identical outputs. Even then, budget roughly `1/J²` more sample: 2.1x at 85% judge accuracy.
3. **Otherwise the judge is a secondary or diagnostic metric.** Use it to explain a result, not to make the ship decision.

---

*Code, tests, and results: [`llm_judge.py`](https://github.com/wavde/experiment-toolkit/blob/main/src/experiment_toolkit/llm_judge.py) · [`test_llm_judge.py`](https://github.com/wavde/experiment-toolkit/blob/main/tests/test_llm_judge.py) · [`llm_judge_sim.py`](https://github.com/wavde/experiment-toolkit/blob/main/scripts/llm_judge_sim.py) · [`llm_judge_results.md`](https://github.com/wavde/experiment-toolkit/blob/main/docs/llm_judge_results.md) · References: Angelopoulos, Bates, Fannjiang, Jordan, Zrnic (2023), Prediction-powered inference · Angelopoulos, Duchi, Zrnic (2023), PPI++ · Chaganty, Mussmann, Liang (2018), The price of debiasing automatic metrics in natural language evaluation · Bross (1954); Rogan & Gladen (1978) on misclassification · Deng, Xu, Kohavi, Walker (2013), CUPED · Zheng et al. (2023), Judging LLM-as-a-judge.*
