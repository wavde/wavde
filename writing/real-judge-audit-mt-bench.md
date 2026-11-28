# A real LLM judge favours the winner: auditing GPT-4 against expert humans

*~7 min read. Companion code is [`judge_audit`](https://github.com/wavde/experiment-toolkit/blob/main/src/experiment_toolkit/judge_audit.py) in experiment-toolkit, with [tests](https://github.com/wavde/experiment-toolkit/blob/main/tests/test_judge_audit.py), a [one-command audit](https://github.com/wavde/experiment-toolkit/blob/main/scripts/judge_audit_mtbench.py), and the [generated results](https://github.com/wavde/experiment-toolkit/blob/main/docs/judge_audit_results.md). It checks the assumption behind [LLM-as-judge is a noisy metric](llm-judge-noise-in-ab-tests.md) and the judge loops that followed, and updates [Which number supports which decision](which-number-supports-which-decision.md).*

## The recommendation

Don't ship on an LLM judge's side-by-side win rate. On public data where expert humans and GPT-4 judged the same comparisons, the judge's error depended on which model the humans preferred, which is exactly the arm-dependent error the earlier simulations warned about. Its win rate for the stronger model was above the experts' in **all 15** model pairs, by up to **+0.185**.

The fix from the simulations holds up on real verdicts: PPI++ with human labels drawn at random kept its confidence intervals honest. But the real judge is a weaker shortcut than the simulations assumed. It correlated with expert verdicts at 0.14 to 0.56, against 0.69 for the simulated judge in loop 1, so PPI++ saved only **2–28%** of the human labels, not the 43% the simulated judge implies. **Budget for human labels as if the judge barely helps.**

## The decision

Six loops ([1](llm-judge-noise-in-ab-tests.md), [5](offline-evals-vs-online-ab.md), [7](ppi-with-covariates-label-budget.md), [9](judge-drift-mid-test.md), [11](silent-judge-drift.md), [12](canary-set-judge-drift.md)) rest on simulated judges whose errors can depend on the arm. A reviewer's first question is whether real judges behave that way at all, and how much a real judge actually helps once you correct it. If real judge error doesn't depend on the arm, the judge-only metric is safe and most of those rules are over-cautious. If it does, the rules stand, and the label budget depends on how informative the real judge is.

## The setup

No model API calls and no credentials. The data are the public [MT-Bench human judgments](https://huggingface.co/datasets/lmsys/mt_bench_human_judgments) (Zheng et al., 2023, CC BY 4.0), downloaded at runtime from a pinned revision and checked against SHA-256 hashes.

- **What was judged:** answers from six 2023 models (GPT-4, Claude-v1, GPT-3.5, Vicuna-13B, Alpaca-13B, LLaMA-13B) to 80 two-turn MT-Bench questions, compared in pairs.
- **Who judged:** expert annotators, mostly graduate students in each question's field, and GPT-4 with a pairwise prompt run in both answer orders. An inconsistent pair counts as a tie.
- **As an A/B test:** each pair of models is a side-by-side test, with the stronger model (by the paper's human ranking) as treatment. Per item (one question and turn), the verdict is 1 if the stronger model wins, 0.5 for a tie and 0 if it loses, averaged over experts when several judged it. The target is the experts' win rate.
- **Items:** 1,814 comparisons judged by both, across 15 pairs.
- **A known-answer check on real verdicts.** Each replication draws 2,000 items with replacement from one pair's real (human, GPT-4) verdicts, so the true win rate is known exactly. Every item gets a GPT-4 verdict and a random subset gets its human verdict. Then the judge alone, the human labels alone and PPI++ are compared against the truth, 1,000 times per pair.

Seed `20261127`. `python scripts/judge_audit_mtbench.py` reproduces it in seconds after a 1.4 MB download, and CI smoke-runs it on synthetic data with the same schema.

![Judge audit](https://raw.githubusercontent.com/wavde/experiment-toolkit/main/docs/judge_audit.png)

## The judge inflates the winner

| Pair (stronger first) | Experts | GPT-4 | Bias (95% CI) |
|---|---:|---:|---:|
| GPT-4 vs GPT-3.5 | 0.615 | 0.800 | +0.185 (+0.117, +0.252) |
| Claude-v1 vs GPT-3.5 | 0.509 | 0.641 | +0.131 (+0.062, +0.203) |
| GPT-4 vs Alpaca-13B | 0.839 | 0.955 | +0.116 (+0.058, +0.178) |
| Claude-v1 vs Alpaca-13B | 0.861 | 0.873 | +0.012 (-0.041, +0.064) |
| GPT-3.5 vs Vicuna-13B | 0.646 | 0.664 | +0.018 (-0.037, +0.076) |

(Five of the 15 pairs; the [results](https://github.com/wavde/experiment-toolkit/blob/main/docs/judge_audit_results.md) have all of them. The CIs resample questions.)

The bias is positive in 15 of 15 pairs and its CI excludes zero in 9. It's larger when GPT-4 is one of the contestants, +0.117 on average against +0.057 for the other 10 pairs, which fits the self-enhancement bias the MT-Bench authors reported. But it isn't only self-preference: Claude-v1 vs GPT-3.5 is +0.131.

That pair is the decision-changing one. Experts split it almost evenly, 0.509. GPT-4 gives Claude-v1 0.641. In the known-answer check at 2,000 prompts, the judge's confidence interval sat above 0.5, meaning "ship Claude", in **1.000** of replications. With 200 human labels, PPI++ did so in 0.060 and the human labels alone in 0.051.

## The error depends on the arm

The judge doesn't just add noise. It errs in one direction:

| Pairs | Experts prefer the stronger model: GPT-4 agrees | Experts prefer the weaker model: GPT-4 agrees |
|---|---:|---:|
| All pairs | 0.823 (1,188 items) | 0.221 (249 items) |
| GPT-4 is a contestant | 0.864 | 0.161 |
| GPT-4 isn't a contestant | 0.804 | 0.256 |

When experts preferred the stronger model, GPT-4 agreed 82% of the time. When they preferred the weaker one, GPT-4 agreed 22% of the time. In loop 1's terms, the judge is far more lenient on the treatment arm's outputs than on control's. That is the assumption every judge loop simulated, here on real data.

## The fix holds, but saves fewer labels

In the known-answer check, with 200 human labels per pair:

- **GPT-4 alone** covered the true win rate in at most 0.005 of replications in 13 of 15 pairs, and at most 0.436 in the other two.
- **Human labels alone** covered it 0.934 to 0.957 of the time.
- **PPI++** (judge plus the same human labels) covered it 0.930 to 0.956 of the time.

So PPI++ stays valid when the judge's error depends on the arm, as the simulations said. What it buys is smaller than they implied. PPI++ helps in proportion to how well the judge tracks the human verdicts:

| | Judge–human correlation | Labels PPI++ saves (200 of 2,000) |
|---|---:|---:|
| Loop 1's simulated judge (sensitivity = specificity = 0.85) | 0.69 | 43% |
| MT-Bench, median pair | 0.35 | 11% |
| MT-Bench, range over pairs | 0.14 to 0.56 | 2% to 28% |

The closed form matched the simulation: PPI++'s standard error relative to human labels alone was 0.847 to 0.990 simulated against 0.848 to 0.992 predicted.

Part of the low correlation is the humans, not the judge. Expert verdicts on a single comparison are noisy, and PPI++ is calibrated to those verdicts, so this is the correlation that matters for the label budget.

## Assumptions and limitations

- **2023 models and GPT-4 (2023) as judge.** Current judges may be better calibrated, or biased differently. This is one judge on one benchmark.
- **Pairwise judgments, not per-output scores.** Loop 1 simulated a judge scoring each output alone. A side-by-side win rate is a common way LLM changes are evaluated, but per-output scoring could behave differently.
- **"Stronger" comes from the paper's human ranking,** so the direction of the bias is stated relative to a ranking from the same human data. The arm-dependent agreement in the second table doesn't depend on that ranking: it conditions on which model the experts preferred on each item.
- **The resampling check treats the 1,814 judged items as the population.** It shows that PPI++ is valid on the joint distribution of real human and judge verdicts; it doesn't show the items represent your traffic.
- **Which items humans judged was chosen by the dataset authors.** The bias compares judge and humans on the same items, so selection doesn't bias the comparison, but it limits how far the win rates generalise.

## When the conclusion breaks

- **The two arms' outputs are indistinguishable to the judge,** such as a serving change with identical text. Then arm-dependent error can't arise, as loop 1 said.
- **A judge validated on your own traffic** with judge–human agreement that doesn't depend on which arm humans prefer. This audit shows the property can't be assumed, not that every judge lacks it.
- **Much more informative judges.** If a current judge reaches a correlation near 0.7 with your human labels, loop 1 and loop 7's label budgets apply again.

## What would change my mind

- **An audit of current judges and models,** on public paired human and judge verdicts, finding agreement that doesn't depend on which model humans preferred.
- **Evidence that the low correlation is mostly human noise** that a better labeling protocol removes, for example several raters per item. Then the budget question becomes how to label, not how many.

## The decision rule

1. **Don't ship on a judge-only win rate,** even from a strong judge. On MT-Bench, GPT-4 inflated the stronger model's win rate in 15 of 15 pairs and called a near tie (experts 0.509) for one model in every replication.
2. **Anchor the judge with human labels drawn at random and analyse with PPI++.** On real verdicts it kept 0.930 to 0.956 coverage where the judge alone kept at most 0.005 in most pairs.
3. **Size the label budget from the judge–human correlation measured on your own labels,** not from simulated judges. At the correlations seen here, plan for PPI++ to save about a tenth of the labels, not about half.
4. **Expect more bias when the judge is related to a contestant.** It was +0.117 where GPT-4 judged itself, against +0.057 otherwise.

---

*Code, tests, and results: [`judge_audit.py`](https://github.com/wavde/experiment-toolkit/blob/main/src/experiment_toolkit/judge_audit.py) · [`test_judge_audit.py`](https://github.com/wavde/experiment-toolkit/blob/main/tests/test_judge_audit.py) · [`judge_audit_mtbench.py`](https://github.com/wavde/experiment-toolkit/blob/main/scripts/judge_audit_mtbench.py) · [`judge_audit_results.md`](https://github.com/wavde/experiment-toolkit/blob/main/docs/judge_audit_results.md) · Data: Zheng, Chiang, Sheng et al. (2023), Judging LLM-as-a-judge with MT-Bench and Chatbot Arena, [lmsys/mt_bench_human_judgments](https://huggingface.co/datasets/lmsys/mt_bench_human_judgments), CC BY 4.0 · References: Angelopoulos, Duchi, Zrnic (2023), PPI++.*
