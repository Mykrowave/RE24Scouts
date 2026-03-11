# RE24 Run Expectancy Table

## What Is RE24?
RE24 measures a batter's contribution to run scoring by tracking how much the **expected runs for the remainder of an inning** changes as a result of their plate appearance.

**Formula:** RE24 (per PA) = (RE after PA + runs scored during PA) − RE before PA

- **Positive RE24**: the batter left the team in a better scoring position than they inherited
- **Negative RE24**: the batter left the team in a worse scoring position
- A player's **season RE24** is the sum of all their individual PA RE24 values — it never resets

---

## Base State Notation
Each state is written as three characters representing 1st, 2nd, and 3rd base:

| Symbol | Meaning |
|--------|---------|
| `---`  | Bases empty |
| `1--`  | Runner on 1st only |
| `-2-`  | Runner on 2nd only |
| `--3`  | Runner on 3rd only |
| `12-`  | Runners on 1st and 2nd |
| `1-3`  | Runners on 1st and 3rd |
| `-23`  | Runners on 2nd and 3rd |
| `123`  | Bases loaded |

---

## Run Expectancy Matrix (MLB Averages — Baseline)

> **Note:** These values are derived from MLB data and used as a stable baseline for relative player comparisons. Youth baseball has a higher run environment, so absolute RE24 values will be lower than what the actual run environment produces. Relative comparisons between players on the same team remain valid. This table may be updated with empirical data after a full season.

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

**When an inning ends (3rd out recorded), RE after = 0.000** regardless of base state.

---

## Example Calculation
Situation: runner on 1st, 1 out. Batter hits a double, runner scores.

- RE before = 0.509 (`1--`, 1 out)
- RE after = 0.664 (`-2-`, 1 out) — batter now on 2nd, 1 out still
- Runs scored during PA = 1
- **RE24 = (0.664 + 1) − 0.509 = +1.155**
