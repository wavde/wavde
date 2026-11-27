# The targeting model already knows who would convert: propensity vs uplift on real randomized data

*~7 min read. Companion code is [case 07](https://github.com/wavde/paid-media-playbook/tree/main/case-studies/07-criteo-uplift) in paid-media-playbook, with [tests](https://github.com/wavde/paid-media-playbook/blob/main/tests/test_criteo_uplift.py) and the [generated results](https://github.com/wavde/paid-media-playbook/blob/main/case-studies/07-criteo-uplift/results.md). It checks the simulation in [Measuring marketing when the algorithm runs the campaign](ai-mediated-marketing-measurement.md) against real data.*

> **Update:** a replication on a second randomized dataset, a retailer's e-mail experiment, did not reproduce the concentration finding below. There, the top 20% of customers by propensity reached 25.9% of incremental visits, not 72.2%. Credit inflation and falling marginal returns did replicate, but they were weaker. See [It didn't replicate](uplift-targeting-replication-hillstrom.md). Treat the uplift-vs-propensity conclusion here as specific to this dataset.

## The recommendation

Treat platform-credited results from a tightly targeted campaign as an upper bound, not as incrementality. Price any budget increase with a marginal test, never with the average lift at the current reach. And don't assume an uplift model beats the platform's propensity targeting: test that against a randomized holdout at the reach you actually buy.

The reason comes from 6,988,166 held-out users in the public Criteo uplift dataset, where treatment was randomized:

- Targeting the top 5% by predicted visit propensity makes platform-credited visits **8.58x** the incremental visits, against 4.71x when everyone is targeted.
- Extending propensity targeting from 10% to 40% reach adds **7.30** incremental visits per 1,000 extra users. The top 10% averaged **49.75** per 1,000.
- An uplift model reaches 45.7% of all incremental visits at 5% reach, against 28.9% for propensity. By 30% reach the two are within 2 points.

## The decision

The [loop 4 simulation](ai-mediated-marketing-measurement.md) reached three conclusions about platforms that target by predicted conversion propensity:

- their credited conversions overstate incrementality, and more so as targeting sharpens;
- the next dollar returns less than the average dollar;
- the fix is randomized measurement at the decision being made.

A simulation can only show what its assumptions allow. The question here is whether those conclusions hold on real data, and whether a team should replace propensity targeting with an uplift model.

## The data and the check

The Criteo uplift dataset (Diemert et al. 2018) contains 13,979,592 users from incrementality tests in which a random 15% were withheld from targeting. It has 12 anonymized features and `visit` and `conversion` labels. Treatment is randomized, so for any set of users the treated-minus-control difference within that set is an unbiased estimate of its incremental outcomes. That is the baseline a skeptical reviewer would accept.

Across all users, treatment raises the visit rate from 0.03820 to 0.04854 (+27.1%) and the conversion rate from 0.00194 to 0.00309 (+59.4%). Everything is intent-to-treat: only 3.1% of all users were actually exposed.

On a seeded half of the data I fit two simple models (ridge logistic regression on the features and their squares):

- a **propensity** model, `P(outcome | treated, x)`, which is what a platform optimizing for outcomes learns;
- a **T-learner uplift** model, which subtracts a second model fit on the control group.

On the other half I rank users by each score and read out every reach level against its own randomized control. Standard errors come from 20 folds of the test half. Seed `20261127`; `python case-studies/07-criteo-uplift/src/run.py` downloads the data (SHA-256 verified) and reproduces everything in about a minute.

## What replicated

**Credit inflation grows as targeting sharpens.** Here "credited" means every outcome among treated users, the most generous attribution rule.

| Targeting (visits) | Reach | Credited / incremental |
|---|---:|---:|
| Everyone | 100% | 4.71x |
| Propensity | 20% | 5.90x |
| Propensity | 5% | 8.58x |
| Uplift model | 5% | 3.94x |

For conversions the pattern is the same but weaker: 2.58x for everyone and 3.37x for the top 5% by propensity. Targeting by uplift doesn't fix it for conversions (3.36x).

**The marginal user is worth less than the average one.** This is the case-06 finding that should matter most for budget decisions, and the real data makes it much stronger:

| Propensity targeting | Avg per 1k, top 10% | Marginal per 1k, 10% → 40% | Marginal / average |
|---|---:|---:|---:|
| Visits | 49.75 | 7.30 | 0.15 |
| Conversions | 8.59 | 0.33 | 0.04 |

The loop 4 simulation assumed a ratio of 0.81. Under random targeting the ratio is 1.03 for visits and 1.01 for conversions, as it should be. A lift test on a tightly targeted campaign measures the users the algorithm was surest about. It says almost nothing about the users a bigger budget would add.

## What the simulation got wrong

The simulation assumed high-intent users respond slightly *less* to ads, in log-odds terms. On Criteo, absolute uplift rises steeply with propensity: the top 20% by visit propensity hold 72.2% of all incremental visits, and the top 20% by conversion propensity hold 86.2% of all incremental conversions. The "they'd have converted anyway" story is true for the *credited* number. It isn't true for the people the targeting model picks: they are also the ones most moved by the ad.

That's why the uplift model's advantage is narrow:

| Share of all incremental reached | 5% reach | 10% | 20% | 30% |
|---|---:|---:|---:|---:|
| Visits, propensity | 28.9% | 48.3% | 72.2% | 83.5% |
| Visits, uplift model | 45.7% | 62.5% | 76.0% | 85.3% |
| Conversions, propensity | 59.1% | 71.6% | 86.2% | 90.1% |
| Conversions, uplift model | 53.1% | 68.8% | 80.4% | 85.1% |

For visits, the uplift model wins clearly at 5–10% reach and roughly ties from 20% reach on. For rare conversions it doesn't beat propensity at any reach shown. The T-learner subtracts two noisy models of a 0.3% event, and the difference is mostly noise.

## Assumptions and limitations

- **One advertiser's tests and anonymized features.** How much uplift concentrates in high-propensity users may differ for other products, funnels and audiences.
- **Intent-to-treat, with generous credit.** "Credited" counts every outcome among treated users. Real attribution windows and last-touch rules credit less, but they credit the same high-propensity users.
- **Simple models.** A stronger uplift learner (X-learner, causal forest, a direct uplift objective) could widen the gap over propensity, especially for conversions. This check shows what a straightforward uplift model buys, not the ceiling.
- **Test-set sampling error only.** The standard errors don't include model-fitting noise, because the models are fit once.
- **Shares are relative to targeting everyone,** not to a budget-optimal policy.

## When the conclusion breaks

- **Uplift is concentrated in users the propensity model ranks low,** for example new customers whom the ad persuades but who rarely convert on their own. Then propensity targeting wastes most of the budget and an uplift model wins at every reach. This dataset doesn't show that pattern, but nothing guarantees a given advertiser won't.
- **The campaign already runs at broad reach.** At 30% reach and above, the targeting rule barely matters for visits. The budget-level question (marginal return) matters more.
- **Credit is defined differently.** View-through or modeled attribution can push credited numbers above or below these ratios. The direction (more inflation at tighter targeting) is the robust part.

## What would change my mind

- **The same check on other randomized advertising datasets, with uplift concentrated in low-propensity users.** That would change the recommendation from "test an uplift model first" to "replace propensity targeting".
- **A stronger uplift learner that beats propensity for conversions** on this data, by more than the fold standard errors (about 2 to 2.5 points of share at 5–10% reach).
- **Platform-side holdouts showing a marginal / average ratio near 1** at the reach changes a team actually makes. Then the average lift at current reach would be a usable budget guide.

## The decision rule

1. **Platform-credited results are an upper bound,** and a looser one the more tightly the campaign is targeted (4.71x everyone, 8.58x for the top 5% by propensity, for visits).
2. **Price budget changes with a marginal test,** such as case 06's two-cell budget split, at the reach levels you're choosing between. Under propensity targeting, the marginal return from 10% to 40% reach was 15% of the average for visits and 4% for conversions.
3. **Treat an uplift model as a hypothesis, not an upgrade.** Before switching from propensity targeting, compare the two on a randomized holdout at the reach you buy. Here it paid at 5–10% reach on visits and not on conversions.

---

*Code, tests, and results: [`criteo_uplift.py`](https://github.com/wavde/paid-media-playbook/blob/main/case-studies/07-criteo-uplift/src/criteo_uplift.py) · [`run.py`](https://github.com/wavde/paid-media-playbook/blob/main/case-studies/07-criteo-uplift/src/run.py) · [`test_criteo_uplift.py`](https://github.com/wavde/paid-media-playbook/blob/main/tests/test_criteo_uplift.py) · [`results.md`](https://github.com/wavde/paid-media-playbook/blob/main/case-studies/07-criteo-uplift/results.md) · References: Diemert, Betlei, Renaudin, Amini (2018), A large scale benchmark for uplift modeling · Künzel, Sekhon, Bickel, Yu (2019), Metalearners for estimating heterogeneous treatment effects · Radcliffe (2007), Using control groups to target on predicted lift.*
