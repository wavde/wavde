# A canary set catches a drifting judge without blaming real quality changes

*~6 min read. Companion code is [`judge_drift`](https://github.com/wavde/experiment-toolkit/blob/main/src/experiment_toolkit/judge_drift.py) in experiment-toolkit, with [tests](https://github.com/wavde/experiment-toolkit/blob/main/tests/test_judge_drift.py), a [one-command simulation](https://github.com/wavde/experiment-toolkit/blob/main/scripts/canary_drift_sim.py), and the [generated results](https://github.com/wavde/experiment-toolkit/blob/main/docs/canary_drift_results.md). It follows [When the judge drifts silently](silent-judge-drift.md) and tests the canary idea that essay and [Which number supports which decision](which-number-supports-which-decision.md) left open.*

## The recommendation

If an A/B metric is scored by a hosted LLM judge, store a few hundred outputs from **every arm** at launch and have the production judge re-score them every day. Their quality can't change, so if their scores move, the judge moved. This is the drift check to run, in place of watching the judge's pass rate on live traffic.

In a simulation with known drift times, a canary of **400 stored outputs per arm** and no human labels:

- caught drift that made the judge more lenient on treatment only **0.932** of the time, a median of 6 days after it started. A judge-score monitor on both arms caught it 0.838 of the time, also in 6 days;
- stayed quiet when true quality fell and the judge didn't change, alarming **0.021** of the time against **1.000** for the score monitor.

It doesn't replace [loop 11's rule](silent-judge-drift.md): human labels drawn at random from the whole test are still what keep the readout valid. The canary tells you whether the judge changed, so you know whether a judge-only dashboard or an interim read can be trusted.

## The decision

Loop 11 found that monitoring a judge cheaply forces a trade-off. A judge-score monitor on both arms catches drift in about a week but can't tell a drifting judge from a real quality change: it fires on both. Splitting the judge's agreement with human labels by the human label separates the two, but needs thousands of labels per arm. Does re-judging a fixed canary set daily get both: fast, specific, and cheap? And how big does the canary need to be?

## The setup

There is no model API. Everything is simulated, so the drift day is known; the monitors aren't told it. The test and scenarios are loop 11's:

- **The test:** 28 days, 500 units per arm per day, control quality rate 0.60, no true effect.
- **The judge** starts at sensitivity = specificity = 0.85. From day 15 its specificity drops to 0.75 on the arms named, at once (step) or gradually until day 28.
- **Five scenarios:** no change; judge drift on both arms; on treatment only (step); on treatment only (gradual); and true quality falling by 0.08 on both arms with the judge unchanged.
- **The canary:** stored outputs saved before the change, re-scored daily by the current judge. The monitor compares each output's average verdict over the latest 7 days with its own average over the first 7, and tests the mean of those paired differences across outputs.
- **Repeatability:** how often the judge gives the same stored output the same verdict on different days, from 0 (a fresh, independent verdict each day) to 1 (deterministic). It doesn't change the judge's error rates. The main comparison uses 0, the worst case for the canary.
- **Every monitor** checks daily from day 14 to 28, with a Bonferroni threshold that keeps the chance of any false alarm at or below 5% when the judge doesn't change.

It's 2,000 replications per scenario, seed `20261127`. `python scripts/canary_drift_sim.py` reproduces it in about 90 seconds, and CI smoke-runs it.

![Canary drift simulation](https://raw.githubusercontent.com/wavde/experiment-toolkit/main/docs/canary_drift.png)

## Specific, not faster

Alarm rates by the end of the test (median days after the change, for judge drift):

| Scenario | Score monitor, both arms | Agreement monitor, 500 labels per arm | Canary, 800 control outputs | Canary, 400 outputs per arm |
|---|---:|---:|---:|---:|
| No change | 0.025 | 0.036 | 0.018 | 0.028 |
| Judge drifts on both arms (step) | 0.971 (6) | 0.153 (8) | 1.000 (4) | 0.996 (5) |
| Judge drifts on treatment only (step) | 0.838 (6) | 0.086 (8) | 0.020 | 0.932 (6) |
| Judge drifts on treatment only (gradual) | 0.363 (12) | 0.043 (8) | 0.022 | 0.543 (12) |
| True quality falls, judge unchanged | 1.000 | 0.032 | 0.019 | 0.021 |

**The canary is specific by construction.** Stored outputs don't get worse when live quality does, so the quality drop left the canary at 0.021, the same as no change. The score monitor fired every time. That's the difference between "something changed, go find out what" and "the judge changed".

**It's more powerful, not faster.** On treatment-only drift the canary caught 0.932 against 0.838, and on gradual drift 0.543 against 0.363. But the median delay was the same: 6 days for a step, 12 for gradual. Both compare a 7-day window with a 7-day reference, and that window, not the data source, sets the delay.

**A canary built from control outputs misses the drift that matters.** A canary of 800 control outputs, which is what a team gets if it saves production outputs before the test, caught drift on both arms every time but treatment-only drift only 0.020 of the time, no more than a false alarm. Loop 11 showed that drift which treats the arms differently is the drift that biases the readout. The canary has to contain treatment outputs.

**It costs judge calls, not labels.** 400 outputs per arm is 800 judge calls a day, against the 1,000 units a day the test already judges. The agreement monitor needed human labels and still caught only 0.086.

## How big the canary needs to be

Treatment-only step drift, 500 replications per cell:

| Canary outputs per arm | Repeatability 0.0 | Repeatability 0.5 | Repeatability 0.9 | No change (repeatability 0.0) |
|---:|---:|---:|---:|---:|
| 50 | 0.142 | 0.180 | 0.112 | 0.024 |
| 100 | 0.278 | 0.386 | 0.470 | 0.034 |
| 200 | 0.606 | 0.780 | 0.920 | 0.020 |
| 400 | 0.940 | 0.980 | 1.000 | 0.020 |
| 800 | 1.000 | 1.000 | 1.000 | 0.018 |

With fully independent re-judging, one check on a fully drifted week flags with 80% probability at **420** outputs per arm (closed form; 0.773 at 400). A judge that mostly repeats its verdicts needs fewer: at repeatability 0.9, 200 outputs per arm caught the drift 0.920 of the time. The paired test compares each output with itself, so the more consistent the judge, the more clearly a real change stands out. False alarms stayed between 0.006 and 0.034 in every size and repeatability tested.

## Assumptions and limitations

- **Loop 11's model:** one drift starting on day 15, a judge that becomes lenient on bad outputs, binary quality, confusion-matrix judges.
- **The canary outputs represent each arm.** Drift that only affects outputs the canary doesn't contain, such as a new prompt type added mid-test, is invisible to it, just as treatment-only drift was invisible to the control-only canary.
- **Repeatability is a modelling choice.** Real judges at temperature 0 are close to deterministic but not exactly; the sweep covers 0 to 0.9. It doesn't model drift that changes *which* outputs the judge errs on without changing its error rates.
- **The canary is re-judged with the same prompt and settings as production.** If the canary pipeline differs (a cached response, a different system prompt), it can miss a change the production judge has.
- **A drift inside the reference week is invisible,** as with every monitor here. Starting the canary before launch avoids that.

## When the conclusion breaks

- **Outputs can't be stored,** for example for privacy reasons. Then only live-traffic monitors are available, and loop 11's trade-off applies.
- **The provider pins model versions.** If the judge can't change without a version flag, loop 9's stratified analysis applies and a canary is a cheap confirmation, not a necessity.
- **Drift in the human labels.** The canary watches the judge, not the labelers. A labeling vendor or guideline change still needs its own check.

## What would change my mind

- **A real hosted judge whose daily verdicts on fixed outputs vary far more than its error rates imply,** for example from load-dependent sampling. Then the canary would need to be larger, or its threshold re-calibrated on a no-change period.
- **Evidence that real judge drift is concentrated in rare output types.** A canary of a few hundred outputs would then miss it, and a canary sampled to cover those types would be needed.

## The decision rule

1. **Keep loop 11's rule for the readout:** human labels drawn at random from the whole test, analysed with PPI++. The canary detects drift; it doesn't correct it.
2. **Store a canary set from every arm at launch and re-judge it daily** with the production judge. About 400 outputs per arm if the judge's verdicts are independent from day to day, about 200 if they mostly repeat. A canary of control outputs alone caught treatment-only drift 0.020 of the time.
3. **Treat a canary alarm as judge drift.** It stayed at 0.021 when real quality fell. Stop trusting judge-only dashboards and interim reads from the alarm day.
4. **Treat a score alarm without a canary alarm as a real change,** not a judge problem, as long as the canary covers every arm's outputs. With both monitors running, the pair says which one happened.

---

*Code, tests, and results: [`judge_drift.py`](https://github.com/wavde/experiment-toolkit/blob/main/src/experiment_toolkit/judge_drift.py) · [`test_judge_drift.py`](https://github.com/wavde/experiment-toolkit/blob/main/tests/test_judge_drift.py) · [`canary_drift_sim.py`](https://github.com/wavde/experiment-toolkit/blob/main/scripts/canary_drift_sim.py) · [`canary_drift_results.md`](https://github.com/wavde/experiment-toolkit/blob/main/docs/canary_drift_results.md) · References: Chen, Zaharia, Zou (2023), How is ChatGPT's behavior changing over time? · Angelopoulos, Duchi, Zrnic (2023), PPI++.*
