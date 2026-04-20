# E-values: the one-number sensitivity check every observational memo should have

*~5 min read. Builds on [case 04 of the causal-inference-playbook](https://github.com/wavde/causal-inference-playbook/tree/main/case-studies/04-propensity-score) and the new `e_value` helper in [experiment-toolkit v0.2.0](https://pypi.org/project/experiment-toolkit/).*

---

## The question every skeptical reviewer asks

You've done the observational analysis. Maybe propensity score matching,
maybe IPW, maybe a dose-response regression. You land on:

> "Users in cohort A have a 40% higher 90-day retention than cohort B,
> after matching on onboarding channel, signup device, and first-session
> activity."

Somewhere between draft 2 and the review meeting, the question shows
up: *"How big would the unmeasured confounder have to be to explain
this away?"*

You can wave your hands about "well, we controlled for everything we
could," or you can give them a number. The **E-value** (VanderWeele &
Ding, 2017, *Annals of Internal Medicine*) is that number.

## What it is

For an observed risk ratio `RR`, the E-value is defined as:

$$
\text{E-value} = RR + \sqrt{RR \cdot (RR - 1)}
$$

(with the convention that `RR < 1` is replaced by `1/RR` — the scale is
symmetric). Its interpretation is exact and beautifully conservative:

> An unmeasured confounder `U` would need to have a risk ratio of at
> least *E-value* with **both** the treatment and the outcome — above
> and beyond the measured covariates — to fully explain away the
> observed association.

Crucially, it makes **no assumption** about the distribution of `U`, its
direction, its binary-vs-continuous nature, or its functional form.
That's what makes it bulletproof for a memo: there's no
"well, but if you assume X" a skeptic can hang you on.

## Using it in a memo

```python
from experiment_toolkit import e_value

# Observed RR = 1.4 with 95% CI [1.15, 1.70]
result = e_value(point_estimate=1.40, lower_ci=1.15)

print(result)
# E-value (point)=2.159, E-value (CI)=1.559
```

What goes in the memo:

> The observed risk ratio of 1.40 has an E-value of 2.16 at the point
> estimate and 1.56 at the lower bound of the 95% CI. In plain English:
> an unmeasured confounder would need to be associated with both
> treatment assignment and outcome with a risk ratio of at least 1.56 —
> after accounting for the channel, device, and first-session controls —
> to move the lower CI bound to the null. That's a bigger association
> than any of our measured confounders have with either treatment or
> outcome, which makes the confounder story a hard sell.

The second sentence is the move. You're not just reporting a number,
you're benchmarking it against *the confounders you already measured*.
If your biggest measured confounder has RR 1.3 with the outcome, and
you'd need RR 1.56 for a hidden one, the reader has something concrete
to reason about.

## The three things people get wrong

**1. "E-value is the probability the result is real."** It isn't.
It's a threshold on confounder strength. A large E-value makes the
confounder story implausible but doesn't make the causal story true.

**2. Reporting only the point-estimate E-value.** Always also report
the CI-based E-value. An observed RR of 2.0 with a CI of [1.02, 3.9]
has a CI-based E-value of ~1.16 — the result is fragile at the lower
bound regardless of how impressive the point estimate looks.

**3. Stopping there.** E-values and Rosenbaum bounds (next section)
complement each other. E-values work on effect estimates; Rosenbaum
bounds work on hypothesis tests after matching. Use both.

## Rosenbaum bounds: the matched-pair version

When your design is **matched pairs** — as in PSM — there's an even
more mechanical alternative. Rosenbaum (2002) asks: by what odds ratio
`gamma` could a hidden confounder shift the probability of being the
treated member of a pair, and at what `gamma` does the one-sided
Wilcoxon signed-rank test stop rejecting?

```python
from experiment_toolkit import rosenbaum_gamma_threshold

paired_differences = treated_outcomes - control_outcomes
gamma_star = rosenbaum_gamma_threshold(paired_differences, alpha=0.05)
print(f"Robust up to gamma = {gamma_star:.2f}")
# Robust up to gamma = 1.85
```

A `gamma*` of 1.85 means: a hidden confounder would need to make one
member of each pair **1.85x more likely** than the other to be treated,
above and beyond the matching variables, for the result to stop being
statistically significant. In health and marketing studies, `gamma*`
values of 1.3–2.0 are considered moderately robust; >2 is strong;
>3 is unusually strong.

## The take-home

Observational causal claims live or die on the plausibility of the
no-unmeasured-confounding assumption. You can't prove it, but you can
*quantify the sensitivity* to its violation with two lines of code.
Make E-values (for effect estimates) and Rosenbaum `gamma*` (for
matched-pair tests) a standard row in your memo template. Your
reviewers will stop asking the question.

---

*Code and references: [case study 04](https://github.com/wavde/causal-inference-playbook/tree/main/case-studies/04-propensity-score) · [experiment-toolkit](https://github.com/wavde/experiment-toolkit) · VanderWeele & Ding (2017) · Rosenbaum (2002)*
