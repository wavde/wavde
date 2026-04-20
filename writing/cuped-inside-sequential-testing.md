# CUPED inside sequential testing: two variance reductions that compound

*~6 min read. Builds on [case 05 of the causal-inference-playbook](https://github.com/wavde/causal-inference-playbook/tree/main/case-studies/05-sequential-testing) and the new `msprt_cuped_pvalue` in [experiment-toolkit v0.2.0](https://pypi.org/project/experiment-toolkit/).*

---

## Two favourite tools, both about variance

Most of the variance-reduction folklore in experimentation reduces to
two tricks:

1. **CUPED** (Deng et al. 2013): subtract off the part of the outcome
   predicted by a pre-experiment covariate. The residual has smaller
   variance, so your sample-size requirements shrink by a factor of
   `(1 - rho^2)`.

2. **Sequential testing** (Johari, Pekelis, Walsh 2015): use an always-
   valid p-value like mSPRT so you can peek whenever you want. In
   practice, you typically stop when the p-value crosses your alpha
   threshold — in expectation, that's earlier than the fixed-horizon
   test would have ended.

Each one on its own will cut experiment duration in a well-configured
system by 20-50%. The natural question: **do they compose?** If my
outcome has a pre-experiment covariate with `rho=0.7`, and I'm running
mSPRT with `tau=sigma`, do I get both savings at once, or does one eat
the other?

## The answer is yes, with one caveat

Under randomisation, CUPED is a deterministic linear transformation of
the outcome that preserves unbiasedness. mSPRT only requires that the
per-user sequence of test statistics be a martingale under the null. A
fixed linear transformation doesn't break that martingale property. So
running mSPRT on the CUPED-adjusted outcome:

$$
\widetilde{Y}_i = Y_i - \theta \cdot (X_i - \bar{X})
$$

gives you a valid always-valid p-value on a lower-variance outcome, and
the same `(1 - rho^2)` sample-size reduction that CUPED gives you in
the fixed-horizon setting carries over to the expected stopping time.

The caveat: **`theta` has to be chosen independently of the sequential
peeking**. If you re-estimate `theta` at every peek from an ever-growing
stream, the sequence of test statistics is no longer a martingale — you
leak information from the future into the past. The fix is the same one
the CUPED paper suggests: estimate `theta` once on a **warm-up sample**
(say the first 5-10% of users) and freeze it.

## What the v0.2.0 helper does

```python
from experiment_toolkit import msprt_cuped_pvalue, compute_theta

# Warm-up phase: first 2,000 users per arm
theta = compute_theta(
    y=np.concatenate([y_t[:2000], y_c[:2000]]),
    x=np.concatenate([x_t[:2000], x_c[:2000]]),
)

# Production peeking: pass frozen theta in on every peek
for n in (5_000, 10_000, 25_000, 50_000):
    p = msprt_cuped_pvalue(
        y_t=y_t, y_c=y_c, x_t=x_t, x_c=x_c,
        n_per_arm=n, tau=1.0, theta=theta,
    )
    if p < 0.05:
        print(f"Stop at n={n}, p={p:.4f}")
        break
```

The function strips out the CUPED centring (`theta * (X - X_pooled_mean)`),
computes the adjusted `delta_hat` and `sigma`, and hands them to the
classical mSPRT formula. If you pass `theta=None`, it estimates theta
from the pooled sample at that peek — fine for a single analysis, but
**not** for repeated peeking.

## How big is the compounded effect

In a simulation with `rho=0.8` between the covariate and outcome, and a
true lift of 0.05 standard deviations:

| Test | Median stopping n per arm |
|---|---|
| Fixed-horizon z-test | 6,280 |
| mSPRT (plain) | 4,100 |
| mSPRT + CUPED | 1,500 |

The sequential test alone saves about a third. CUPED alone would save
`1 - (1-0.64) = 64%` of the fixed-horizon sample. Compounded, you end
up needing about a quarter of the fixed-horizon sample to declare
significance, with full peeking validity. That's the difference between
a 4-week experiment and a 1-week experiment.

## When this backfires

The usual CUPED pitfalls still apply:

- **Weak covariate (`rho < 0.3`)**: the variance-reduction factor is
  close to 1 and the overhead of the warm-up isn't worth it.
- **Covariate that predicts the treatment effect**: if `X` is correlated
  not just with `Y` but with the magnitude of the *effect*, CUPED can
  bias the estimate. The fix is to de-mean by arm, or to use a method
  like ML-CUPED that models the heterogeneity.
- **Non-stationary user population**: if the users who arrive later in
  the experiment have systematically different `X`, the pooled mean
  shifts and your frozen `theta` becomes stale. Re-estimating `theta`
  on a *fresh holdout window* periodically (not on the peeking stream)
  is the clean fix.

And the sequential pitfall: **mSPRT's `tau` matters**. If you set
`tau >> true_effect`, the test is under-powered. If you set
`tau << true_effect`, you give up optimality. A reasonable default is
`tau = sigma` on the *adjusted* outcome, which is what the helper uses
by default.

## The take-home

CUPED and sequential testing both reduce variance, but in different
ways: CUPED shrinks the variance of a single-peek estimate, mSPRT
converts calendar time into a valid early-stopping rule. They compose
cleanly if — and only if — `theta` is frozen. When both apply, the
compounded effect on experiment duration is often dramatic enough to
be worth the warm-up infrastructure.

---

*Code and references: [case study 05](https://github.com/wavde/causal-inference-playbook/tree/main/case-studies/05-sequential-testing) · [experiment-toolkit](https://github.com/wavde/experiment-toolkit) · Deng, Xu, Kohavi, Walker (2013) · Johari, Pekelis, Walsh (2015, 2021)*
