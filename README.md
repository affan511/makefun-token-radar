# Token Radar — radar-2

A token launch radar for make.fun's `token-radar-01` bounty. It reads a sealed
DefiLlama snapshot of 758 protocols, ranks every genuinely pre-token protocol by
what its launch would be worth, and hands in a static dashboard that shows the
board, a detail view for each protocol, and the steps to farm it.

## The model (radar-2)

Implied value follows only from the snapshot's own figures; no protocol name is
ever special-cased:

```
implied = round( tvl_usd
  x category_multiple
  x (1 + clamp(change_7d, -60, 120) / 100 x 0.5)
  x (0.6 + min(age_days, 1095) / 1095 x 0.4)
  x (1 + 0.02 x min(chains - 1, 5)) )
```

- **Category multiple**: the median market-cap-to-TVL ratio of the token-bearing
  protocols in the same category, read from the snapshot itself. A category with
  fewer than three live-token protocols uses the snapshot-wide median. A DEX's
  token trades at a different multiple of its TVL than a lender's, and the board
  prices them as the market does.
- **Growth**: the seven-day TVL change. Capital arriving now is worth more than
  capital that arrived years ago.
- **Maturity**: the time since the protocol was first listed — the best record of
  how long it has run with no token.
- **Chain breadth**: more chains reach more users, capped small on purpose.

A protocol is ranked only when it is genuinely pre-token: its record carries no
token, and it is not a wrapped, staked or bridged product of a token that already
trades, not an exchange, and not a versioned pool. Wrapped, staked, bridged and
exchange products are set aside with their reasons, and the live-token protocols
are shown with their market caps — the market's own values, never a model's.

## Farm plans

Every row carries a seven-step farm plan derived from its own category, chains,
TVL and age, with what each step costs and how long it takes. Gas is priced for
the protocol's own chain: cents on Solana and the rollups, dollars on Ethereum
mainnet.

## Run it

```
mkdir -p out && MAKEFUN_OUTPUT_DIR=out node main.ts --input <snapshot.json>
```

`main.ts` (Node 22, standard library only) reads the snapshot, writes
`board.json` and `data.js`, copies `dashboard/` and `screenshots/` to the output
directory, and writes `dossier.json` with the vendored make.fun library.

## Files

- `main.ts` — the entrypoint
- `lib/model.ts` — the radar-2 model, the board, and the farm plans
- `vendor/makefun-board.ts` — make.fun's dossier library, as the kit ships it
- `dashboard/` — the static page (index.html, app.css, app.js), no network
- `screenshots/` — the board, a protocol's detail view, and a filtered view

## License

MIT. See `LICENSE`.
