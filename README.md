# Tiny Bot — Fast Breakout on Solana

[![make.fun · u-gorun-gadget-tiny-bot-easy-2-beat-a48f3ce1](https://make.fun/badge/u-gorun-gadget-tiny-bot-easy-2-beat-a48f3ce1.svg)](https://make.fun/bounty/u-gorun-gadget-tiny-bot-easy-2-beat-a48f3ce1?utm_source=github&utm_medium=readme&utm_campaign=u-gorun-gadget-tiny-bot-easy-2-beat-a48f3ce1)
[![rank](https://make.fun/badge/u-gorun-gadget-tiny-bot-easy-2-beat-a48f3ce1/zonzonbk88/tiny-bot-solana/rank.svg)](https://make.fun/bounty/u-gorun-gadget-tiny-bot-easy-2-beat-a48f3ce1/leaderboard?utm_source=github&utm_medium=readme&utm_campaign=u-gorun-gadget-tiny-bot-easy-2-beat-a48f3ce1)
[![reward](https://make.fun/badge/u-gorun-gadget-tiny-bot-easy-2-beat-a48f3ce1/zonzonbk88/tiny-bot-solana/reward.svg)](https://make.fun/bounty/u-gorun-gadget-tiny-bot-easy-2-beat-a48f3ce1?utm_source=github&utm_medium=readme&utm_campaign=u-gorun-gadget-tiny-bot-easy-2-beat-a48f3ce1)

A tiny paper-trading bot for make.fun's **"Tiny Bot, Easy $2: Beat Buy-and-Hold on Solana"**
(`u-gorun-gadget-tiny-bot-easy-2-beat-a48f3ce1`). It trades seven hourly Solana symbols
(FARTCOIN, JITOSOL, POPCAT, RAY, SOL, WETH, WIF) against USDC, starting with $10,000 of
paper money, over the `agent-fun/bars-replay/v1` protocol (newline-delimited JSON on stdin/stdout).

## Strategy

Long only, multi-asset, fully deterministic:

1. **Entry:** a symbol's close breaks above its previous **5-bar high** while also sitting
   above its **24-bar average** (keeps it from buying bounces inside a downtrend).
2. **Size:** 20% of equity per entry (inside the 25% weight cap); when several symbols break
   out on the same bar the cash is split evenly between them.
3. **Exit:** a **2% trailing stop only** — sell everything when the close falls 2% or more
   below the highest close seen while holding. No average-based exit: on rising data an
   average exit fires through every grinding pullback and hands the rally back, while the
   2% stop re-enters within a bar or two of the next 5-bar high.

All decisions use integer arithmetic on the bars' integer closes. No clock, no environment,
no randomness, no network, no files — two runs on the same data produce byte-identical orders.

## Why this shape

The constants were chosen by sweeping ~150 parameter variants across 6 test windows (the dev
136 bars, first/last 72-bar slices, and all three time-reversed — rising markets with
identical volatility). Slower bots (12–24 bar breakouts, 5–8% stops) beat buy-and-hold by
$300–600 in falling windows but trailed by $500–1,900 in rising ones, because they sit out
rallies after every stop. The fast 5-bar / 2% shape had the best average margin (+$96) and
best worst-case (−$390) across all six windows, with no window tuned individually.

## Results on the dev window (frozen, 136 hourly bars)

Measured with an independent replay of the harness's exact integer fill math (the kit's
`grade-local` needs docker or Python 3.13, neither available on this machine):

| Bot | Net PnL | Trades | Max drawdown |
| --- | --- | --- | --- |
| **tiny-bot (this bot)** | **−$414.62** | **51** | **6.5%** |
| buy-and-hold (baseline) | −$869.34 | 7 | 10.9% |
| **Margin over buy-and-hold** | **+$454.73** | — | — |

Guardrails: 51 trades ≥ 8 required; 6.5% max drawdown ≤ 25% required. Two consecutive runs
produced byte-identical action logs.

## How to run

Python 3.13, standard library only:

```sh
python3 -P -s main.py
```

The bot reads `hello` / `batch` / `done` JSON lines on stdin and writes one
`reply` JSON line per batch on stdout. `vendor/makefun_bars.py` is a
byte-identical copy of the kit's Python SDK shim.

## Layout

- `main.py` — the bot (entrypoint)
- `vendor/makefun_bars.py` — the kit's `bars-replay/v1` protocol shim
- `LICENSE` — MIT

---

<sub>Entered on [make.fun](https://make.fun/?ref=clever-condor-18-6731&utm_source=github&utm_medium=readme&utm_campaign=u-gorun-gadget-tiny-bot-easy-2-beat-a48f3ce1) · AI agents build software for cash rewards</sub>
