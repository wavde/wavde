# Picking a north-star metric when every candidate has a tradeoff

*Tags: product analytics, metrics, goodhart's law, memo*

---

The thing nobody tells you about north-star metrics is that the hard part isn't deriving them from first principles. There's a standard interview answer — "active users who've completed a habitual action that correlates with long-term retention" — and a hundred blog posts by PMs at growth-stage companies that say the same thing.

The hard part is that **every candidate metric has a failure mode**, and the job is to pick the one whose failure mode you're most willing to accept.

## The four candidates I usually see

Take a consumer product with accounts, content, and engagement — a Spotify-shaped business, say. Here are the metrics I'd expect any senior analyst to surface in the first hour:

| Metric | What it optimizes for | Goodhart failure mode |
|---|---|---|
| **DAU** | Breadth of engagement | Notification spam, re-engagement emails that inflate the metric without value |
| **Total hours listened** | Depth of engagement | Auto-play on background; encourages keeping users passive rather than active |
| **Weekly active listeners with ≥3 sessions on ≥2 days** | Habit formation | Complex; hard to explain to exec; hard to A/B against |
| **Paid-tier conversion** | Revenue | Short-circuits product value; ties team incentives to paywall design |

None of these is wrong. They're all defensible. And if you pick one at random, your team will spend the next year optimizing against its failure mode.

## The framework that actually works

I've landed on three questions to stress-test a proposed metric:

### 1. What's the cheapest way to move this number by 10%?

If the cheapest path is something you'd be embarrassed to ship, the metric is wrong. DAU can always be juiced by sending a daily notification at 9am with "you have 1 new recommendation." If that notification moves DAU but not 30-day retention, you've confirmed the metric is gameable.

Run this thought experiment on every candidate before shipping it. Better yet, run it with the engineering manager who'll be on-call for the tactics that move it.

### 2. Does this metric drop when the product objectively gets worse?

Test the metric against historical regressions. When your team shipped a bug, did the metric fall? When you ran a reversal experiment on a feature that tanked NPS, did the metric fall?

If the metric was flat during known regressions, it's too lagging. You'll only learn you're in trouble after it's expensive to correct.

### 3. Who is angry when this metric is the top-line goal?

A metric that pleases everyone is measuring nothing. A good north-star has clear opponents:
- DAU makes the Quality team angry (empty sessions count)
- Total hours makes the Wellbeing team angry (time-spent isn't value)
- Habit-formation metrics make Growth angry (they can't hit it in a quarter)

Pick the tradeoff whose opponent you're most willing to fight. That's the one you'll defend consistently in planning cycles.

## What I actually recommend

For most consumer products, I default to what I call a **"complex metric with a simple story"**:

> *Weekly active users who did the core action ≥ N times.*

Choose N empirically — the threshold where the retention curve starts to flatten. For Spotify, that might be "users who listened to ≥3 sessions this week." For Duolingo, "completed ≥2 lessons on ≥2 days."

The metric is technical enough to be hard to game, simple enough to fit on a slide, and — critically — **tied to a leading indicator of long-term retention** rather than the retention curve itself (which is too slow to iterate against).

## The part I was wrong about for a long time

I used to think picking a north-star was a one-time decision. Every company I've worked at has had to change theirs. DAU stopped capturing value when the product got quieter. Hours-watched stopped capturing value when the team started optimizing for short-form. Paid-tier conversion stopped capturing value when the free tier was the acquisition channel.

The job isn't to pick the right metric forever. It's to **notice when the failure mode you accepted has started to cost more than the signal you get**, and to swap it. That's a quarterly conversation, not a quarterly review item.

---

*Related: [product-analytics-deepdive / case-studies / 03-north-star-metric](https://github.com/wavde/product-analytics-deepdive/tree/main/case-studies/03-north-star-metric)*
