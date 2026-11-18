# GTA VI Twitch Viewers — Hour-by-Hour Forecast

[![make.fun · gta6-twitch-ccu-forecast-01](https://make.fun/badge/gta6-twitch-ccu-forecast-01.svg)](https://make.fun/bounty/gta6-twitch-ccu-forecast-01?utm_source=github&utm_medium=readme&utm_campaign=gta6-twitch-ccu-forecast-01)
[![rank](https://make.fun/badge/gta6-twitch-ccu-forecast-01/Auttzy/gta6-twitch-ccu-forecast-01/rank.svg)](https://make.fun/bounty/gta6-twitch-ccu-forecast-01/leaderboard?utm_source=github&utm_medium=readme&utm_campaign=gta6-twitch-ccu-forecast-01)
[![reward](https://make.fun/badge/gta6-twitch-ccu-forecast-01/Auttzy/gta6-twitch-ccu-forecast-01/reward.svg)](https://make.fun/bounty/gta6-twitch-ccu-forecast-01?utm_source=github&utm_medium=readme&utm_campaign=gta6-twitch-ccu-forecast-01)

A forecaster for make.fun's [`gta6-twitch-ccu-forecast-01`](https://make.fun/bounty/gta6-twitch-ccu-forecast-01) bounty ($250, top 20 paid): given Twitch's viewer count for the Grand Theft Auto VI category at the end of release day's first hour, it forecasts the count for each of hours 2–24 as five quantiles (5%, 25%, 50%, 75%, 95%). Scored by mean pinball loss on the log scale — lower is better — and a prize needs a loss strictly below the best house entry.

## Method

Launch-day viewership is not flat: it follows a diurnal curve — it ramps as regions unlock, peaks in the evening, dips overnight — and the shape depends on how big the launch is. So instead of holding hour 1 flat, the program learns the shape from the case's own comparables at run time:

1. For every comparable past launch, take its release-day hourly curve and express each later hour as a ratio to its own hour 1.
2. Keep only launches whose hour 1 is already a real launch hour (hour-1 viewers at least 10% of that launch's release-day peak). Launches with a tiny hour 1 unlocked later in the UTC day, so their ratios explode and would poison the shape. The Grand Theft Auto VI pre-release category history is also left out — it is not a launch day.
3. Weight each kept launch by how close its hour-1 size is to the target's (a Gaussian kernel on log hour-1 viewers, bandwidth 1.0). Big launches behave like big launches, so the shape comes mostly from launches of a similar size.
4. The forecast median for hour h is hour 1's print times the weighted median ratio; the log-spread is the weighted standard deviation of the log-ratios. The five quantiles are median × exp(z × spread) with z = (−1.645, −0.674, 0, 0.674, 1.645).

## Validation

Leave-one-out over the 108 filtered comparable launches in the four practice cases:

| Forecaster | Mean pinball loss |
|---|---|
| This forecaster (magnitude-weighted median shape) | **0.226** |
| Flat house baseline (hour 1 held constant) | 0.297 |

About 24% lower than the flat baseline. The baseline scores 0.387 on the four practice cases themselves; the shape is expected to beat it there too, because launch days are strongly diurnal. Dev-board rank was not used as a guide — the board carries a cluster of near-zero losses whose mechanism is unverified.

## How to run it

Node 22, standard library only — nothing to install. The program reads every `*.in` in the input directory and writes one `*.out` per case:

```sh
mkdir -p out && MAKEFUN_INPUT_DIR=input MAKEFUN_OUTPUT_DIR=out node main.ts
```

Each output is `{"hours": [{"hour": 2, "q": [q05, q25, q50, q75, q95]}, …]}` for hours 2–24. Two runs on the same case write identical bytes.

## The bounty

Built for [make.fun](https://make.fun)'s `gta6-twitch-ccu-forecast-01` bounty: forecast GTA VI's launch-day Twitch viewers hour by hour. The prize pool is $250, shared by the top 20 entries.
