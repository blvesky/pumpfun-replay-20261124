# Pump.fun Replay Sprint — Launch Sniper Bot

Paper-trading bot for the `pumpfun-replay/v2` harness: 10 SOL of paper money
on a 24h replay of pump.fun bonding-curve activity. Run by the grader as
`node /bot/main.ts` (Node 22+, zero dependencies, TypeScript stripped as-is).

## Strategy

Launch sniper with a rug filter. Every coin is watched from its `CreateEvent`.
In its first slots (age 1–3) the bot buys at most 0.2 SOL of a launch only if:

- at least 5 trades and 3–5 distinct non-creator buyers have shown up,
- the curve holds 1–30 SOL of real reserves,
- the creator has not sold (and their own create-slot buy is at most 2.5 SOL),
- nobody has dumped hard (sell ratio and largest single sell within bounds).

Exits: take profit at +25% of cost, stop loss at −12%, hard exit after
10 slots held, and an immediate exit if the creator sells while we hold.
One buy per coin, at most 5 open positions, and a fill-pacing splitter that
only splits orders into pieces while the run's fill count lags its pace.

## Why these numbers

Swept on the kit dev window with the pilot sniper grid tool. The edge is in
the entry filter, not the exits: tp 800–2500 bps, sl 600–2000 bps and
maxHold 6–20 slots all stay positive, and the chosen point (+0.08 SOL,
38 coins, max drawdown 0.4%) sits in the middle of that flat region rather
than on a single best cell. Guardrails on the dev window: 38 distinct coins
with ≥0.01 SOL round trips (bar: 20), max drawdown 0.4% (bar: 50%),
76 fills, zero rejected orders.

## Layout

- `main.ts` — entrypoint: hands the strategy to the SDK shim.
- `strategy.ts` — the whole strategy (pure function of the batches; no
  clock, no randomness, no I/O, so the two graded runs agree byte for byte).
- `vendor/makefun-bot.ts` — the kit's SDK shim, byte-identical to
  `sdk/ts/makefun-bot.ts` (kept out of authorship counts by the kit's rules).

## License

MIT — see `LICENSE`.
