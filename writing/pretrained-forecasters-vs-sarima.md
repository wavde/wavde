# Pretrained forecasters vs SARIMA: when zero-shot wins, and when it doesn't

*~7 min read. Companion code lives in [demand-forecasting](https://github.com/wavde/demand-forecasting): the [comparison script](https://github.com/wavde/demand-forecasting/blob/main/run_pretrained.py), the [model wrapper](https://github.com/wavde/demand-forecasting/blob/main/src/demand_forecasting/pretrained.py), [tests](https://github.com/wavde/demand-forecasting/blob/main/tests/test_pretrained.py), and the [generated results](https://github.com/wavde/demand-forecasting/blob/main/docs/pretrained_results.md).*

## The recommendation

With a year or two of daily history on a strongly seasonal series, keep SARIMA in production. A zero-shot pretrained forecaster doesn't beat it, whether SARIMA uses a fixed order or one re-selected by AIC on every fold, and it loses most in the peak-season window, which is the one that matters most. Where a pretrained model earns a place is **short history**: with only the last 56 days of data it beat both SARIMAs on both series tested, and its accuracy barely moved as history shrank while SARIMA's degraded. Re-tuning SARIMA on short history didn't close that gap; on UK it made things worse. Use it as the default for new or cold-start series and as a strong extra baseline, and score it against seasonal-naive with MASE under the same rolling-origin backtest as everything else.

## The decision

Time-series foundation models promise forecasts with no training, tuning or feature engineering. A team that already runs a tuned SARIMA has to decide whether to switch, add the new model alongside it, or ignore it. The question is narrow: under the backtest the team already trusts, does the pretrained model forecast better, and under what conditions?

## The setup

The comparison reuses the backtest from [When boosting loses to SARIMA](when-boosting-loses-to-sarima.md) and changes nothing else:

- **Data.** UCI Online Retail II, daily units from December 2009 to December 2011 (739 days). Two series: `UK`, which is the main backtest series, and `Rest of world`, which is every other country combined and is smaller and noisier.
- **Backtest.** Six rolling-origin folds with a 28-day horizon and an expanding training window. The folds are identical for every model, and Saturday forecasts are forced to zero because the retailer doesn't trade on Saturdays.
- **Metric.** MASE, scaled by each fold's in-sample seasonal-naive error. Below 1 beats the baseline.
- **Models.** Seasonal-naive; SARIMA(1,1,1)(1,1,1)₇ with the same fixed order as before; `SARIMA-tuned`, which re-selects (p,q)(P,Q) from {0,1,2}×{0,1,2}×{0,1}×{0,1} by AIC on each fold's training data only, with d = D = 1 so the AICs are comparable; ETS; LightGBM; and **Chronos-Bolt** (`amazon/chronos-bolt-small`, pinned to a fixed commit). Chronos-Bolt is used zero-shot: pretrained weights, no fine-tuning, and each fold's training history as its only context. The point forecast is the median quantile.

One command reproduces it: `pip install -r requirements-pretrained.txt && python run_pretrained.py`. The seed is 7, it runs on CPU, and it takes about eight minutes, mostly the SARIMA order search. Chronos-Bolt predicts quantiles in a single forward pass with no sampling, so two runs give identical output. The results file records the exact model commit and library versions.

![Pretrained forecaster vs SARIMA](https://raw.githubusercontent.com/wavde/demand-forecasting/main/docs/pretrained_vs_sarima.png)

## The result with full history

| Model | UK MASE | Rest of world MASE |
|---|---:|---:|
| SARIMA | **0.741** | **0.584** |
| SARIMA-tuned | 0.748 | **0.584** |
| ETS | 0.773 | 0.605 |
| Chronos-Bolt | 0.784 | 0.589 |
| Seasonal-naive | 0.910 | 0.907 |
| LightGBM | 0.966 | 0.758 |

With zero configuration, Chronos-Bolt beats seasonal-naive by 13.9% on UK and 35.1% on rest of world, so it isn't a weak model. But on UK its MASE is 5.8% above SARIMA's, and it has the lower error in only 1 of 6 folds. On rest of world the gap is 0.8%, with 3 wins in 6 folds. That is a tie. Re-tuning SARIMA on every fold changes little: 0.748 on UK, where Chronos-Bolt beats it in 2 of 6 folds, and 0.584 on rest of world, where it wins 3 of 6. With full history AIC mostly picked (2,1,2) with a seasonal (0,1,1) or (1,1,1), and the fixed order was already close to that.

The UK average hides where the gap comes from. In five of six folds the two models are within 0.05 MASE of each other. Excluding the last fold, both average about 0.68. The last fold, 12 November to 9 December 2011, is the Christmas ramp. There SARIMA scores 1.041, the tuned SARIMA 1.031, and Chronos-Bolt 1.301. On this data, zero-shot loses mostly in the window a retailer most needs to get right. The SARIMA has two years of this retailer's own seasonality to work from. The pretrained model has to infer the ramp from the context window alone.

By lead time, the picture is mixed rather than clean. On UK, Chronos-Bolt does better than SARIMA only 15–21 days ahead (0.626 vs 0.686) and is much worse 8–14 days ahead (0.735 vs 0.564). On rest of world, it's worse 1–7 days ahead (0.750 vs 0.683) and slightly better at every longer lead time.

## When history is short

The more useful finding comes from giving every model only the last N days of history. The folds and test windows stay the same, and MASE is still scaled on the full training split so the rows are comparable:

| History | UK SARIMA | UK SARIMA-tuned | UK Chronos-Bolt | RoW SARIMA | RoW SARIMA-tuned | RoW Chronos-Bolt |
|---|---:|---:|---:|---:|---:|---:|
| Full | **0.741** | 0.748 | 0.784 | **0.584** | **0.584** | 0.589 |
| 365 days | **0.721** | 0.741 | 0.761 | 0.612 | 0.601 | **0.594** |
| 112 days | **0.758** | 0.852 | 0.811 | 0.678 | 0.650 | **0.589** |
| 56 days | 0.850 | 1.048 | **0.729** | 0.737 | 0.702 | **0.600** |

SARIMA gets worse as history shrinks: on rest of world the fixed order goes from 0.584 to 0.737. Chronos-Bolt stays within 0.589–0.600 on rest of world. On UK it moves around (0.729 to 0.811) without a clear trend. At 56 days, Chronos-Bolt beats the better of the two SARIMAs on both series, by 0.121 on UK and 0.102 on rest of world.

Tuning is not a fix for short history. On rest of world the AIC search helps a little (0.702 against 0.737 at 56 days) but still loses to Chronos-Bolt. On UK it overfits: 0.852 at 112 days and 1.048 at 56 days, worse than the fixed order and, at 56 days, worse than seasonal-naive (0.910). Choosing an order by AIC on a few weeks of data rewards fitting noise.

There is one caution in the table. On UK, Chronos-Bolt was best with 56 days and worse with full history. That ordering isn't monotonic (112 days is its worst), so I read it as noise rather than a sign that less context helps. ETS also beat all three models on UK at 112 days (0.723). With six folds, rankings this close shouldn't carry much weight.

## Assumptions and the limits of public data

- **One retailer, two series, six 28-day windows each.** MASE differences of a few percent are inside fold-to-fold noise. Win counts are descriptive, not a significance test.
- **The SARIMA search is a small grid.** `SARIMA-tuned` searches a small AIC grid with fixed differencing, not a full auto-ARIMA search. A wider search could move SARIMA in either direction.
- **Pretraining overlap isn't audited.** UCI Online Retail II is a public dataset, and if it is in the model's pretraining corpus, Chronos-Bolt is flattered. That would strengthen the result where SARIMA wins and weaken it where Chronos-Bolt wins.
- **Point forecasts only.** Chronos-Bolt also produces quantiles, but their coverage isn't scored. A team that needs prediction intervals for inventory decisions hasn't been answered here.
- **One small model.** A larger Chronos variant or a different foundation model might rank differently.
- **No covariates.** Neither model uses promotions, prices or holidays beyond the closed-Saturday rule.
- **CPU floating point.** Results can differ in the last digits across machines and library versions; the versions are recorded.

## When the conclusion breaks

- **Short or new series.** The truncation results point the other way here. If most of your series have about two months of history, which is typical for new products, stores or markets, the zero-shot model is the better default in this evidence.
- **Many series, no time to tune.** The recommendation assumes one or a few series that each justify their own model. Across thousands of SKUs, even a small per-series order search costs real compute (it took most of this run's eight minutes on two series), and a model that needs no per-series configuration changes the cost comparison.
- **Peak season isn't in the test window.** The UK gap is mostly one Christmas fold. If your evaluation excludes the peak, the two models look tied, and you'll draw the wrong conclusion about the window that matters most.

## What would change my mind

- A larger backtest across many series and peak seasons where the zero-shot model matches SARIMA on peak windows. Then the no-tuning advantage would make it the default.
- Fine-tuned or covariate-aware pretrained models closing the Christmas gap on this same backtest.
- Evidence of pretraining overlap with this dataset. That would make the short-history wins less convincing.
- Interval coverage results. If the quantiles are well calibrated and SARIMA's intervals aren't, that could matter more than a few points of MASE.

## The decision rule

1. **Keep SARIMA as primary when you have a year or more of daily history** on a strongly seasonal series. A fixed order or a per-fold AIC search both work here. Add the pretrained model as a baseline next to seasonal-naive, not as a replacement.
2. **Use the zero-shot model as the default when history is short or a series is new.** In this evidence it beat both SARIMAs on both series at 56 days, but on only one of two at 112 and 365 days. Don't try to rescue SARIMA on short history by searching orders; on UK that made it worse. Re-run the backtest as history accumulates, and switch once SARIMA wins.
3. **Either way, decide on the backtest, not the model card.** Use the same rolling-origin folds, MASE against seasonal-naive, and a test window that includes the peak season.

---

*Code, results, and references: [demand-forecasting](https://github.com/wavde/demand-forecasting) · [`run_pretrained.py`](https://github.com/wavde/demand-forecasting/blob/main/run_pretrained.py) · [`pretrained.py`](https://github.com/wavde/demand-forecasting/blob/main/src/demand_forecasting/pretrained.py) · [`test_pretrained.py`](https://github.com/wavde/demand-forecasting/blob/main/tests/test_pretrained.py) · [`pretrained_results.md`](https://github.com/wavde/demand-forecasting/blob/main/docs/pretrained_results.md) · [`pretrained_metrics.json`](https://github.com/wavde/demand-forecasting/blob/main/reports/pretrained_metrics.json) · Chronos: Ansari et al. (2024), *Chronos: Learning the Language of Time Series* · MASE: Hyndman & Koehler (2006) · rolling-origin evaluation: Hyndman & Athanasopoulos, *Forecasting: Principles and Practice*.*
