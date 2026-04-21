# Finding (and fixing) a 17% Type-I error bug in an mSPRT implementation

_Drafted April 2026. Part of [experiment-toolkit](https://github.com/wavde/experiment-toolkit)._

## TL;DR

A first cut of an mSPRT (mixture Sequential Probability Ratio Test) always-valid p-value shipped in the package. It looked correct. Smoke tests passed. The formula matched my notes. Under the null, it still rejected at roughly **17% when the nominal alpha was 5%**. The bug was a factor-of-two in the prior-scaling term. This post walks through how the bug was caught and what changed to keep that class of error from shipping again.

## Why mSPRT

Classical A/B analysis assumes a single look at the end of the experiment. In practice, stakeholders peek at dashboards. Every unscheduled look is a multiple-testing opportunity that inflates Type-I error. mSPRT (Robbins, 1970; Johari, Pekelis, Walsh, 2017) provides a p-value that is valid at any stopping time, so peeking is safe.

The formula:

```
V = 2σ² / n
shrinkage = τ² / (V + τ²)
log Λ = 0.5 · log(V / (V + τ²)) + (δ̂² / 2V) · shrinkage
p = min(1, 1/Λ)
```

where `δ̂` is the observed effect, `σ` is the per-observation SD, `n` is per-arm sample size, and `τ` is the prior SD on the true effect.

## The bug

An earlier draft had the prior-scaling term as `τ²/2` instead of `τ²` in the `V + τ²` denominator. A small algebra slip. Three hand-picked unit tests did not catch it because the output was still a number in `[0, 1]` that moved in the expected direction as `n` grew.

## How it was caught

The test that caught it is a **Type-I error simulation** and now lives in CI:

1. Simulate many experiments under H0 (no true effect).
2. At each sample size along the trajectory, compute the mSPRT p-value.
3. For an always-valid test, the fraction of trajectories that *ever* dip below alpha should be at most alpha.

Against the buggy implementation, that fraction came out near **17% at alpha = 0.05**. The corrected implementation hovers at 3 to 4%, comfortably below alpha. Always-valid tests are conservative by construction; that gap is expected.

```python
# tests/test_toolkit.py
def test_msprt_always_valid_type1_error():
    rng = np.random.default_rng(0)
    n_trials = 2000
    n_max = 5000
    alpha = 0.05
    rejections = 0
    for _ in range(n_trials):
        # H0: both arms drawn from N(0, 1); no true effect.
        x = rng.normal(size=n_max)
        y = rng.normal(size=n_max)
        for n in range(100, n_max + 1, 100):
            delta_hat = x[:n].mean() - y[:n].mean()
            p = msprt_pvalue(delta_hat, sigma=1.0, n_per_arm=n, tau=0.05)
            if p < alpha:
                rejections += 1
                break
    assert rejections / n_trials <= alpha + 0.015  # small Monte Carlo slack
```

It now runs on every push.

## Lessons

1. **Known-answer tests catch typos. Property tests catch the bugs you're actually afraid of.** A "does the number look sensible?" test happily passed a 17%-false-positive implementation. A direct Type-I error simulation did not.
2. **For statistical code, write the test that would fail on the property you care about.** The worry here was losing the always-valid guarantee, so the test measures that property directly.
3. **Document the bug in the changelog.** See [`CHANGELOG.md`](https://github.com/wavde/experiment-toolkit/blob/main/CHANGELOG.md). Not because advertising a bug is fun, but because anyone running an older tag needs to know which release actually ships the corrected formula.

## The cost of peeking, as a reference

The same simulation harness against H0, with one-look and naive-peeking variants:

| Test | Type-I error |
|---|---|
| Fixed-horizon z-test (one look at n=5000) | ~5% |
| Naive z-test, peeking every 100 users | ~26% |
| mSPRT, peeking every 100 users | ~3% |

Peeking at the naive test roughly quintuples the false-positive rate. mSPRT holds the guarantee.

## Try it

```bash
pip install git+https://github.com/wavde/experiment-toolkit.git
experiment-toolkit sample-size --mde 0.02 --sd 1.0
```

Or read [the implementation](https://github.com/wavde/experiment-toolkit/blob/main/src/experiment_toolkit/sequential.py) and the [tests](https://github.com/wavde/experiment-toolkit/blob/main/tests/test_toolkit.py).

## References

- Robbins, H. (1970). *Statistical Methods Related to the Law of the Iterated Logarithm.*
- Johari, Pekelis, Walsh (2017). *Peeking at A/B Tests: Why it matters, and what to do about it.* (KDD)
- Howard, Ramdas, McAuliffe, Sekhon (2021). *Time-uniform, Nonparametric, Nonasymptotic Confidence Sequences.* (Annals of Statistics)
