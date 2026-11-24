# Pairs Trading in Crypto: Does the Gatev Rule Still Work?

Does the classic pairs trading rule of Gatev, Goetzmann and Rouwenhorst (2006) work on cryptocurrencies, and can smarter pair selection rescue it? Tested on 41 coins with daily prices, March 2018 to September 2026, against random pairs, random clusters and a held-out period.

Group project (QF600 Asset Pricing, Group 8) for the Master of Science in Quantitative Finance at Singapore Management University (SMU).

## Key results

Top 20 pairs, mean monthly return on committed capital, out-of-sample (Aug 2024 to Sep 2026, 26 months):

|                                              | Model 1: Gatev baseline | Model 2: clustering first | Model 3: wider trigger (3 SD) |
| -------------------------------------------- | ----------------------- | ------------------------- | ----------------------------- |
| In-sample, before fees                       | 1.59%                   | 1.40%                     | 1.59%                         |
| Out-of-sample, before fees                   | 0.08%                   | 0.35%                     | 0.49%                         |
| Out-of-sample, after 10 bp fee per trade leg | 0.00%                   | 0.28%                     | 0.44%                         |
| t-statistic after fees                       | 0.01                    | 0.67                      | 1.11                          |
| Break-even fee (bp per leg)                  | 11                      | 48                        | above 100                     |

- **In-sample it works; out-of-sample it fades.** The baseline falls from 1.59% to 0.08% a month.
- **No model is significantly profitable after fees.** All t-statistics are below the 1.65 bar set in advance.
- **Clustering and a wider trigger help, but neither gain is significant** (t = 0.52 and 0.65). With 26 months, the differences cannot be told apart from noise.

Pre-registered hypotheses (fixed before the final run, one-sided 5% level, out-of-sample):

| Hypothesis                                 | Result                                                       |
| ------------------------------------------ | ------------------------------------------------------------ |
| H1a: clustering beats the Gatev baseline   | Not supported (t = 0.52)                                     |
| H1b: AP clusters beat random clusters      | Supported out-of-sample (1% of 100 random clusterings do as well). In-sample, random clusters do as well or better, so treat this as weak evidence |
| H2: the 3 SD trigger beats 2 SD after fees | Not supported (t = 0.65)                                     |
| H3: the pairs beat random pairs            | Not supported (8-11% of 200 random portfolios do as well)    |
| H4: profitable after a 10 bp fee           | Not supported (t = 0.01, 0.67, 1.11)                         |

Plain-language terms:

- **Pairs trading:** find two assets that move together. When they drift apart, buy the one that fell behind and short the one that ran ahead. Close when they meet again.
- **Short:** borrow an asset and sell it, hoping to buy it back cheaper.
- **Committed capital:** returns measured on the capital set aside for all selected pairs, including pairs that never traded. This is the conservative measure.
- **In-sample / out-of-sample:** data used to build and choose the rules, versus later data held back to test them.
- **Basis point (bp):** 0.01%. A 10 bp fee is 0.10% per trade leg.
- **Break-even fee:** the fee at which the average return reaches zero.
- **t-statistic:** how strong the evidence is. Around 2 or above is usually treated as significant. The bar here is 1.65.
- **Newey-West standard errors:** a correction for returns that are correlated from month to month (6 lags used).
- **SD (standard deviation):** a measure of how far a price gap is from normal.

## The three models

1. **Model 1: Gatev baseline.** Each coin is paired with the coin whose normalised price path is closest (smallest sum of squared gaps). Pairs are ranked by that distance.
2. **Model 2: cluster first.** Coins are grouped using PCA (5 factors that summarise how coins move together) and affinity propagation (a method that picks the number of groups itself). Coins are then paired only within their own group.
3. **Model 3: tune the trigger.** Same as Model 2, but the entry threshold is chosen from 1.0, 1.5, 2.0, 2.5 and 3.0 SD using in-sample data only (best Sharpe ratio after a 10 bp fee). The pick was 3.0 SD.

## How the trading works

- **Formation:** 12 months of data to find pairs. **Trading:** the next 6 months. A new batch of pairs (a cohort) starts every month, so up to six run at once and their returns are averaged. There are 91 cohorts.
- **Open** when the gap exceeds the entry threshold: buy the laggard and short the leader, $1 each. **Close** when the prices cross, or at the end of the 6 months.
- **Trades execute one day after the signal.**
- **Crypto adaptations:** calendar days (crypto trades 7 days a week), no dividends, bad-price filtering (13 spikes repaired, 24 permanent level shifts removed), and a rule that closes a pair if a short squeeze wipes out its capital.
- **Split:** 75% in-sample (Mar 2018 to Jul 2024) and 25% out-of-sample (Aug 2024 to Sep 2026), by month.

## Falsification tests

- **Random-pairs bootstrap:** both coins in every trade are replaced by random coins with similar recent returns, on the same dates (200 runs per model).
- **Placebo clusters:** coin labels are shuffled across clusters of the same sizes (100 runs).
- **Transaction costs:** a fee grid from 0 to 100 bp per leg, with break-even fees.
- **Other checks:** sensitivity to the number of PCA factors (3, 5, 10), sub-periods, and exposure to crypto market, size, momentum and reversal factors.

## Repository contents

| File or folder                             | Description                                   |
| ------------------------------------------ | --------------------------------------------- |
| `QF600_Pairs_Trading_Final_executed.ipynb` | The full analysis, with outputs               |
| `crypto_cache/`                            | One CSV per coin with daily prices and volume |
| `requirements.txt`                         | Python packages needed                        |

## How to run

1. Install Python 3, then run `pip install -r requirements.txt`.
2. Keep `crypto_cache/` in the same folder as the notebook, or set `DATA_DIR` in the settings cell to its full path.
3. Open the notebook with `jupyter notebook` and choose **Kernel > Restart & Run All**. The placebo and bootstrap sections are the slowest.

All settings (dates, thresholds, fees, number of runs, random seed 600) are in one cell near the top.

## Data

Daily prices and dollar volume for 41 coins, 2018-03-01 to 2026-09-30 (3,136 days). Source: CoinDesk API. Each CSV has the columns `symbol`, `date`, `open`, `high`, `low`, `close`, `volumefrom`, `volumeto`.

The stablecoin USDT is excluded because it has no relative price movement to trade. Coins that start after March 2018 are dropped. Files ending in `.partial.csv` are skipped.

## Limitations

- **Survivorship bias:** only coins that still have prices in 2026 are included, which flatters returns.
- **No short-funding costs:** borrowing costs on the short leg are not modelled, so true break-even fees are lower than shown.
- **Short holdout:** 26 months of out-of-sample data, so small differences cannot be detected.
- **Out-of-sample not fully clean for Models 1 and 2:** their 2 SD results had been seen before this notebook was built. The threshold grid, selection rule and pass/fail rules were fixed before running it.
- **Multiple testing:** the threshold grid adds 5 trials per model. The best threshold (3.0 SD) sits at the edge of the grid, so a higher value might do better.
- **Fewer pairs than the original paper:** with about 35 coins per window, each window gives about 20-30 pairs. When Model 2 has fewer than 20 pairs, "Top 20" holds all of them.
- **Short losses are uncapped:** the $1 legs are never rebalanced, so a short leg can lose more than the pair's capital on a big up-move.
- **Size factor is a proxy:** there is no market cap data, so dollar volume is used instead.

## Reference

Gatev, E., Goetzmann, W. N. and Rouwenhorst, K. G. (2006). Pairs Trading: Performance of a Relative-Value Arbitrage Rule. *Review of Financial Studies*, 19(3), 797-827.

## Authors

- Davidson Mundt
- Eric Jansen
- Diyaa Rajeev Valiyaparambath Kuttikattil
