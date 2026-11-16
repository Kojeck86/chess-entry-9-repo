# Ember SEE — a 4 KB chess engine (tiny-chess-01, round 6)

From-scratch single-file Python chess engine, packed as an
lzma+base85 payload with a one-line unpacker. Plays the env
protocol (`POST /game`, `/move`, `/resign` at `MAKEFUN_ENV_URL`).

Round 6 builds on round 5 ("Ember", v5): same search, plus
static exchange evaluation (SEE) pruning in quiescence.

## How it works

- 0x88 board, negamax alpha-beta, iterative deepening (depth 1-8).
- Fixed node budget: 25,000 nodes/move (S + quiescence). No wall
  clock anywhere, so live and replay runs are identical.
- Transposition table (Zobrist, exact/lower/upper), killer move +
  history heuristic, null-move pruning (R=2), late-move reductions,
  in-search repetition draws.
- Quiescence: captures only (all evasions in check), delta pruning.
- **SEE pruning (new in v6):** in quiescence, a capture with a
  negative static exchange value is skipped. Classic swap
  algorithm (least-valuable-attacker order, minimax over the
  capture sequence), reusing the attack test, which now returns
  the least-valuable attacker (square + 1, or 0) instead of a
  boolean. SEE runs only when victim + promotion value is below
  the attacker's value (necessary for a losing exchange), so
  winning captures cost nothing. As in classic SEE: pins ignored;
  a recapture promotion scores as a pawn (safe direction).
- Eval: material + centrality/advance + passed-pawn bonus +
  bishop pair + rook on 7th + king centralisation (mid- vs
  endgame). Ordering: TT move, MVV-LVA captures, killer, history.

## What changed vs v5

- SEE pruning in quiescence (above).
- Attack test returns the least-valuable attacker; the search
  uses it as its check test too (behaviour unchanged).
- History indexed directly by 0x88 squares; one killer per ply
  (2nd slot dropped for the byte budget).
- Node budget unchanged at 25,000/move; main.py 4,063 bytes.

## Verification (local, no Docker)

- Perft: startpos, kiwipete, pos3, pos4, pos5, pos6, depths 1-3,
  all match the reference counts.
- Zobrist hash: make/unmake restores it; matches a from-scratch
  recompute (200 random plies).
- SEE: 7 hand-computed exchanges all match; SEE leaves board,
  hash, side, castling, en-passant and king squares untouched
  (200 random positions).
- `main.py` 4,063 bytes (cap 4,096); snapshot under 8,192.
- Self-play: vs v5 4-4 (8 games); vs v4 2W/2L/4D (8 games).
- Practice trials (official scorer, 16 games each): trial-0 Elo
  989 (2W/2D/12L), trial-1 Elo 933 (2W/1D/13L), trial-2 Elo
  1087 (4W/0D/12L); mean 1,003 (v5 mean 1,018). Statistically
  indistinguishable (9.5 vs 10.0 points over 48 games); SEE did
  not measurably move the score. Both below the 1,154 floor.

## Build

`engine6.py` is the readable source (kept out of the entry);
`main.py` is produced by minifying it and packing it with lzma
preset 9 + base85. Rebuild with the local `build6.py`.

## License

MIT. See LICENSE.
