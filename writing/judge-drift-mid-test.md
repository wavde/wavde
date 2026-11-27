# Upgrading the judge mid-test is safe only if your labels know about it

*~6 min read. Companion code is [`judge_drift`](https://github.com/wavde/experiment-toolkit/blob/main/src/experiment_toolkit/judge_drift.py) in experiment-toolkit, with [tests](https://github.com/wavde/experiment-toolkit/blob/main/tests/test_judge_drift.py), a [one-command simulation](https://github.com/wavde/experiment-toolkit/blob/main/scripts/judge_drift_sim.py), and the [generated results](https://github.com/wavde/experiment-toolkit/blob/main/docs/judge_drift_results.md). It follows [LLM-as-judge is a noisy metric](llm-judge-noise-in-ab-tests.md) and [A covariate stacks on the judge](ppi-with-covariates-label-budget.md).*

## The recommendation

If the LLM judge is upgraded while a test is running, you don't need to restart the test. But you do need either:

- human labels spread across the whole test and analysed with PPI++, or
- labels in each judge-version period, with PPI++ run per period.

Front-loaded labeling is the trap. If all the labels were collected before the upgrade, PPI++ calibrates the old judge and applies that calibration to the new one. When the new judge treats the two arms differently, that analysis keeps about 70% of the naive bias. Its false-positive rate also climbs with every extra label you buy: at 20,000 units per arm it goes from **0.064** at 250 labels to **0.392** at 4,000.

The pooled judge score with no labels is worse. If the new judge is more lenient on treatment, it rejects a true null **82.6%** of the time.

## The decision

The earlier loops assumed one fixed judge. In practice the judge is a model and a prompt, and both get upgraded, sometimes mid-test, because a better model shipped or someone fixed a prompt. The upgrade hits both arms at the same moment, so it feels harmless. Is it? And what should the analysis do about it?

## The setup

There is no model API. Everything is simulated, and the true effect is known.

- **True quality** is binary, with a control rate of 0.60. The effect is +0.03 or 0, and it stays constant through the test.
- **The old judge** has sensitivity = specificity = 0.85 on both arms.
- **The upgrade:** halfway through, both arms switch to a new judge. Each arm's new judge can have its own error rates.
- **Labels:** every unit is judged, and every PPI++ variant gets the same 500 human labels per arm, out of 10,000 units per arm.

Four upgrades:

- **none**;
- **more lenient on both arms** (specificity 0.75);
- **more accurate on both arms** (0.92 / 0.92);
- **more lenient on treatment only** (specificity 0.75 on treatment, 0.85 on control). For example, the new model may prefer the treatment's style.

Five analyses:

- the **pooled** judge score;
- the judge score from **before the upgrade only** (truncating the test);
- PPI++ with labels **spread** across the test;
- PPI++ with labels **before the upgrade only**;
- PPI++ **stratified** by judge version, with labels split in proportion to each period.

It's 2,000 replications per cell, seed `20261127`. `python scripts/judge_drift_sim.py` reproduces it in about a minute, and CI smoke-runs it.

![Judge drift simulation](https://raw.githubusercontent.com/wavde/experiment-toolkit/main/docs/judge_drift.png)

## When the upgrade is harmless

If the new judge shifts both arms the same way, the upgrade doesn't break anything under the null. With a more lenient judge on both arms and no true effect, every analysis has Type-I error between 0.048 and 0.051. A more accurate judge on both arms is the same (0.048–0.053), and it helps the analyses that use it: the pooled judge score's power rises to 0.910, from 0.866 with no upgrade. The estimate still means something different after the upgrade (+0.0231 instead of +0.0210 for a true +0.03, because the new judge attenuates less), but it isn't biased toward one arm.

## When it isn't

Now let the new judge be more lenient on treatment only, with no true effect:

| Analysis | Mean estimate (expected) | Type-I error |
|---|---:|---:|
| Judge only, pooled | +0.0201 (+0.0200) | 0.826 |
| Judge only, pre-upgrade data | +0.0001 (+0.0000) | 0.044 |
| PPI++, labels across whole test | +0.0014 (+0.0000) | 0.042 |
| PPI++, labels before upgrade only | +0.0141 (+0.0139) | 0.095 |
| PPI++, stratified by judge version | +0.0003 (+0.0000) | 0.050 |

The pooled judge score carries half of the new judge's extra false positives on treatment: 0.10 × 0.40 × 0.5 = +0.020. Truncating at the upgrade avoids that, at the cost of half the data.

PPI++ with front-loaded labels is the subtle failure. PPI++ corrects the judge with the gap between judge scores on labeled and unlabeled units. When every labeled unit came before the upgrade, that gap is between the old judge and a mix of old and new judges. The correction is calibrated to the wrong judge. It still removes some of the bias, but about 70% remains (+0.0141 against +0.0201).

That bias doesn't shrink as you add labels, but the standard error does, so Type-I error rises with the label budget:

| Labels per arm (20,000 units) | Labels before upgrade only | Labels across whole test | Stratified by judge version |
|---:|---:|---:|---:|
| 250 | 0.064 | 0.050 | 0.060 |
| 1,000 | 0.120 | 0.036 | 0.042 |
| 4,000 | 0.392 | 0.044 | 0.054 |

The same mechanism also inflates a real effect: with a true +0.03, front-loaded PPI++ averages +0.0421 (expected +0.0427).

## Why spreading or stratifying labels works

PPI++ is unbiased whenever the labeled units are a random sample of the units being corrected. With labels spread across the whole test, the labeled units see the same mix of old and new judge scores as the unlabeled ones, so the correction calibrates the mix. Stratifying by judge version calibrates each judge on its own units and weights the two periods by their shares. Both stay unbiased whatever the new judge does. Here, stratifying cost nothing over spreading: Type-I 0.050 and 0.042, with SEs of 0.0234.

## Assumptions and limitations

- **One upgrade, at a known time, applied to both arms at once.** A gradual rollout of the new judge, or a judge that drifts without a version change (for example, a hosted model updated silently), is harder because there is no clean boundary to stratify on.
- **Binary quality, a constant true effect, and confusion-matrix judges.** The new judge's bias is a fixed per-arm leniency.
- **Human labels as ground truth,** drawn at random within each (arm, period) cell when stratified.
- **One label budget split by period size.** A team could put more labels after the upgrade if the new judge is noisier. That isn't tested here.

## When the conclusion breaks

- **The judge changes without a version boundary.** Stratifying needs to know when the judge changed. Spreading labels across the test still works, as long as the labels really are spread in time.
- **Labels can't be collected after the upgrade,** for example because the labeling vendor contract ended. Then the honest options are truncating at the upgrade or re-scoring the whole test with one judge version.
- **Re-scoring is cheap.** If the stored outputs can be re-judged with the new judge, re-scoring everything with one version removes the problem entirely. It also needs labels on the new judge, which is the same requirement as above.

## What would change my mind

- **Evidence that real judge upgrades only shift both arms equally.** For example, audits across many upgrades showing that the new judge's error doesn't depend on which arm produced the output. Then front-loaded labels would be safe, like the "lenient on both arms" row (0.048).
- **Coverage failures for stratified PPI++** with few labels in the post-upgrade period, for example if an upgrade lands near the end of a test.

## The decision rule

1. **A mid-test judge upgrade doesn't require a restart,** but it does require the analysis to know about it.
2. **Spread human labels across the whole test, or stratify by judge version.** Either keeps PPI++ unbiased whatever the new judge does.
3. **Never apply pre-upgrade labels to post-upgrade judge scores.** The remaining bias grows into a false positive as the label budget grows (0.064 to 0.392 Type-I from 250 to 4,000 labels).
4. **Without labels, truncate at the upgrade or re-score everything with one judge.** Never pool judge scores across versions. That rejected a true null 82.6% of the time when the new judge favoured treatment.

---

*Code, tests, and results: [`judge_drift.py`](https://github.com/wavde/experiment-toolkit/blob/main/src/experiment_toolkit/judge_drift.py) · [`test_judge_drift.py`](https://github.com/wavde/experiment-toolkit/blob/main/tests/test_judge_drift.py) · [`judge_drift_sim.py`](https://github.com/wavde/experiment-toolkit/blob/main/scripts/judge_drift_sim.py) · [`judge_drift_results.md`](https://github.com/wavde/experiment-toolkit/blob/main/docs/judge_drift_results.md) · References: Angelopoulos, Bates, Fannjiang, Jordan, Zrnic (2023), Prediction-powered inference · Angelopoulos, Duchi, Zrnic (2023), PPI++ · Zheng et al. (2023), Judging LLM-as-a-judge.*
