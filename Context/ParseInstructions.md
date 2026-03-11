# Game Feed Parse Instructions

## Overview
This document explains how to parse a game feed file and produce:
1. A per-game detail file at `Spring2026/Games/[YYYY-MM-DD]_vs_[Opponent].md`
2. Updated cumulative stats in `Spring2026/PlayerStats.md`

Always reference `Context/RE24Table.md` for RE values. Only process **our team's half-innings**.

---

## Step 1 — Identify Our Half-Innings

Our team is **USA Scout 8u Prospects**. Our batting half-innings begin with a header line matching either form (Top when visiting, Bottom when home):

```
Top [N] - USA Scout 8u Prospects
Bottom [N] - USA Scout 8u Prospects
```

Skip all half-innings headed by the **opponent's** team name entirely.

---

## Step 2 — Understand the Game Feed Format

Each play in a half-inning is described by a **block of lines**:

```
[Play type]                   ← Single, Strikeout, Ground Out, Error, Fielder's Choice, etc.
[Score/out update]            ← e.g., "MCBB 3 - USSC 0 | 1 Out" or just "2 Outs"
[Pitch sequence]              ← e.g., "Ball 1, Foul, In play." or "Strike 1 swinging, ..."
[Narrative description]       ← The authoritative line; describes who batted and what happened
```

The **narrative description** is the most important line. It tells you:
- Who batted (abbreviated name)
- What happened to the batter (reaches, out, etc.)
- How each base runner moved

Score/out update lines may appear before or after the pitch sequence. They are supplementary — use them to cross-check, but rely on the narrative for base state tracking.

---

## Step 3 — Player Name Mapping

Game feeds use abbreviated names (first initial + last name). Map them to roster players:

| Feed Name   | Full Name        | # |
|-------------|:-----------------|:-:|
| L Brown     | Liam Brown       | 1 |
| C Hayes     | Cooper Hayes     | 2 |
| B Herrera   | Balyn Herrera    | 3 |
| H Steed     | Henry Steed      | 11|
| B Hipp      | Benjamin Hipp    | 13|
| B Parker    | Baxter Parker    | 14|
| L Watkins   | Levi Watkins     | 23|
| P Morrison  | Porter Morrison  | 27|
| J Ham       | Jase Ham         | 28|
| C Newton    | Cooper Newton    | 42|
| C Griffin   | Charlie Griffin  | 44|
| L Meyer     | Logan Meyer      | 99|

Only players in the season `Roster.md` get tracked. If a name appears that isn't on the roster, they are from the opposing team — ignore them.

---

## Step 4 — Track Base-Out State Through the Half-Inning

At the **start of each half-inning**, initialize:
- Base state: `---` (nobody on base)
- Outs: 0

For each PA, before recording anything, note the **current base state and out count** — that is RE_before's lookup state.

After the play resolves, update:
- Which bases have runners (and who, for RBI tracking)
- How many outs (add 1 for standard outs; add 2 for double plays)
- How many runs scored

That updated state is RE_after's lookup state. If the PA ends the inning (3rd out), RE_after = 0.000.

**Base state resets to `---` with 0 outs at the start of every half-inning.** This is just baseball — it has no effect on player RE24 totals.

---

## Step 5 — Calculate RE24 for Each PA

```
RE24 = (RE_after + runs_scored_during_PA) − RE_before
```

Look up RE_before and RE_after from `Context/RE24Table.md` using the base state and out count for each.

**When inning ends:** RE_after = 0.000 (regardless of base state).

**Runs scored during a PA:** Count runners who cross home plate as a direct result of this PA (including runs scored on errors during the same play).

---

## Step 6 — Narrative Parsing Patterns

### Batter safely reaches base
| Outcome | Batter reaches? | Out recorded? |
|---------|----------------|---------------|
| Single / Double / Triple / Home Run | Yes | No |
| Walk / HBP | Yes | No |
| Error (reaches on error) | Yes (no hit credited) | No |
| Fielder's Choice | Yes | Yes — for a base runner, not the batter |
| Double Play | No (batter out) | Yes — 2 outs total |

### Batter is out
| Outcome | Batter reaches? | Out recorded? |
|---------|----------------|---------------|
| Strikeout | No | Yes — 1 out |
| Ground Out / Fly Out / Line Out / Pop Out | No | Yes — 1 out |
| Double Play | No | Yes — 2 outs |

### Reading runner movements from the narrative
Narrative lines follow this pattern:
```
[Batter] [action], [Runner A] [advances to / scores / remains at / out advancing to] [base], [Runner B] ...
```

Key phrases:
- `scores` → that runner crossed home, +1 run
- `advances to 2nd / 3rd / home` → runner moved up
- `remains at 1st / 2nd / 3rd` → runner did not move
- `out advancing to [base]` → runner was thrown out (add 1 out, remove from bases)
- `advances to 2nd on error` → runner advanced due to error (not a put-out)

### Errors
When a batter "reaches on an error", treat them as reaching base (goes to 1st or wherever the narrative says) but **do not credit a hit** in the stats. Record as "Error (reached)" in play type.

### Home Run
Batter and all runners score. Post-state is always `--- [same out count]`.

---

## Step 7 — Stats to Track Per PA

For each PA, record:

| Field | Source |
|-------|--------|
| Batter | Narrative first name-initial + last name → full name via mapping |
| Inning | Which inning (Bottom N) |
| Outs Before | Current out count before this PA |
| Runners Before | Base state before this PA (e.g., `1--`) |
| Play Type | Single / Double / Triple / HR / Walk / K / GO / FO / LO / PO / FC / Error / DP |
| Runs Scored | Count of "scores" in narrative |
| RE Before | Lookup in RE24Table.md |
| RE After | Lookup in RE24Table.md (or 0.000 if inning ended) |
| RE24 | (RE After + Runs Scored) − RE Before |

**Counting stats to update in PlayerStats.md:**
- PA: every batter appearance (all play types above)
- H: Single, Double, Triple, Home Run
- 2B: Double
- 3B: Triple
- HR: Home Run
- BB: Walk or HBP (batter reaches without a hit)
- RBI: runs that score as a result of this batter's PA (including on errors during the PA)
- R: credited when the player themselves crosses home plate (tracked on other batters' PAs)
- RE24: accumulated — add this PA's RE24 to the player's season total

---

## Step 8 — Files to Create / Update

### Create: `Spring2026/Games/[YYYY-MM-DD]_vs_[Opponent].md`

**Header:**
```
# [Date] vs [Opponent]
Final Score: USA Scout 8u Prospects [N] — [Opponent] [N]  (W / L)
```

**PA Table:**
| # | Inning | Batter | Outs | Runners | Play | R | RE Before | RE After | RE24 |
|---|--------|--------|------|---------|------|---|-----------|----------|------|

**Game Summary Table** (per player, this game only):
| Player | # | PA | H | 2B | 3B | HR | BB | RBI | R | RE24 |
|--------|---|----|----|----|----|----|----|-----|---|------|

### Update: `Spring2026/PlayerStats.md`

Column order: `Player | # | PA | H | 2B | 3B | HR | BB | RBI | R | OBP | SLG | OPS | RE24 | RE24/PA`

For each player who had at least one PA in this game:
- Add to their PA, H, 2B, 3B, HR, BB, RBI, R counts
- **Add this game's RE24 to their season RE24 total** (never replace — always add)
- Recalculate derived stats from cumulative totals:
  - OBP = (H + BB) / PA  → format as `.XXX`
  - SLG = (H + 2B + 2×3B + 3×HR) / PA  → format as `.XXX`
  - OPS = OBP + SLG  → format as `.XXX` (may exceed 1.000)
  - RE24/PA = RE24 / PA  → format as `+0.XXX` / `−0.XXX`
- Re-sort the table by RE24 descending
- Update the "Last updated" date in the header

---

### Create/Update: `Spring2026/CurrentSuggestedBattingOrder.md`

Run the batting order algorithm (Step 9) to regenerate the suggested lineup.

---

## Step 9 — Generate Suggested Batting Order

After `PlayerStats.md` is updated, regenerate `Spring2026/CurrentSuggestedBattingOrder.md`.

**Reference:** `Context/StatDrivenLineup.md` defines the 9 roles. This system extends them to 12 spots.

**Speed values** are read from `Spring2026/Roster.md` (1–10 scale; blank = treat as 0 for tiebreaking).

### Algorithm

Pool = all players with PA > 0. Assign spots greedily in order — each player used exactly once.

**Tiebreaker order** (when the primary stat is tied): Speed (higher wins) → RE24/PA (higher wins) → PA (higher wins).
Speed tiebreaker only applies at speed-sensitive spots (1, 6, 9); ignored elsewhere.

```
Spot 1  — The Catalyst        → highest OBP from pool           [tiebreak: Speed, RE24/PA]
Spot 2  — Best All-Around     → highest OPS from remaining       [tiebreak: RE24/PA]
Spot 3  — High-Value Bat      → highest OPS from remaining       [tiebreak: RE24/PA]
Spot 4  — The Cleanup         → highest SLG from remaining       [tiebreak: RE24/PA]
Spot 5  — Second Cleanup      → highest OPS from remaining       [tiebreak: RE24/PA]
Spot 6  — Second Lead-off     → highest OBP from remaining       [tiebreak: Speed, RE24/PA]
Spot 7  — Mid-Range           → highest OPS from remaining       [tiebreak: RE24/PA]
Spot 8  — Lower Tier          → highest OPS from remaining       [tiebreak: RE24/PA]
Spot 9  — Table Setter        → highest OBP from remaining       [tiebreak: Speed, RE24/PA]
Spots 10–11 — Extended Tier   → remaining PA>0 players by OPS descending
Spot 12 — No PA / Pending     → players with 0 PA, by jersey number
```

> "High Defense" (spot 8 in StatDrivenLineup.md) is not tracked in this system — use OPS as the proxy.

### Output Format

```markdown
# Suggested Batting Order — Spring 2026

**Philosophy:** Role-based (Context/StatDrivenLineup.md). Each spot filled by the best
available player for that role's primary stat. Speed (from Roster.md) breaks ties at
speed-sensitive spots (1, 6, 9).
**Updated after:** [Date] vs [Opponent]

| # | Player | Jsy | OBP | SLG | OPS | RE24/PA | Role |
|--:|--------|----:|----:|----:|----:|--------:|------|
| 1 | ...    |     |     |     |     |         | The Catalyst       |
...through 12...

## Rationale

**1. [Player]** — The Catalyst: [one sentence explaining the slot assignment].
...through 12...
```

---

## Quick Reference — RE24 Table

| Base State | 0 Outs | 1 Out | 2 Outs |
|:----------:|:------:|:-----:|:------:|
| `---`      | 0.481  | 0.254 | 0.098  |
| `1--`      | 0.859  | 0.509 | 0.224  |
| `-2-`      | 1.100  | 0.664 | 0.319  |
| `--3`      | 1.351  | 0.950 | 0.362  |
| `12-`      | 1.437  | 0.884 | 0.429  |
| `1-3`      | 1.784  | 1.130 | 0.486  |
| `-23`      | 1.964  | 1.376 | 0.580  |
| `123`      | 2.292  | 1.541 | 0.752  |
