# A covariate stacks on the judge, but a better judge beats both: cutting the label budget of PPI++

*~6 min read. Companion code is [`ppi_covariates`](https://github.com/wavde/experiment-toolkit/blob/main/src/experiment_toolkit/ppi_covariates.py) in experiment-toolkit, with [tests](https://github.com/wavde/experiment-toolkit/blob/main/tests/test_ppi_covariates.py), a [one-command simulation](https://github.com/wavde/experiment-toolkit/blob/main/scripts/ppi_covariates_sim.py), and the [generated results](https://github.com/wavde/experiment-toolkit/blob/main/docs/ppi_covariates_results.md). It follows [LLM-as-judge is a noisy metric](llm-judge-noise-in-ab-tests.md).*

## The recommendation

If an A/B test scored by an LLM judge has a pre-treatment covariate that predicts quality, add it to PPI++ as a second control variate. It can't introduce bias, and it cuts the human-label budget. Size the budget from the **combined** R² of the judge and the covariate, not from each one separately, because they overlap. When the judge is already good, put the next unit of effort into a better judge rather than a better covariate: in this simulation, improving the judge from 85% to 95% accuracy cut the label budget more than any covariate did.

The numbers (20,000 judged units per arm, true effect +0.03 on a 0.60 base rate, labels needed for 80% power):

- **Judge at 85% accuracy:** PPI++ alone needs **2,397** labels per arm.
  - Adding a covariate with R² 0.25 brings that to **2,072** (−14%).
  - A covariate with R² 0.50 brings it to **1,625** (−32%).
- **Judge at 95% accuracy:** PPI++ alone needs **982** labels per arm, fewer than any covariate achieved at 85%.

## The decision

[Loop 1](llm-judge-noise-in-ab-tests.md) ended with a correct but expensive answer: PPI++ with human labels drawn within each arm removes judge bias, but the label budget caps power. Teams often have a cheap, pre-treatment signal for every unit, such as a query's difficulty bucket or a user's prior-period outcome. That is the CUPED idea (Deng et al. 2013). How much does that signal cut the label budget, and when does it stop helping?

## The setup

Everything is simulated, so every answer has a closed form.

- **True quality** is binary, with a 0.60 control rate.
- **The covariate X** is a binary pre-treatment attribute known for every unit. It is set so that it explains a chosen share R²(Y; X) of the variance of true quality.
- **The judge** errs through a confusion matrix that depends only on true quality. So the judge score and X are independent signals about the same outcome.
- **Labels:** every unit is judged, and human labels go to a random sample within each arm.

The estimator is PPI++ with a vector of control variates. In each arm:

```
theta_hat = mean_lab(Y) - lam' (mean_lab(Z) - mean_unlab(Z))
```

`Z` is the judge score, the covariate, or both. With the variance-minimising weights the per-arm variance is

```
Var(Y) / n * (1 - R² * N / (N + n))
```

where `R²` is the population R² of true quality on `Z`, `n` is the number of labeled units and `N` the number of unlabeled units. Two limits are visible right away: R² can't exceed 1, and the factor `N / (N + n)` shrinks when labels are a large share of the arm.

The closed form reproduces loop 1's PPI++ budgets exactly (982, 2,397 and 3,610 labels at judge accuracy 0.95, 0.85 and 0.70; 4,126 with human labels alone), and that is a test in the repo. Seed `20261127`; `python scripts/ppi_covariates_sim.py` reproduces everything in under two minutes, and CI smoke-runs the same script.

![PPI++ with a covariate](https://raw.githubusercontent.com/wavde/experiment-toolkit/main/docs/ppi_covariates.png)

## The check against known answers

At 20,000 judged units and 500 labels per arm, with judge accuracy 0.85 and a covariate with R² 0.25, all four estimators are unbiased over 2,000 replications. Their spread matches the closed-form SE:

| Estimator (true effect +0.03) | Mean estimate | SD (closed-form SE) | Power (closed form) |
|---|---:|---:|---:|
| Human labels only | +0.0294 | 0.0309 (0.0308) | 0.163 (0.164) |
| Human labels + covariate | +0.0297 | 0.0266 (0.0267) | 0.201 (0.203) |
| PPI++ | +0.0300 | 0.0227 (0.0225) | 0.268 (0.266) |
| PPI++ + covariate | +0.0300 | 0.0207 (0.0208) | 0.316 (0.302) |

With no true effect, coverage is 0.954–0.956 and Type-I error 0.044–0.046. With a judge 5 points more lenient on treatment, the PPI++ rows show Type-I error of 0.060–0.063. The human-labels-only row on the same draws shows 0.056, even though it doesn't use the judge at all. So the excess, about 2.6 Monte Carlo SEs, is in the draws, not leakage from the judge.

## Why the gains don't add up

The judge alone explains R² = 0.480 of true quality. A covariate with R² 0.25 doesn't add 0.25 on top: together they explain 0.557. Both are noisy readings of the same outcome, so the covariate only helps with the part of quality the judge misses. That is why the saving from adding the covariate depends on the judge:

| Judge accuracy | Covariate R² 0.10 | 0.25 | 0.50 |
|---:|---:|---:|---:|
| no judge | 8% | 21% | 45% |
| 0.95 | 2% | 6% | 16% |
| 0.85 | 5% | 14% | 32% |
| 0.70 | 7% | 19% | 42% |

The weaker the judge, the more the covariate adds, and the most it adds is with no judge at all. At judge accuracy 0.95 a strong covariate saves 16%. At 0.70 it saves 42%.

The same table shows where effort pays. At covariate R² 0.25:
- moving from a 0.85 judge to a 0.95 judge takes the budget from 2,072 to 923 labels;
- moving from covariate R² 0.25 to 0.50 with a 0.85 judge takes it from 2,072 to 1,625.

A better judge is the bigger lever, as long as it can be had.

## When it stops helping

- **Labels are a large share of the arm.** At judge accuracy 0.85 and covariate R² 0.25, adding the covariate saves 15% at 100,000 judged units per arm and 14% at 20,000, but only 5% at 5,000. With 5,000 units, PPI++ already needs 3,561 labels, 71% of the arm, so there are few unlabeled units left to borrow strength from.
- **The judge is already strong.** See the table above: 2–16% at 0.95 accuracy.
- **Very small label budgets.** Each control variate is one more weight estimated from the labeled units. At 30 labels per arm, coverage is 0.934 for PPI++ and 0.923 with the covariate. By 100 labels they are 0.952 and 0.944, and by 300 they are 0.952 and 0.949.

## Assumptions and limitations

- **Synthetic, binary, and one covariate.** Real covariates are often continuous, several at once, and related to the judge's errors. A covariate that predicts *where the judge is wrong*, not just quality, could help more than this model shows.
- **Judge errors depend only on true quality.** If the judge is more lenient on easy queries and the covariate is query difficulty, the two signals overlap differently.
- **The covariate must be pre-treatment and measured the same way for labeled and unlabeled units.** That is what keeps the estimator unbiased. A post-treatment covariate (one the treatment can change) breaks this.
- **Human labels are treated as ground truth,** drawn at random within each arm, as in loop 1.
- **Normal approximations.** Coverage holds from about 100 labels per arm up.

## What would change my mind

- **Real experiments where the combined R² of judge and covariate is much higher than either alone.** That would happen if the covariate predicts the judge's mistakes. Then the covariate would be worth more than this simulation says, even with a strong judge.
- **Coverage failures at realistic label budgets** with continuous or many covariates, where the estimated weights are noisier than one binary covariate.
- **A judge that can't be improved cheaply.** Then the covariate is the only lever left, and the "better judge first" advice doesn't apply.

## The decision rule

1. **Add any pre-treatment covariate you have to PPI++.** It can't add bias, and it lowers the label budget by roughly the share of quality it explains beyond the judge.
2. **Size the label budget from the combined R² of judge and covariate.** Don't add the two R² values: 0.48 plus 0.25 gave 0.557 here.
3. **If the judge can be improved, do that first.** Going from 85% to 95% accuracy roughly halved the budget or better in every column of the grid. No covariate in the grid did as much.
4. **Below about 100 labels per arm, expect coverage below 95%** whether or not you add the covariate (0.923–0.934 at 30 labels). Plan a larger label budget instead of relying on the extra control variate.

---

*Code, tests, and results: [`ppi_covariates.py`](https://github.com/wavde/experiment-toolkit/blob/main/src/experiment_toolkit/ppi_covariates.py) · [`test_ppi_covariates.py`](https://github.com/wavde/experiment-toolkit/blob/main/tests/test_ppi_covariates.py) · [`ppi_covariates_sim.py`](https://github.com/wavde/experiment-toolkit/blob/main/scripts/ppi_covariates_sim.py) · [`ppi_covariates_results.md`](https://github.com/wavde/experiment-toolkit/blob/main/docs/ppi_covariates_results.md) · References: Angelopoulos, Duchi, Zrnic (2023), PPI++ · Deng, Xu, Kohavi, Walker (2013), CUPED · Chaganty, Mussmann, Liang (2018), The price of debiasing automatic metrics in natural language evaluation.*
