# Game Theory FIFA ⚽ — a game-theory football playground

> A World Cup predictor where the tactics aren't hidden inside the model. You can watch a Nash
> equilibrium *form*, duel the engine, take a nation through a tournament, and see for yourself why
> no single tactic ever wins.

Most match predictors are a black box: feed in two teams, get back a percentage. Game Theory FIFA treats a
match as what it really is — **a game between two managers**, where the best tactic for one depends
on what the other does — and puts that reasoning on screen. It's a single self-contained HTML file,
no build step, no dependencies.

---

## Contents

- [The five modes](#the-five-modes)
- [The game theory (the actual point)](#the-game-theory-the-actual-point)
- [How the engine works](#how-the-engine-works)
- [Accuracy: validated on the 2022 World Cup](#accuracy-validated-on-the-2022-world-cup)
- [Run it](#run-it)
- [Project files](#project-files)
- [Roadmap](#roadmap)
- [Notes & disclaimer](#notes--disclaimer)

---

## The five modes

**1. Predict.** Pick two teams, get win/draw/loss odds, the most likely scoreline, and — the part
other predictors hide — each side's *optimal tactical mix*.

**2. Manager Duel — "Can you beat Nash?"** You're the manager. Pick a tactic each round; the engine
answers. Cumulative expected goal difference is the score over 10 rounds. The engine's brain is
switchable, and that's where the lesson lives:
- **Nash** — unexploitable; you drift around zero no matter what you do.
- **Adaptive** — reads your pattern and punishes it. Spam Attack and it parks the bus and counters.
- **Guardiola / Mourinho / Klopp** — fixed, *exploitable* styles. Once you spot Mourinho parking the
  bus 60% of the time, you play Balanced to unlock him and win systematically.

The gap between "can't beat Nash" and "can beat Mourinho" is the whole idea of equilibrium, made
playable.

**3. Watch Nash Emerge.** An animation of *fictitious play*: both managers start 100% committed to
one tactic, then learn by best-responding, and you watch the bars converge to the equilibrium with a
live iteration counter. Very few projects visualize an algorithm actually converging.

**4. Battlefield.** An animated rock-paper-scissors triangle next to a colour-coded payoff heatmap
for any matchup. Hover a cell for the expected goal difference and a plain-English reason. It shows
*why* no tactic dominates — so mixing is forced.

**5. Campaign.** Take a nation through a randomized 16-team knockout. Before each tie you see your
opponent's likely tactics in a "press conference," choose your counter, and the match resolves with a
real sampled scoreline (ties go to penalties). Advance R16 → QF → SF → Final, with a live path
tracker, and lift the trophy if you win four. Pick a giant for ~1-in-5 title odds; an underdog can
still pull off a penalty-shootout Cinderella run.

---

## The game theory (the actual point)

A naive model says "Argentina are stronger, so Argentina win." But managers make *choices*, and the
right choice is conditional — and your opponent is reasoning about you at the same time. When both
sides choose best responses to each other simultaneously, you're no longer doing arithmetic; you're
solving a game.

### Three tactics, no winner — a cycle

Each manager picks one of three tactics, wired so none is always best:

| If you play… | you beat… | but lose to… |
|---|---|---|
| **Attack** | Balanced (you overrun them) | Park the bus (they counter) |
| **Balanced** | Park the bus (you control a passive side) | Attack |
| **Park the bus** | Attack (you absorb and break) | Balanced |

Because the advantage runs in a circle (rock-paper-scissors), there is usually **no dominant pure
strategy** — which is exactly when a *mixed* strategy (a probability over tactics) is the right
answer.

### The payoff matrix and the equilibrium

Each cell is the home side's **expected goal difference** for a tactic pairing, built from the two
teams' Elo-based expected goals and the cycle above. The game is zero-sum on goal difference, so an
equilibrium is guaranteed to exist.

**Worked example — Argentina vs France** (Elo 2105 vs 2085 → baseline 1.35 vs 1.25 goals):

|                | FRA Attack | FRA Balanced | FRA Park bus |
|----------------|-----------:|-------------:|-------------:|
| **ARG Attack**   | +0.16 | **+0.63** | −0.15 |
| **ARG Balanced** | −0.38 | +0.10 | **+0.37** |
| **ARG Park bus** | **+0.32** | −0.23 | +0.04 |

No row is always best (the bolded best responses jump around), so the solution is a genuine mix:

- **Argentina:** 26% Attack · 28% Balanced · **45% Park the bus**
- **France:** 32% Attack · 17% Balanced · **51% Park the bus**

Two elite, evenly matched sides should both lean cautious and counter-punching — exactly how cagey
knockout football between top teams tends to look.

### Solving it: fictitious play

Rather than linear programming, the equilibrium is found with **fictitious play**: each manager keeps
a tally of what the other has done, and on each round plays the single best response to that tally.
For a zero-sum game this converges, and the long-run tactic frequencies *are* the Nash equilibrium.
The "Watch Nash Emerge" mode animates this exact process.

---

## How the engine works

```
Elo ratings ─► baseline expected goals
                     │
                     ▼
        a 3×3 tactical game between the two managers
                     │
                     ▼
        Nash equilibrium  (fictitious play)
                     │
                     ▼
        equilibrium expected goals  (λ_A, λ_B)
                     │
                     ▼
        Poisson scoreline grid + Dixon–Coles draw correction
                     │
                     ▼
        Win / Draw / Loss probabilities + most likely scoreline
```

Key constants (all tunable in the source): Elo→goals slope `210`, home edge `+20` Elo, Dixon–Coles
`rho = −0.11`. The Dixon–Coles term nudges probability back into the realistic low-scoring draws
(0–0, 1–1) that an independent Poisson model under-predicts.

---

## Accuracy: validated on the 2022 World Cup

The engine was backtested neutral-venue over **37 verified 2022 matches** (the full knockout bracket
plus the key group games, including the upsets). Two versions are compared: **v1** (hand-set
attack/defence ratings + plain Poisson) and **v2** (Elo ratings + Dixon–Coles).

| Metric | v1 | v2 |
|---|---:|---:|
| Outcome accuracy (W/D/L) | 59.5% | **62.2%** |
| — group stage | 57.1% | **61.9%** |
| — knockout (90') | 62.5% | 62.5% |
| Avg Brier score (lower = better) | 0.561 | **0.560** |

**Honest reading:** the Elo ratings did the heavy lifting on accuracy. Dixon–Coles improved the
*realism* of scorelines (Spain–Germany and several knockout ties now correctly read as 1–1) but
rarely flips a favoured side to a "draw" pick, so it helps calibration more than the headline number.
And 37 games from the most upset-heavy World Cup on record is a small sample — a trustworthy read
needs many tournaments. Full match-by-match detail and the improvement roadmap are in
`game-theory-fifa-accuracy.xlsx`.

---

## Roadmap

Built so far: Predict, Manager Duel / Can-You-Beat-Nash, Watch Nash Emerge, Battlefield, Campaign,
Elo + Dixon–Coles engine, animated pitch background.

Next up (the engine is structured to take these on top of the same core):

- **5-formation strategy layer** — turn the 3×3 game into a richer 5×5 (4-3-3 press, 5-4-1 low
  block, etc.).
- **Historical Upset explorer** — replay famous shocks and find the tactical equilibrium that made
  them possible.
- **Match-replay commentary generator** — turn a predicted scoreline into a minute-by-minute story.
- **Blend with bookmaker odds** — the cheapest real accuracy gain (a strong ensemble baseline).

---

## Notes & disclaimer

Elo ratings here are illustrative estimates on a World-Football-Elo-style scale, not a live feed. The
model predicts the 90-minute result; knockout ties are settled by a simple shootout model. This is a
**toy for exploring game theory**, not betting advice.

Built with vanilla HTML/CSS/JS. The interesting bits — fictitious play, the payoff game,
Dixon–Coles, and the Monte Carlo campaign — are all in the single `<script>` block of
`game-theory-fifa.html`, readable top to bottom.

© 2026 **wuisabel-gif** · [github.com/wuisabel](https://github.com/wuisabel-gif)
