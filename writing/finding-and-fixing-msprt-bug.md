# Finding (and fixing) a 17% Type-I error bug in my own mSPRT implementation

_Drafted April 2026. Part of [experiment-toolkit](https://github.com/wavde/experiment-toolkit)._

## TL;DR

I shipped a first cut of an mSPRT (mixture Sequential Probability Ratio Test)
always-valid p-value. It looked correct — passed smoke tests, matched the
formula in my notes. Under the null, **it flagged false positives ~17% of the
time at a nominal α = 5%.** The bug was a factor-of-2 in the prior-scaling
term. This post walks through how I caught it and what I changed to keep it
from happening again.

## Why mSPRT

Classical A/B analysis assumes you look once at the end. Product
stakeholders peek at dashboards. Every peek is a multiple-testing opportunity
that inflates Type-I error. mSPRT (Robbins 1970, Johari-Pekelis-Walsh 2017)
gives you a p-value that is valid **at any stopping time**, so peeking is
safe.

The core formula is:

```
V = 2σ² / n
shrinkage = τ² / (V + τ²)
log Λ = 0.5 · log(V / (V + τ²)) + (δ̂² / 2V) · shrinkage
p = min(1, 1/Λ)
```

where δ̂ is the observed effect, σ is the per-observation SD, n is per-arm
sample size, and τ is the prior SD on the true effect.

## The bug

An earlier draft of mine had the prior-scaling term written as `τ²/2` instead
of `τ²` inside the `V + τ²/2` denominator. It's a small algebra slip — the
kind of thing unit tests on three hand-picked cases won't catch, because the
output is still a valid-looking p-value in [0, 1].

## How I caught it

I wrote a **Type-I error simulation** and made it a unit test. The test is:

1. Simulate many experiments under H0 (zero true effect).
2. At each sample size along the trajectory, compute the mSPRT p-value.
3. For an always-valid test, the fraction of trajectories that **ever** fall
   below α should be ≤ α.

Against the buggy implementation, this fraction came out near **17% for
α = 0.05**. The correct implementation hovers at 3–4% — comfortably below α
(always-valid tests are conservative, and that's the point).

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

This is the test that caught the bug. It now ships in the package and runs
on every CI push.

## Lessons

1. **Known-answer tests are table stakes; property tests are what actually
   catch subtle bugs.** A "does the number look sensible?" test would have
   passed a 17%-false-positive implementation. A Type-I error simulation
   doesn't.
2. **For statistical code, write the test that would fail on the bug you're
   afraid of.** I was afraid of getting the always-valid property wrong —
   so the test directly measures that property.
3. **Capture the bug in the CHANGELOG.** I documented this explicitly in
   [`CHANGELOG.md`](https://github.com/wavde/experiment-toolkit/blob/main/CHANGELOG.md).
   Not because I enjoy advertising a bug, but because future me (or future
   users) need to know that the formula shipped today is the corrected one.

## The cost of peeking

As a reference point, here's what naive peeking costs you under H0, on the
same simulation harness:

| Test | Type-I error |
|---|---|
| Fixed-horizon z-test (one look at n=5000) | ~5% ✅ |
| Naive z-test with peeking every 100 users | ~26% ❌ |
| mSPRT with peeking every 100 users | ~3% ✅ |

Peeking with the naive test quintuples your false-positive rate. With mSPRT
you can peek as often as you like and the Type-I guarantee holds.

## Try it

```bash
pip install git+https://github.com/wavde/experiment-toolkit.git
experiment-toolkit sample-size --mde 0.02 --sd 1.0
```

Or read [the implementation](https://github.com/wavde/experiment-toolkit/blob/main/src/experiment_toolkit/sequential.py)
and the [tests that validate it](https://github.com/wavde/experiment-toolkit/blob/main/tests/test_toolkit.py).

## References

- Robbins, H. (1970). *Statistical Methods Related to the Law of the Iterated
  Logarithm.*
- Johari, Pekelis, Walsh (2017). *Peeking at A/B Tests: Why it matters, and
  what to do about it.* (KDD)
- Howard, Ramdas, McAuliffe, Sekhon (2021). *Time-uniform, Nonparametric,
  Nonasymptotic Confidence Sequences.* (Annals of Statistics)
