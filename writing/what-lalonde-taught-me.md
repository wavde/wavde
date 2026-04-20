# What LaLonde taught me about propensity score matching

*Tags: causal inference, PSM, IPW, AIPW, NSW, replication*

---

The LaLonde (1986) / Dehejia-Wahba (1999) dataset is the lab rat of observational causal inference. You have a randomized benchmark — the National Supported Work (NSW) training program — which tells you the true effect on 1978 earnings is around **+$1,794** (NSW-treated vs NSW-control). Then you throw away the experimental control group, replace it with non-experimental controls from CPS or PSID, and ask: **can modern observational methods recover the experimental number?**

I wrote a replication script that runs four estimators against the NSW-treated + CPS-control composite and scored them:

| Estimator | Estimate | vs experimental ($1,794) |
|---|---|---|
| Naive mean difference | −$8,500 | Off by $10k with the wrong sign |
| PSM (1:1, caliper) | ~$1,600 | Within ~$200 |
| IPW (stabilized) | ~$1,750 | Within ~$50 |
| AIPW (doubly robust) | ~$1,800 | Within ~$10 |

That first row is the reason this dataset matters. Men in the NSW program were unemployed minorities with low earnings histories. CPS controls were the broader labor force. If you ignore selection and just subtract the means, **you conclude the training program destroyed $8,500 in earnings.** You'd shut it down.

## Three things that surprised me

### 1. How far off naive comparisons really are

I'd read "selection bias is a problem in observational studies" a hundred times. Writing the script and watching the terminal print `-8500` next to an experimental benchmark of `+1794` lands differently. It's not a 20% overshoot — it's a **sign flip of $10k magnitude.**

Every time someone shows me a +X% lift from a dashboard filter ("users who did Y saw +5% retention") without an experimental comparison, I now think of LaLonde.

### 2. AIPW is worth the extra line of code

AIPW is the "doubly robust" estimator — it combines a propensity model and an outcome model, and you get consistency if either one is right. It's two extra lines over IPW (fit a regression on controls, use it as a baseline) and it reliably comes closest to the truth.

IPW alone can be unstable when propensities get close to 0 or 1. You end up with a few observations with weights of 50 driving your entire estimate. AIPW pulls those in by leveraging the outcome model.

If you only implement one method from the playbook, implement AIPW. It's not more complicated — it's just less forgiving about which of your two models happens to be wrong.

### 3. Overlap is the whole ballgame

PSM only works if treated and control units overlap in propensity-score space. LaLonde's NSW-treated population is **nearly non-overlapping** with CPS — their propensities to have been in the treatment arm are near zero. If you don't trim or enforce common support, you're extrapolating a propensity model into regions where the data doesn't exist.

The fix is simple: drop controls below the 1st percentile of treated propensities and above the 99th. The number of dropped rows tells you something important about what you're doing — if you're dropping 80% of your control group, you should be nervous.

## What this means for practice

LaLonde convinced me of three rules I now follow:

1. **Always benchmark if you can.** If you have *any* randomized variant — even a prior A/B test on the same population — compare your observational estimate to it first. Trust no method that can't recover an experimental baseline.

2. **Always plot the propensity overlap.** If the histograms of treated and control propensities don't meaningfully overlap, no amount of statistical machinery saves you.

3. **Default to AIPW, not PSM.** Matching is intuitive and sells well in memos, but IPW and AIPW use all the data and are asymptotically more efficient. Reserve matching for when audience comprehension matters more than estimator variance.

---

*Code: [causal-inference-playbook / case-studies / 04-propensity-score / lalonde](https://github.com/wavde/causal-inference-playbook/tree/main/case-studies/04-propensity-score/lalonde)*

*References:*
- *LaLonde, R. (1986). "Evaluating the Econometric Evaluations of Training Programs with Experimental Data." AER.*
- *Dehejia, R. & Wahba, S. (1999). "Causal Effects in Non-Experimental Studies: Reevaluating the Evaluation of Training Programs." JASA.*
