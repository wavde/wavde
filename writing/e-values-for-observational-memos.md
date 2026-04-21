# E-values: a one-number sensitivity check for observational memos

*~5 min read. Companion code in [case 04 of the causal-inference-playbook](https://github.com/wavde/causal-inference-playbook/tree/main/case-studies/04-propensity-score) and `e_value` / `rosenbaum_gamma_threshold` in [experiment-toolkit](https://pypi.org/project/experiment-toolkit/).*

## The question every observational memo has to answer

You've run the observational analysis. Propensity-score matching, IPW, a dose-response regression, pick your flavour. The headline reads something like:

> Users in cohort A have 40% higher 90-day retention than cohort B, after matching on onboarding channel, signup device, and first-session activity.

The question the reviewer asks, sooner or later: *how strong would an unmeasured confounder have to be to explain this away?* Handwaving about "we controlled for everything we had" is not an answer. An E-value is.

## Definition

For an observed risk ratio `RR >= 1`,

$$
E = RR + \sqrt{RR \cdot (RR - 1)}
$$

(for `RR < 1`, apply the formula to `1/RR`; the scale is symmetric). The interpretation, from VanderWeele & Ding (2017), is the strong part:

> An unmeasured confounder `U` would have to be associated with both treatment assignment and the outcome with a risk ratio of at least `E`, above and beyond the measured covariates, to fully explain away the observed association.

The result assumes nothing about the distribution of `U`, its functional form, or whether it is binary or continuous. That is what makes it a workable number in a memo: no "well, but if you assume X" that a reviewer can hang an objection on.

## Using it

```python
from experiment_toolkit import e_value

# Observed RR = 1.40, 95% CI [1.15, 1.70]
result = e_value(point_estimate=1.40, lower_ci=1.15)
print(result)
# E-value (point)=2.16, E-value (CI)=1.56
```

What the memo paragraph then looks like:

> The observed risk ratio of 1.40 has an E-value of 2.16 at the point estimate and 1.56 at the lower bound of the 95% CI. An unmeasured confounder would need to be associated with both cohort assignment and the outcome with a risk ratio of at least 1.56, after accounting for the channel, device, and first-session controls, to move the lower bound to the null. The strongest measured confounder in this analysis has an outcome RR of roughly 1.3, so a hidden confounder at that scale would not be enough.

The benchmark sentence is the part that earns the paragraph its keep. Reporting the E-value in isolation is easy to dismiss. Reporting it next to the strongest measured confounder gives the reader something concrete to compare against.

## Common mistakes

1. **Treating `E` as a probability that the effect is real.** It isn't. It's a threshold on confounder strength. A large E-value makes the confounding story implausible; it does not prove the causal story.
2. **Reporting only the point-estimate E-value.** Always report the CI-based E-value as well. An RR of 2.0 with a CI of [1.02, 3.9] has a CI-based E-value near 1.16, which is fragile regardless of how impressive the point estimate looks.
3. **Stopping at one sensitivity statistic.** E-values and Rosenbaum bounds answer different questions. Use both where the design supports it.

## Rosenbaum bounds: the matched-pair version

When the design is matched pairs (PSM), Rosenbaum (2002) gives a more mechanical alternative. For each odds ratio `gamma`, ask: could a hidden confounder at that strength flip the result of a one-sided Wilcoxon signed-rank test? Find the smallest `gamma` at which the test no longer rejects.

```python
from experiment_toolkit import rosenbaum_gamma_threshold

paired_differences = treated_outcomes - control_outcomes
gamma_star = rosenbaum_gamma_threshold(paired_differences, alpha=0.05)
print(f"Robust up to gamma = {gamma_star:.2f}")
# Robust up to gamma = 1.85
```

A `gamma* = 1.85` means: to overturn significance, a hidden confounder would have to make one member of each matched pair roughly 1.85x more likely than the other to have been treated, beyond the matching variables. In applied work, values of 1.3 to 2.0 are usually called moderately robust; above 2 is strong; above 3 is unusually so.

## Where sensitivity analysis stops helping

Both tools quantify robustness to a specific class of violation: unmeasured confounding. They do not help with model misspecification, selection bias in who ends up in the analysis set, or measurement error in the treatment variable. If the concern in the room is that the cohorts are fundamentally different populations doing different things, a bigger E-value does not resolve it. That conversation belongs upstream, in the matching design and the common-support diagnostics.

## The take-home

Observational causal claims live or die on the plausibility of no-unmeasured-confounding. The assumption can't be proved, but its sensitivity can be quantified. Putting an E-value (for effect estimates) and, where applicable, a Rosenbaum `gamma*` (for matched-pair tests) next to every observational point estimate pushes a predictable reviewer question into the body of the memo, with a number attached.

---

*Code and references: [case study 04](https://github.com/wavde/causal-inference-playbook/tree/main/case-studies/04-propensity-score) · [experiment-toolkit](https://github.com/wavde/experiment-toolkit) · VanderWeele & Ding (2017) · Rosenbaum (2002)*
