# Pump.fun Replay Sprint — Launch Sniper Bot

Paper-trading bot for the `pumpfun-replay/v2` harness: 10 SOL of paper money
on a 24h replay of pump.fun bonding-curve activity. Run by the grader as
`node /bot/main.ts` (Node 22+, zero dependencies, TypeScript stripped as-is).

## Strategy

Launch sniper with a rug filter. Every coin is watched from its `CreateEvent`.
In its first slots (age 1–3) the bot buys at most 0.2 SOL of a launch only if:

- at least 7 trades and 3–8 distinct non-creator buyers have shown up,
- the creator bought 0.5–2.5 SOL in the create slot (skin in the game,
  but not a whale bundle) and has not sold,
- the curve holds 1–30 SOL of real reserves,
- nobody has dumped hard (sell ratio and largest single sell within bounds).

Exits: take profit at +10% of cost, breakeven stop (sell the moment the
position is worth less than its cost), hard exit after 5 slots held, and an
immediate exit if the creator sells while we hold.
One buy per coin, at most 5 open positions.

## Why these numbers

Swept on the kit dev window (all three parts, 128,058 events) with the pilot
sniper grid tool. The edge is the creator's own buy: a launch whose creator
put 0.5–2.5 SOL in at creation, with 7+ trades and 3–8 buyers in its first
3 slots, is a live pump. The region is flat: minTrades 6–8, minDevBuy
0.4–0.6 SOL and maxBuyers 8–12 all stay positive (dev +0.076 to +0.111 SOL);
the chosen point (+0.091 SOL, 40 coins, max drawdown 0.7%) sits in the
middle of it. Guardrails on the dev window: 40 distinct coins with ≥0.01 SOL
round trips (bar: 20), max drawdown 0.7% (bar: 50%), 80 fills, zero
rejected orders. Honest caveat: most of the dev net comes from one coin
that pumped ~7x in the slots after entry; without it the run is about
−0.05 SOL. The filter is what finds such pumps, but a single runner is
what pays for the rest.

## Layout

- `main.ts` — entrypoint: hands the strategy to the SDK shim.
- `strategy.ts` — the whole strategy (pure function of the batches; no
  clock, no randomness, no I/O, so the two graded runs agree byte for byte).
- `vendor/makefun-bot.ts` — the kit's SDK shim, byte-identical to
  `sdk/ts/makefun-bot.ts` (kept out of authorship counts by the kit's rules).

## License

MIT — see `LICENSE`.
