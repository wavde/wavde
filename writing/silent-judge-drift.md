# When the judge drifts silently, random labels protect the readout and monitors only tell you where to look

*~7 min read. Companion code is [`judge_drift`](https://github.com/wavde/experiment-toolkit/blob/main/src/experiment_toolkit/judge_drift.py) in experiment-toolkit, with [tests](https://github.com/wavde/experiment-toolkit/blob/main/tests/test_judge_drift.py), a [one-command simulation](https://github.com/wavde/experiment-toolkit/blob/main/scripts/silent_drift_sim.py), and the [generated results](https://github.com/wavde/experiment-toolkit/blob/main/docs/silent_drift_results.md). It follows [Upgrading the judge mid-test is safe only if your labels know about it](judge-drift-mid-test.md).*

## The recommendation

If your A/B metric is scored by a hosted LLM judge that might change under the same model name, protect the readout and don't rely on detecting the change. Draw the human labels at random from the whole test and analyse with PPI++. You can draw them at the end, from logged outputs. That readout needs no detection. In every scenario tested, including drift that hits only the treatment arm, its Type-I error stayed between **0.048** and **0.053**.

Monitors are still worth running, but as tripwires for dashboards and interim reads, not as the defence:

- A judge-score monitor on the **control arm only** can't see drift that hits only treatment. It fired **0.021** of the time, no more than with nothing changing.
- A score monitor on **both arms** caught a step change **0.847** of the time, a median of 6 days after it started. It also fired **1.000** of the time when true quality fell and the judge didn't change. An alarm means "something changed", not "the judge changed".
- Telling the two apart takes human labels, split by the human label. At a normal budget of 500 labels per arm, a continuous agreement monitor caught the treatment-only drift only **0.089** of the time. A one-off label burst after an alarm is cheaper.

## The decision

[Loop 9](judge-drift-mid-test.md) handled a judge upgrade at a known time: spread labels across the test, or stratify by judge version. A hosted judge can change with no version flag: the provider updates the model, the name stays the same, and nobody tells the experiment. Chen, Zaharia and Zou (2023) found that the behaviour of models served under the same ChatGPT API names changed substantially within three months. There's no boundary to stratify on. How should a team detect a silent change, and is detection needed at all if labels are spread across the test?

## The setup

There is no model API. Everything is simulated, so both the true effect and the drift day are known. The analyses and monitors are told neither.

- **The test:** 28 days, 500 units per arm per day (14,000 per arm). True quality is binary with a control rate of 0.60; the true effect is +0.03 or 0.
- **The judge** starts at sensitivity = specificity = 0.85 on both arms. From day 15 it becomes more lenient (specificity 0.75) on the arms named in the scenario.
- **Five scenarios:** no change; judge drift on both arms (step); on treatment only (step); on treatment only, gradually from day 15 to day 28; and true quality falling by 0.08 on both arms from day 15 with the judge unchanged. The last one is a real change that a drift monitor should *not* blame on the judge.
- **Three readouts:** the judge score alone; PPI++ with 500 labels per arm from the launch week; and PPI++ with 500 labels per arm drawn at random from the whole test.
- **Three monitors.** Each compares the latest 7 days with the first 7 days, every day from day 14, with a Bonferroni threshold that keeps the chance of any false alarm at or below 5%. They track: the judge pass rate on the control arm; the judge pass rate on each arm; and, in each arm, the judge pass rate among labeled units humans passed and among those humans failed (the judge's sensitivity and false-positive rate). The third uses the same randomly drawn labels as the readout.

It's 2,000 replications per cell, seed `20261127`. `python scripts/silent_drift_sim.py` reproduces it in about two minutes, and CI smoke-runs it.

![Silent drift simulation](https://raw.githubusercontent.com/wavde/experiment-toolkit/main/docs/silent_drift.png)

## The readout doesn't need to know when the judge changed

With no true effect:

| Scenario | Judge only | PPI++, launch-week labels | PPI++, labels across whole test |
|---|---:|---:|---:|
| No change | 0.048 | 0.050 | 0.048 |
| Judge drifts on both arms (step) | 0.046 | 0.044 | 0.049 |
| Judge drifts on treatment only (step) | 0.922 | 0.097 | 0.052 |
| Judge drifts on treatment only (gradual) | 0.435 | 0.054 | 0.053 |
| True quality falls, judge unchanged | 0.052 | 0.059 | 0.053 |

Drift that hits both arms equally is harmless under the null, as in loop 9. Drift that hits one arm is not. The judge-only readout carries half of the extra false positives on treatment, +0.0201 (closed form +0.0200), and rejects a true null 92.2% of the time. Launch-week labels calibrate the old judge and apply that calibration to the new one, leaving +0.0139 of bias (closed form +0.0139). With a real +0.03 effect, launch-week PPI++ averages +0.0434 (closed form +0.0427).

Labels drawn at random from the whole test don't care when, or whether, the judge changed. The labeled units see the same mix of old and new judge scores as the unlabeled ones, so the PPI++ correction calibrates the mix. Its coverage stayed between 0.944 and 0.956 in all ten cells. It costs almost nothing: its SE under treatment-only drift was 0.0233, against 0.0227 for launch-week labels.

A gradual drift is half as damaging on average (judge-only bias +0.0105, closed form +0.0107) but still rejects a true null 43.5% of the time with the judge alone.

## Score monitors are fast but can't tell drift from real change

| Scenario (no true effect) | Score monitor, control arm | Score monitor, both arms | Agreement monitor, 500 labels per arm |
|---|---:|---:|---:|
| No change | 0.022 | 0.022 | 0.031 |
| Judge drifts on both arms (step) | 0.871 | 0.968 | 0.144 |
| Judge drifts on treatment only (step) | 0.021 | 0.847 | 0.089 |
| Judge drifts on treatment only (gradual) | 0.019 | 0.365 | 0.043 |
| True quality falls, judge unchanged | 0.993 | 1.000 | 0.029 |

Three things stand out.

**Watching control alone misses the dangerous case.** The drift that biases the readout is the one that treats the arms differently. A control-arm monitor only sees drift on control, and its 0.021 alarm rate under treatment-only drift is the false-alarm rate.

**Watching both arms catches a step change about a week in.** The both-arm score monitor fired a median of 6 days after the change, around day 20 of 28. That's early enough to stop trusting a judge-only dashboard before the readout, but not early enough to keep a week of interim reads clean. In closed form, the judge pass rate moves from 0.570 to 0.610, so one check on a fully changed week flags with probability 0.603. A gradual drift was caught only 0.365 of the time, a median of 12 days in.

**Both score monitors fire on real change.** When true quality fell and the judge stayed the same, they fired 0.993 and 1.000 of the time. A shift in judge pass rate is a shift in judge, quality, or both, and judge scores alone can't separate them. Watching treatment too doesn't fix this: a true effect that grows or fades over the test would also move treatment's pass rate (not simulated here).

## Separating drift from real change costs labels

The agreement monitor tracks the judge's pass rate among units humans passed and among units humans failed, separately. Those rates don't move when quality shifts and the judge stays the same. That's why it stayed at 0.029 when quality fell. Raw agreement, or the gap between judge and human pass rates, would move, because a judge's mistakes depend on how many good and bad outputs it sees.

But at 500 labels per arm, a week holds only about 50 human-failed labels per arm, too few to see the false-positive rate move from 0.15 to 0.25. With treatment-only drift and no true effect:

| Labels per arm | Agreement monitor alarm rate | Median days after change | False alarms when quality falls |
|---:|---:|---:|---:|
| 500 | 0.076 | 6 | 0.030 |
| 1,000 | 0.180 | 7 | 0.032 |
| 2,000 | 0.398 | 7 | 0.030 |
| 4,000 | 0.788 | 6 | 0.016 |
| 8,000 | 0.986 | 5 | 0.034 |

(500 replications per row, so the 500-label rate differs slightly from the 0.089 in the main run.)

Running this continuously means labeling thousands of units per arm, far more than the readout needs. A cheaper diagnosis is a one-off burst after a score alarm: label units from the launch week and from the flagged week, and test the judge's false-positive rate between them. For this drift, 80% power at a two-sided 5% level needs about 619 labels per arm in each of the two weeks (closed form).

## Assumptions and limitations

- **One drift, starting on day 15, from a lenient-on-one-arm judge.** Binary quality, confusion-matrix judges, equal daily traffic.
- **One monitor design:** a 7-day window against the launch week with a Bonferroni threshold. A CUSUM or a scan over change points would detect faster at the same false-alarm rate. The qualitative findings, that control-only monitoring misses arm-specific drift and that score monitors can't separate drift from quality change, don't depend on the test statistic.
- **Drift inside the launch week** is invisible to these monitors, because the launch week is the reference.
- **Human labels as ground truth.** If the labelers drift too (new guidelines, a new vendor), labels drawn across the test inherit that drift.
- **Not tested:** re-judging a fixed canary set of stored outputs every day. Any change in its scores would be pure judge drift, with no labels and no confusion with quality change. It's likely the cheapest good monitor, but it isn't simulated here.

## When the conclusion breaks

- **Outputs aren't kept long enough to label at the end.** Then labels must be collected during the test, at a steady rate, so they still form a random sample of the whole test.
- **Decisions are made on interim reads.** A random sample of the test so far is still valid for an interim read, but launch-week labels and judge-only dashboards are not. Here the score monitor is the only early warning.
- **The judge feeds more than one test.** A silent change affects every test and offline eval that uses it, so detection matters beyond this readout, even when the readout is protected.

## What would change my mind

- **Evidence that silent hosted-model updates move both arms equally.** Then the judge-only readout would be safe under the null, like the both-arm drift row (0.046), and score monitors would only matter for interpretation.
- **A canary-set monitor** that catches realistic drift within days at a false-alarm rate near 5%. That would replace the label burst as the diagnosis step.

## The decision rule

1. **Protect the readout with labels drawn at random from the whole test,** analysed with PPI++. It needs no detection: Type-I 0.048 to 0.053 under every drift tested. Draw them at the end from logged outputs if that's easier.
2. **Never ship on the judge-only readout or on launch-week labels** if the judge is hosted and might change. Treatment-only drift gave them Type-I errors of 0.922 and 0.097.
3. **Run the score monitor on both arms, not just control,** as a tripwire for dashboards and interim reads. It caught a step change 0.847 of the time, a median of 6 days in. Control alone caught 0.021.
4. **Treat a score alarm as "something changed" and diagnose it with labels.** Label a burst from the launch week and the flagged week, and compare the judge's pass rate within human-passed and within human-failed units (about 619 labels per arm per week for this drift). Don't run that comparison continuously at a normal label budget: it caught the drift 0.089 of the time at 500 labels per arm.

---

*Code, tests, and results: [`judge_drift.py`](https://github.com/wavde/experiment-toolkit/blob/main/src/experiment_toolkit/judge_drift.py) · [`test_judge_drift.py`](https://github.com/wavde/experiment-toolkit/blob/main/tests/test_judge_drift.py) · [`silent_drift_sim.py`](https://github.com/wavde/experiment-toolkit/blob/main/scripts/silent_drift_sim.py) · [`silent_drift_results.md`](https://github.com/wavde/experiment-toolkit/blob/main/docs/silent_drift_results.md) · References: Angelopoulos, Duchi, Zrnic (2023), PPI++ · Page (1954), Continuous inspection schemes · Chen, Zaharia, Zou (2023), How is ChatGPT's behavior changing over time?*
