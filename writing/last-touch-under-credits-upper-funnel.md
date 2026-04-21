# Last-touch attribution systematically under-credits upper funnel

*~5 min read. Companion code lives in [case 02 of paid-media-playbook](https://github.com/wavde/paid-media-playbook/tree/main/case-studies/02-mta-comparison).*

## The setup

A user sees a display ad on Monday. A video ad on Wednesday. On Friday they type your brand name into Google, click the sponsored result, and convert. Last-touch attribution credits brand-search with 100% of the conversion. Display gets zero. Video gets zero.

On a single user, that looks defensible. "The last thing they did was click brand-search, so that's what closed the sale." On a million users, it is the single largest source of channel-level mis-measurement I've watched drive paid-media budget decisions.

## The numbers

The [MTA reproducer](https://github.com/wavde/paid-media-playbook/tree/main/case-studies/02-mta-comparison) generates 10k user journeys against a known DGP: display truly drives 28% of conversions, video 18%, paid-search 22%, brand-search 14%, organic 10%, direct 8%. Last-touch reports:

| Channel | True share | LTA share | Error |
|---|---|---|---|
| display | 28% | **4.3%** | −24 pp |
| video | 18% | **2.3%** | −16 pp |
| brand-search | 14% | **25.3%** | +11 pp |
| direct | 8% | **21.8%** | +14 pp |

This is not a noisy Monte Carlo draw. The shape is structural: any channel whose job is to *create* intent gets its credit eaten by whichever channel *captures* the intent at the moment of conversion. Display and video create demand. Brand-search and direct reap it.

## Why it compounds into budget

Every channel has an apparent CPA computed off whatever attribution the stack reports. LTA hands brand-search a flattering CPA because brand-search is pocketing credit that display and video generated. Budget moves toward the "efficient" channel. Display and video keep their spend but lose their credit, so their apparent CPA gets worse. Budget moves further away.

A year later the brand-search line is over-budgeted, the display line is starved, and aggregate spend efficiency is visibly worse than where the brand started. The team runs a post-mortem. Nobody points at the attribution rule, because the attribution rule did not change; it was wrong from the start.

## Markov as the realistic alternative

The reproducer also runs Anderl et al. (2016) style Markov removal-effect on the same paths. For each channel, remove it from the graph, recompute the conversion probability from first-touch to the absorbing "converted" state, take the delta as that channel's contribution, then normalise.

On the same 10k users:

| Channel | True | LTA | Markov |
|---|---|---|---|
| display | 28% | 4.3% | **21.1%** |
| video | 18% | 2.3% | **15.0%** |
| brand-search | 14% | 25.3% | **17.4%** |

Markov still under-shoots display by ~7 pp because its first-order stationarity assumption smooths the real DGP's cohort dynamics. But the error band is now single-digit percentage points, not two-and-a-half decades.

Shapley would be axiomatic-correct, but at N=10k users and seven channels it is badly behaved. Coalition-level conversion rates for rare combinations (display-without-search) have noisy marginals and the method collapses credit onto the coalitions it can see. In the reproducer, Shapley dumps most of the credit on paid-search. It stabilises at N ≥ 250k users and ≤ 6 channels, which is a real constraint when the paid media team wants a weekly readout.

## What to do about it

Three moves, in order of cost:

1. **Report the delta.** Every LTA dashboard should carry a Markov column next to the LTA column. The first time a VP sees a 24 pp gap on display, the conversation is over.
2. **Never promote a channel on LTA alone.** If brand-search's LTA CPA looks great, that is consistent with (a) brand-search being genuinely efficient and (b) display paying the bill. You cannot distinguish those with LTA.
3. **Calibrate against an experiment.** A geo holdout or conversion-lift test on display once a year tells you which story is true. If incrementality confirms display is efficient and LTA says it isn't, the stack is wrong, not the channel.

## The take-home

LTA is not a neutral reporting choice; it is a *model* with a known directional bias. Reporting it without the Markov (or Shapley, or lift-test) companion is like reporting an A/B test's raw delta without a p-value. The number is there, the number is even technically correct, and the decision it induces is routinely wrong.

---

*Code and references: [case 02 — MTA comparison](https://github.com/wavde/paid-media-playbook/tree/main/case-studies/02-mta-comparison) · Anderl, Becker, von Wangenheim, Schumann (2016) · Shao & Li (2011) · Shapley (1953)*
