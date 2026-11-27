# Opt-in AI features: when an observational impact estimate is usable

*~8 min read. Companion code is [case 07 of the causal-inference-playbook](https://github.com/wavde/causal-inference-playbook/tree/main/case-studies/07-ai-feature-adoption): the [simulation](https://github.com/wavde/causal-inference-playbook/blob/main/case-studies/07-ai-feature-adoption/src/adoption_simulate.py), the [one-command runner](https://github.com/wavde/causal-inference-playbook/blob/main/case-studies/07-ai-feature-adoption/src/run.py), [known-answer tests](https://github.com/wavde/causal-inference-playbook/blob/main/tests/test_adoption.py), and the [generated results](https://github.com/wavde/causal-inference-playbook/blob/main/case-studies/07-ai-feature-adoption/results.md).*

## The recommendation

For an opt-in AI feature, the default way to measure impact is a randomized holdout or a randomized staggered rollout. An observational estimate is a screening number unless four things hold: it uses pre-launch outcomes (prefer difference-in-differences), that estimator's own fake-launch placebo passes, you can argue that users don't opt in because their need just went up, and the E-value is compared against an explicit bound on a post-launch confounder.

The third condition is the one that fails for AI tools, and no diagnostic catches it. In a simulation where users adopt when a writing-heavy week arrives and the true effect is **zero**, the placebo-gated difference-in-differences estimate passed its placebo and excluded zero in **97.2%** of replications. It was confidently wrong almost every time.

## The decision

A team launches an opt-in AI writing assistant to everyone. A month later, adopters look far more engaged than non-adopters. Leadership wants a number for the feature's impact to use in roadmap and headcount decisions. Does the team report an adjusted observational estimate, or pay for a holdout?

## The setup

The simulation knows the answer by construction. Each replication draws 4,000 users with weekly sessions in three periods: two before launch and one after. Adoption depends on tenure and prior activity, which are logged, plus one unobserved channel that varies by scenario:

| Scenario | What drives adoption beyond logged data | Visible before launch? |
|---|---|---|
| `observables` | Nothing | n/a |
| `fixed_trait` | A stable "power user" trait | Yes, as a level |
| `pre_trend` | A user-specific growth trend already under way | Yes, as a slope |
| `post_shock` | A rise in need that starts only after launch | No |

Two more scenarios repeat `pre_trend` and `post_shock` with a true effect of zero. Four estimators are compared against the true effect on adopters (the ATT): the raw adopter-minus-non-adopter gap, AIPW on the logged covariates, AIPW that also adjusts for both pre-launch outcomes, and difference-in-differences from the last pre-launch period to the post-launch period. Each one gets a **fake-launch placebo**, the same estimator run one period earlier where the true effect is zero, and an E-value (VanderWeele & Ding 2017).

One command reproduces everything with seed 0 and 1,000 replications per scenario: `python case-studies/07-ai-feature-adoption/src/run.py`. At 1,000 replications, a 5% or 95% rate has a Monte Carlo standard error of about 0.7 points.

![Opt-in AI adoption simulation](https://raw.githubusercontent.com/wavde/causal-inference-playbook/main/case-studies/07-ai-feature-adoption/docs/hero.png)

## The raw gap is never usable

The adopter gap overstated the effect by +1.898 to +3.129 sessions per week across the four scenarios with a real effect, against true effects of 1.243 to 1.283. The bias is bigger than the effect. Its 95% interval covered the truth in 0.0% of replications in every scenario.

There is a second, quieter overstatement. Users who gain most from the feature are also the ones most likely to adopt it, so even an unbiased estimate of the effect on adopters (1.283 in `observables`) overstates what turning it on for everyone would deliver (1.000). That gap is real, not bias, but it matters if the number feeds a forecast for a default-on launch.

## What pre-launch outcomes buy you

**With selection on observables only,** both AIPW and DiD work: bias within ±0.001, coverage 95.1% to 96.4%.

**With a stable unobserved trait,** adding pre-launch outcomes to AIPW is not enough. Noisy pre-period outcomes only partly proxy the trait, and AIPW with both of them was still biased by +0.628. DiD, which differences the trait out, was biased by −0.001 with 94.8% coverage. The placebo told them apart: it flagged AIPW-with-pre-periods in 100.0% of replications and DiD in 4.7%.

**With a pre-existing growth trend,** DiD is biased too (+0.360, coverage 4.0%), but its placebo flagged in 95.3% of replications. A gate that acts only when the placebo passes and the interval excludes zero acted on DiD in just 4.7% of replications, even though DiD rejected zero in 100.0%. With a true effect of zero and the same trend, the gate acted in 3.9%.

So far the placebo is doing its job. Pre-launch outcomes plus a placebo catch selection that leaves a footprint before launch.

## The case the placebo can't see

In `post_shock`, users opt in when their need rises, for example when a writing-heavy project starts. Nothing about it shows up before launch, so the placebo has nothing to detect. Every adjusted estimator was biased by +0.724 to +0.844 against a true effect of 1.243, with 0.0% coverage, while placebo flag rates stayed at 6.2% to 6.5%, close to nominal.

The gated rule made this worse, because it lent the wrong answer a passed check:

| Scenario | Gate acts on DiD | …with a CI that misses the truth |
|---|---:|---:|
| `observables` | 95.6% | 3.0% |
| `fixed_trait` | 95.3% | 4.1% |
| `pre_trend` | 4.7% | 4.7% |
| `post_shock` | 93.5% | 93.5% |
| `null_pre_trend` | 3.9% | 3.9% |
| `null_post_shock` | 97.2% | 97.2% |

With no real effect at all, DiD reported +0.726 sessions per week, passed its placebo, and excluded zero in 97.2% of replications. AIPW, with or without pre-period outcomes, did the same in 95.3% to 95.5%.

For an AI writing assistant, this is the mechanism to expect, not an edge case. Wanting help with writing is the reason to opt in.

## The E-value isn't a pass/fail check

It's tempting to read a large E-value as robustness. In this simulation biased estimates had *larger* E-values, because the bias inflates the estimate. In `observables` the raw gap had a median E-value of 5.11, against 2.42 for the unbiased DiD. The spurious DiD estimate in `null_post_shock` still had a median E-value of 1.83 (1.65 for the interval). An E-value only means something when you compare it against a stated bound on how strongly an unmeasured post-launch driver could affect both adoption and engagement. ([E-values: a one-number sensitivity check](e-values-for-observational-memos.md) covers the mechanics.)

## Assumptions and the limits of synthetic data

- **Linear trends.** In `pre_trend`, the pre-launch slope predicts the post-launch slope. A trend that bends at launch behaves like `post_shock`, and the placebo won't see it.
- **Two pre-launch periods.** One placebo is a weak check. More periods would allow an event-study test of trends, but that still can't see a post-launch shock.
- **Fixed selection strengths.** The direction of each result follows from the mechanism. The bias sizes and flag rates don't transfer to other selection strengths or sample sizes.
- **Correctly specified models.** The outcome model is linear and the propensity model logistic, as in the simulation. Real data adds misspecification on top.
- **One unobserved channel at a time.** Real adoption likely mixes all three, so a passing placebo rules out only the channels it can see.
- **AIPW standard errors treat the fitted models as fixed.** They were calibrated here (mean SE 0.085 against an empirical SD of 0.086 in `observables`), but cross-fitting would be safer with flexible models.
- **Synthetic users.** Nothing here is measured on a real product.

## When the conclusion breaks

- **Adoption isn't driven by need.** If opt-in is driven by something stable, such as tenure, plan or a fixed preference, the evidence supports DiD with a passing placebo: bias −0.001 and 94.8% coverage under a stable trait.
- **The trend bends at launch.** Then even a passing placebo on a trend scenario is no protection. That is the `post_shock` problem under another name.
- **You need the all-users effect.** An unbiased adopter estimate (1.283 in `observables`) still overstates a default-on launch (1.000). A holdout of the opt-in feature measures the effect on people who choose it. If the decision is a default-on launch, the experiment has to test default-on exposure.

## What would change my mind

- A product where adoption timing is set externally and logged: an admin rolls the feature out by team, or a random invite wave. Then adoption isn't timed to need, condition 3 holds, and the observational estimate becomes usable as a decision number.
- A real holdout that matched the placebo-passing DiD estimate across several AI launches. That would suggest the `post_shock` channel is weak in practice.
- A logged proxy for rising need, such as document counts or calendar load, that, once adjusted for, makes the `post_shock` bias disappear in a version of this simulation.

## The decision rule

An observational estimate of an opt-in AI feature's impact is usable only if all four hold:

1. **It uses pre-launch outcomes.** Prefer DiD or another estimator that differences out stable traits. Never report the raw adopter gap.
2. **That estimator's own fake-launch placebo passes.** If it flags, selection the placebo can see is biasing the estimate.
3. **You can argue that adoption isn't timed to a change in need.** The placebo can't test this. If users plausibly opt in when their workload rises, it fails.
4. **The E-value is compared against an explicit bound** on how strongly a post-launch driver could act on both adoption and engagement. A "large" E-value on its own is not a pass.

Otherwise, run a randomized holdout or a randomized staggered rollout, and treat the observational number as a screen. For a writing assistant, condition 3 is the weak link, so the holdout is the default.

---

*Code, results, and references: [case 07](https://github.com/wavde/causal-inference-playbook/tree/main/case-studies/07-ai-feature-adoption) · [`adoption_simulate.py`](https://github.com/wavde/causal-inference-playbook/blob/main/case-studies/07-ai-feature-adoption/src/adoption_simulate.py) · [`run.py`](https://github.com/wavde/causal-inference-playbook/blob/main/case-studies/07-ai-feature-adoption/src/run.py) · [`test_adoption.py`](https://github.com/wavde/causal-inference-playbook/blob/main/tests/test_adoption.py) · [`results.md`](https://github.com/wavde/causal-inference-playbook/blob/main/case-studies/07-ai-feature-adoption/results.md) · VanderWeele & Ding (2017), *Introducing the E-value* · Ashenfelter (1978) on pre-program dips · Angrist & Pischke (2009), *Mostly Harmless Econometrics*, §5.4 · Imbens & Wooldridge (2009), *Recent Developments in the Econometrics of Program Evaluation*.*
