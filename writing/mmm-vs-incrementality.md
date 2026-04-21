# MMM vs incrementality: what each answers, and what neither does

*~7 min read. Companion code in [case 03](https://github.com/wavde/paid-media-playbook/tree/main/case-studies/03-incrementality) and [case 04](https://github.com/wavde/paid-media-playbook/tree/main/case-studies/04-media-mix-modeling) of paid-media-playbook.*

## Two camps, one argument that never ends

Every paid-media measurement shop eventually sorts itself into two camps. The MMM camp builds a weekly time-series model that maps channel spend onto outcomes, reads off per-channel ROAS, and allocates next quarter's budget from the response curves. The incrementality camp runs ghost-ad and PSA-controlled conversion-lift tests on specific campaigns, reports a lift with a confidence interval, and trusts that far more than any regression coefficient.

Each camp thinks the other is answering the wrong question with the wrong tool. Both are right about the other, and both are wrong about themselves. The short version of what each does and what it hides is worth writing down.

## What MMM is actually computing

MMM fits a weekly (or daily) regression of outcome on transformed spend per channel. The transforms are the whole ball game: an adstock carryover $\alpha$ per channel (last week's spend still matters this week), and a Hill or log saturation with half-saturation $k$ and shape $s$ (the 11th dollar is less productive than the 10th). Plus a base trend, seasonality, macro covariates, and a linear channel coefficient $\beta$ that becomes the ROAS.

The case-04 reproducer generates 104 weeks of data for five channels under a known DGP — adstock decay, Hill half-saturation, and ROAS all specified per channel — and then fits it back two ways. A MAP optimiser (Nelder-Mead over 18 parameters) gets R² = 0.70. A Bayesian fit (PyMC, HalfNormal priors on $\beta$ and Beta priors on decay) gets a similar fit with posterior HDIs.

Here is the part that disappoints everyone at first read: the MAP fit's individual parameters are substantially off. SEM's fitted half-saturation comes back at 562k when the truth is 120k — a 4.7× error. Video's decay fits to 0.47 when the truth is 0.70. The model finds a point in parameter space with low squared error, but that point does not coincide with the ground truth, because different combinations of decay and half-saturation produce nearly identical weekly fits. This is the *adstock-vs-saturation identification problem*, and every honest MMM paper mentions it in the limitations section.

R² = 0.70 is a *real* fit on this data. The per-channel ROAS is still useful for ranking. Expecting the point estimates to land on the true parameters was the mistake.

## What the Bayesian fit earns its keep doing

The Bayesian version does not magically fix identification. It makes the failure *legible*. On the same data, the posterior HDI on $\beta_\text{video}$ should span roughly $[0.010, 0.080]$ — an 8× range. That is the identification problem showing up as uncertainty, which is the right place for it to show up.

The planning implication is enormous. At the top of video's HDI it is a top-three channel. At the bottom it is marginal. A MAP point estimate hides this. A posterior summary table forces the planner to price the uncertainty into the budget decision, instead of pretending the regression produced one number.

This is why "Bayesian MMM is overkill for my team" is usually wrong. The point is not the MCMC. The point is that the single-number MAP ROAS is understating uncertainty on exactly the channels where uncertainty matters most.

## What incrementality is actually computing

Conversion lift is a randomised experiment. One group sees the real ad; a matched group sees a PSA placebo (or is placed in a ghost-ad cell that tracks which users *would have* seen the ad). The lift is the conversion-rate delta. Under the usual randomisation assumptions, the lift is causal, full stop.

What it costs you is N. At a 0.5% base conversion rate and a target of +10% relative lift, the [case-03 reproducer](https://github.com/wavde/paid-media-playbook/tree/main/case-studies/03-incrementality) returns the required per-arm sample size as **327,922**. Rounded, that's 656k users for one channel's one campaign. For brands with that kind of traffic, fine. For most brands, that is one campaign per quarter — and then you still need to read the sensitivity.

The reproducer's readout on seed 0 returns +8.25% relative lift with a 95% CI of [+1.32%, +15.70%] and p = 0.0195. The CI is what the report should lead with. A single conversion-lift test is one draw from a wide sampling distribution. Two successive tests on the same campaign are routinely on opposite sides of zero.

## What neither answers

*What happens if we cut this channel entirely for a year?*

MMM can tell you the model's answer. The model has no data outside the historical range of spend, so the answer is an extrapolation of a Hill curve past where any observed point supported it. Not real.

Incrementality can run the year-long zero-spend experiment in principle, and no brand's paid-media budget survives it in practice.

The answer in between is the geo holdout. Pause the channel in 20% of DMAs for a quarter. Read the [geo-lift case study](https://github.com/wavde/paid-media-playbook/tree/main/case-studies/01-geo-lift) — it's the one piece of this measurement stack that can see what the MMM has to extrapolate to and what the conversion-lift test can't afford to run.

## The stack that actually works

None of these tools is the answer. The stack is the answer. The version I've seen earn credibility in front of finance:

1. **MMM quarterly**, Bayesian, with the HDIs visible to the planner. Drives next-quarter allocation.
2. **Two geo tests a year** on the two or three largest channels. Outputs an incremental lift with a CI. Used to calibrate the MMM: if MMM's ROAS disagrees with geo lift by more than 20%, the MMM is refit with a tighter prior on the affected channel.
3. **Matched-market incrementality** on always-on campaigns that have a plausible ghost-ad control. Reported with CIs, not point estimates. Used for campaign-level go/no-go.
4. **MTA (Markov, not last-touch)** as the weekly in-quarter dashboard. Reconciled against MMM quarterly.

The MMM camp and the incrementality camp both think the other is redundant. They are both wrong. MMM allocates. Incrementality validates. Geo calibrates. MTA makes the story visible week to week. Pull any one out and the stack loses something the remaining three cannot recover.

## The take-home

Ask of any measurement output: *what question is it answering, and what uncertainty is it hiding?* MMM answers the allocation question and hides parameter-level identification. Incrementality answers the single-campaign question and hides sampling noise inside a narrow CI. Geo answers the "what if we cut this" question and hides cluster-level variance. No tool in this family ships without a hidden cost. The job is not to pick a winner. The job is to run the combination where each tool's hidden cost is covered by another tool's honest answer.

---

*Code and references: [case 03 — incrementality](https://github.com/wavde/paid-media-playbook/tree/main/case-studies/03-incrementality) · [case 04 — MMM](https://github.com/wavde/paid-media-playbook/tree/main/case-studies/04-media-mix-modeling) · Jin, Wang, Sun, Chan, Koehler (2017) Bayesian MMM · Hanssens, Parsons, Schultz (2001) · Meta / Google conversion-lift methodology whitepapers*
