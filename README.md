# Amoxliatl fight simulator — level 3 / 10 hitpoints

A single-file, dependency-free tick simulator for the **Amoxliatl** quest-variant fight in Old School
RuneScape, built for a **combat level 3, 10 hitpoint** account that deals damage **only through
recoil**.

Open `osrs-amox-sim.html` in any browser. No build step, no dependencies, no network calls — the
whole engine and UI are one inline script.

## What it models

The account cannot attack (attacking grants combat experience, which is the one thing the run cannot
undo), so all 400 of the boss's hitpoints have to come from **ring-of-recoil damage** — one point per
hit absorbed. That turns the fight into a scheduling problem: how many damage instances can be
absorbed with the healing an inventory can carry.

Simulated tick by tick:

- the attack rhythm (autos every 8 ticks, 3 per cycle, then a special)
- the arena floor — **two** distinct sources with different timings and caps (a spike at auto+3, and
  a pattern from auto+7 to auto+104), including which one suppresses the other
- **stalls** — inventory items that defer queued damage with the roll cap frozen at your hitpoints
  when the stall was armed
- the hunter-food cycle, the poison/cure cycle, and the phoenix-necklace phase
- poison as an internal counter, movement, eat cooldowns, and the one-tick click delay

## Current state

Engine marker `POISONCOUNTER-V16`, page marker `DIAGPANEL-V16A` (earlier markers `CUREATDUMP-V15`,
`FORCEDOFF-V14`, `DUMPSEED-V13` are also in the file).

At the stock 23 antelope the engine reaches **118 recoil**. Recoil rises monotonically with food
carried, and the modelled kill lands at **115 antelope** across 40/40 seeds.

The acceptance sweep, 40 seeds — if this row does not reproduce, the build is not this one:

| carried | 23 | 40 | 60 | 80 | 100 | 110 | 115 | 120 |
|---|---|---|---|---|---|---|---|---|
| recoil | 118 | 176 | 207 | 272 | 373 | 394 | **400 — 40/40 kill** | 401 |

The theoretical ceiling with stock supplies is **319** against the boss's 400, so the fight is not
winnable on the current plan without more consumables — that gap is the open problem, not a bug.

## Diagnostics

Under **Route** there is a diagnostics panel with three readouts, all running against the same batch
loop as the Monte Carlo (`mulberry32(i+1)`, `simulate(false)`):

- **Ceiling vs achieved** — food / cure / null, what the consumables allow against what the run
  actually banked, with the working shown for each.
- **Death causes** — a tally across N seeds giving the tick, hitpoints, tile, and the **age of every
  ice patch on that tile**. The age names the source: age 3 is a spike worth up to 8, age 7–104 is
  the floor pattern worth up to 5, anything else means the attack killed you. It also splits deaths
  into "bag empty" (resource exhaustion) and "supplies left" (worth diagnosing).
- **Stall usage** — arms per length per phase, and average instances against the capacity they were
  planned against.

The engine appends to the arm log and the death record and **never reads either back**, so the panel
cannot move a result. Verified by running the previous build and this one side by side in one
process: identical at every food count over 100 seeds.

## Routes

Four are shipped: three individual phases (cure stall, delayed heal, phoenix null) and
**the level 3 plan** — cures, then delayed food, then nulls, then RNG — which runs them in turn.
Running a phase alone is how you read that phase in isolation.

Five exploratory routes (ice cycle, tank at the ceiling, greedy, kite, CCQ burn) were removed once
the plan settled; they are in the git history. Each was a baseline for a question since answered —
her 16 max hit against a 15 hitpoint ceiling closes tanking outright, and floor space is not the
binding constraint, so kiting is not a route in its own right.

## Reading the code

The inline script is heavily commented, and the comments are the actual record: each one explains a
mechanic that was **measured in game** and what the engine got wrong before it. Anything marked ★ is
a practitioner correction and takes precedence over documentation.

`AMOXLIATL-START-HERE.md` carries the investigation state — the settled mechanics, the deaths that
were found and closed, what has been measured and reverted, and the open questions.

## Caveat

The numbers are the **simulator's**, not a promise. Every antelope count quoted in this project's
history has been withdrawn at least once when a mechanic turned out to be measured wrong. Treat any
figure here as the current best model, and re-measure before spending anything on it.
