# When TWFE lies: staggered rollouts and what Callaway-Sant'Anna buys you

*~6 min read. Companion code lives in [case 03 of the causal-inference-playbook](https://github.com/wavde/causal-inference-playbook/tree/main/case-studies/03-diff-in-diff) and `cs_staggered_att` in [experiment-toolkit](https://pypi.org/project/experiment-toolkit/).*

## The setup

A feature rolls out to three markets on different dates: A in March, B in July, C in November. The reflexive move is to pool the panel into a two-way fixed-effects regression,

```python
y ~ unit_fe + period_fe + treated_post
```

and read the `treated_post` coefficient as "the average treatment effect." For a long time, that was the default recipe.

Then Goodman-Bacon (2021) and de Chaisemartin & D'Haultfoeuille (2020) showed, from different angles, the same uncomfortable thing: when treatment effects vary across cohorts or evolve over time, the TWFE coefficient is a weighted average of underlying 2×2 DiDs, and some of those weights can be negative. Already-treated units act as *controls* for later-treated cohorts. A growing true effect in the March cohort can partly cancel a real effect in the November cohort.

This isn't a small correction. In a three-cohort simulation with true static effects of 1, 3, and 5, TWFE tends to return something near 1.8 when the equal-weighted truth is 3. That gap is big enough to flip which variant looks better in a writeup.

## What CS does differently

Callaway & Sant'Anna (2021) decompose the problem. Instead of one global coefficient, they estimate a grid of group-time average treatment effects,

$$
\widehat{ATT}(g, t) = \mathbb{E}[Y_t - Y_{g-1} \mid \text{cohort}=g] - \mathbb{E}[Y_t - Y_{g-1} \mid \text{never treated}]
$$

one number per (cohort `g`, period `t`) cell with `t >= g`. Each `ATT(g, t)` is a clean 2×2 DiD between cohort-`g` treated units and never-treated controls, anchored at period `g-1`. Because the control group is always the never-treated, early-cohort dynamics never leak into later-cohort estimates.

You then aggregate the grid however the question calls for it: by relative time (an event study), by cohort, or equal-weighted across the grid for a single overall number.

```python
from experiment_toolkit import cs_staggered_att, simulate_staggered_panel

panel = simulate_staggered_panel(seed=42)
res = cs_staggered_att(
    unit=panel["unit"], period=panel["period"],
    y=panel["y"], cohort=panel["cohort"], n_bootstrap=500,
)

print(res)
# CS(overall ATT=+3.02, 95% CI [+2.61, +3.43], n_units=75, boot=500)
```

Standard errors come from a unit-level cluster bootstrap: resample units with replacement and recompute every `ATT(g, t)` on each replicate. The pedagogy here is easier than the code. A proper CS standard error has to respect both within-unit correlation across time and the shared control group, and a cluster bootstrap handles both without asking you to write out an analytic sandwich.

## When you still don't need CS

If the rollout is a single on-date for everyone (a classic pre/post), TWFE and CS agree. CS matters when adoption is staggered. Randomised A/B tests don't need any of this.

## When CS still lies

CS rests on parallel trends between cohort `g` and the never-treated control, conditional on being in the baseline period. If the never-treated units drift for reasons unrelated to the intervention, no staggered-DiD estimator saves you. The usual responses are a synthetic control or a falsification test on the event-study pre-period coefficients.

CS also assumes no anticipation. If users in cohort `g` change behaviour before period `g` because they see what's coming, `g-1` is already contaminated and the baseline is off. In rollouts where treatment is invisible until it goes live, anticipation is usually negligible. In announced changes (pricing, policy), it is not.

## The take-home

TWFE isn't wrong because the algebra is wrong. It's wrong because the quantity it computes stopped matching what most readers mean by "the treatment effect" once staggered adoption became the norm. CS returns an interpretable number per cell, leaves the aggregation choice explicit, and, with a cluster bootstrap, fits comfortably into a single function call. For a DiD writeup against a staggered rollout, it's a better default.

---

*Code and references: [case study 03](https://github.com/wavde/causal-inference-playbook/tree/main/case-studies/03-diff-in-diff) · [experiment-toolkit](https://github.com/wavde/experiment-toolkit) · Callaway & Sant'Anna (2021) · Goodman-Bacon (2021) · de Chaisemartin & D'Haultfoeuille (2020)*
