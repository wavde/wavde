# Five classic data science case studies, re-read in 2026

*~12 min read. New evidence for this essay is [`bandit_eval`](https://github.com/wavde/experiment-toolkit/blob/main/src/experiment_toolkit/bandit_eval.py) in experiment-toolkit, with [tests](https://github.com/wavde/experiment-toolkit/blob/main/tests/test_bandit_eval.py), a [one-command simulation](https://github.com/wavde/experiment-toolkit/blob/main/scripts/bandit_eval_sim.py), and the [generated results](https://github.com/wavde/experiment-toolkit/blob/main/docs/bandit_eval_results.md). The other sections reuse evidence from earlier essays, linked where it's used. [Which number supports which decision](which-number-supports-which-decision.md) maps the same rule across all of them; this essay applies it to five well-known write-ups.*

## The recommendation

Read these five write-ups for their methods. Most of the methods held up. What aged is an assumption they share: that the analyst decides who sees what, so the number on the dashboard comes from an assignment the analyst controls.

In 2026 something with its own objective usually sits between the treatment and the measurement. It might be a bandit choosing among generated creatives, a seller (or a tool acting for one) accepting or overriding a price, a pretrained forecaster nobody fitted to your data, an LLM judge producing the metric, or a user deciding whether to opt in. Each one makes what is shown depend on what is likely to happen next. The staff-level move is the same in all five cases: **decide where randomization survives, protect it from day one, and quote numbers only from it.** Applied to each theme:

1. **Artwork and creative.** Let the bandit choose the creative, but take the lift only from impressions it never learned from. In a simulation where 20 generated variants are all exactly as good as the control and click rates drift ±20% during the test, the leader's raw lift claimed a significant win in **100%** of replications. A 20% evaluation slice that the bandit never learned from claimed one in **1.7–2.0%**.
2. **Pricing.** Measure what the suggestion is worth, not how accurate the price model is. Randomize who is offered the suggestion, because sellers who accept it aren't a random sample, and randomize by market and time block when sellers compete for the same guests.
3. **Forecasting.** Keep the backtest as the arbiter. Bring in a pretrained model as an extra baseline and as the default for short history. On my backtest it lost to SARIMA with two years of history and beat it with 56 days.
4. **Leading.** The leverage has moved from running analyses to owning how AI features are measured: what the judge may score, when an offline eval is enough, and who labels. With a judge 5 points more lenient on the treatment arm, a test of a change with no effect came out significant **49%** of the time at 5,000 units per arm.
5. **Propensity.** First ask which propensity. A propensity to buy is a targeting score. A propensity score models who got treated. Neither replaces randomizing the encouragement when the treatment is something users choose.

## Where the five come from

Amney Mounir's [LinkedIn post](https://www.linkedin.com/posts/amney-mounir_when-i-worked-as-the-only-data-analyst-i-activity-7532095209111855105-tUSP) lists five case studies that helped when there was no senior analyst around to learn from. I followed every link. Two of them go somewhere other than their labels say:

- "Predicting a Listing's Price at Airbnb" links to a post on listing lifetime value. For pricing I use Airbnb's 2018 pricing post instead.
- "Using Propensity Score at Spotify" links to a third-party marketing podcast that happens to be hosted on Spotify. It isn't a Spotify write-up, so I don't cite it. For that theme I use a Spotify Engineering post that fits.

| Theme | Original | Published |
|---|---|---|
| Netflix artwork | [The Power of a Picture](https://about.netflix.com/en/news/the-power-of-a-picture); [Selecting the best artwork for videos through A/B testing](https://netflixtechblog.com/selecting-the-best-artwork-for-videos-through-a-b-testing-f6155c4595f6); [Artwork Personalization at Netflix](https://netflixtechblog.com/artwork-personalization-c589f074ad76) | May 2016; May 2016; Dec 2017 |
| Airbnb pricing | [Learning Market Dynamics for Optimal Pricing](https://medium.com/airbnb-engineering/learning-market-dynamics-for-optimal-pricing-97cffbcc53e3) | Aug 2018 |
| Uber forecasting | [Forecasting at Uber: An Introduction](https://www.uber.com/blog/forecasting-introduction/) | Sep 2018 |
| Meta data science | [How data scientists lead and drive impact at Meta](https://medium.com/@AnalyticsAtMeta/how-data-scientists-lead-and-drive-impact-at-meta-6b5b896821b2) | Jan 2025 |
| Spotify | [Encouragement Designs and Instrumental Variables for A/B Testing](https://engineering.atspotify.com/2023/8/encouragement-designs-and-instrumental-variables-for-a-b-testing) | Aug 2023 |

Read the originals first; I don't summarize them here. Each section says what I think held up, what changed, and what I would do now, with the evidence that changed my answer. Nothing here describes what these companies do today. I'm reading their public write-ups, not their current practice.

## 1. Netflix: picking artwork

**What held up.** The 2017 post logs the randomization behind every artwork choice, so a selection policy can be evaluated offline without bias. That is the right instinct, and it's the step most teams skip. It also makes the cost of exploration explicit and argues it's small when it is spread over a very large audience.

**What changed: the candidate pool.** The 2017 post describes up to a few dozen candidate images per title, made by artists and designers. Generative tools now make 20 or 50 variants cheap for any team, including teams with a small fraction of Netflix's traffic. Two problems follow, and both land on the number that gets reported:

- **Winner's curse.** The reported lift belongs to the leader, the variant that looked best. The best of many noisy estimates is biased upward even when no variant is better than the control.
- **Time confounding.** A bandit moves traffic as it learns, so the leader's impressions come disproportionately from late in the test. If the base click rate drifts during the test, comparing the leader with a control that was served evenly over time mixes up the creative's effect and the drift.

**The evidence.** I built a simulation with known ground truth for this essay ([results](https://github.com/wavde/experiment-toolkit/blob/main/docs/bandit_eval_results.md)). It runs 480,000 impressions in 12 batches. The control keeps a fixed 20% of traffic, and a batched Thompson sampler with logged assignment probabilities spreads the rest over 20 generated variants. In this table every variant is exactly as good as the control, so any claimed win is false:

| Estimate of the leader's lift | No drift: bias | No drift: false wins | Drift ±20%: bias | Drift ±20%: false wins |
|---|---:|---:|---:|---:|
| Leader's raw mean minus control's | +3.5% | 16.1% | +12.6% | **100.0%** |
| Doubly robust (DR), same log | +4.6% | 22.5% | +3.8% | 8.2% |
| DR, log split in half after the fact | −2.1% | 1.0% | −1.4% | 1.9% |
| DR, 20% evaluation slice the bandit never learns from | −0.4% | 1.2% | −0.1% | 2.0% |
| Confirmation A/B, 50,000 impressions per arm | +0.2% | 3.6% | −0.1% | 1.6% |

Four results changed what I'd recommend:

- **Off-policy estimators on the same log fix the drift, not the selection.** Under drift, DR cut the bias from +12.6% to +3.8%. Without drift it was *worse* than the raw comparison (+4.6% against +3.5%). Reweighting corrects for how traffic was allocated. It can't correct for the fact that the arm was picked because it looked good on the same data. IPS and self-normalized IPS behaved the same way as DR.
- **Splitting the log after the fact doesn't rescue it.** Both halves shaped the allocation, so the held-out half is biased downward. It also picks worse variants: when variants really differ, the leader picked this way had a true lift of +3.1%, against +6.7% for the bandit's own pick.
- **More variants, more curse.** With every variant equal to the control, same-log DR's bias grew from +0.6% with 2 variants to +8.0% with 50, and its false-win rate from 3.7% to 25.5%. The evaluation slice stayed within ±0.7% at every count.
- **The slice is cheap in pick quality and expensive in precision.** Holding back 20% of traffic from learning barely changed the pick: the leader's true lift was +6.4%, against +6.7% when the bandit learned from everything. But the slice's standard error was 6.4%, against 2.8% for a confirmation A/B of similar size, because most of its traffic went to variants that lost. Without a propensity floor, or with a floor of 1% of variant traffic, its 95% interval coverage dropped to 0.910–0.930.

**What I'd do now.** Let the bandit choose the creative, but take the lift only from impressions it never learned from. If the number feeds a ship decision or a forecast, run a short confirmation A/B of the leader against the control. If there's no time for that, reserve an evaluation slice the bandit never updates on, log its assignment probabilities, and set a floor from day one; treat what it says as a check on direction and rough size. Never quote the leader's raw lift. On an ad platform, the platform's own creative split also compares different audiences, so fix the audience before randomizing the creative ([Measuring marketing when the algorithm runs the campaign](ai-mediated-marketing-measurement.md)).

## 2. Airbnb: pricing suggestions

**What held up.** Modelling how demand arrives as check-in approaches, rather than treating demand as one number, and combining machine learning with structural modelling so the model stays interpretable. Both are still how I'd build a pricing model.

**What changed: the question I'd be asked.** Today the question I'd expect from leadership is less "how accurate is the price model?" and more "what is the suggestion worth?" That's a causal question, and the seller's decision to accept, ignore or override the suggestion sits in the middle of it. Whether the suggestion reaches the calendar through a click, an "apply automatically" switch, or an agent acting for the seller, it's still a choice the seller makes. That creates two problems the accuracy metric can't see:

- **Acceptance is self-selected.** Sellers who turn on automatic pricing differ from those who don't, and the difference may be timed: a seller who switches it on because bookings have slowed has adopted exactly when need rose. That is the mechanism that broke every observational estimator in my [opt-in AI adoption](opt-in-ai-adoption-observational-bias.md) simulation. With a true effect of zero, a difference-in-differences estimate that passed its own placebo check still excluded zero in **97.2%** of replications.
- **Sellers compete for the same guests.** A lower suggested price for treated listings takes bookings from control listings in the same market, so a listing-level A/B overstates what rolling it out to everyone would deliver. In my [switchback case](https://github.com/wavde/causal-inference-playbook/tree/main/case-studies/06-switchback), a user-level A/B overstated the policy effect about threefold and its interval almost never covered the truth. Randomizing whole market-time blocks recovered it.

**What I'd do now.**

1. **Name the estimand:** the effect of *offering* the suggestion, which is what a launch changes. The effect on sellers who accept it is a different number.
2. **Randomize the offer or the default,** not the acceptance. If the effect on acceptors matters, use the randomized offer as an instrument and report it as an effect on compliers (section 5).
3. **Randomize by market and time block when listings compete,** and accept the lost power, because the effective sample size is the number of blocks.
4. **Keep price-model accuracy as a diagnostic.** Overrides tell you where the model is wrong, but sellers choose when to override, so evaluating the model only on overridden prices is biased in the same way as the acceptance comparison.

## 3. Uber: forecasting demand

**What held up.** Nearly all of it: choose the method by how well the underlying mechanism is understood, backtest over expanding or sliding windows, score against a naive seasonal forecast, and report prediction intervals, not just point forecasts. The advice that a simple statistical model is usually the better choice when the mechanism isn't known aged best of anything in the five posts.

**What changed: pretrained forecasters.** Time-series foundation models forecast with no fitting, tuning or feature engineering. The temptation is to trust the model card and switch. On my backtest ([Pretrained forecasters vs SARIMA](pretrained-forecasters-vs-sarima.md)), zero-shot Chronos-Bolt didn't beat SARIMA with two years of history (UK MASE 0.784 against 0.741), and most of the gap came from the Christmas fold (1.301 against 1.041). With only the last 56 days, it beat SARIMA on both series (UK 0.729 against 0.850; rest of world 0.600 against 0.737). Re-selecting SARIMA's order by AIC on 56 days made UK worse (1.048), worse than seasonal-naive (0.910).

One thing I'd add, which the classic framing underweights: when a forecast triggers an action, such as moving supply, offering an incentive or running a promotion, the actuals it's scored against include the action's effect. A backtest then mixes forecast error with intervention effect. Log the actions, and estimate their effect separately. In [demand-forecasting](https://github.com/wavde/demand-forecasting), a two-way fixed-effects difference-in-differences recovers a known +15% promotion uplift (14.9%) on a synthetic store panel built from the same retail data's demand shape.

**What I'd do now.** Decide on the backtest, not the model card: the same rolling-origin folds, MASE against seasonal-naive, and a test window that includes the peak season. Keep SARIMA (or whatever wins your backtest) on long, strongly seasonal series. Use the zero-shot model as the default for new series and short history, and re-run the backtest as history accumulates. Score the intervals too; my backtest still doesn't.

## 4. Meta: how data scientists lead

**What held up.** This is the most recent of the five (January 2025), so the bar is low, but three ideas are worth keeping: data scientists embedded in product teams and accountable for their outcomes; a habit of distrusting the data in front of you; and the ecosystem score, which models downstream effects so a short experiment can inform a long-run decision.

**What changed: the metric is often a model.** For a team shipping AI features, the primary metric is often produced by another model: an LLM judge scoring responses, or an offline eval deciding what reaches users. That is the ecosystem score's problem in a new form, a model standing in for an outcome you can't observe directly, with one difference that matters. The treatment can change how the model scores it. A candidate that writes longer answers can win the judge without being better.

My evidence on how far that goes:

- **A lenient judge produces false wins that grow with traffic.** With the judge 5 points more lenient on treatment and no real effect, the naive test came out significant 15% of the time at 1,000 units per arm, 49% at 5,000 and 99% at 20,000. PPI++ with 500 human labels drawn within each arm stayed between 3.6% and 5.8% ([LLM-as-judge is a noisy metric](llm-judge-noise-in-ab-tests.md)).
- **An offline eval answers a question about the eval set.** When the eval set over-represented the segment a candidate helped, the raw offline eval shipped a change that hurt live traffic in 100% of replications. A candidate that added latency was never stopped by any offline readout, while the online test stopped it 96% of the time ([An offline eval win is evidence about the eval set](offline-evals-vs-online-ab.md)).
- **Upgrading the judge mid-test is a design change.** Labelling only at the start of the test kept about 70% of the naive bias when the new judge was more lenient on treatment ([Upgrading the judge mid-test](judge-drift-mid-test.md)). When the judge drifts with no version boundary, labels drawn at random from the whole test kept PPI++ valid (Type-I 0.048 to 0.053), while score monitors couldn't tell drift from a real quality drop ([When the judge drifts silently](silent-judge-drift.md)).

**What I'd do now.** The leadership the Meta post describes, challenging assumptions across disciplines, now has to be pointed at the evaluation itself. Before the first test of an AI feature, I would write down and get agreement on:

- what the judge may score, and that human labels are drawn at random **within each arm**;
- which changes an offline eval may gate on its own (none that change latency, cost or the traffic mix);
- the label budget, sized from the judge's agreement with humans on the product's own traffic;
- what happens to an analysis when the judge model changes during a test, flagged or silent: draw labels at random across the whole test, not just in launch week.

## 5. Spotify: propensity, and which kind

The post's label said "propensity score at Spotify", and its link went to a podcast about propensity-to-buy models. That mismatch is itself the lesson. The word means two different things:

- **Propensity to buy or convert,** P(outcome | features): a targeting score.
- **Propensity score,** P(treated | features): a tool for adjusting an observational comparison.

**What held up.** For observational adjustment, propensity scores are still the workhorse, and the old lesson still applies. On the LaLonde data, a naive comparison put a job-training program's effect at about −$8,500 against an experimental +$1,794, and AIPW landed within about $10 of the experimental number ([LaLonde, revisited](what-lalonde-taught-me.md)). Overlap, not the estimator, decides whether the answer can be trusted.

**What changed, in both senses of the word.**

- **For targeting,** whether a propensity model already finds the users an ad persuades turned out to depend on the dataset. On Criteo, the top 20% of users by propensity held 72.2% of the incremental visits. On the Hillstrom e-mail experiment they held 25.9% ([It didn't replicate](uplift-targeting-replication-hillstrom.md)).
- **For causal adjustment,** the selection that matters most for AI features isn't on anything logged. When users opt in because their need just went up, AIPW with pre-launch outcomes and DiD were each confidently wrong in 95.3–97.2% of replications with no real effect ([Opt-in AI features](opt-in-ai-adoption-observational-bias.md)).

The Spotify post's answer is the one I'd generalize. When you can't, or don't want to, withhold a feature, randomize the **encouragement** to use it, such as a banner, a default or an invitation, and use that as an instrument. You change what you randomize, not the estimator. The Meta post points at the same move for networked products with peer encouragement designs. It also covers section 2: a randomized pricing suggestion is an encouragement.

**What I'd do now.**

1. **Ask which propensity** before reading the first slide.
2. **Use propensity scores to diagnose overlap and inside AIPW,** not as proof that selection is handled.
3. **When the treatment is a choice, randomize the encouragement.** Report the effect on compliers with the first-stage strength next to it, and argue the exclusion restriction: a banner can change behavior by itself, not only through use of the feature. When you can withhold the feature, prefer a fixed holdout; in my [rollout comparison](staggered-rollout-vs-holdout.md), a staggered rollout shipped a fading novelty effect in 100.0% of replications, against 4.7% for a fixed holdout.
4. **Test propensity targeting against an uplift model** on a randomized holdout at the reach you actually buy.

## Assumptions and limitations

- **Public write-ups, not current practice.** Nothing here claims what these five companies do today.
- **Mostly synthetic evidence.** The simulations have known ground truth, which is what makes them checkable, and one mechanism at a time, which is where they can mislead. The public datasets are UCI Online Retail II, Criteo, Hillstrom and LaLonde.
- **The bandit simulation is deliberately simple:** one batched Thompson sampler, binary clicks, constant relative lifts, a linear drift and a fixed 20% control. The 2017 Netflix post is about a *contextual* bandit, which personalizes the choice. Its evaluation question is the value of a policy, not the lift of one arm, and I didn't simulate it. The selection problem carries over to choosing a policy on the same log.
- **Borrowed evidence for two themes.** The pricing and leadership sections reuse simulations built for other questions (an opt-in AI feature, a generic switchback, judge-scored A/B tests). They show the mechanisms exist, not their size in a pricing or ecosystem setting.
- **No evidence yet on encouragement designs.** Section 5's main recommendation rests on the Spotify post and standard IV results, not on my own check.

## When the conclusion breaks

- **Few variants and lots of traffic.** With 2 variants, same-log DR had +0.6% bias and a 3.7% false-win rate. There the evaluation slice's lost precision may cost more than the bias it removes.
- **No number is needed.** If the only decision is which creative to keep, and nobody will quote or forecast its lift, let the bandit run. Holding out a slice barely changes the pick.
- **You really do control assignment,** with nothing choosing between treatment and measurement. Then the classic methods apply as written.
- **Long, stable, seasonal history.** Keep the fitted model that wins the backtest. The pretrained model is a baseline.

## What would change my mind

- **A winner's-curse correction that holds on the same adaptive log.** Corrections for the selected arm exist (Andrews, Kitagawa & McCloskey 2024), but they assume approximately normal estimates, which a bandit's adaptive allocation can break, and I haven't tested one on this log. One that kept 95% coverage with 20–50 variants on a bandit's own log would make the evaluation slice unnecessary.
- **A simulation of encouragement designs at realistic uptake** showing that the first stage is too weak to be useful. Then section 5 would tilt from "randomize the encouragement" to "keep a holdout".
- **A record of past launches** in which judge-scored or offline readouts matched the online result for a class of changes. Then those readouts could gate that class alone.

## The decision rule

1. **Before reading any lift, name what decided who got treated:** you, a bandit, a seller, a user or a platform. If it wasn't you, treat the number as a screen.
2. **Keep randomization at the decision being made, from day one:** an evaluation slice with logged propensities and a floor, a fixed holdout, a randomized offer or encouragement, or a switchback block.
3. **Quote effects only from that randomized slice,** and run a short confirmation test when the number feeds a ship decision or a forecast.
4. **Let models that sit in the measurement path, such as pretrained forecasters, LLM judges and bandits, compete as candidates** under the same backtest or test as everything else, never as the referee.

---

*Code, tests, and results: [`bandit_eval.py`](https://github.com/wavde/experiment-toolkit/blob/main/src/experiment_toolkit/bandit_eval.py) · [`test_bandit_eval.py`](https://github.com/wavde/experiment-toolkit/blob/main/tests/test_bandit_eval.py) · [`bandit_eval_sim.py`](https://github.com/wavde/experiment-toolkit/blob/main/scripts/bandit_eval_sim.py) · [`bandit_eval_results.md`](https://github.com/wavde/experiment-toolkit/blob/main/docs/bandit_eval_results.md) · [causal-inference-playbook cases 04, 06, 07, 08](https://github.com/wavde/causal-inference-playbook) · [demand-forecasting pretrained results](https://github.com/wavde/demand-forecasting/blob/main/docs/pretrained_results.md) · [paid-media-playbook case 07 (Criteo)](https://github.com/wavde/paid-media-playbook/tree/main/case-studies/07-criteo-uplift) · References: Andrews, Kitagawa, McCloskey (2024), Inference on winners, QJE · Dudík, Langford, Li (2011), Doubly robust policy evaluation and learning · Li, Chu, Langford, Wang (2011), Unbiased offline evaluation of contextual-bandit-based news article recommendation algorithms · Angrist, Imbens, Rubin (1996), Identification of causal effects using instrumental variables · Bojinov, Simchi-Levi, Zhao (2023), Design and analysis of switchback experiments.*
