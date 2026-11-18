# Token Radar

A token launch radar: it reads a frozen snapshot of every protocol DefiLlama tracks,
ranks the protocols that record no token by what their launch would be worth, and hands
in a dashboard a reader can use — the board, a detail view for each protocol, and the
steps to farm it.

Built for make.fun's `token-radar-01` bounty. The entry's program is `main.ts` (Node 22,
standard library only, no network, deterministic: two runs write the same bytes).

## The board

- **226 rows**: the whole 40-protocol watchlist (pre-token, TVL at or above $50M) plus
  every other rankable pre-token protocol below the floor (186 early rows). Rank 1 is
  Steakhouse Financial at $465M implied; the last row is Valdora Finance at $214K.
- **24 live-token protocols set aside** in their own view, shown with the market's own
  values (market cap, market-cap-to-TVL), never mixed into the pre-token ranking.
- Every row carries a 6-step farm plan with a cost and a time.

## The model (radar-2)

```
implied = round( tvl_usd x multiple x (1 + clamp(change_7d, -60, 120) / 100 x 0.5)
                 x (0.6 + min(age_days, 1095) / 1095 x 0.4) )
```

- **Multiple** — the market's own price of a dollar of TVL: the median market-cap-to-TVL
  ratio of the token protocols in the same category, computed from this snapshot at run
  time (a category needs at least 5 trading protocols to set its own multiple; any other
  category uses the whole market's median, 0.21x from 128 protocols). Multiples are
  clamped to 0.05–1.50x so one outlier token cannot set a category's price. On this
  snapshot: Canonical Bridge 0.75x, Basis Trading 0.65x, Bridge 0.56x, Dexs 0.34x,
  RWA 0.27x, Yield 0.19x, Lending 0.13x, CEX 0.07x, Liquid Staking 0.05x.
- **Growth** — seven-day TVL change, clamped to −60%..+120%, so one week's spike cannot
  carry a protocol.
- **Maturity** — age since the protocol was first listed, the best record of how long it
  has run with no token (unlisted age counts as 365 days).
- No protocol name is ever special-cased. The dashboard recomputes every row from the
  snapshot figures and shows the multiple, the growth and the maturity behind each number.

## The farm plan

Six steps per protocol, generated from its own record (category, chains, TVL, age):
a fresh wallet, the category's action (supply, provide liquidity, bridge, stake, mint,
vault deposit …) with a capital tier from TVL, one qualifying action a week, a hold of
30–60 days, a check of the protocol's own points page (claim only from its official
link; never pay anyone to claim), and a farm log. Gas costs follow the chain
(Ethereum $3–8, Solana ~$0.01, L2s $0.10–0.50). Nothing here promises a token:
the steps are how to farm, and the page says so.

## Honesty

- "No token on record" is what DefiLlama records, not proof a protocol has none.
- Implied value is this radar's own claim about what a token would be worth if the
  protocol's record stayed as it is. It does not predict whether or when a token is
  issued, and no farm step guarantees one.

## Run it

```sh
mkdir -p out && MAKEFUN_OUTPUT_DIR=out node main.ts --input <snapshot.json|directory>
```

The default input is `/input` (the sealed snapshot) and the default output is `/output`.
The program writes `board.json`, `data.js`, `dashboard/`, `screenshots/` and
`dossier.json` (built with the vendored `vendor/makefun-board.ts`), and refuses to write
a board the gate would refuse.

Open `out/dashboard/index.html` in a browser (it reads `out/data.js`, no network).
Deep links: `#p=<slug>` opens a protocol's detail, `#category=<name>` and
`#chain=<name>` filter the board, `#tab=live` shows the live-token view.

## Files

```
main.ts            the entrypoint: reads the snapshot, writes /output
lib/model.ts       the model: multiples, implied value, farm plans, the board
vendor/makefun-board.ts   the dossier library, the kit's copy byte for byte
dashboard/         index.html, app.css, app.js — the static page, no network
screenshots/       1-board.png, 2-detail.png, 3-filtered.png
```

## License

MIT. See LICENSE.
