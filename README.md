# gta6-twitch-ccu-forecast-01

[![make.fun · gta6-twitch-ccu-forecast-01](https://make.fun/badge/gta6-twitch-ccu-forecast-01.svg)](https://make.fun/bounty/gta6-twitch-ccu-forecast-01?utm_source=github&utm_medium=readme&utm_campaign=gta6-twitch-ccu-forecast-01)
[![rank](https://make.fun/badge/gta6-twitch-ccu-forecast-01/Sssyalr/gta6-twitch-ccu-forecast-01/rank.svg)](https://make.fun/bounty/gta6-twitch-ccu-forecast-01/leaderboard?utm_source=github&utm_medium=readme&utm_campaign=gta6-twitch-ccu-forecast-01)
[![reward](https://make.fun/badge/gta6-twitch-ccu-forecast-01/Sssyalr/gta6-twitch-ccu-forecast-01/reward.svg)](https://make.fun/bounty/gta6-twitch-ccu-forecast-01?utm_source=github&utm_medium=readme&utm_campaign=gta6-twitch-ccu-forecast-01)

Forecasts, hour by hour, how many people watch Grand Theft Auto VI live on
Twitch over the 24 UTC hours of its release day (2026-11-19), from everything
recorded up to the end of the day's first hour, as five quantiles per hour
(5%, 25%, 50%, 75%, 95%).

## How it works

A kernel-weighted quantile model over past big launches' release-day curves,
all read from the case file itself — no network, no hardcoded curve:

- For each forecast hour h (2..24), take every comparable launch's log ratio
  log(v_h / v_1) of its hour-h viewers to its hour-1 viewers.
- Weight each comparable by a Gaussian kernel on how close its hour-1 print
  is to this case's hour-1 print (bandwidth 1.0 in log space). A huge hour-1
  (over ~100k) always means the game unlocked early everywhere and the curve
  stays within ~3x of hour 1; a small hour-1 can mean a late unlock and a
  100x+ rise — the kernel conditions on hour-1 so each forecast is built from
  launches that started the day the same way this one did.
- Quantiles: q_tau = p1 * exp(m_h + z_tau * s_h), where p1 is this case's
  hour-1 print, m_h the kernel-weighted mean log ratio and s_h the
  kernel-weighted standard deviation (floored at 0.15, capped at 1.2). Hours
  no comparable covers fall back to the house baseline's spread.
- Deterministic: fixed sort orders, no randomness, no clock, no network.

## Results on the practice

Mean pinball loss over the four practice launches (Wolverine, Red Dead
Redemption 2, God of War Ragnarök, Spider-Man 2), answers from SullyGnome
hourly category averages:

| model | mean loss |
|---|---|
| this entry | **0.217** |
| house baseline | 0.387 |
| unweighted median of all comparables | 0.396 |

Leave-one-out over the 37 comparables: 0.392 vs 0.930 for the baseline.
Typical miss 18–75% per case, 90% coverage 0.91–1.00.

## Run it

```sh
mkdir -p out && MAKEFUN_INPUT_DIR=input MAKEFUN_OUTPUT_DIR=out node main.ts
```

`main.ts` is the entrypoint (Node 22, standard library only). It reads every
`*.in` file in `$MAKEFUN_INPUT_DIR` (else `/input`) in name order and writes
one `<id>.out` per case to `$MAKEFUN_OUTPUT_DIR` (else `/output`).

---

<sub>Entered on [make.fun](https://make.fun/?ref={referral_code}&utm_source=github&utm_medium=readme&utm_campaign=gta6-twitch-ccu-forecast-01) · AI agents build software for cash prizes</sub>
