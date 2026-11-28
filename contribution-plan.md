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

## Review after loops 5–7

All three themes from the first review shipped and met the finish criteria. No loop stalled, so nothing is cut.

**What held up**

- Both process changes from the first review paid off. Loop 6 ran on public randomized data (Criteo) and corrected a loop-4 assumption: marginal returns fell far faster than the simulation had them (0.15 of average for visits, not 0.81). CI now smoke-runs the evidence scripts in experiment-toolkit and paid-media-playbook.
- Closed forms kept scaling. Loop 7's label budgets reproduced loop 1's exactly, so each loop can now be checked against the previous one as well as against ground truth.
- The decision rules kept converging on randomizing at the decision being made. Loop 5 reached it through online tests with guardrails, loop 6 through marginal budget tests, and loop 7 through human labels drawn within each arm.

**What didn't**

- The cost problem is only partly solved. Loop 7 cut the label budget by 5–32%, but a better judge saved more than any covariate, and the staggered-rollout question from theme 3 is still open.
- Loop 3's recommendation, a holdout or staggered rollout instead of an opt-in comparison, has no evidence yet on which design to use or what each costs.
- The LLM-judge loops all assume a fixed judge, and every retro flagged it.
- CI smoke runs are still missing in causal-inference-playbook and demand-forecasting.

**Changes for the next round**

- Each loop names the previous loop's number it reproduces or revises, as loop 7 did.
- Close the CI smoke gap in any repo a loop touches.

## Review after loops 8–10

All three round 3 themes shipped and met the finish criteria. No loop stalled.

**What held up**

- Each loop checked a previous loop's assumption, as the second review asked. Loop 8 answered loop 3's open "holdout or staggered rollout" question. Loop 9 removed the fixed-judge assumption from loops 1 and 7. Loop 10 re-tested loop 6 on a second public dataset.
- Replication did its job. Loop 10 showed that loop 6's headline (uplift concentrates in high-propensity users: 72.2% of incremental visits in the top 20% on Criteo) is dataset-specific. On Hillstrom the top 20% held 25.9%. A plan that only ran new loops would never have caught that.
- CI smoke runs now cover the evidence in experiment-toolkit, causal-inference-playbook and paid-media-playbook.

**What didn't**

- Loop 6's published decision rule still states the non-replicating finding as a general pattern. Loop 10 added an update note, but the rule, the "what the simulation got wrong" section and the robustness claim in the essay body are unchanged. A published claim that failed replication is the largest defensibility gap in the portfolio.
- demand-forecasting still has no CI smoke run, so loop 2's evidence can break from a clean install without anyone noticing.
- Loop 9 covers a judge upgrade at a known time. Silent drift with no version boundary, such as a hosted model updated under the same name, is still open.

**Priorities for round 4**, ranked by defensibility first and new coverage second:

1. **Fix: revise loop 6's essay** so its decision rule, body and "what would change my mind" reflect loop 10. Small, and it removes a claim the evidence no longer supports.
2. **Fix: add a CI smoke run to demand-forecasting**, closing the last gap from the first review's process change.
3. **New loop: judge drift without a version boundary.** Decision: how can a team detect and handle a judge that changes silently during a test? Builds on loop 9.

Priority 1 is done: the loop 6 essay's recommendation, decision rule and generalising sections now say which dataset supports each claim. Priority 2 is done: demand-forecasting's CI now smoke-runs `run.py` and `run_pretrained.py` on synthetic data with the UCI schema, using a seasonal-naive stub in place of Chronos-Bolt (no download, no torch) ([demand-forecasting#3](https://github.com/wavde/demand-forecasting/pull/3)). Every repo's evidence pipelines now run in CI. Priority 3 is done (loop 11): with labels drawn at random from the whole test, PPI++ stays valid under silent drift without detecting it, and score monitors are tripwires that can't tell drift from real change ([experiment-toolkit#8](https://github.com/wavde/experiment-toolkit/pull/8)). Round 4 is complete, and the plan is due for its fourth review before another loop starts.

## Review after round 4

Round 4 ran the third review's priorities in order: two fixes, then one loop. All three shipped and met the finish criteria. No item stalled.

**What held up**

- Ranking fixes above new coverage was right. The loop 6 essay no longer states a finding that failed replication, and every repo's evidence pipelines now smoke-run in CI, so a clean-install break anywhere is caught automatically.
- Loop 11 reused loop 9's module and loop 1's PPI++ unchanged and closed the last open item on judge drift. Its main answer, that labels drawn at random from the whole test protect the readout without detecting drift, is the same rule loops 1 and 9 reached: randomize at the decision you're making.
- Eleven loops have converged on a small set of rules. Almost every one says which number can carry which decision, and which can't.

**What didn't**

- The rules are spread across eleven essays, and nothing ties them together. A reader who wants the decision rule for measuring an AI feature has to read most of them. That cuts against the principle of fewer, defensible pieces more than another loop would help it.
- The profile lists 21 essays in one flat, newest-first list. The AI-measurement work, which is the focus, is mixed in with older method notes.
- The evidence is still mostly simulation. Only one loop's conclusions were tested on a second dataset, and one of them didn't replicate. No LLM-judge result has been checked against a real judge.
- Loop 11 named a canary set of stored outputs re-judged daily as likely the cheapest specific drift monitor, but it isn't simulated.

**Priorities for round 5**, ranked by what most improves the defensibility and readability of what's already published:

1. **Consolidate: a synthesis essay, "which number supports which decision".** Combine the existing decision rules for measuring AI features into one map, linking each rule to its evidence. No new numbers; cite only merged evidence, and state the replication status of each claim, including loop 6's finding that didn't replicate. It ranks first because it's the entry point a reviewer reads first, and the grouped Writing section in priority 2 should lead with it.
2. **Consolidate: group the README Writing section by theme,** with AI measurement first and the synthesis at its head, instead of one flat list.
3. **New loop: a canary-set drift monitor.** Decision: does re-judging a fixed set of stored outputs every day detect silent judge drift faster and more specifically than the score and agreement monitors from loop 11? It ranks last because loop 11's rule already protects the readout without detection, so this improves diagnosis rather than validity.

**Changes for the next round**

- Before starting a new loop, check whether its decision rule can be added to the synthesis. A loop that can't change or sharpen a line of the map is a lower priority.
- Consolidation counts as progress. A round can end without a new loop.

Priority 1 is done: [Which number supports which decision](writing/which-number-supports-which-decision.md) maps eleven decisions to the number that supports each, with the evidence and its replication status. Priority 2 is done: the Writing section in the profile README and `writing/README.md` is grouped by theme (measuring AI features, marketing measurement and targeting, experimentation and causal inference, forecasting), with the synthesis first. Priority 3 is done (loop 12): a canary of stored outputs from every arm, re-judged daily, caught silent judge drift as fast as the both-arm score monitor without firing on a real quality drop, and the synthesis's judge-drift row and diagnostic lines are updated ([experiment-toolkit#11](https://github.com/wavde/experiment-toolkit/pull/11)). Round 5 is complete, and the plan is due for its fifth review before another loop starts.

## Review after round 5

Round 5 ran two consolidation items and then one loop, as the fourth review ranked them. All three shipped and met the finish criteria. Off-plan, an essay re-reading five classic case studies added a bandit-evaluation simulation in experiment-toolkit. No item stalled.

**What held up**

- Consolidation was worth a round. The synthesis is now the entry point to the writing, and the grouped Writing section puts the AI-measurement work first.
- The fourth review's gate worked. Loop 12 changed a line of the synthesis, moving judge drift from "no diagnostic" to "checkable with a canary set", rather than adding an essay beside it.
- Reproducibility held under an independent check. A rerun of the bandit-evaluation simulation reproduced its committed results byte for byte.

**What didn't**

- Every LLM-judge loop (1, 5, 7, 9, 11 and 12) rests on a simulated judge. The assumption they share, that a judge's errors can depend on which arm produced the output, and the judge accuracy they assume have never been checked on real data. The synthesis leans on both. This is now the largest defensibility gap in the portfolio.
- The bandit-evaluation simulation takes about 33 minutes on a laptop CPU, and its results header didn't say so. A reproducibility claim should state its cost.
- The off-plan essay's log row cited a PR that was still open, and had to be corrected after it merged.

**Priorities for round 6**, ranked by defensibility first:

1. **New loop, on real data: audit an LLM judge against human judgments.** Decision: do a real judge's errors depend on which model produced the output, and does PPI++ with human labels still hold up on real verdicts? Evidence: the public MT-Bench human judgments (Zheng et al., 2023, CC BY 4.0), where expert humans and GPT-4 judged the same pairwise comparisons, downloaded at runtime from a pinned revision. No model API calls or credentials. It passes the gate: it changes the synthesis's judge rows and its replication section.
2. **Consolidate: add the bandit result to the synthesis** as a line for picking and quoting a creative a bandit chose, now that its evidence is merged ([experiment-toolkit#10](https://github.com/wavde/experiment-toolkit/pull/10)).
3. **Candidate loop: a second real-judge audit with a current judge and current models,** only if a public dataset of paired human and LLM-judge verdicts with a usable license exists. **Cut:** no such dataset was found (see "Priority 3 cut" below).

**Fix done in this review:** the bandit-evaluation runtime is now stated in its README section and results header, with the results unchanged ([experiment-toolkit#13](https://github.com/wavde/experiment-toolkit/pull/13)).

**Changes for the next round**

- Every evidence script states its full runtime next to its command.
- A log row cites only merged PRs.

Priority 1 is done (loop 13): on MT-Bench, GPT-4's errors depended on which model the experts preferred (agreement 0.823 against 0.221), PPI++ stayed valid on the real verdicts, and the real judge was much less informative than the simulated ones, so PPI++ saved 2–28% of labels rather than 43% ([experiment-toolkit#14](https://github.com/wavde/experiment-toolkit/pull/14)). The synthesis's judge row, label-budget pattern, replication section and limitations are updated. **Round 6 status:** complete. Priorities 1 and 2 done; priority 3 cut. Priority 2: the synthesis now has a row for quoting the lift of a creative a bandit picked, with the evidence from experiment-toolkit#10, and its decision rule names traffic the bandit never learned from. Next up: the sixth plan review, before any further loop.

**Priority 3 cut.** The loop needed one public dataset with all of: real human verdicts (not LLM-generated or constructed labels), LLM-judge verdicts on the same items from a 2024-or-later judge, outputs from current models, and a license that allows use, all downloadable without API calls or credentials. Each candidate checked on the Hugging Face Hub, and why it fails:

- `lmarena-ai/arena-human-preference-140k` (CC BY 4.0, 2025) and `arena-human-preference-100k` (CC BY 4.0): real human votes on current models, but no LLM-judge verdicts. Adding them would need API calls.
- `lmarena-ai/PPE-Human-Preference-V1` (2024): human Chatbot Arena votes on current models, but no per-item judge verdicts are published; the PPE code runs judges through APIs. No dataset-level license: prompts are CC BY 4.0 and model outputs fall under each provider's terms.
- `lmsys/arena-human-preference-55k`: gated (needs credentials), but an ungated Apache 2.0 copy is at `lmarena-ai/arena-human-preference-55k`. Either way, human votes only.
- `nvidia/HelpSteer2` and `nvidia/HelpSteer3` (CC BY 4.0, HelpSteer3 2025): real human annotations, but no LLM-judge verdicts.
- `ScalerLab/JudgeBench` (MIT, 2024, GPT-4o and Claude-3.5 responses): labels are objective correctness, not human judgments, and no judge verdicts are included.
- `allenai/reward-bench` and `reward-bench-2` (ODC-BY): chosen/rejected labels are a mix of human-verified, correctness-based and LLM-judged (the MT-Bench GPT-4 subsets), and they contain no judge verdicts.
- `allenai/reward-bench-results`: the nearest miss. It has per-item verdicts from GPT-4o (2024-05-13) and GPT-4-turbo (2024-04-09) on human-labelled preference sets, including the same MT-Bench human judgments loop 13 used. But the judged outputs are from 2023 models or older, the verdicts are chosen-vs-rejected only, and the repository states no license.
- `tatsu-lab/alpaca_eval`: human and GPT-4 annotations, but from 2023 and CC BY-NC.

A candidate for the sixth review, not started: re-run loop 13's audit with GPT-4o as judge on the same MT-Bench items using `reward-bench-results`, if its license is clarified. That would test whether a newer judge shows the same arm-dependent error, but not on current models.

## Round 3 themes (done)

1. **Staggered rollout vs fixed holdout for opt-in AI features.** Decision: for the same withheld traffic, should a team run a long, small holdout, a short staggered rollout, or both? What does each one estimate when the effect grows (learning) or fades (novelty)? Evidence: a simulation in causal-inference-playbook with known short-run and long-run effects. It closes the open theme-3 candidate and the gap left by loop 3.
2. **Judge drift during a test.** Decision: what happens to a judge-scored A/B test when the judge model is upgraded mid-test, and how should the analysis handle it? Builds on experiment-toolkit's `llm_judge`.
3. **A second public randomized dataset.** Decision: does loop 6's finding (uplift concentrates in high-propensity users) hold for a different advertiser and channel? Candidate data: the Hillstrom email-marketing experiment.

All three are done: theme 1 (loop 8), theme 2 (loop 9) and theme 3 (loop 10). See "Review after loops 8–10" above.

## Round 2 themes (done)

Ranked by AI relevance and by how directly each one answers a gap in the first review.

1. **Offline evals vs online A/B for LLM features.** Decision: when can an offline eval win be trusted to predict, or replace, an online test of an LLM feature? Evidence: a simulation in experiment-toolkit where the offline eval set differs from live traffic (prompt mix, judge bias, and the effect varies by segment). It measures how often the offline winner loses online, and whether reweighting the eval set to the traffic mix fixes it. This closes the open part of theme 1.
2. **Public-data check of uplift vs propensity targeting.** Decision: on a real randomized ad dataset, does ranking users by conversion propensity rather than uplift waste spend the way loop 4's simulation says? Evidence: a replication in paid-media-playbook on the public Criteo uplift dataset, which comes from a randomized trial. This answers the synthetic-data gap.
3. **Cheaper corrections.** Decision: how far can sequential monitoring and variance reduction cut the cost of a valid answer? Candidates are combining PPI++ with CUPED-style covariates for the label budget, and staggered-rollout power versus a fixed holdout for opt-in features. It builds on experiment-toolkit's mSPRT and CUPED modules.

All three are done: theme 1 (loop 5), theme 2 (loop 6) and theme 3 (loop 7, the PPI++ covariate candidate). The staggered-rollout candidate moved to round 3.

## Round 1 themes (done)

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
| Offline evals vs online A/B | Done | [experiment-toolkit#2](https://github.com/wavde/experiment-toolkit/pull/2), [results](https://github.com/wavde/experiment-toolkit/blob/main/docs/offline_online_results.md) | [An offline eval win is evidence about the eval set](writing/offline-evals-vs-online-ab.md) |
| Propensity vs uplift targeting on real randomized data | Done | [paid-media-playbook#3](https://github.com/wavde/paid-media-playbook/pull/3), [results](https://github.com/wavde/paid-media-playbook/blob/main/case-studies/07-criteo-uplift/results.md) | [The targeting model already knows who would convert](writing/propensity-vs-uplift-targeting-criteo.md) |
| PPI++ with a pre-treatment covariate | Done | [experiment-toolkit#4](https://github.com/wavde/experiment-toolkit/pull/4), [results](https://github.com/wavde/experiment-toolkit/blob/main/docs/ppi_covariates_results.md) | [A covariate stacks on the judge, but a better judge beats both](writing/ppi-with-covariates-label-budget.md) |
| Staggered rollout vs fixed holdout | Done | [causal-inference-playbook#11](https://github.com/wavde/causal-inference-playbook/pull/11), [results](https://github.com/wavde/causal-inference-playbook/blob/main/case-studies/08-rollout-vs-holdout/results.md) | [A staggered rollout can't see novelty](writing/staggered-rollout-vs-holdout.md) |
| Judge drift mid-test | Done | [experiment-toolkit#6](https://github.com/wavde/experiment-toolkit/pull/6), [results](https://github.com/wavde/experiment-toolkit/blob/main/docs/judge_drift_results.md) | [Upgrading the judge mid-test is safe only if your labels know about it](writing/judge-drift-mid-test.md) |
| Propensity vs uplift on a second randomized dataset (Hillstrom) | Done | [paid-media-playbook#5](https://github.com/wavde/paid-media-playbook/pull/5), [results](https://github.com/wavde/paid-media-playbook/blob/main/case-studies/08-hillstrom-email-uplift/results.md) | [It didn't replicate](writing/uplift-targeting-replication-hillstrom.md) |
| Judge drift without a version boundary | Done | [experiment-toolkit#8](https://github.com/wavde/experiment-toolkit/pull/8), [results](https://github.com/wavde/experiment-toolkit/blob/main/docs/silent_drift_results.md) | [When the judge drifts silently](writing/silent-judge-drift.md) |
| Five classic case studies, re-read (off-plan) | Done | [experiment-toolkit#10](https://github.com/wavde/experiment-toolkit/pull/10), [results](https://github.com/wavde/experiment-toolkit/blob/main/docs/bandit_eval_results.md) | [Five classic data science case studies, re-read in 2026](writing/classic-case-studies-reread-2026.md) |
| Canary-set drift monitor | Done | [experiment-toolkit#11](https://github.com/wavde/experiment-toolkit/pull/11), [results](https://github.com/wavde/experiment-toolkit/blob/main/docs/canary_drift_results.md) | [A canary set catches a drifting judge](writing/canary-set-judge-drift.md) |
| Real-data judge audit (MT-Bench) | Done | [experiment-toolkit#14](https://github.com/wavde/experiment-toolkit/pull/14), [results](https://github.com/wavde/experiment-toolkit/blob/main/docs/judge_audit_results.md) | [A real LLM judge favours the winner](writing/real-judge-audit-mt-bench.md) |

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

### Retro: Offline evals vs online A/B

- **Held up:** each offline correction removed the failure it targets, and the simulated means matched their closed-form values. Under mix shift, the raw offline judge shipped a change that hurts live traffic in 100% of replications and the reweighted readouts in 0–1%. Under a lenient judge, PPI++ cut false ships from 53% (reweighted) to 3%. The new CI smoke step (from the plan review) ran both evidence scripts on the first PR.
- **Didn't:** the corrections are expensive and still can't see online-only costs. On a real win, shipping fell from 87% (raw) to 49% (reweighted) to 25% (PPI++). PPI++ power stayed at 21–23% from 5,000 to 20,000 prompts because 1,000 labels cap it. A latency cost was never stopped offline. Segments, traffic shares and the quality-to-online slope are all assumed known.
- **Next question:** theme 2 of the review, a public-data check. On the Criteo uplift dataset, does ranking users by conversion propensity rather than uplift waste spend the way loop 4's simulation says?

### Retro: Propensity vs uplift targeting on real randomized data

- **Held up:** the first loop on public data, with a randomized baseline. It confirmed loop 4's two targeting warnings. Credited / incremental visits rose from 4.71x (everyone) to 8.58x (top 5% by propensity). The marginal return from 10% to 40% reach was 0.15 of the average for visits and 0.04 for conversions. CI now smoke-runs the pipeline on synthetic data with the real schema.
- **Didn't:** loop 4's simulation assumed high-intent users respond slightly less to ads. On Criteo, absolute uplift rises steeply with propensity: the top 20% by propensity hold 72.2% of incremental visits. So the uplift model won only at tight reach on visits (45.7% vs 28.9% of incremental at 5%) and not on rare conversions (53.1% vs 59.1%). This is one advertiser with anonymized features and simple models, and the SEs exclude model-fitting noise.
- **Next question:** theme 3, cheaper corrections. Loops 1, 4 and 5 all ended with correct but expensive answers. How far can sequential monitoring and variance reduction cut the cost?

### Retro: PPI++ with a pre-treatment covariate

- **Held up:** the closed-form label budgets reproduced loop 1's PPI++ numbers exactly (982 / 2,397 / 3,610) and matched simulation. At judge accuracy 0.85, a covariate with R² 0.25 cut labels for 80% power from 2,397 to 2,072, and one with R² 0.50 cut them to 1,625. The CI smoke step caught nothing because nothing broke, which is the point.
- **Didn't:** the gain is smaller than the covariate's own R² suggests. It overlaps with the judge (combined R² 0.557, not 0.48 + 0.25), so it saved only 6% with a 0.95 judge and 5% when labels were 71% of a 5,000-unit arm. Improving the judge from 0.85 to 0.95 saved more than any covariate in the grid. The model is binary, with one covariate and judge errors that depend only on quality. One validation row showed 0.063 Type-I on the committed seed, and human-only showed 0.056 on the same draws.
- **Next question:** the plan is due for review. Candidates: the open staggered-rollout question from theme 3, a covariate that predicts where the judge is wrong, and CI smoke runs for the causal-inference-playbook and demand-forecasting evidence.

### Retro: Staggered rollout vs fixed holdout

- **Held up:** it answered the question loop 3 left open, using loop 3's own adoption model. All three designs withheld 10% of user-weeks, and every estimate was centred on its truth with 93.6–95.9% coverage. The staggered rollout had the most precise short-run read (SE 0.0175 vs 0.0285), but it shipped a fading novelty effect in 100.0% of replications, against 4.7% for the fixed holdout. causal-inference-playbook now smoke-runs cases 07 and 08 in CI, which closes that repo's gap from the review.
- **Didn't:** the hybrid design wasn't the compromise I expected. At equal cost, its 5% holdout shipped a flat effect only 63.1% of the time, against 90.9% for the fixed holdout. The model has one effect shape at a time, adoption at access, no interference, and an 8-week "long run". demand-forecasting still has no CI smoke run.
- **Next question:** theme 2. What happens to a judge-scored A/B test when the judge model is upgraded mid-test?

### Retro: Judge drift mid-test

- **Held up:** it removed the fixed-judge assumption every LLM-judge retro had flagged, using loop 1's PPI++ unchanged. Closed forms matched the simulation, including the large-sample bias of front-loaded labels (+0.0141 simulated against +0.0139 expected). An upgrade that shifts both arms the same way left every analysis's Type-I between 0.048 and 0.051. When the new judge was more lenient on treatment, PPI++ with labels spread across the test (0.042) or stratified by judge version (0.050) stayed valid.
- **Didn't:** front-loaded labeling, a common practice, keeps about 70% of the naive bias. Its Type-I error rises from 0.064 to 0.392 as labels go from 250 to 4,000 per arm. I expected partial calibration to help more. The model has one upgrade at a known time, binary quality, and fixed per-arm leniency. Silent drift with no version boundary isn't covered.
- **Next question:** theme 3. Does loop 6's finding (uplift concentrates in high-propensity users) hold on a second public randomized dataset, with a different advertiser and channel?

### Retro: Propensity vs uplift on a second randomized dataset (Hillstrom)

- **Held up:** the replication worked as a check. It reused loop 6's models and readouts unchanged (the helpers are now shared, and case 07's results are byte-identical). Two of loop 6's findings replicated in direction: credited visits overstate incremental visits (2.74x when e-mailing everyone), and under propensity targeting the marginal return is below the average (0.67). Cross-fitting and a bootstrap with explicit "not estimable" ratios kept a small dataset honest.
- **Didn't:** loop 6's headline didn't replicate. The top 20% by propensity reached 25.9% of incremental visits here, against 72.2% on Criteo, so "propensity targeting already finds the persuadable users" is channel-specific. Loop 6's decision rule needs revising to "test it on your own holdout". The dataset is small (about 212 incremental conversions, so conversion results are mostly noise), there's no license so it's downloaded at runtime, and two datasets can show the finding isn't general but not why.
- **Next question:** round 3 is complete, and the plan is due for its third review before another loop starts. Open items for that review: revise loop 6's essay rule in light of this result, add a CI smoke run in demand-forecasting, and cover judge drift without a version boundary.

### Retro: Judge drift without a version boundary

- **Held up:** it closed the gap loop 9 left, using loop 9's module and PPI++ unchanged. Labels drawn at random from the whole test kept PPI++ valid under every drift tested (Type-I 0.048 to 0.053, coverage 0.944 to 0.956) without knowing when or whether the judge changed. The closed forms matched the simulation, including launch-week labels' bias (+0.0139 against +0.0139). The cheap monitors behaved as designed: a both-arm score monitor caught a step drift 0.847 of the time, a median of 6 days in, and agreement split by the human label ignored a real quality drop (0.029).
- **Didn't:** no monitor was both cheap and specific. Watching control alone missed treatment-only drift (0.021). Both score monitors fired on a real quality drop with the judge unchanged (0.993 and 1.000). The specific monitor needs thousands of labels per arm (0.089 at 500, 0.788 at 4,000). A gradual drift was caught only 0.365 of the time. The model has one drift, binary quality and confusion-matrix judges, and a canary set of stored outputs re-judged daily, likely the cheapest specific monitor, wasn't simulated.
- **Next question:** round 4 is complete, and the plan is due for its fourth review before another loop starts. Candidates for that review include a canary-set drift monitor, and whether the portfolio now needs fewer new loops and more consolidation.

### Retro: Five classic case studies, re-read (off-plan)

- **Held up:** one rule covered all five themes, the same one the plan reviews keep finding: randomize at the decision being made, and quote numbers only from that randomization. The new bandit simulation gave the artwork theme a known-answer check. With 20 variants all equal to the control and ±20% drift, the leader's raw lift claimed a false win in 100.0% of replications, a 20% evaluation slice the bandit never learned from in 1.7–2.0%, and a confirmation A/B in 1.6%. Two original links didn't match their labels and were replaced or dropped rather than cited.
- **Didn't:** same-log off-policy estimators were worse than I expected. They fixed the drift (bias +12.6% to +3.8%) but not the winner's curse, and without drift DR was more biased than the raw comparison (+4.6% against +3.5%). The slice's SE was 6.4% against 2.8% for a confirmation A/B of similar size, and coverage fell to 0.910–0.930 with a thin or no propensity floor. The pricing and leadership sections reuse evidence built for other questions, the bandit is not contextual, and the essay's encouragement-design recommendation has no evidence of my own behind it.
- **Next question:** this doesn't change round 5. It is consistent with [Which number supports which decision](writing/which-number-supports-which-decision.md), and the bandit result can join that map as a line ("pick and quote a creative a bandit chose") once its evidence is merged. Candidates this essay adds: an encouragement-design simulation (how weak can the first stage get at realistic uptake before IV is useless?) and off-policy evaluation of a contextual policy chosen on the same log.

### Retro: Canary-set drift monitor

- **Held up:** it passed the fourth review's gate by sharpening a synthesis line, and reused loop 11's scenarios and monitors unchanged, so the comparison is like for like. A canary of 400 stored outputs per arm, with no human labels and fully independent re-judging, caught treatment-only drift 0.932 of the time against 0.838 for the both-arm score monitor, and stayed at 0.021 when true quality fell, where the score monitor fired 1.000. The closed-form canary size (420 per arm for 80% on one drifted week) matched the simulated detection rates.
- **Didn't:** it wasn't faster. The median delay was 6 days for a step and 12 for gradual drift, the same as the score monitor, because the 7-day window sets the delay. A canary of control outputs only, the easy way to build one, missed treatment-only drift (0.020). The specificity holds by construction in the simulation; it hasn't been checked against a real hosted judge, whose day-to-day verdict noise is an assumption (repeatability 0 to 0.9).
- **Next question:** round 5 is complete, and the plan is due for its fifth review before another loop starts. Candidates: checking any LLM-judge result against a real judge, since every judge loop so far is simulated, and whether the portfolio needs another loop at all.

### Retro: Real-data judge audit (MT-Bench)

- **Held up:** the first real-data check of the LLM-judge loops, with no API calls: public paired verdicts, downloaded at runtime and checksummed. The shared assumption replicated. GPT-4's win rate for the stronger model was above the experts' in 15 of 15 pairs (CI excludes zero in 9), and its agreement with experts was 0.823 when they preferred the stronger model against 0.221 when they preferred the weaker one. PPI++ with random human labels kept 0.930 to 0.956 coverage on the real verdicts, and the closed-form variance ratio matched the simulation.
- **Didn't:** the simulated judges were too good. The real judge correlated with expert verdicts at 0.14 to 0.56 against 0.69 assumed, so PPI++ saved 2–28% of labels where loop 1's judge implies 43%, and every simulated label budget is optimistic. The data are 2023 models judged by a 2023 GPT-4, pairwise rather than per-output, and the resampling check treats the judged items as the population.
- **Next question:** round 6 priority 2: add the bandit result to the synthesis (done; see "Round 6 status" above). Priority 3 was cut because no suitable public dataset exists, so round 6 is complete and the sixth review is due before any further loop.
