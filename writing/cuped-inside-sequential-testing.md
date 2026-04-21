# CUPED inside sequential testing: two variance reductions that compound

*~6 min read. Companion code in [case 05 of the causal-inference-playbook](https://github.com/wavde/causal-inference-playbook/tree/main/case-studies/05-sequential-testing) and `msprt_cuped_pvalue` in [experiment-toolkit](https://pypi.org/project/experiment-toolkit/).*

## Two tools, both about variance

The variance-reduction folklore in online experimentation is largely two techniques.

1. **CUPED** (Deng et al., 2013): subtract the part of the outcome predicted by a pre-experiment covariate. The residual has lower variance, and the sample-size requirement scales by `(1 - rho^2)`.
2. **Sequential testing** (Johari, Pekelis, Walsh, 2015): use an always-valid p-value like mSPRT so stakeholders can look at the number whenever they want. The expected stopping time is generally shorter than a fixed-horizon test at the same alpha and power.

Each one on its own can cut experiment duration meaningfully. The natural question is whether they compose: if the outcome has a pre-experiment covariate with `rho = 0.7` and mSPRT is running with `tau = sigma`, do you get both savings at once, or does one eat the other?

## Why they compose

Under randomisation, CUPED is a deterministic linear transformation of the outcome that preserves unbiasedness. mSPRT only requires that the per-user sequence of test statistics be a martingale under the null. A fixed linear transformation of the outcome preserves that property. So running mSPRT on the CUPED-adjusted outcome,

$$
\widetilde{Y}_i = Y_i - \theta \cdot (X_i - \bar{X}),
$$

produces a valid always-valid p-value on a lower-variance outcome. The `(1 - rho^2)` factor that CUPED buys you in the fixed-horizon setting carries through to the expected stopping time of the sequential test.

## The caveat that matters

`theta` has to be chosen independently of the peeking schedule. If `theta` is re-estimated at each peek from the full accumulated stream, the sequence of statistics is no longer a martingale: information from later observations leaks into the adjustment at earlier ones, and the always-valid guarantee breaks.

The clean fix is the one the CUPED paper already recommends: estimate `theta` once on a warm-up sample (say the first 5 to 10 percent of enrolled users) and freeze it for the rest of the experiment.

## The helper

```python
from experiment_toolkit import msprt_cuped_pvalue, compute_theta

# Warm-up: first 2,000 users per arm
theta = compute_theta(
    y=np.concatenate([y_t[:2000], y_c[:2000]]),
    x=np.concatenate([x_t[:2000], x_c[:2000]]),
)

# Peeking: pass frozen theta on every peek
for n in (5_000, 10_000, 25_000, 50_000):
    p = msprt_cuped_pvalue(
        y_t=y_t, y_c=y_c, x_t=x_t, x_c=x_c,
        n_per_arm=n, tau=1.0, theta=theta,
    )
    if p < 0.05:
        print(f"Stop at n={n}, p={p:.4f}")
        break
```

The function applies the CUPED centring (`theta * (X - X_pooled_mean)`), computes the adjusted `delta_hat` and `sigma`, and hands both to the mSPRT formula. Passing `theta=None` causes it to estimate `theta` from the pooled sample at that peek; that is valid for a single analysis, and invalid for repeated peeking.

## Rough magnitude

In a simulation with `rho = 0.8` and a true lift of 0.05 standard deviations:

| Test | Median stopping n per arm |
|---|---|
| Fixed-horizon z-test | 6,280 |
| mSPRT (plain) | 4,100 |
| mSPRT + CUPED | 1,500 |

The sequential test alone saves roughly a third against the fixed-horizon baseline. CUPED alone would save about `1 - (1 - 0.64) = 64%` of the fixed-horizon sample. Compounded, the median stopping sample is about a quarter of the fixed-horizon requirement, with full peeking validity. In calendar time, that is the difference between a one-month experiment and a one-week one.

## When this backfires

Standard CUPED pitfalls still apply.

- **Weak covariate (`rho < 0.3`).** The variance-reduction factor is close to 1 and the warm-up overhead is not worth the complexity.
- **Covariate correlated with the treatment effect, not just the outcome.** If `X` predicts *how big* the effect is (not just the level of `Y`), a single `theta` can bias the estimate. De-mean `theta` by arm, or use a method that models the heterogeneity.
- **Non-stationary user mix.** If the users who arrive later in the experiment have systematically different `X`, the pooled mean drifts and a frozen `theta` becomes stale. Re-estimating `theta` on a fresh holdout window (not on the peeking stream) is the clean response.

The sequential-test knob to watch is `tau`. Setting `tau` much larger than the true effect under-powers the test; setting it much smaller gives up optimality. `tau = sigma` on the adjusted outcome is a sensible default and is what the helper uses when `tau` is not specified.

## The take-home

CUPED and sequential testing reduce variance in different places: CUPED on the variance of a single estimate, mSPRT on the calendar time to a valid stop. They compose cleanly when `theta` is estimated once and then frozen. When the covariate is good and the infrastructure is in place, the compounded reduction in expected runtime is usually worth the warm-up step.

---

*Code and references: [case study 05](https://github.com/wavde/causal-inference-playbook/tree/main/case-studies/05-sequential-testing) · [experiment-toolkit](https://github.com/wavde/experiment-toolkit) · Deng, Xu, Kohavi, Walker (2013) · Johari, Pekelis, Walsh (2015, 2021)*
