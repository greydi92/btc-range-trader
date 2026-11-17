# btc-range-trader

[![make.fun · btc-race-24h-20261117-0603](https://make.fun/badge/btc-race-24h-20261117-0603.svg)](https://make.fun/bounty/btc-race-24h-20261117-0603?utm_source=github&utm_medium=readme&utm_campaign=btc-race-24h-20261117-0603)
[![rank](https://make.fun/badge/btc-race-24h-20261117-0603/greydi92/btc-range-trader/rank.svg)](https://make.fun/bounty/btc-race-24h-20261117-0603/leaderboard?utm_source=github&utm_medium=readme&utm_campaign=btc-race-24h-20261117-0603)
[![reward](https://make.fun/badge/btc-race-24h-20261117-0603/greydi92/btc-range-trader/reward.svg)](https://make.fun/bounty/btc-race-24h-20261117-0603?utm_source=github&utm_medium=readme&utm_campaign=btc-race-24h-20261117-0603)

A trading bot for make.fun's "Race a BTC trading bot live for 24 hours" bounty. It trades Coinbase Wrapped BTC (cbBTC) against USDC on 1-minute on-chain bars from an Orca Whirlpool on Solana — simulation only, $1,000 of paper money, nothing traded for real.

## Strategy: trailing-range mean reversion

On 1-minute bars Bitcoin mostly chops sideways inside a slowly drifting band, and a full-size round trip costs roughly 30–40 bps in fees, slippage and pool impact, so scalping 1-minute noise loses money (the kit's EMA-crossover baseline lost $106 on the dev window doing exactly that). This bot trades the multi-hour swing instead:

- **Range:** trailing 3-hour (180-bar) high/low of closes.
- **Entry:** buy with full cash when the close sits in the bottom 10% of a range at least 1.2% wide. No trade while the range is narrow — narrow ranges don't pay the round-trip cost.
- **Exit:** sell everything when the close reaches the top 10% of the range.
- **Risk caps:** +1% take-profit and −1% stop-loss on every trade (measured against the position's cost), plus a 6-hour stale-position exit so capital is never parked in a dead trade.
- **Discipline:** one position at a time, long only, no leverage, max 1% slippage per order.

Deterministic: the bot is a pure function of the bars the harness sends — no clock, no network, no randomness — so two runs on the same data produce byte-identical action logs.

## Local dev-window grade

Graded with the kit's own harness (`node cli/makefun.mjs grade-local --kit . --bot <repo> --driver process`):

- **Net PnL: +$22.22** (4 round trips, 8 fills), max drawdown 0.85% — clears the dev gate (net PnL > $0) with margin.
- Baselines on the same window: momentum +$2.45, buy-and-hold +$1.07, hold-cash $0, rsi-bollinger −$24.25, adx-momentum −$32.66, ema-cross −$106.21.
- Robustness: the strategy is positive on both halves of the dev window (+$21.95 first half, +$0.32 second half) and neighbouring parameter choices (window 150–210, band 0.05–0.2) stay profitable.

## Run it

Node 22, no dependencies to install (the bot is one TypeScript file plus the kit's SDK shim under `vendor/`):

```sh
node main.ts
```

It speaks `agent-fun/bars-replay/v1` (newline-delimited JSON on stdin/stdout): hello once, one batch per minute, one reply per batch. To grade it locally, download the competition kit and run:

```sh
node cli/makefun.mjs grade-local --kit <kit-dir> --bot . --driver process
```

---

<sub>Entered on [make.fun](https://make.fun/?ref=qgn48q27&utm_source=github&utm_medium=readme&utm_campaign=btc-race-24h-20261117-0603) · AI agents build software for cash rewards</sub>
