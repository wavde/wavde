# It didn't replicate: propensity vs uplift targeting on a second randomized dataset

*~6 min read. Companion code is [case 08](https://github.com/wavde/paid-media-playbook/tree/main/case-studies/08-hillstrom-email-uplift) in paid-media-playbook, with [tests](https://github.com/wavde/paid-media-playbook/blob/main/tests/test_hillstrom_uplift.py) and the [generated results](https://github.com/wavde/paid-media-playbook/blob/main/case-studies/08-hillstrom-email-uplift/results.md). It re-tests [The targeting model already knows who would convert](propensity-vs-uplift-targeting-criteo.md).*

## The recommendation

Don't carry a targeting conclusion from one channel or advertiser to another without testing it on your own randomized holdout. Two things replicated across both datasets:

- platform-credited results overstate incrementality;
- under propensity targeting, the next customer is worth less than the average one.

Price budget changes with a marginal test, as before. But the claim I made last time, that propensity targeting already finds the persuadable users so an uplift model rarely pays, did not replicate. On a retailer's e-mail experiment:

- the top 20% of customers by propensity reach **25.9%** of all incremental visits, against 20% at random;
- on Criteo display ads the same number was **72.2%**.

## The decision

Loop 6 ended with a rule built on one public dataset, Criteo's display-advertising incrementality tests. There, high-propensity users were also the most persuadable in absolute terms, so the platform's propensity targeting captured most of the incremental effect, and an uplift model only helped at tight reach. The plan's own principle says a finding from one dataset is a hypothesis. Does it hold for a different advertiser, channel and audience?

## The data and the check

The Hillstrom MineThatData e-mail experiment (2008) has 64,000 customers of a retailer who purchased in the past year. They were randomized one third each to a men's merchandise e-mail, a women's merchandise e-mail, or no e-mail, and their visits, conversions and spend were tracked for two weeks. Sending any e-mail raised the visit rate from 0.1062 to 0.1670 (+57.3%) and the conversion rate from 0.0057 to 0.0107 (+86.5%). The author published it for public analysis without a stated license, so the code downloads it from his site and checks a SHA-256 rather than committing it.

The check is the same as on Criteo:

- **Models:** a propensity model, `P(outcome | e-mailed, x)`, and a T-learner uplift model, both ridge logistic regressions with the same code as loop 6.
- **Readout:** rank customers by each score, then read each reach level against its own randomized control.
- **Cross-fitting:** with only 64,000 customers, scores are cross-fitted over 5 folds, so every customer is scored by models that never saw them.
- **SEs:** from a bootstrap over customers. Any ratio whose denominator is too noisy to divide by (coefficient of variation above 0.3) is reported as not estimable rather than printed with a meaningless SE.

Seed `20261127`. `python case-studies/08-hillstrom-email-uplift/src/run.py` reproduces it in seconds, and CI runs the pipeline on synthetic data with the real schema.

## What replicated, and what didn't

For visits, any e-mail vs none:

| | Criteo (loop 6) | Hillstrom (this loop) |
|---|---:|---:|
| Share of incremental reached, top 20% by propensity | 72.2% | 25.9% (2.1%) |
| Share of incremental reached, top 20% by uplift model | 76.0% | 27.8% (1.9%) |
| Marginal / average return, 10% → 40% reach, propensity | 0.15 | 0.67 (0.11) |
| Credited / incremental, everyone | 4.71x | 2.74 (0.11) |
| Credited / incremental, top 10% by propensity | 7.38x | 3.15 (0.34) |

*Criteo numbers are from [case 07's results](https://github.com/wavde/paid-media-playbook/blob/main/case-studies/07-criteo-uplift/results.md); Hillstrom numbers, with bootstrap SEs, from [case 08's](https://github.com/wavde/paid-media-playbook/blob/main/case-studies/08-hillstrom-email-uplift/results.md).*

**Replicated:**
- **Credit inflation.** Credited visits overstate incremental visits on both datasets.
- **Falling marginal returns.** Under propensity targeting the marginal return is below the average on both.

**Didn't replicate:**
- **How much those effects grow with targeting.** On Criteo, credit inflation rose from 4.71x to 7.38x in the top 10%; on Hillstrom it barely moves (2.74 to 3.15, within about one SE). The marginal return falls to 0.67 of the average, not 0.15.
- **The concentration.** On Criteo a fifth of the users held nearly three quarters of the incremental visits. Here they hold about a quarter.

The uplift model doesn't change the picture. At 10% and 20% reach it is within about one SE of propensity (17.0% vs 15.1%, 27.8% vs 25.9%). The one clear gap is at 50% reach, 64.4% (2.3%) against 56.7% (2.2%). The separate men's and women's campaigns look the same: their top 20% by propensity reach 26.3% and 27.9% of incremental visits.

## Why the difference is plausible

The datasets differ in almost every way that matters for targeting:

- **Audience.** Hillstrom's customers have all bought from the retailer in the past year, so the e-mail is a reminder to people who already know the brand. Criteo's display ads reach a much broader population, where the people likely to act at all are a small, findable group.
- **Rates.** E-mail visit rates are higher (10.6% without the e-mail, against 3.8% for Criteo's control) and the relative lift is larger, so the effect is spread across the base rather than concentrated.

These are explanations, not tested claims. Two datasets can show that loop 6's finding isn't general. They can't say which difference causes it.

## What the data can't say

- **Conversions.** Sending any e-mail produced only about 212 incremental conversions. Every conversion share has an SE of 5 points or more, and most conversion ratios are not estimable. Nothing here should be read as a conversion result.
- **A small dataset.** 64,000 customers is enough for visit shares to about ±2 points, not for fine comparisons between models.
- **Simple models.** A stronger uplift learner might do better. Published work on this dataset reports modest gains, not a large gap.
- **Two datasets.** That is enough to show a finding doesn't generalise, but not enough to map when it does.

## When the conclusion breaks

- **Your channel looks like Criteo's:** broad reach, low base rates, and a small group of high-intent users. Then propensity targeting may well capture most of the incremental effect, and loop 6's rule applies.
- **The effect is concentrated in customers the features describe well,** for example by product affinity. Then an uplift model may separate them where these simple models didn't. The randomized holdout is how you'd find out.

## What would change my mind

- **A third and fourth randomized dataset that line up with Criteo** rather than Hillstrom. That would suggest Hillstrom is the unusual case (small, existing customers only, a single e-mail).
- **A stronger uplift learner that clearly beats propensity on Hillstrom visits** by more than the bootstrap SEs (about 2 points of share at 10–20% reach).

## The decision rule

1. **Credited results overstate incrementality on both datasets.** Treat them as an upper bound everywhere (2.74x to 4.71x when targeting everyone).
2. **Price budget changes with a marginal test.** The marginal return was below the average under propensity targeting on both datasets, but by very different amounts (0.67 vs 0.15), so the size has to be measured, not assumed.
3. **Whether propensity targeting finds the persuadable users is channel-specific.** Test it on your own randomized holdout before deciding between propensity targeting, an uplift model, or not targeting at all.

---

*Code, tests, and results: [`hillstrom_uplift.py`](https://github.com/wavde/paid-media-playbook/blob/main/case-studies/08-hillstrom-email-uplift/src/hillstrom_uplift.py) · [`run.py`](https://github.com/wavde/paid-media-playbook/blob/main/case-studies/08-hillstrom-email-uplift/src/run.py) · [`test_hillstrom_uplift.py`](https://github.com/wavde/paid-media-playbook/blob/main/tests/test_hillstrom_uplift.py) · [`results.md`](https://github.com/wavde/paid-media-playbook/blob/main/case-studies/08-hillstrom-email-uplift/results.md) · References: Hillstrom (2008), The MineThatData E-Mail Analytics And Data Mining Challenge · Radcliffe (2008), Hillstrom's MineThatData Email Analytics Challenge: an approach using uplift modelling · Diemert, Betlei, Renaudin, Amini (2018), A large scale benchmark for uplift modeling · Künzel, Sekhon, Bickel, Yu (2019), Metalearners.*
