# When boosting loses to SARIMA: baselines, MASE, and honest backtests

*~6 min read. Companion code lives in [demand-forecasting](https://github.com/wavde/demand-forecasting), with the full write-up at [wavde.github.io/demand-forecasting](https://wavde.github.io/demand-forecasting.html).*

## TL;DR

On a single daily retail series, a LightGBM with lag, rolling, and calendar features came in **last** — MASE 0.97, slightly *worse* than a seasonal-naive baseline — while a plain SARIMA reached **MASE 0.74, 18.5% below baseline**. That ordering is not a bug, and hiding it would have been the real mistake. It's what you should expect when you point a flexible learner at one rigid-weekly aggregate, and it's only visible because the evaluation used the right baseline (seasonal-naive), the right metric (MASE), and the right backtest (rolling-origin, no leakage).

## The series

UCI Online Retail II: a UK online gift retailer, cleaned and aggregated to one **daily total-units** series of 739 days (Dec 2009 – Dec 2011). Two features dominate it:

- **Hard weekly seasonality.** Saturday is a structural zero — the retailer doesn't process orders that day. That's a business rule, so forecasts for Saturday are forced to zero and the seasonal-naive baseline inherits the same zero from the prior week. The comparison stays fair on both sides.
- **Heavy-tailed volume.** Occasional large wholesale orders push the daily coefficient of variation to 0.86, versus 0.44 once you aggregate to weekly. Those spikes are close to unpredictable from calendar structure alone.

So the *signal* is overwhelmingly "what day of the week is it, and are we in the Christmas ramp?" — and that is exactly the signal a seasonal model captures for free.

## The result

Six-fold rolling-origin backtest, 28-day horizon, each model trained only on its own past:

```text
model            sMAPE     RMSE    MASE   MASE vs naive
SARIMA           29.67   6981.8   0.741        +18.5%
ETS              30.63   7174.3   0.773        +15.0%
seasonal_naive   37.73   8104.9   0.910          0.0%
LightGBM         36.77   8501.9   0.966         −6.1%
```

The two classical seasonal models (SARIMA, Holt-Winters ETS) beat the baseline comfortably and on every metric. The gradient-boosted model, with lags (1,2,3,7,14,21,28), rolling mean/std/min/max (7,14,28), and holiday flags, does not.

## Why this is the expected outcome, not a failure

It's tempting to read "LightGBM lost" as "the ML was undertuned." More often, on a problem like this, it's structural:

1. **One series is a data-poor regime for a flexible learner.** Boosting earns its keep by learning patterns *across* many related series — thousands of SKUs, stores, regions. Given a single aggregate with ~700 daily points, there isn't enough signal to justify its flexibility, so it spends variance modelling noise that SARIMA simply differences away with its $(1,1,1)(1,1,1)_7$ structure.
2. **Recursive multi-step forecasting compounds error.** The ML model forecasts 28 days out by feeding its own predictions back in as lags. Small early errors propagate. The state-space seasonal models project the horizon in one coherent shot.
3. **The signal is mostly seasonal, and seasonality is the one thing the baseline already nails.** When the baseline is strong, the marginal value a learner has to *add* is small and noisy.

None of this means boosting is bad. It means it was the wrong tool for *this* shape of problem — and saying so is more useful than a leaderboard where every model conveniently beats the baseline.

## What made the finding trustworthy

Three evaluation choices did the real work:

- **A seasonal-naive baseline, not a flat mean.** Against a naive last-value or a mean, almost anything looks brilliant. The honest bar for a weekly series is "last week, same day." Three of four models clearing it — and one failing to — is informative precisely because the bar is high.
- **MASE as the headline.** Mean Absolute Scaled Error divides your error by the in-sample seasonal-naive error, so **1.0 is the baseline by construction**: below 1 you've added value, above 1 you've subtracted it. It's scale-free and reads the same across series, unlike RMSE, and it doesn't blow up on the near-zero Saturdays the way MAPE would.
- **Rolling-origin cross-validation, leakage-checked.** Six consecutive 28-day windows, each model trained only on data before its window. Every lag and rolling feature is shifted at least one day, so a row dated *t* never sees *t* or later — a property that's unit-tested, not just asserted in prose. A single random train/test split would have leaked future seasonality into the past and quietly flattered every model.

## The causal bonus, and why it's labelled a bonus

The dataset has no marked promotion, so I didn't pretend to measure one from observational data. Instead a synthetic 24-store × 180-day panel is built from the real demand *shape*, a known **+15%** uplift is applied to treated stores, and a two-way fixed-effects DiD on log demand recovers it as **14.9% (95% CI [14.2%, 15.6%])**. It's a calibration check on the estimator against a ground truth I control — not a claim about the retailer — and it's framed that way on purpose.

## The take-home

The interesting result here isn't "SARIMA won." It's that a fair setup let the data say *which* model fits the regime, including the inconvenient answer that the fanciest one doesn't. Pick a baseline that's genuinely hard to beat, score against it with a metric centred on it, and back-test the way the model will actually be used. Do that, and a negative result on the ML model becomes a feature of the analysis rather than something to bury.

---

*Code, figures, and references: [demand-forecasting](https://github.com/wavde/demand-forecasting) · MASE: Hyndman & Koehler (2006) · rolling-origin evaluation: Hyndman & Athanasopoulos, *Forecasting: Principles and Practice*.*
