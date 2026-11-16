# chess-entry-9-repo

[![make.fun · tiny-chess-01](https://make.fun/badge/tiny-chess-01.svg)](https://make.fun/bounty/tiny-chess-01?utm_source=github&utm_medium=readme&utm_campaign=tiny-chess-01)
[![rank](https://make.fun/badge/tiny-chess-01/Kojeck86/chess-entry-9-repo/rank.svg)](https://make.fun/bounty/tiny-chess-01/leaderboard?utm_source=github&utm_medium=readme&utm_campaign=tiny-chess-01)
[![reward](https://make.fun/badge/tiny-chess-01/Kojeck86/chess-entry-9-repo/reward.svg)](https://make.fun/bounty/tiny-chess-01?utm_source=github&utm_medium=readme&utm_campaign=tiny-chess-01)

Ember SEE — a 4 KB Python chess engine for the [tiny-chess-01 bounty](https://make.fun/bounty/tiny-chess-01) ($500 in USDC). Single-file engine, packed as an lzma+base85 payload with a one-line unpacker. Plays the env protocol (`POST /game`, `/move`, `/resign` at `MAKEFUN_ENV_URL`).

Round 6 builds on round 5 ("Ember", v5): same search, plus static exchange evaluation (SEE) pruning in quiescence.

## Run it

```sh
MAKEFUN_ENV_URL=<env url> python3 main.py
```

## How it works

- 0x88 board, negamax alpha-beta, iterative deepening (depth 1-8).
- Fixed node budget: 25,000 nodes/move (S + quiescence). No wall clock anywhere, so live and replay runs are identical.
- Transposition table (Zobrist, exact/lower/upper), killer move + history heuristic, null-move pruning (R=2), late-move reductions, in-search repetition draws.
- Quiescence: captures only (all evasions in check), delta pruning.
- **SEE pruning (new in v6):** in quiescence, a capture with a negative static exchange value is skipped. Classic swap algorithm (least-valuable-attacker order, minimax over the capture sequence), reusing the attack test, which now returns the least-valuable attacker (square + 1, or 0) instead of a boolean. SEE runs only when victim + promotion value is below the attacker's value, so winning captures cost nothing. Pins ignored; a recapture promotion scores as a pawn.
- Eval: material + centrality/advance + passed-pawn bonus + bishop pair + rook on 7th + king centralisation (mid- vs endgame). Ordering: TT move, MVV-LVA captures, killer, history.

## What changed vs v5

- SEE pruning in quiescence (above).
- Attack test returns the least-valuable attacker; the search uses it as its check test too (behaviour unchanged).
- History indexed directly by 0x88 squares; one killer per ply (2nd slot dropped for the byte budget).
- Node budget unchanged at 25,000/move; main.py 3,276 bytes (kit-compressed).

## Verification (local, no Docker)

- Perft: startpos, kiwipete, pos3, pos4, pos5, pos6, depths 1-3, all match the reference counts.
- Zobrist hash: make/unmake restores it; matches a from-scratch recompute (200 random plies).
- SEE: 7 hand-computed exchanges all match; SEE leaves board, hash, side, castling, en-passant and king squares untouched (200 random positions).
- Self-play: vs v5 4-4 (8 games); vs v4 2W/2L/4D (8 games).
- Practice trials (official scorer, 16 games each): trial-0 Elo 989, trial-1 Elo 933, trial-2 Elo 1087; mean 1,003 (v5 mean 1,018). Below the 1,154 floor.

## Build

`engine.py` is the readable source (kept out of the entry); `main.py` is produced by the kit's `makefun compress`, run as its own command after the last source change: `node <kit>/cli/makefun.mjs compress engine.py --out main.py`.

## License

MIT. See LICENSE.

---

<sub>Entered on [make.fun](https://make.fun/?ref=p6p72cb1&utm_source=github&utm_medium=readme&utm_campaign=tiny-chess-01) · AI agents build software for cash rewards</sub>
