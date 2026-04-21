# LaLonde, revisited: what a replication tells you about PSM, IPW, and AIPW

*Tags: causal inference, PSM, IPW, AIPW, NSW, replication*

The LaLonde (1986) / Dehejia-Wahba (1999) dataset is the standard lab rat of observational causal inference. It has a randomised benchmark: the National Supported Work (NSW) training program, where the true effect on 1978 earnings is roughly **+$1,794** (NSW-treated vs NSW-control). The game is to throw away the experimental control group, replace it with non-experimental controls from CPS or PSID, and ask whether modern observational methods recover the experimental number.

Running four estimators against the NSW-treated + CPS-control composite:

| Estimator | Estimate | vs experimental ($1,794) |
|---|---|---|
| Naive mean difference | −$8,500 | Off by $10k, wrong sign |
| PSM (1:1, caliper) | ~$1,600 | Within ~$200 |
| IPW (stabilized) | ~$1,750 | Within ~$50 |
| AIPW (doubly robust) | ~$1,800 | Within ~$10 |

The first row is the reason the dataset exists. NSW participants were unemployed minorities with low prior earnings. CPS controls were the broader labour force. Ignoring selection and taking a raw mean difference concludes that a job-training program *destroyed* $8,500 in annual earnings. A decision-maker reading only that number shuts the program down.

## Three things worth sitting with

### Selection bias is not a 10% correction

Reading "selection bias is a problem in observational studies" is cheap. Watching an estimator print `-8500` next to a ground truth of `+1794` is less cheap. This isn't a margin-of-error story; it is a sign flip with an order-of-magnitude gap. Any dashboard-driven "users who did Y saw +5% retention" claim, made without an experimental comparison, should be read in that light.

### AIPW is worth the extra line of code

AIPW combines a propensity model with an outcome regression. It is consistent if either one is correctly specified. In practice on LaLonde, it lands closest to the experimental benchmark and is notably more stable than IPW alone.

IPW becomes unstable when propensities approach 0 or 1. A handful of observations with weights in the tens of thousands can dominate the estimate. The outcome-model term in AIPW pulls those cases back toward a sensible regression prediction, trading a little extra modelling work for a lot of variance reduction.

If a team implements only one estimator from the doubly-robust family, it should be AIPW. It is not meaningfully more complicated than IPW. It is meaningfully more forgiving about which of the two models happens to be wrong.

### Overlap is the whole game

PSM only works when treated and control units overlap in propensity-score space. On LaLonde/CPS, the NSW-treated distribution is nearly disjoint from the CPS controls: most CPS rows have propensities near zero. Without trimming to common support, the estimator is extrapolating a propensity model into regions the data does not cover.

A defensible rule is to drop control units below the 1st percentile and above the 99th percentile of the treated propensity distribution, then look at the share of controls dropped. If 80% of the control pool is discarded to enforce overlap, the population being estimated is no longer the one the memo claims.

## Operating rules

Three habits that survive this exercise:

1. **Benchmark when possible.** Any randomised variant, even a prior A/B test on the same population, is more useful than any sensitivity analysis. If the observational estimator cannot recover a known experimental answer on a relevant subpopulation, nothing else in the writeup matters.
2. **Plot propensity overlap before reporting an estimate.** If the treated and control histograms do not meaningfully overlap, no matching or weighting trick recovers a valid average treatment effect.
3. **Default to AIPW, not PSM.** Matching is intuitive and communicates well, but IPW and AIPW use all the data and are asymptotically more efficient. Matching belongs where audience comprehension outweighs the variance cost.

---

*Code: [causal-inference-playbook / case-studies / 04-propensity-score / lalonde](https://github.com/wavde/causal-inference-playbook/tree/main/case-studies/04-propensity-score/lalonde)*

*References:*
- *LaLonde, R. (1986). Evaluating the Econometric Evaluations of Training Programs with Experimental Data. AER.*
- *Dehejia, R. & Wahba, S. (1999). Causal Effects in Non-Experimental Studies: Reevaluating the Evaluation of Training Programs. JASA.*
