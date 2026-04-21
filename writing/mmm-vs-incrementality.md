# MMM vs incrementality: what each answers and what neither does

Every paid-media measurement program eventually runs into the MMM-vs-incrementality debate. Teams take sides. Neither side is wrong, because they are answering different questions.

This is the short version.

## What MMM answers

*"Given what we spent across all channels over the last two years, what is each channel's response curve — decay, saturation, and coefficient?"*

MMM fits a weekly time-series model: spend per channel → outcome, with adstock transforms to account for lagged response and a saturating function (Hill, log, or similar) to account for diminishing returns. The output is ROAS per channel and a response curve per channel. The reproducer in the [MMM case study](https://github.com/wavde/paid-media-playbook/tree/main/case-studies/04-media-mix-modeling) does exactly this on 104 weeks of synthetic data with five channels.

**What MMM is good for**: quarterly budget allocation. Sensitivity of ROAS to spend changes. Relative ranking of channels.

**What MMM is bad at**: single-campaign questions. Causal identification (the parameters are observationally identified, not experimentally). Anything that requires a narrow time window.

**What MMM hides**: the adstock-vs-saturation identification problem. Multiple parameter combinations produce similar model fits. The point estimate can be substantially wrong — the MAP reproducer fits R² = 0.70 but mis-estimates SEM's half-saturation by 4.7×. This is why a Bayesian fit matters: it shows the uncertainty MAP's single number hides.

## What incrementality answers

*"If we ran this specific campaign again, and ran a matched control without the campaign, how much more conversion would we get?"*

The [incrementality case study](https://github.com/wavde/paid-media-playbook/tree/main/case-studies/03-incrementality) covers the matched-market ghost-ad / PSA design. You show the real ad to one matched group, a PSA placebo to the other. The lift is the difference in conversion rate. Under the usual randomisation assumptions the lift is causal.

**What incrementality is good for**: one-campaign, one-decision questions. Validating channels. Detecting when MMM is wrong about a specific line.

**What incrementality is bad at**: cross-channel allocation. Anything that is not a single campaign. Low base-rate outcomes without large N (at a 0.5% base rate with +10% target lift, you need ~328k users per arm for 80% power — the reproducer computes this explicitly).

## What neither answers

*"What happens if we cut this channel entirely for a year?"*

MMM extrapolates — dangerously — beyond the historical range of spend. Incrementality can't observe a zero-spend regime for a year on a running business.

This is where geo holdouts help: you can observe the zero-spend counterfactual in a subset of DMAs, at a cost and duration that is less extreme than a national pause. See the [geo-lift case study](https://github.com/wavde/paid-media-playbook/tree/main/case-studies/01-geo-lift) and [essay on geo holdouts](geo-holdouts-as-paid-media-ab.md).

## The combined stack

A measurement program that works, in my experience:

1. **Quarterly MMM** drives baseline budget allocation and sensitivity analysis. Bayesian, not point estimates. Bring the uncertainty.
2. **One or two large geo tests per year** calibrate the MMM on the two or three most important channels. If the MMM's ROAS for a channel diverges from the geo's lift by more than 20%, something in the MMM is mis-specified — don't trust the other channels until you've fixed it.
3. **Matched-market incrementality** for any always-on campaign that has a plausible ghost-ad control. Report with CIs, not point estimates.
4. **MTA (Markov, not last-touch)** as the intra-quarter dashboard for channel-level trends. Calibrate quarterly against 1 and 2.

Any single one of these is defensible. The combination is what earns a measurement team credibility.

## Related

- [MMM case study](https://github.com/wavde/paid-media-playbook/tree/main/case-studies/04-media-mix-modeling) — MAP + Bayesian side-by-side
- [Incrementality case study](https://github.com/wavde/paid-media-playbook/tree/main/case-studies/03-incrementality) — sample-size planning, readout, sensitivity
- [Geo holdouts as paid-media A/B](geo-holdouts-as-paid-media-ab.md)
- [Last-touch systematic bias](last-touch-under-credits-upper-funnel.md)
