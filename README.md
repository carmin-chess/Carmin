# Carmin

**Carmin** is a UCI chess engine written in Rust by **Cawl**.

---

## Evaluation

Carmin uses a hand-crafted evaluation (HCE), tuned with Texel-style methods.

The evaluation scores a position from the side to move by combining several terms:

- **Material** — piece values, including material imbalance tables
- **Mobility** — how many safe squares pieces can move to
- **King safety / king danger** — pawn shield, open files, attacking pieces near the king
- **Pawns** — structure, passed pawns, advanced pawns, weak pawns
- **Pieces** — outposts, poorly placed pieces, patterns
- **Threats** — hanging pieces and tactical pressure
- **Space & initiative** — control of the board and who is dictating the play
- **Endgame knowledge** — special cases and fortress detection when little material remains

A **personality** system can scale these terms (and contempt) so the engine plays more aggressively, defensively, positionally, etc., without rewriting the evaluation.

Static evaluation is also refined by a **correction history** table that learns small adjustments from search outcomes.

---

## Search

Carmin searches with **negamax alpha-beta** and **principal variation search (PVS)**.

### Iterative deepening
The engine searches depth 1, then 2, then 3, … up to the time or depth limit. Each deeper iteration reuses ordering and transposition-table information from the previous one.

Before the main search, a shallow ranking pass is run (depth `ROOT_MOVE_CUT_DEPTH`). Only the top `ROOT_MOVE_CUT_COUNT` root moves are kept for the full search, which saves time on clearly inferior moves.

### Main techniques
- **Transposition table** — multi-bucket hash table storing score, bound, depth, and best move
- **Quiescence search** — extends the search on captures (and similar tactical moves) so evaluations are not taken in the middle of exchanges
- **Null-move pruning** — if even “passing” the move still fails high, the position is cut
- **Razoring & reverse futility pruning** — cheap cutoffs near the leaves when the static eval is far from alpha/beta
- **ProbCut** — reduced-depth probes to prove a beta cutoff early
- **Late move reductions (LMR)** and **late move pruning (LMP)** — search late, quiet moves with less depth (or skip them)
- **Futility & SEE pruning** — skip moves that cannot improve alpha according to static exchange evaluation
- **Singular extensions** — extend the search when one move is clearly better than all others
- **Move ordering** — TT move, captures (MVV-LVA / SEE), killers, counters, history and continuation history
- **Aspiration windows** — search with a narrow score window around the previous iteration’s score and widen only on fails

### Time management
Soft and hard time limits are computed from UCI `wtime` / `btime` / `winc` / `binc` / `movetime`. The search stops between depths when the soft limit expires, and aborts immediately when the hard limit (or node limit) is hit.

### Parallelism
Lazy SMP: multiple threads search the same position, sharing the transposition table, with the main thread reporting UCI info and the best move.

---

## Building

```bash
cargo build --release
```

Binary: `target/release/carmin`

---

## Usage

Carmin speaks the UCI protocol. Point any UCI GUI or bot at the binary, or run it in a terminal:

```
uci
isready
position startpos
go depth 15
```
