# RE24 Scouts

A stat-tracking system for the **USA Scout 8u Prospects** youth baseball team. Game feeds are parsed into RE24-based hitting statistics, giving a more meaningful picture of each player's offensive contribution than traditional counting stats alone.

---

## What is RE24?

**RE24** (Run Expectancy based on 24 base-out states) measures how much a plate appearance changed the team's expected run output.

Baseball has 24 possible base-out states — 8 base configurations (nobody on, runner on 1st, runners on 1st and 2nd, etc.) crossed with 3 out counts (0, 1, 2). Each state has a known *run expectancy* — the average number of runs a team will score from that point to the end of the inning. RE24 is the delta a batter creates:

```
RE24 per PA = (RE after PA + runs scored during PA) − RE before PA
```

A walk with the bases loaded and 2 outs is worth far more than a leadoff walk — RE24 captures that. A strikeout with nobody on is far less damaging than one with the bases loaded — RE24 captures that too. It rewards batters for what they actually *did to the scoreboard*, not just whether they got a hit.

RE values come from MLB long-run averages (see `Context/RE24Table.md`). RE24 accumulates across all plate appearances for the entire season — it never resets between games.

---

## How It Works

### 1. Game Feeds
Raw game feeds are plain-text play-by-play files dropped into `Spring2026/GameFeed/`. Each feed covers one game and includes every at-bat with a narrative description of what happened.

### 2. Compact Feed (Step 0)
Before any RE24 math is done, the raw feed is distilled into a compact one-line-per-PA format — our half-innings only, pitch sequences stripped, just the essential state information. This file is also saved in `GameFeed/` with a `_compact` suffix.

### 3. RE24 Calculation
Working from the compact feed, each plate appearance is evaluated: base-out state before, base-out state after, runs scored. RE24 is calculated and attributed to the batter.

### 4. Output Files
- **`GameBreakDown/[GameName].md`** — full PA-by-PA table with RE24 for every plate appearance, plus a per-player game summary
- **`PlayerStats.md`** — cumulative season stats for all 12 players, sorted by RE24
- **`CurrentSuggestedBattingOrder.md`** — a stat-driven batting order generated after every game

---

## Directory Structure

```
RE24Scouts/
├── README.md
├── CLAUDE.md                              # AI assistant instructions
├── Context/
│   ├── RE24Table.md                       # 24-state run expectancy matrix
│   ├── ParseInstructions.md               # Step-by-step game processing guide
│   └── StatDrivenLineup.md                # Batting order role philosophy
├── Spring2026/
│   ├── Roster.md                          # 12 players + speed ratings
│   ├── PlayerStats.md                     # Season cumulative stats
│   ├── CurrentSuggestedBattingOrder.md    # Latest suggested lineup
│   ├── GameFeed/
│   │   ├── [GameName].txt                 # Raw game feed
│   │   └── [GameName]_compact.txt         # Compact feed (one PA per line)
│   └── GameBreakDown/
│       └── [GameName].md                  # Per-game PA-by-PA detail
└── GameFeeds/
    └── ExampleGame.md                     # Reference example
```

---

## Stats Tracked

| Stat | Description |
|------|-------------|
| PA | Plate appearances |
| H | Hits (1B, 2B, 3B, HR) |
| 2B / 3B / HR | Extra-base hits |
| BB | Walks and hit-by-pitch |
| RBI | Runs batted in |
| R | Runs scored |
| OBP | On-base percentage = (H + BB) / PA |
| SLG | Slugging = (H + 2B + 2×3B + 3×HR) / PA |
| OPS | OBP + SLG |
| RE24 | Season cumulative run expectancy added |
| RE24/PA | RE24 per plate appearance (rate stat) |

---

## Suggested Batting Order

After each game, a batting order is generated using a role-based algorithm defined in `Context/StatDrivenLineup.md`. The system places the highest on-base players at the top and bottom of the order, clusters the highest run-producing batters (by RE24/PA) in spots 2–4, and uses speed as a tiebreaker at speed-sensitive spots (1, 6, 9).

---

## Current Season

**Spring 2026** — files live in `Spring2026/`. The roster has 12 players.
