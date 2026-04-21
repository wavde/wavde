# Geo holdouts as paid-media A/B

You cannot randomise users into "saw the ad" and "did not see the ad" at the individual level without invasive infrastructure. You *can* randomise geographies. That is the whole idea behind geo holdouts, and it is the most underused measurement tool in paid media.

## The design

Pick a set of DMAs (or countries, or cities — the unit just has to be targetable in the ad platform). Randomly assign some to "treatment" (run the campaign as planned) and some to "holdout" (pause the campaign entirely). Measure the outcome — trial starts, activations, revenue — and compare.

This is a cluster-randomised A/B test. The unit of randomisation is the geo, not the user. That has consequences for statistical power (small N if you have few DMAs) and for what you can generalise to (you are measuring the lift on the population of *geos like these*, not a specific user segment). But it has the enormous advantage that the counterfactual is *real*: it is what actually happened in the holdout geos, not an assumption from a model.

## Synthetic control when you can't randomise

Paid-media teams don't always get to pause a campaign in half the country. When the campaign is already running, you can build a *synthetic control* from the weighted combination of donor DMAs that best matches the treatment DMA's pre-period outcome. Abadie-Diamond-Hainmueller (2010) is the original reference. SLSQP with unit-simplex constraints gets you the weights in a few lines of Python.

In the [geo-lift case study](https://github.com/wavde/paid-media-playbook/tree/main/case-studies/01-geo-lift), synthetic control on a 36-DMA donor pool recovers an 8% lift (truth: 8%) with a placebo-in-space p-value of 0.025. That's from 12 treated DMAs running for 12 weeks. If you run it for 24 weeks you get tighter CIs. If you run it on 40 DMAs you get tighter CIs again.

## MDE before the test

Always compute the minimum detectable effect *before* committing to a geo test. The MDE depends on the geo-level coefficient of variation, the number of treated + donor geos, and the number of pre/post weeks. If the MDE is 15% and you expect a 5% true lift, don't run the test — you will not detect anything and you will have wasted the campaign budget.

The MDE formula is in the case study's code. Rule of thumb for a productivity SaaS with 30+ DMAs and 12 weeks pre/post: MDE is typically 6–10%.

## Why this is better than MMM for big decisions

MMM gives you ROAS by channel. Geo gives you a single channel's incremental contribution with a confidence interval. The first answers "how do I allocate next quarter's budget?". The second answers "was this campaign worth running?".

Do MMM every quarter. Do geo tests for the two or three largest campaigns of the year. They answer different questions. Don't let the MMM team tell you the geo test is redundant, and don't let the geo team tell you the MMM is sloppy. They are complements.

## Related

- [Geo-lift case study](https://github.com/wavde/paid-media-playbook/tree/main/case-studies/01-geo-lift) — MDE, SLSQP synthetic control, placebo-in-space
- [MMM vs incrementality](mmm-vs-incrementality.md) — what each answers
