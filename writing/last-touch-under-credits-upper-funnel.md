# Last-touch attribution systematically under-credits upper funnel

Last-touch attribution (LTA) is still the reporting default in most ad-measurement stacks. It is also the single largest source of channel-level mis-measurement I have repeatedly watched drive paid-media budget decisions.

This essay is the short version of what the [MTA case study](https://github.com/wavde/paid-media-playbook/tree/main/case-studies/02-mta-comparison) in `paid-media-playbook` is designed to make concrete: LTA is not a neutral reporting choice. It systematically under-credits display and video, and systematically over-credits brand-search and direct. The magnitude matters.

## What LTA actually does

LTA assigns 100% of conversion credit to the last paid touch in the user journey. On paper that is a well-defined rule. In practice, a user who sees a display ad on Monday, a video ad on Wednesday, searches your brand on Friday and converts, gets fully credited to brand-search. Display and video get zero.

That is the mechanism. It looks mild on a single user. It is not mild in aggregate.

## The systematic bias

In the reproducer, with a generative process where display truly drives 28% of the lift and brand-search truly drives 14%, LTA reports:

- brand-search: **25.3%** (true 14%) — over-credit of +11 pp
- direct: **21.8%** (true 8%) — over-credit of +14 pp
- display: **4.3%** (true 28%) — under-credit of −24 pp
- video: **2.3%** (true 18%) — under-credit of −16 pp

This is not noise. It is the structural consequence of measuring credit at the end of the journey. Any channel whose role is to create intent (display, video, sometimes paid-social) will be mis-reported. Any channel whose role is to capture intent (brand-search, direct) will be over-reported.

## Why it matters for budget

Every channel has a CPA. LTA gives brand-search a CPA that looks lower than it truly is, because brand-search is receiving credit for demand other channels created. Budget moves toward the "efficient" channel. The demand-creating channels' apparent CPA gets worse (they have lost credit but kept their spend). Budget moves further away from them. The feedback loop ends with an over-budgeted brand-search line and an under-budgeted display line — and a total-spend efficiency that is silently worse than where the brand started.

## What to reach for instead

Markov removal-effect is the method to trust at realistic sample sizes. In the same reproducer, with the same data, Markov puts display at 19.2% (vs truth 28%) and video at 12.8% (vs truth 18%). It still under-shoots — but by ~8 pp rather than ~24 pp.

Shapley is axiomatic-correct but noisy at N=10k users. At N ≥ 250k with a small channel set (< 6), it stabilises.

The general rule: if you have the paths, run Markov alongside LTA, expose the delta on a channel-level dashboard, and never let LTA be the sole input to a budget decision.

## Related

- [MTA case study](https://github.com/wavde/paid-media-playbook/tree/main/case-studies/02-mta-comparison) — code, full output, method comparison
- [When to use which paid-media method](https://github.com/wavde/paid-media-playbook/tree/main/case-studies/00-method-selection) — decision matrix
