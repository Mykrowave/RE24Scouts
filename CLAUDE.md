# RE24 Scouts — Claude Context

## What This Project Is
This project tracks **RE24 hitting statistics** for the **USA Scout 8u Prospects** youth baseball team. Game feeds are provided as markdown play-by-play files. Claude parses these feeds and maintains structured markdown stat files.

**Only players listed in the current season's `Roster.md` are tracked.** The opposing team is never tracked.

---

## RE24 — Key Rule
RE24 is **cumulative per player across all plate appearances for the entire season**. It never resets — not between innings, not between games. Each PA produces a single RE24 value that is added to the player's running season total.

RE24 per PA = (RE after PA + runs scored during PA) − RE before PA

Values come from `Context/RE24Table.md`. Base-out state resets at the start of each half-inning (that's just baseball), but player totals accumulate indefinitely.

---

## Directory Structure
```
RE24Scouts/
├── CLAUDE.md                         # This file
├── Context/
│   ├── RE24Table.md                  # 24-state run expectancy matrix (season-agnostic)
│   └── ParseInstructions.md          # How to parse game feeds and update stats
├── Spring2026/                       # Current season
│   ├── Roster.md                     # Active roster for this season
│   ├── PlayerStats.md                # Season cumulative stats (updated after each game)
│   ├── GameFeed/
│   │   ├── [GameName].txt            # Raw game feed (as provided)
│   │   └── [GameName]_compact.txt    # Compact feed generated in Step 0
│   └── GameBreakDown/
│       └── [GameName].md             # Per-game PA-by-PA detail (same name as feed, .md)
└── GameFeeds/
    └── ExampleGame.md                # Reference example only
```

---

## Current Season
**Spring2026** — all season files live in `Spring2026/`.

To start a new season: create a new `SeasonYear/` folder with a fresh `Roster.md` and `PlayerStats.md`, and new `GameFeed/` and `GameBreakDown/` subfolders.

---

## Processing a New Game
See `Context/ParseInstructions.md` for the complete step-by-step guide.

**Short version:**
1. Read the game feed
2. Find all `Bottom X - USA Scout 8u Prospects` or `Top X - USA Scout 8u Prospects` half-innings
3. For each PA in those half-innings, calculate RE24
4. Create `Spring2026/GameBreakDown/[GameName].md` with PA-by-PA detail
5. Update `Spring2026/PlayerStats.md` with cumulative totals
