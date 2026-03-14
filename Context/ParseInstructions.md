# Game Feed Parse Instructions

## Overview

This document explains how to parse a game feed and produce three outputs:
1. A compact feed file at `Spring2026/Games/[original_filename]_compact.txt`
2. A per-game detail file at `Spring2026/Games/[YYYY-MM-DD]_vs_[Opponent].md`
3. Updated cumulative stats in `Spring2026/PlayerStats.md`

**Two-pass process:** Step 0 converts the raw feed to compact format. Steps 1–5 operate on that compact file — never re-read the raw feed after Step 0.

Only process **our team's half-innings**. Our team is **USA Scout 8u Prospects**.

---

## Step 0 — Convert Raw Feed to Compact Format

Read the raw game feed. Identify our half-innings (headers containing `USA Scout`). For each PA in our half-innings only, emit **one line** in this format:

```
Inn | Batter | Play | Outs_bef | State_bef | Outs_aft | State_aft | R | Scorers
```

### Column Definitions

| Column     | Values                                          | Notes |
|:-----------|:------------------------------------------------|:------|
| Inn        | B1, B2… or T1, T2…                             | Bottom/Top + inning number, our half-innings only |
| Batter     | Abbreviated name (e.g., `P Morrison`)           | From feed, as-is |
| Play       | 1B / 2B / 3B / HR / BB / HBP / K / GO / FO / LO / PO / FC / Err / DP | From play-type line |
| Outs_bef   | 0, 1, or 2                                      | Outs before this PA |
| State_bef  | --- / 1-- / -2- / --3 / 12- / 1-3 / -23 / 123  | Bases occupied before this PA |
| Outs_aft   | 1, 2, or 3                                      | Outs after; **3 = inning ended** |
| State_aft  | same notation                                   | Bases after; use `---` when inning ends |
| R          | integer                                         | Runs scoring on this PA |
| Scorers    | comma-separated abbreviated names               | Only when R > 0; omit when R = 0 |

### Rules

- **Skip all opponent half-innings entirely.**
- The **narrative line** is authoritative. Ignore pitch sequences and most score-update lines.
- Track base state and outs cumulatively within each half-inning, resetting to `--- / 0 outs` at the start of each.
- When Outs_aft = 3, State_aft is always `---` regardless of runners on base.
- "Inning Ended by scorekeeper" — treat as if the inning ended normally on the last PA shown.
- Scorers = players (abbreviated) who cross home plate on this PA (for R and RBI attribution).

### Outcome Reference

| Play-type line          | Batter reaches? | Outs added |
|:------------------------|:---------------:|:----------:|
| Single / Double / Triple / HR | Yes       | 0          |
| Walk / HBP              | Yes             | 0          |
| Error (reaches)         | Yes (no H)      | 0          |
| Fielder's Choice        | Yes             | 1 (runner) |
| Strikeout / GO / FO / LO / PO | No       | 1          |
| Double Play             | No              | 2          |

### Example Compact Lines

```
B1 | P Morrison | GO  | 0 | --- | 1 | ---  | 0
B1 | L Meyer    | 1B  | 1 | --- | 1 | 1--  | 0
B1 | B Parker   | Err | 1 | 1-- | 1 | 12-  | 0
B1 | C Griffin  | PO  | 1 | 12- | 2 | 12-  | 0
B1 | J Ham      | BB  | 2 | 12- | 2 | 123  | 0
B1 | H Steed    | K   | 2 | 123 | 3 | ---  | 0
B2 | C Hayes    | 1B  | 0 | 123 | 0 | -23  | 2 | B Hipp, L Watkins
```

**Save to:** `Spring2026/GameFeed/[original_filename]_compact.txt`

---

## Step 1 — Map Player Names

Compact feed uses abbreviated names. Map to full names:

| Feed Name   | Full Name        |  # |
|:------------|:-----------------|---:|
| L Brown     | Liam Brown       |  1 |
| C Hayes     | Cooper Hayes     |  2 |
| B Herrera   | Balyn Herrera    |  3 |
| H Steed     | Henry Steed      | 11 |
| B Hipp      | Benjamin Hipp    | 13 |
| B Parker    | Baxter Parker    | 14 |
| L Watkins   | Levi Watkins     | 23 |
| P Morrison  | Porter Morrison  | 27 |
| J Ham       | Jase Ham         | 28 |
| C Newton    | Cooper Newton    | 42 |
| C Griffin   | Charlie Griffin  | 44 |
| L Meyer     | Logan Meyer      | 99 |

Names not in this table are opposing players — ignore them.

---

## Step 2 — Calculate RE24 for Each PA

Work through the compact feed line by line:

```
RE24 = (RE_after + R) − RE_before
```

- **RE_before**: look up `State_bef` + `Outs_bef` in the RE24 table below
- **RE_after**: look up `State_aft` + `Outs_aft`; **use 0.000 when Outs_aft = 3**
- **R**: taken directly from the compact feed

### RE24 Table

| State | 0 out | 1 out | 2 out |
|:-----:|------:|------:|------:|
| ---   | 0.481 | 0.254 | 0.098 |
| 1--   | 0.859 | 0.509 | 0.224 |
| -2-   | 1.100 | 0.664 | 0.319 |
| --3   | 1.351 | 0.950 | 0.362 |
| 12-   | 1.437 | 0.884 | 0.429 |
| 1-3   | 1.784 | 1.130 | 0.486 |
| -23   | 1.964 | 1.376 | 0.580 |
| 123   | 2.292 | 1.541 | 0.752 |

---

## Step 3 — Counting Stats Per PA

| Stat | Increment when |
|:-----|:---------------|
| PA   | always (every line) |
| H    | Play = 1B, 2B, 3B, or HR |
| 2B   | Play = 2B |
| 3B   | Play = 3B |
| HR   | Play = HR |
| BB   | Play = BB or HBP |
| RBI  | R > 0 on this PA — batter gets credit for all R on their PA |
| R    | player's abbreviated name appears in Scorers on any line |
| RE24 | add this PA's RE24 to the player's running season total |

---

## Step 4 — Files to Create / Update

### A. Create: `Spring2026/GameBreakDown/[original_filename].md`

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

### B. Update: `Spring2026/PlayerStats.md`

Column order: `Player | # | PA | H | 2B | 3B | HR | BB | RBI | R | OBP | SLG | OPS | RE24 | RE24/PA`

- Add game totals to each player's cumulative counts.
- **Add** this game's RE24 to their season total (never replace).
- Recalculate derived stats from cumulative totals:
  - OBP = (H + BB) / PA → format `.XXX`
  - SLG = (H + 2B + 2×3B + 3×HR) / PA → format `.XXX`
  - OPS = OBP + SLG → format `.XXX` (may exceed 1.000)
  - RE24/PA = RE24 / PA → format `+0.XXX` or `−0.XXX`
- Re-sort by RE24 descending.
- Update "Last updated" date in the header.

### C. Create/Update: `Spring2026/CurrentSuggestedBattingOrder.md`

Run Step 5 to regenerate the suggested lineup.

---

## Step 5 — Generate Suggested Batting Order

After `PlayerStats.md` is updated, regenerate `Spring2026/CurrentSuggestedBattingOrder.md`.

**Reference:** `Context/StatDrivenLineup.md` defines the 9 roles. This system extends them to 12 spots.
**Speed values:** from `Spring2026/Roster.md` (1–10 scale).

### Algorithm

Pool = all players with PA > 0. Assign spots greedily — each player used exactly once.

**Tiebreaker order:** Speed (higher wins) → RE24/PA (higher wins) → PA (higher wins).
Speed tiebreaker applies only at speed-sensitive spots (1, 6, 9).

```
Spot 1       — The Catalyst       → highest OBP from pool              [tiebreak: Speed, RE24/PA]

Spots 2–4 filled as a GROUP:
  Step A: Select top 3 remaining players by RE24/PA → "core group"
  Step B: Assign within core group:
    Spot 2   — Best All-Around    → highest OPS in core group           [tiebreak: RE24/PA]
    Spot 3   — High-Value Bat     → next-highest OPS in core group      [tiebreak: RE24/PA]
    Spot 4   — The Cleanup        → remaining core group player

Spot 5       — Second Cleanup     → highest OPS remaining               [tiebreak: RE24/PA]
Spot 6       — Second Lead-off    → highest OBP remaining               [tiebreak: Speed, RE24/PA]
Spot 7       — Mid-Range          → highest OPS remaining               [tiebreak: RE24/PA]
Spot 8       — Lower Tier         → highest OPS remaining               [tiebreak: RE24/PA]
Spot 9       — Table Setter       → highest OBP remaining               [tiebreak: Speed, RE24/PA]
Spots 10–11  — Extended Tier      → remaining PA>0 players by OPS desc
Spot 12      — No PA / Pending    → players with 0 PA, by jersey number
```

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
