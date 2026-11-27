# Measuring marketing when the algorithm runs the campaign

*~8 min read. Companion code is [case 06 of the paid-media-playbook](https://github.com/wavde/paid-media-playbook/tree/main/case-studies/06-ai-mediated-measurement): the [simulator](https://github.com/wavde/paid-media-playbook/blob/main/case-studies/06-ai-mediated-measurement/src/ai_measurement.py), the [one-command runner](https://github.com/wavde/paid-media-playbook/blob/main/case-studies/06-ai-mediated-measurement/src/run.py), [known-answer tests](https://github.com/wavde/paid-media-playbook/blob/main/tests/test_ai_measurement.py), and the [generated results](https://github.com/wavde/paid-media-playbook/blob/main/case-studies/06-ai-mediated-measurement/results.md).*

## The recommendation

When the ad platform's model chooses the audience, the creative mix and the pacing, match each decision to the one tool that can answer it:

- **Platform-reported numbers** are for within-policy pacing and day-to-day tuning only. They aren't incrementality.
- **Budget changes** need a holdout at both budget levels: a two-cell budget-split lift test. A single lift test prices the budget it ran at, not the next dollar.
- **Creative decisions** need a test that fixes the audience first and then randomizes the creative. The platform's own A/B test compares audiences.
- **MMM with algorithm-paced spend**: when the MMM coefficient conflicts with a lift test, replace it with the lift result rather than averaging the two. In the simulation, using the lift test as a prior still left the ROI overstated by 48% at a spend–demand correlation of 0.6.

## The decision

Automated campaign types (Advantage+, Performance Max and their peers) pick who sees the ad, which creative they see and how fast the budget is spent. A team still has to decide whether the campaign works, whether to raise its budget, which creative to keep, and how much to trust the MMM. Every number it looks at is now produced by the same optimizer it is trying to evaluate. Which number can support which decision?

## The setup

All data is synthetic, and the truth is known by construction. Each simulated user has a conversion probability with and without the ad, so true incremental conversions are exact. Four mechanisms are switched on one at a time:

1. **Targeting.** The platform serves the top 20% of 400,000 users by a score whose correlation with baseline conversion intent is the *targeting sharpness*. A 50% ghost-ads holdout runs among the users it would have served (Johnson, Lewis & Nubbemeyer 2017).
2. **Budget transport.** The same users at 10% and 40% reach. The cost per served user is set so that, at sharpness 0.8, scaling up is profitable on average and unprofitable at the margin.
3. **Creative delivery.** Creatives A and B have identical per-person effects, but the platform delivers A to high-baseline, low-lift users and B to low-baseline, high-lift users.
4. **MMM pacing.** 104 weeks of sales with latent demand the MMM can't see, and log spend correlated with that demand. The MMM gets the exact adstock and saturation transforms, which is its best case.

One command reproduces it: `pip install -r requirements.txt && python case-studies/06-ai-mediated-measurement/src/run.py`, with seed 20261127. Two runs produce byte-identical output.

## Platform credit isn't incrementality, and sharper targeting makes it worse

The platform finds people who would have converted anyway. The better it gets at that, the more conversions it credits to itself:

| Targeting sharpness | Platform-credited / true incremental | Lift-test bias | Lift-test CI coverage |
|---:|---:|---:|---:|
| 0.00 | 2.50x | -0.7% | 96.0% |
| 0.50 | 3.03x | -0.5% | 95.0% |
| 0.80 | 3.31x | -0.2% | 92.0% |
| 0.95 | 3.43x | +0.1% | 94.5% |

Even with random targeting, credited conversions are 2.5 times the true incremental count, because most served users would have converted anyway. At sharpness 0.95 the overstatement reaches 3.43x. The ghost-ads holdout stays unbiased throughout. As the platform's targeting improves, its own report becomes less trustworthy, while a holdout's accuracy doesn't change.

## A lift test prices the budget it ran at

Sharp targeting serves the best prospects first, so each extra dollar reaches people who respond less. At sharpness 0.8, the true average iROAS at 10% reach is 1.10, but the marginal iROAS from 10% to 40% reach is 0.90, below break-even. Under random targeting the two are equal (marginal/average 1.00). For sharpness 0.5 and above, the marginal return is 0.81–0.85 of the average.

So the question "should we scale reach from 10% to 40%?" has a true answer of no. How often did each kind of evidence say yes?

| Evidence used | Says scale up (truth: don't) |
|---|---:|
| Platform-credited ROAS at 10% reach | 100% |
| Platform-credited marginal ROAS (two-cell) | 100% |
| Lift test at 10% reach (average iROAS) | 94% |
| Lift test at 40% reach (average iROAS) | 1% |
| Two-cell budget-split lift test (marginal iROAS) | 0% |

The single lift test at current budget is unbiased for the question it answers, which is whether the campaign as it runs today is incremental. It still says scale up 94% of the time, because that is a different question. The two-cell split runs both reach levels on random halves of users, each with its own holdout. It estimated the marginal iROAS at 0.90 against a true 0.90, with 98% CI coverage, and never recommended scaling.

## The platform's creative test compares audiences

With identical per-person creative effects, the true creative difference is zero. The platform delivers A to high-baseline users and B to high-lift users, which Braun & Schwartz (2023) call divergent delivery:

| Design | A minus B: estimate (truth) | Picks A | Picks B | No winner |
|---|---:|---:|---:|---:|
| In-platform A/B, served conversion rate | +0.0551 (+0.0000) | 100% | 0% | 0% |
| In-platform A/B, per-arm ghost-ads lift | -0.0233 (-0.0232) | 0% | 100% | 0% |
| Audience fixed first, creative randomized | +0.0000 (+0.0000) | 4% | 4% | 93% |

The served conversion rate picks A every time, which is wrong. Per-arm holdouts are correct, but they answer a different question: campaign B *as delivered* is more incremental, because the platform sent it to the people who respond. Only fixing the audience first answers which creative is better. That test finds no winner 93% of the time, which is the right answer.

## MMM can't separate spend from the demand that drives it

When the platform raises spend as it sees intent rise, spend and latent demand move together, and the MMM credits media with demand. The true ROI is about 0.71:

| Corr(spend, demand) | MMM alone | MMM with lift test as prior | Lift-test coefficient | MMM vs lift conflict flagged |
|---:|---:|---:|---:|---:|
| 0.0 | 0.72 (+1%) | 0.71 (+0%) | 0.70 (-1%) | 6% |
| 0.3 | 1.11 (+56%) | 0.87 (+22%) | 0.71 (+0%) | 50% |
| 0.6 | 1.40 (+98%) | 1.05 (+48%) | 0.71 (+0%) | 94% |
| 0.9 | 1.59 (+125%) | 1.28 (+81%) | 0.72 (+1%) | 100% |

At a correlation of 0.6, the MMM says ROI exceeds 1 in 100% of replications when it doesn't. Using the lift test as a precision-weighted prior helps, but the result is still +48% biased and says ROI > 1 in 63% of replications. The MMM's estimate is precise, so it pulls the average toward its biased value. Replacing the coefficient with the lift-test estimate gives 0.71 with 95% coverage. A simple conflict test between the MMM and the lift result flagged 94% of those panels, so the conflict itself is the signal to replace rather than blend. (For what each tool answers in general, see [MMM vs incrementality](mmm-vs-incrementality.md).)

## Assumptions and the limits of synthetic data

- **One mechanism at a time.** Real automated campaigns combine all four, and the platform's model drifts during a test.
- **Chosen effect sizes.** The break-even cost in the budget test was placed between average and marginal uplift so the decision flips. With a different cost, the gap may not change the call. The size of the gap (marginal/average 0.81–0.85 at sharpness 0.5 and above) is the part that transfers.
- **Clean ghost-ads holdouts.** The platform must log who it *would* have served, and holdout and treated users must not interfere. Many platforms offer only coarser holdouts.
- **Scale.** The two-cell result uses 2,000,000 users per test. Smaller tests will separate average from marginal less reliably.
- **A best-case MMM.** It's OLS with the true transforms. A Bayesian MMM that learns its transforms adds its own error. Calibration acts on one coefficient, and the lift window is assumed to measure the same effect the MMM estimates.
- **Point decisions.** Wrong-decision rates compare point estimates with break-even. Requiring the interval to clear break-even gives fewer wrong calls and more "no decision" calls.

## When the recommendation breaks

- **Targeting isn't sharp.** Under random targeting, average and marginal iROAS match (0.71 and 0.71), so a single lift test does price a budget change.
- **Spend isn't paced with demand.** At a spend–demand correlation of 0, the MMM is close to unbiased (+1%), so its coefficient needn't be replaced. The recommendation applies only when pacing is algorithmic.
- **The lift test measures a different effect than the MMM.** If the effect changes over time or the lift window isn't representative, replacing the coefficient imports a different bias.
- **The question is campaign-level.** If the decision is "keep campaign A or campaign B as the platform delivers them", per-arm holdouts are the right tool, and the audience-fixed test answers a question nobody asked.

## What would change my mind

- Field evidence that platform-reported conversions track holdout-measured incrementality within a stable ratio for a given account. Platform numbers could then be scaled rather than ignored. The simulation says the ratio moves with targeting sharpness, so it would need to be re-measured whenever targeting changes.
- An MMM that observes a good demand proxy, such as search interest or organic traffic, and loses its bias when spend is paced with demand.
- Two-cell tests at realistic advertiser scale that can't separate marginal from average return. Then the budget rule would need a different design, such as geo-level budget splits.

## The decision rule

1. **Platform-reported numbers** are for within-policy pacing only.
2. **Budget changes** need a two-cell budget-split holdout, measured at both budget levels.
3. **Creative decisions** need audience-randomized tests: fix the audience first, then randomize creative.
4. **When spend is algorithm-paced,** an MMM coefficient that conflicts with a lift test should be replaced by the lift result, not averaged with it.

---

*Code, results, and references: [case 06](https://github.com/wavde/paid-media-playbook/tree/main/case-studies/06-ai-mediated-measurement) · [`ai_measurement.py`](https://github.com/wavde/paid-media-playbook/blob/main/case-studies/06-ai-mediated-measurement/src/ai_measurement.py) · [`run.py`](https://github.com/wavde/paid-media-playbook/blob/main/case-studies/06-ai-mediated-measurement/src/run.py) · [`test_ai_measurement.py`](https://github.com/wavde/paid-media-playbook/blob/main/tests/test_ai_measurement.py) · [`results.md`](https://github.com/wavde/paid-media-playbook/blob/main/case-studies/06-ai-mediated-measurement/results.md) · Braun & Schwartz (2023), *Where A/B testing goes wrong* · Johnson, Lewis & Nubbemeyer (2017), *Ghost ads* · Gordon, Zettelmeyer, Bhargava & Chapsky (2019), *A comparison of approaches to advertising measurement* · Jin, Wang, Sun, Chan & Koehler (2017), *Bayesian methods for media mix modeling*.*
