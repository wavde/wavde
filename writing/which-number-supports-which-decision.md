# Which number supports which decision: measuring AI features

*~8 min read. A synthesis of the eleven loops in the [contribution plan](../contribution-plan.md). It adds no new evidence: every number below appears in the linked essay, and each essay links to the code that reproduces it with one command and a fixed seed.*

## The recommendation

When someone shows you a number about an AI feature, ask two questions before acting on it: **what was randomized, and who labeled the outcome?** A number supports a decision only if the randomization sits at the decision being made and the outcome is anchored to human judgment by a random sample. Otherwise it's a screen: useful for deciding what to test, not for deciding what to ship.

Across eleven loops, the numbers teams are usually shown failed that test in the same ways:

- An LLM judge's score difference, when the treatment changes what the judge sees.
- An offline eval win, read without reweighting to live traffic.
- The gap between users who opted in to an AI feature and those who didn't.
- The first weeks of a staggered rollout, for a feature whose effect fades.
- Platform-credited conversions, and the average lift at the current budget.
- A model card, instead of a backtest on your own series.

Each one was wrong in at least one check where the answer was known by construction or came from a randomized experiment, and several got *more* confident as data grew.

## The decision

A team measuring an AI feature is rarely short of numbers. It's short of a way to say which number can carry which decision. This piece lines up the decision rules from the individual loops so that question has one answer, and says where the evidence for each rule is strong and where it's thin.

## What this draws on

Eight of the eleven loops are simulations where the true effect is known by construction, so every estimate can be checked against the truth. Two use public randomized experiments: the Criteo uplift dataset and the Hillstrom e-mail experiment. One is a rolling-origin backtest on the public UCI Online Retail II data. A reduced-size run of every evidence pipeline runs in CI.

That makes the evidence good at one thing: showing that a method fails, or works, under a mechanism I chose. It's weaker at saying how often that mechanism occurs in practice. "Assumptions and limitations" below comes back to this.

## The map

| Decision | Number you'll often be shown | What went wrong with it | Number that supports the decision | Essay |
|---|---|---|---|---|
| Ship on a judge-scored metric | Judge-score difference | Judge 5 points more lenient on treatment, no true effect: rejected 15% of the time at 1,000 units per arm, 49% at 5,000, 99% at 20,000 | PPI++ with human labels drawn at random within each arm: 3.6% to 5.8% | [LLM-as-judge is a noisy metric](llm-judge-noise-in-ab-tests.md) |
| Keep reading a test after a judge upgrade | Judge scores pooled across versions; labels from before the upgrade | Pooled: rejected a true null 82.6% of the time. Pre-upgrade labels: Type-I rose from 0.064 to 0.392 as labels went from 250 to 4,000 | PPI++ with labels spread across the test or stratified by judge version | [Upgrading the judge mid-test](judge-drift-mid-test.md) |
| Trust a test when the judge may change silently | Judge-only readout; launch-week labels | Treatment-only drift: Type-I 0.922 and 0.097 | PPI++ with labels drawn at random from the whole test: 0.048 to 0.053 under every drift tested | [When the judge drifts silently](silent-judge-drift.md) |
| Ship from an offline eval | Raw offline eval win | Shipped a change that hurts live traffic in 100% of replications; never stopped a latency cost | Reweighted offline readouts (0–1%) to choose what to test; an online A/B with guardrails to ship (stopped the latency cost in 96%) | [An offline eval win is evidence about the eval set](offline-evals-vs-online-ab.md) |
| Estimate the impact of an opt-in AI feature | Adopter vs non-adopter gap; DiD that passed its placebo | With adoption timed to a rise in need and a true effect of zero, placebo-gated DiD excluded zero in 97.2% of replications | A randomized holdout or randomized staggered rollout | [Opt-in AI features](opt-in-ai-adoption-observational-bias.md) |
| Launch on the long-run effect | Staggered rollout readout | Shipped a fading effect in 100.0% of replications | A fixed holdout for the whole window: 4.7% | [A staggered rollout can't see novelty](staggered-rollout-vs-holdout.md) |
| Change the budget of an algorithm-run campaign | Platform-credited ROAS; a lift test at the current budget | In simulation, with scaling up truly unprofitable, credited ROAS said scale up 100% of the time and a lift test at current budget 94%. On real data, credited visits were 4.71x incremental on Criteo and 2.74x on the e-mail experiment when targeting everyone | A two-cell budget split: estimated the marginal return at 0.90 against a true 0.90 and never recommended scaling. On real data the marginal return was 0.15 (Criteo) and 0.67 (e-mail) of the average, so measure it | [Measuring marketing when the algorithm runs the campaign](ai-mediated-marketing-measurement.md), [It didn't replicate](uplift-targeting-replication-hillstrom.md) |
| Pick a creative when the platform picks the audience | In-platform A/B on served conversion rate | With neither creative better, picked creative A in 100% of replications | Fix the audience, then randomize creative: no winner in 93% when there was none | [Measuring marketing when the algorithm runs the campaign](ai-mediated-marketing-measurement.md) |
| Read an MMM with algorithm-paced spend | MMM with a lift test as a prior | ROI still overstated by 48% at a spend–demand correlation of 0.6 | The lift test result, replacing the MMM coefficient | [Measuring marketing when the algorithm runs the campaign](ai-mediated-marketing-measurement.md) |
| Replace a forecaster with a pretrained model | Model card | With full history, Chronos-Bolt lost to SARIMA (UK MASE 0.784 vs 0.741) | A rolling-origin backtest on your own series; zero-shot won at 56 days of history | [Pretrained forecasters vs SARIMA](pretrained-forecasters-vs-sarima.md) |
| Choose between propensity and uplift targeting | "Propensity targeting already finds the persuadable users" | True on Criteo (top 20% held 72.2% of incremental visits), false on the e-mail experiment (25.9%) | A comparison on your own randomized holdout at the reach you buy | [It didn't replicate](uplift-targeting-replication-hillstrom.md), [revised Criteo essay](propensity-vs-uplift-targeting-criteo.md) |

## Three patterns

**Biased numbers get more confident with more data.** The naive judge metric's false-positive rate went from 15% to 99% as traffic grew. Front-loaded labels went from 0.064 to 0.392 Type-I as the label budget grew. Neither bias shrinks with data, but the standard error does. If a number measures the wrong thing, a bigger test makes it wrong with more certainty. "Run it longer" is not a fix for a design problem.

**The valid number is expensive, so the budget is the real decision.** With a noisy judge, power at 5,000 units and 500 labels per arm was 0.59 for the naive judge metric and 0.25 for PPI++. On a real offline win, the share of replications that shipped fell from 87% (raw) to 49% (reweighted) to 25% (PPI++). The two-cell budget test used 2,000,000 users per test. A staggered rollout gives a more precise short-run read than a fixed holdout (SE 0.0175 vs 0.0285) and is still the wrong design for a fading effect. The cheapest lever was usually upstream: improving the judge from 85% to 95% accuracy cut the label budget from 2,397 to 982 labels per arm, more than any covariate did.

**Some failures have no diagnostic, so the design has to carry them.** The fake-launch placebo can't see adoption timed to a rise in need (97.2% confidently wrong). Score monitors can't tell a drifting judge from a real quality drop: both fired in 0.993 and 1.000 of replications when quality fell and the judge didn't change. Offline evals can't see latency. In each case, no amount of analysis on the existing data fixes it; randomizing at the decision does.

## What replicated, and what didn't

Only one loop's conclusions were tested on a second dataset, and the result was mixed:

- **Replicated:** platform-credited results overstated incrementality on both datasets (4.71x and 2.74x when targeting everyone), and under propensity targeting the marginal return was below the average on both (0.15 and 0.67). The direction held; the size did not, so it has to be measured.
- **Didn't replicate:** that propensity targeting already concentrates the persuadable users. The top 20% by propensity held 72.2% of incremental visits on Criteo and 25.9% on the e-mail experiment. The Criteo essay's rule was revised to say so.

Everything else in the map rests on simulation. The simulations show what a method does under a stated mechanism, with the answer known, but they haven't been replicated on real data.

## Assumptions and limitations

- **Mechanisms are chosen, one at a time.** Each simulation switches on one failure (a lenient judge, mix shift, need-timed adoption, a fading effect, algorithm-paced spend) at a chosen size. Real systems combine them, at sizes I don't know.
- **Simple outcome models.** Quality is binary, judges are confusion matrices, effects are constant or follow one shape.
- **Two public randomized datasets,** both in marketing. None of the LLM-judge results has been checked against a real judge on real outputs.
- **The numbers are illustrations, not benchmarks.** "82.6%" is what happens under that loop's settings, not a rate to expect in production. The direction and the mechanism are what carry over.

## When the conclusion breaks

- **The treatment can't change what the judge sees,** such as a pure serving or latency change with identical outputs. Then the judge-only metric is valid, at roughly `1/J²` more sample (2.1x at 85% judge accuracy).
- **Randomization is impossible.** An observational estimate of an opt-in feature can be used if all four conditions in the opt-in essay hold, including an argument that adoption isn't timed to need.
- **Judge changes hit both arms equally.** Then a judge upgrade or drift is harmless under the null, though the estimate's scale changes.
- **Long history on a seasonal series.** Then SARIMA stays primary and the pretrained model is an extra baseline.

## What would change my mind

- **Real-judge audits showing that judge errors don't depend on which arm produced the output.** Most of the judge rules guard against arm-dependent error. If real judges rarely have it, the judge-only metric would be safe more often.
- **A replication of any simulation result on real data that comes out the other way,** as loop 10 did for loop 6.
- **A cheap diagnostic for a failure listed above as undiagnosable,** such as a canary set of stored outputs re-judged daily to separate judge drift from quality change. That would move a decision from "design it in" to "check it".

## The decision rule

1. **Ask what was randomized.** A number supports a decision only if randomization happened at that decision: users for a feature's impact, a fixed holdout for a long-run effect, budget levels for a budget change, creative within a fixed audience for a creative choice. Otherwise treat it as a screen.
2. **Ask who labeled the outcome.** A judge-only metric supports a ship decision only if the treatment can't change what the judge sees. Otherwise anchor it with human labels drawn at random within each arm and across the whole test, and analyse with PPI++.
3. **Ask whether more data would make it more confident even if it were wrong.** If yes, a bigger test won't help; change the design.
4. **Ask whether it has been replicated where you'll use it.** Carry a conclusion to a new channel, advertiser or product only after testing it on your own randomized holdout.
5. **Budget for the valid number up front.** Labels, holdouts and marginal tests are the cost of a defensible answer. Improving the judge is often cheaper than buying more labels.

---

*Sources, newest first: [When the judge drifts silently](silent-judge-drift.md) · [It didn't replicate](uplift-targeting-replication-hillstrom.md) · [Upgrading the judge mid-test](judge-drift-mid-test.md) · [A staggered rollout can't see novelty](staggered-rollout-vs-holdout.md) · [A covariate stacks on the judge](ppi-with-covariates-label-budget.md) · [The targeting model already knows who would convert](propensity-vs-uplift-targeting-criteo.md) · [An offline eval win is evidence about the eval set](offline-evals-vs-online-ab.md) · [Measuring marketing when the algorithm runs the campaign](ai-mediated-marketing-measurement.md) · [Opt-in AI features](opt-in-ai-adoption-observational-bias.md) · [Pretrained forecasters vs SARIMA](pretrained-forecasters-vs-sarima.md) · [LLM-as-judge is a noisy metric](llm-judge-noise-in-ab-tests.md). Code: [experiment-toolkit](https://github.com/wavde/experiment-toolkit), [causal-inference-playbook](https://github.com/wavde/causal-inference-playbook), [paid-media-playbook](https://github.com/wavde/paid-media-playbook), [demand-forecasting](https://github.com/wavde/demand-forecasting).*
