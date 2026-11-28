# An offline eval win is evidence about the eval set: offline LLM evals vs online A/B

*~7 min read. Companion code is [`offline_eval`](https://github.com/wavde/experiment-toolkit/blob/main/src/experiment_toolkit/offline_eval.py) in experiment-toolkit, with [tests](https://github.com/wavde/experiment-toolkit/blob/main/tests/test_offline_eval.py), a [one-command simulation](https://github.com/wavde/experiment-toolkit/blob/main/scripts/offline_online_sim.py), and the [generated results](https://github.com/wavde/experiment-toolkit/blob/main/docs/offline_online_results.md).*

## The recommendation

Use an offline LLM eval to decide **what to test online**, not whether to ship. Before reading an offline win as a prediction, do three things:

- Reweight the eval to the live traffic mix, segment by segment.
- If the candidate changes how its outputs look, correct the judge with human labels drawn at random from the eval set (PPI++).
- Make the final ship decision on an online A/B with guardrails. Some costs only exist online.

The reason, from a simulation with known ground truth: a candidate that helps coding prompts and hurts chat prompts has a negative effect on live traffic. The raw offline eval ships it in **100%** of replications, and the reweighted readouts ship it in **0–1%**. A candidate that adds latency also hurts live traffic. No offline readout ever stops it, while the online test stops it in **96%**.

## The decision

A team has a candidate model or prompt, B, to replace the baseline, A. They run both on an offline eval set, an LLM judge scores each output, and B wins. Can they ship on that? If not, what has to be true before the offline win predicts the online result?

## The setup

The simulation makes no model API calls. Everything is known by construction.

- **Three prompt segments**: chat, coding and reasoning. The eval set is 20% / 60% / 20%, because eval sets tend to over-sample the hard, checkable cases. Live traffic is 70% / 20% / 10%.
- **True quality** is binary per output. The baseline's rate is 0.60, 0.50 and 0.40 by segment. B changes it by a known amount per segment.
- **The judge** has sensitivity = specificity = 0.85 on A's outputs. In one scenario its specificity on B's outputs drops to 0.80, so it waves through more of B's bad answers.
- **Online**, a user's success probability is 0.1 + 0.8 × quality − online cost. The online cost stands for something the eval can't see, such as latency causing abandonment.

Each replication uses 10,000 eval prompts (1,000 of them human-labeled) and an online test with 200,000 users per arm. Four readouts are compared:

- the raw offline judge difference on the eval mix;
- the judge difference reweighted to the traffic mix;
- PPI++ per segment (judge scores corrected with the human labels, as in the [LLM-as-judge post](llm-judge-noise-in-ab-tests.md)), reweighted to the traffic mix;
- the online A/B.

A readout *ships* B when its 95% CI is above zero and *stops* it when the CI is below zero. Each simulated mean sits next to its closed-form value, and the two agree. Seed `20261127`; `python scripts/offline_online_sim.py` reproduces it in about 20 seconds.

![Offline vs online simulation](https://raw.githubusercontent.com/wavde/experiment-toolkit/main/docs/offline_online.png)

## Three ways an offline win misleads

| Scenario (true online effect) | Raw judge | Reweighted judge | PPI++, reweighted | Online A/B |
|---|---:|---:|---:|---:|
| Aligned win (+0.0240) | ships 87% | ships 49% | ships 25% | ships 100% |
| No effect (0) | ships 1% | ships 3% | ships 3% | ships 3% |
| Mix shift (−0.0064) | **ships 100%** | ships 0% | ships 1% | stops 99% |
| Judge lenient on B, no effect (0) | **ships 93%** | **ships 53%** | ships 3% | ships 2% |
| Online-only cost (−0.0060) | **ships 87%** | **ships 47%** | **ships 22%** | stops 96% |

**Mix shift.** B helps coding by 10 points and hurts chat by 4. On the eval mix that averages to a large win: the judged difference is +0.0362. On the traffic mix it's a loss, and the online effect is −0.0064. The offline eval is answering a different question than the one the launch asks: it reports B's effect on the eval set, not on users. Reweighting each segment's result to the traffic mix fixes the sign. The reweighted judge readout averages −0.0064 against an expected −0.0056.

**Judge bias.** B is no better, but its outputs are easier for the judge to like (for example, they're longer). The raw readout ships 93% of the time. Reweighting can't help, because the bias is present in every segment, so it still ships 53% of the time. PPI++, which uses the human labels to learn how the judge errs on each model's outputs, brings this down to 3%, the same as the no-effect baseline.

**Online-only cost.** B really is better on quality, by 3 points in every segment. But it adds enough latency to cost 3 points of online success, so the net online effect is −0.0060. Every offline readout is correct about quality, so every one of them ships B some of the time and none of them ever stops it. Only the online test sees the cost.

## What the corrections cost

Each correction trades power for protection. On a real, aligned win, the raw readout ships 87% of the time, the reweighted one 49% and PPI++ 25%. Varying the eval set size with the same 1,000 labels shows the trade:

| Eval prompts | Raw: ships the real win | Raw: ships the harmful mix shift | Reweighted: real win | Reweighted: harm | PPI++: real win | PPI++: harm |
|---:|---:|---:|---:|---:|---:|---:|
| 2,000 | 23% | 66% | 13% | 2% | 15% | 2% |
| 5,000 | 53% | 96% | 26% | 1% | 21% | 2% |
| 10,000 | 81% | 100% | 45% | 0% | 23% | 1% |
| 20,000 | 99% | 100% | 77% | 0% | 21% | 1% |

Two things stand out.

- **A bigger eval set makes the raw readout more confident, not more correct.** At 20,000 prompts it ships the real win 99% of the time and the harmful change 100% of the time.
- **PPI++ power stops growing at about 21–23%,** because it is capped by the 1,000 human labels, not by the number of prompts. This is the same binding constraint as in the LLM-as-judge loop.

Reweighting costs power here because chat is 70% of traffic but only 20% of the eval set, so the reweighted estimate leans on the smallest sample. This follows from the post-stratification variance, but the simulation didn't test the remedy: if the eval set is sampled to match the traffic mix, reweighting costs almost nothing.

The offline readouts are also weak at detecting harm. In the mix-shift scenario they correctly decline to ship B, but they stop it in only 6–8% of replications. "The offline eval didn't flag a problem" is not evidence that there isn't one.

## Assumptions and limitations

- **Synthetic, and one mechanism at a time.** Real candidates combine mix shift, judge bias and online costs, and the segments aren't known in advance.
- **Known segments with exact traffic shares.** Reweighting needs a segment label on every live prompt and a stable traffic mix. Real segment definitions are fuzzy, and the mix shifts by day and by surface.
- **Online success is linear in quality.** The 0.8 slope and the 0.1 floor are chosen. If users don't notice small quality differences, online effects shrink and the online test needs more users.
- **Binary quality, one judge, independent outputs.** No graded scores, pairwise preferences or paired analysis, and no judge drift.
- **Human labels as ground truth,** drawn at random from the eval set, as in the LLM-as-judge loop.

## When the conclusion breaks

- **The eval set doesn't cover a traffic segment at all.** Reweighting needs at least a few eval prompts in every live segment. A segment the eval set never samples, such as a new language or use case, is invisible offline.
- **The online metric doesn't reflect quality.** If users can't tell or don't care, a real offline quality win can have no online effect. Then the online test is the arbiter of value, and the offline eval only measures quality.
- **The traffic mix moves after launch.** Reweighting to today's mix predicts today's effect. A launch that changes who uses the feature changes the mix.

## What would change my mind

- **Measured offline-online agreement.** A record across many past launches showing that the reweighted, human-calibrated offline readout matched the online sign, especially for changes with no latency, cost or UI impact. Then those offline readouts could gate low-risk changes on their own.
- **Online-only costs measured separately and found negligible** for a class of changes, such as prompt edits with no change in length or latency.
- **A label budget large enough to give PPI++ real power.** At 1,000 labels it ships a real win only 25% of the time. If labels were cheap enough to make that 80%, the offline gate becomes a useful filter rather than a formality.

*Update after the [real-judge audit](real-judge-audit-mt-bench.md): these power numbers use a simulated judge with sensitivity = specificity = 0.85. On public MT-Bench data, GPT-4's verdicts correlated with expert verdicts at 0.14 to 0.56, against 0.69 for that judge in loop 1's setting, so with the same 1,000 labels a real judge would give PPI++ less power, not more. The audit also found the judge error this essay's lenient-judge scenario assumes: GPT-4's agreement with experts was 0.823 when they preferred the stronger model and 0.221 when they preferred the weaker one.*

## The decision rule

1. **An offline eval win is a reason to run an online test, not a substitute for one.** The final ship decision belongs to an online A/B with guardrails on latency, cost and abandonment. Those costs don't exist offline.
2. **Read offline results per segment and reweight to traffic.** A win concentrated in a segment the eval set over-represents is the signature of mix shift. Better still, sample the eval set to match traffic.
3. **When the candidate changes how its outputs look, correct the judge** with human labels drawn at random from the eval set (PPI++). Size the label budget for the effect you need to detect, because labels, not prompts, cap the power.
4. **Don't treat a quiet offline eval as a safety check.** Offline readouts rarely stop a harmful change; they mostly fail to ship it.

---

*Code, tests, and results: [`offline_eval.py`](https://github.com/wavde/experiment-toolkit/blob/main/src/experiment_toolkit/offline_eval.py) · [`test_offline_eval.py`](https://github.com/wavde/experiment-toolkit/blob/main/tests/test_offline_eval.py) · [`offline_online_sim.py`](https://github.com/wavde/experiment-toolkit/blob/main/scripts/offline_online_sim.py) · [`offline_online_results.md`](https://github.com/wavde/experiment-toolkit/blob/main/docs/offline_online_results.md) · References: Angelopoulos, Bates, Fannjiang, Jordan, Zrnic (2023), Prediction-powered inference · Angelopoulos, Duchi, Zrnic (2023), PPI++ · Kohavi, Tang, Xu (2020), Trustworthy Online Controlled Experiments · Zheng et al. (2023), Judging LLM-as-a-judge.*
