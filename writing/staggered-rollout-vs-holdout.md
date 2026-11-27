# A staggered rollout can't see novelty: holdout vs rollout for opt-in AI features

*~6 min read. Companion code is [case 08](https://github.com/wavde/causal-inference-playbook/tree/main/case-studies/08-rollout-vs-holdout) in causal-inference-playbook, with [tests](https://github.com/wavde/causal-inference-playbook/blob/main/tests/test_rollout.py) and the [generated results](https://github.com/wavde/causal-inference-playbook/blob/main/case-studies/08-rollout-vs-holdout/results.md). It follows [Opt-in AI features: when an observational impact estimate is usable](opt-in-ai-adoption-observational-bias.md).*

## The recommendation

To measure an opt-in AI feature, keep a fixed holdout of users who don't get access for the whole measurement window. A staggered rollout, where some users get access a couple of weeks late, is cheaper per unit of precision, but it only measures the first weeks of exposure. AI features are exactly the kind whose effect changes as people learn the tool or as the novelty wears off. In a simulation where the true long-run effect fades to almost nothing, the staggered rollout ships the feature in **100.0%** of replications. A fixed holdout withholding the same share of traffic ships it in **4.7%**.

## The decision

The [opt-in post](opt-in-ai-adoption-observational-bias.md) ended with a rule: don't trust the adopter comparison; randomize access with a holdout or a staggered rollout. Both cost something, because users who don't have the feature yet are withheld traffic. With the same withheld budget, which design should a team run? And what does each one actually tell you?

## The setup

The simulation has 10,000 users per run, 4 weeks before launch and 8 after.

- **Users** have a stable activity level (SD 1.5) and weekly noise (SD 1).
- **Opt-in:** with access, heavier users are more likely to opt in, as in the opt-in post. The adoption rate is 29.9%.
- **Effect:** adopters gain an effect `tau(d)` in week `d` after getting access. There are four shapes:
  - **none**;
  - **flat**: 0.30 every week;
  - **learning**: grows toward 0.30;
  - **novelty**: starts at 0.40 and fades 40% a week.
- **Truth:** access is randomized, so the true intent-to-treat (ITT) effect is the adoption rate times `tau(d)`, known exactly.

Three designs each withhold 10% of post-launch user-weeks:

- **Fixed holdout:** 10% of users never get access during the 8 weeks.
- **Staggered rollout:** 40% of users get access 2 weeks late.
- **Hybrid:** 20% get access 2 weeks late, and 5% never get access.

Each design is analysed with CUPED-adjusted contrasts (using each user's pre-launch average) for the **short run** (weeks 0–1) and, where possible, the **long run** (weeks 6–7). A **shape test** compares late and early weeks between users with access and holdout users. It's 1,000 replications per cell, seed `20261127`, and `python case-studies/08-rollout-vs-holdout/src/run.py` reproduces it in about a minute.

![Holdout vs rollout](https://raw.githubusercontent.com/wavde/causal-inference-playbook/main/case-studies/08-rollout-vs-holdout/docs/hero.png)

## Every design recovers what it can see

Each estimate is centred on its truth, with coverage between 93.6% and 95.9%. The designs differ in what they can see and how precisely:

| Design | Short-run SE | Long-run SE | Long run identified? |
|---|---:|---:|---|
| Fixed holdout (10%) | 0.0285 | 0.0285 | yes |
| Staggered (40% two weeks late) | 0.0175 | — | no |
| Hybrid (20% late + 5% holdout) | 0.0198 | 0.0393 | yes |

*(SEs are from the flat-effect rows; other shapes are within 0.0002.)*

For the same withheld budget, the staggered rollout's short-run SE is 39% smaller than the holdout's. A 40/60 split for two weeks uses the budget more efficiently than a 10/90 split for eight. But once week 2 arrives, everyone has access, so no one is left to compare against in weeks 6–7.

## What that does to the launch decision

Suppose each design ships when its longest-horizon estimate is significantly positive: the long run if there's a holdout, the short run otherwise.

| Effect shape (true long-run ITT) | Fixed holdout | Staggered | Hybrid |
|---|---:|---:|---:|
| No effect (0) | 2.9% | 2.3% | 2.6% |
| Flat (+0.0897) | 90.9% | 99.9% | 63.1% |
| Learning (+0.0877) | 87.5% | 76.4% | 62.6% |
| Novelty (+0.0045) | **4.7%** | **100.0%** | 3.6% |

The staggered rollout is the best design when the effect is flat and the worst when it isn't:

- **Novelty:** the short-run effect is +0.0957 and the long-run effect is +0.0045. The staggered rollout measures the first number, very precisely, and ships every time.
- **Learning:** it underestimates the effect that will matter (short-run +0.0466 against long-run +0.0877), so it ships less often than the holdout.

The holdout's shape test catches the change: it flags the novelty decline in 76.7% of replications and the learning curve in 22.2%. A staggered rollout can't run that test at all.

## The hybrid is not a free lunch

The hybrid looks like the best of both: an early, precise read from the delayed group and a long-run read from a small holdout. It delivers the early read (short-run SE 0.0198) and avoids the novelty trap (3.6% ships). But at equal cost, its 5% holdout is half the size of the fixed holdout's, so its long-run SE is 0.0393, not 0.0285. It ships a real, flat effect only 63.1% of the time, against 90.9% for the fixed holdout, and it flags the novelty decline in 50.5% of replications instead of 76.7%. An early readout costs about a third of the long-run power.

## Assumptions and limitations

- **Synthetic, with one effect shape at a time.** Real features can mix learning and novelty, and adoption can ramp over weeks. Here users adopt as soon as they get access.
- **No interference.** If users with the feature affect users without it, as with collaborative tools or shared documents, both the holdout and the delayed group are contaminated.
- **Eight weeks is the long run here.** If the effect keeps changing after week 8, the holdout has to run longer, and its cost grows with its length.
- **A single significance test as the ship rule.** Teams that ship on an early read but commit to a long-run check would make fewer novelty mistakes than the staggered column shows. That check needs a holdout.
- **One budget (10%).** A bigger budget raises every design's power. It doesn't give a staggered rollout a long-run read.

## When the conclusion breaks

- **The effect really is flat**, and the team has evidence of it from earlier launches of similar features. Then the staggered rollout is the better design: it shipped the flat effect 99.9% of the time against 90.9%.
- **A long holdout is infeasible**, for example for legal, fairness or competitive reasons. Then the hybrid's small holdout is the minimum that still separates novelty from a lasting effect.
- **The decision is about the rollout itself,** for example operational readiness or early harm detection, not the long-run effect. A staggered rollout's precise short-run read is the right tool for that.

## What would change my mind

- **Evidence from past launches that AI-feature effects are flat after the first two weeks.** That would make the staggered rollout's efficiency the deciding factor.
- **A design that recovers the long run without a holdout**, for example by extrapolating a fitted effect curve from staggered cohorts at different durations. That would only work under strong assumptions about the curve's shape. With only one delayed group, as here, the data can't check those assumptions.

## The decision rule

1. **If the launch decision depends on the long-run effect, keep a fixed holdout for the whole window.** At equal cost it made the best launch call in every shape except flat, and it is the only design here that can test whether the effect changed.
2. **Don't ship an AI feature on a staggered rollout alone.** It measures the first weeks of exposure, which is where novelty lives. It shipped a fading effect every time.
3. **Use a hybrid only if an early readout is worth about a third of the long-run power.** At equal cost, its long-run SE was 0.0393 against 0.0285.

---

*Code, tests, and results: [`rollout.py`](https://github.com/wavde/causal-inference-playbook/blob/main/case-studies/08-rollout-vs-holdout/src/rollout.py) · [`run.py`](https://github.com/wavde/causal-inference-playbook/blob/main/case-studies/08-rollout-vs-holdout/src/run.py) · [`test_rollout.py`](https://github.com/wavde/causal-inference-playbook/blob/main/tests/test_rollout.py) · [`results.md`](https://github.com/wavde/causal-inference-playbook/blob/main/case-studies/08-rollout-vs-holdout/results.md) · References: Deng, Xu, Kohavi, Walker (2013), CUPED · Kohavi, Tang, Xu (2020), Trustworthy Online Controlled Experiments, on novelty and primacy effects and long-term holdouts · Hohnhold, O'Brien, Tang (2015), Focusing on the long-term: it's good for users and business.*
