# Picking a north-star metric when every candidate has a tradeoff

*Tags: product analytics, metrics, Goodhart's law, memo*

The hard part of choosing a north-star metric is not deriving it from first principles. There is a standard interview answer ("active users who have completed a habitual action correlated with long-term retention") and a hundred blog posts that say the same thing in different words.

The hard part is that every reasonable candidate has a failure mode, and the job is to pick the one whose failure mode the team is most willing to live with.

## Four candidates, four failure modes

Consider a consumer product with accounts, content, and engagement, something with a subscription tier and a free tier. The metrics most analysts surface in the first hour:

| Metric | What it optimises for | Goodhart failure mode |
|---|---|---|
| DAU | Breadth of engagement | Re-engagement pings and notification spam that move the number without value |
| Total hours consumed | Depth of engagement | Autoplay and passive sessions; users kept passive rather than active |
| Weekly actives with ≥3 sessions on ≥2 days | Habit formation | Harder to explain; slower to move in an A/B test |
| Paid-tier conversion | Revenue | Team incentives collapse onto paywall design instead of product value |

None of these is wrong. All of them are defensible. Picking one at random usually means spending the next year optimising against its failure mode.

## Three questions that stress-test a candidate

### 1. What is the cheapest way to move the number by 10%?

If the cheapest path is an intervention the team would be embarrassed to ship, the metric is wrong. DAU can be moved by sending a daily 9am notification with "you have one new recommendation." If that notification moves DAU but not 30-day retention, the metric is gameable enough to matter.

Worth running that thought experiment before signing up for the metric, ideally with the engineering manager who will be on-call for whatever tactics end up moving it.

### 2. Does the metric drop when the product objectively gets worse?

Back-test against known regressions. During a known outage or a reversal experiment on a feature that tanked qualitative feedback, did the metric actually fall? If it was flat through known regressions, the metric is too lagging to steer by. The team learns it is in trouble only after the cost of recovery is large.

### 3. Who is angry when this metric is the top-line goal?

A metric that pleases every team is measuring nothing. A good north-star has explicit opponents:

- DAU makes the Quality team angry, because empty sessions count.
- Total hours makes wellbeing and responsible-design functions angry, because time-spent is not value.
- Habit-formation metrics make Growth angry, because they cannot hit them in a single quarter.

Pick the tradeoff whose opponent the team is willing to argue with in planning cycles. That is the one the organisation will actually defend.

## A reasonable default

For most consumer products, a "complex metric with a simple story" tends to work:

> Weekly active users who did the core action `N` or more times.

Choose `N` empirically: the threshold where the retention curve starts to flatten. It is technical enough to be hard to game, simple enough to fit on a slide, and tied to a leading indicator of long-term retention rather than the retention curve itself (which is too slow to iterate against).

## The part that is easy to miss

A north-star metric is not a permanent choice. Products change. DAU stops capturing value when the product gets quieter on purpose. Hours consumed stops capturing value when the format shifts to short-form. Paid conversion stops capturing value when the free tier becomes the acquisition channel.

The job is not to pick the right metric once. It is to notice when the failure mode that was accepted has started to cost more than the signal the metric still provides, and to swap it. That is a recurring conversation, not a one-time decision.

---

*Related: [product-analytics-deepdive / case-studies / 03-north-star-metric](https://github.com/wavde/product-analytics-deepdive/tree/main/case-studies/03-north-star-metric)*
