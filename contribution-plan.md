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
