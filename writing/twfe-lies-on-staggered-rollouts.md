# When TWFE lies: staggered rollouts and what Callaway-Sant'Anna buys you

*~6 min read. Builds on [case 03 of the causal-inference-playbook](https://github.com/wavde/causal-inference-playbook/tree/main/case-studies/03-diff-in-diff) and the new `cs_staggered_att` in [experiment-toolkit v0.2.0](https://pypi.org/project/experiment-toolkit/).*

---

## The scenario that breaks two-way fixed effects

You're at a product company. Three markets turn on a feature in
different months — say market A in March, B in July, C in November. The
natural analyst's instinct is to throw the full panel into a two-way
fixed-effects (TWFE) regression:

```python
y ~ unit_fe + period_fe + treated_post
```

and read the `treated_post` coefficient as "the average treatment
effect." For a generation of applied economists, that was The Way.

Then Goodman-Bacon (2021) and de Chaisemartin & D'Haultfoeuille (2020)
happened. Both showed, in different ways, the same uncomfortable thing:
**when treatment effects differ across cohorts or dynamically over time,
the TWFE coefficient is a weighted average of 2×2 DiDs — and some of
those weights are negative.** Already-treated units end up acting as
*controls* for later cohorts, which means a growing true effect in
cohort A can *cancel out* a real effect in cohort C.

This isn't a small number. In the toy simulation that ships with
`experiment_toolkit.simulate_staggered_panel` — three cohorts with true
static effects of 1, 3, and 5 — TWFE estimates the "average effect" as
something like 1.8 when the equal-weighted truth is 3. It's not a tiny
bias you can wave away; it flips which feature looks better.

## What CS does differently

Callaway & Sant'Anna (2021) decompose the problem. Instead of one global
coefficient, estimate a *grid* of group-time average treatment effects:

$$
\widehat{ATT}(g, t) = \mathbb{E}[Y_t - Y_{g-1} \mid \text{cohort}=g] - \mathbb{E}[Y_t - Y_{g-1} \mid \text{never treated}]
$$

One number per (cohort `g`, period `t`) cell where `t >= g`. Each
`ATT(g, t)` is a clean 2×2 DiD between the cohort-`g` treated units and
the never-treated controls, using period `g-1` as the baseline. Because
the control group is the *never-treated* — never anyone already-treated —
you never mix early-cohort dynamics into later-cohort effects.

You then aggregate the grid however you like: by relative time (an
event study), by cohort, or equal-weighted across the whole grid for a
single overall number. The package ships all three:

```python
from experiment_toolkit import cs_staggered_att, simulate_staggered_panel

panel = simulate_staggered_panel(seed=42)
res = cs_staggered_att(
    unit=panel["unit"], period=panel["period"],
    y=panel["y"], cohort=panel["cohort"], n_bootstrap=500,
)

print(res)
# CS(overall ATT=+3.02, 95% CI [+2.61, +3.43], n_units=75, boot=500)

for row in res.event_study[:3]:
    print(row)
# {'relative_time': 0, 'att': 1.01, 'se': 0.18, 'ci_low': 0.66, 'ci_high': 1.36}
# {'relative_time': 1, 'att': 2.98, 'se': 0.22, 'ci_low': 2.55, 'ci_high': 3.41}
# ...
```

Standard errors come from a unit cluster bootstrap — resample *units*
with replacement and recompute every `ATT(g, t)` on each replicate. This
is one of those places the pedagogy is easier than the code: a proper
CS SE has to respect both the correlation across time within a unit *and*
the shared control group, and a clean cluster bootstrap handles both
without making you write out an analytic sandwich.

## When you still don't need CS

If your rollout is a single on-date for everyone (a classic 2×2
pre/post), TWFE and CS agree. CS is only necessary when you have
**staggered adoption**. The other ~90% of product experiments are
randomised A/B tests where none of this applies — reach for `msprt_pvalue`
instead.

## When CS still lies

CS relies on parallel trends between cohort `g` and the never-treated
control, *conditional on being in the baseline period*. If your
never-treated units drift differently — because they're the markets
where your product was always weaker — no staggered-DiD estimator will
save you. The remedy there is either a synthetic control (case study 06,
forthcoming) or a pre-trend falsification test on the event-study
pre-period coefficients.

Also: CS assumes no anticipation. If users in cohort `g` change
behaviour *before* period `g` because they know what's coming, period
`g-1` is already contaminated and your baseline is off. In a rollout
where treatment is invisible until it's live (most product launches),
anticipation is usually fine.

## The take-home

The default TWFE regression isn't wrong because the math is wrong. It's
wrong because the thing it computes stopped matching what we *mean* by
"the treatment effect" once staggered adoption became the norm. CS
gives you back an interpretable number per cell, leaves the aggregation
to you, and — thanks to a cluster bootstrap — fits comfortably in a
single numpy function. If you're writing a DiD memo in 2026 and your
rollout was staggered, reach for CS first.

---

*Code and references: [case study 03](https://github.com/wavde/causal-inference-playbook/tree/main/case-studies/03-diff-in-diff) · [experiment-toolkit](https://github.com/wavde/experiment-toolkit) · Callaway & Sant'Anna (2021) · Goodman-Bacon (2021)*
