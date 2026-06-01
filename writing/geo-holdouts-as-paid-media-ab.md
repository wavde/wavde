# Geo holdouts as paid-media A/B

*~6 min read. Companion code lives in [case 01 of paid-media-playbook](https://github.com/wavde/paid-media-playbook/tree/main/case-studies/01-geo-lift).*

## The thing individual-level randomisation can't do

You cannot randomly assign a user into "saw the ad" and "did not see the ad" without cooperation from every ad platform the user touches. The platforms will not cooperate. Even if they did, cross-device journeys would leak treatment across arms, and post-ATT on iOS there is no stable identifier to randomise on in the first place.

So you randomise the geography instead. Pick a set of DMAs (or countries, or cities — the unit just has to be targetable in the ad platform). Randomly assign some to "treatment" (campaign runs) and some to "holdout" (campaign paused). Measure the outcome. The lift is the ATT.

This is a cluster-randomised trial. It is the cleanest thing a paid-media team can run, and it is dramatically underused, because pausing a campaign across half of your DMAs feels — to the marketer who commissioned it — like leaving money on the table. Occasionally it is. More often the money was already being left somewhere, and the test is how you find out.

## The numbers you actually need before you commit

The question every planner asks second (after "can we do it?") is "how big does the test have to be?" The MDE calc is the part worth doing on the back of a napkin *before* agreeing to run anything.

For a balanced geo split with $n_t$ treated and $n_c$ control DMAs, pre/post weekly outcome panel, and DMA-level coefficient of variation CV, the approximate two-sided MDE at 80% power and $\alpha = 0.05$ is

$$
\text{MDE} \approx 2.8 \cdot \text{CV} \cdot \sqrt{\tfrac{1}{n_t} + \tfrac{1}{n_c}} \cdot \tfrac{1}{\sqrt{W}}
$$

where $W$ is the number of post weeks. For a productivity SaaS with 40 DMAs split 12/28, CV ≈ 0.25, and a 12-week post period, the MDE comes in at roughly 7%. If the campaign's expected lift is 5%, you will spend the money, end with a wide CI straddling zero, and learn nothing. Do not run it. Either extend the window, use more DMAs, or wait for a bigger campaign.

## Synthetic control when the randomisation didn't happen

The campaign has been running for six months across all 210 DMAs. You were not in the room when it launched. You cannot rewind and add a holdout arm. You still need a measurement.

Abadie, Diamond, Hainmueller (2010) gives you a way. Let the treated unit be a specific DMA (or a group of them). Build a *synthetic control* as a convex combination of donor DMAs whose weights minimise the pre-period outcome gap. The constraint is a unit simplex — weights non-negative and summing to one. Solve with SLSQP. The post-period gap between the treated and synthetic unit is the ATT estimate.

The case-01 reproducer runs this on a 36-DMA donor pool with 12 treated DMAs over 12 pre / 12 post weeks and a true lift of 8%. On seed 0 it returns ATT = +44.91 per DMA-week (+9.6%), with donor weights concentrated on dma_04 (0.31), dma_35 (0.30) and dma_07 (0.15). Placebo-in-space — running the same synthetic-control fit on every donor DMA as a pseudo-treatment — yields an empirical p-value of 0.025.

Two things to notice. First, the point estimate overshoots the truth (9.6% vs 8%) by more than the simulation's nominal noise; placebo p-values from a donor pool that small are lumpy, and a single test with 12 treated DMAs is noisier than the MDE formula above suggests. Second, the donor weights are sparse — two DMAs carry 60% of the synthetic. This is the method being honest: it found two donors that look like the treated unit pre-period, and it is betting on them post-period. If one of those two donors catches an idiosyncratic shock, the ATT is wrong. Worth knowing.

## When geo beats MMM and when it doesn't

MMM quarterly gives you a ROAS curve per channel, which is the right object to allocate budget against. Geo quarterly is impossible to run at the per-channel grain, because you cannot independently pause each of five channels in each of three cities without the combinatorics blowing up the DMA budget.

Running geo once or twice a year gives you causal lift estimates for the biggest campaigns, with real counterfactuals. That is the right object to *calibrate* MMM against. If the MMM reports display at ROAS 2.0 and a geo test on display returns an incremental return of 3.5, the MMM's decay and half-saturation parameters for display are wrong, and the rest of its ROAS estimates are suspect until you refit with the geo lift as an informative prior.

The cleanest pairing I've run:

1. MMM every quarter. Bayesian, with HDIs visible to the consumer.
2. Two geo tests a year on the largest campaigns.
3. Calibrate MMM against the geo lifts. Where they disagree by more than 20%, the MMM gets refit with a tighter prior on the affected channel's ROAS.
4. [MTA](https://github.com/wavde/paid-media-playbook/tree/main/case-studies/02-mta-comparison) (Markov, not last-touch) for in-quarter dashboarding. Reconcile against MMM quarterly and against geo annually.

Any single leg is defensible. The triangulation is what earns the team credibility in front of finance.

## The take-home

The reflex in most organisations is to treat "we can't randomise users" as the end of the causal story. It isn't. Randomise the geo instead; compute MDE honestly before committing; and when even a holdout is off the table, synthetic control gets you a defensible lift with placebo inference. What it costs you is one campaign's worth of paused spend on a handful of markets. What it buys is an answer with a confidence interval, against a real counterfactual, for the two or three budget decisions a year that matter most.

---

*Code and references: [case 01 — geo lift](https://github.com/wavde/paid-media-playbook/tree/main/case-studies/01-geo-lift) · Abadie, Diamond, Hainmueller (2010) · Doudchenko & Imbens (2016) · companion synthetic-control case in [causal-inference-playbook](https://github.com/wavde/causal-inference-playbook/tree/main/case-studies/02-synthetic-control)*
