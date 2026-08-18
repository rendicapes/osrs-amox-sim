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

Build marker `POISONCOUNTER-V16` (earlier markers `CUREATDUMP-V15`, `FORCEDOFF-V14`,
`DUMPSEED-V13` are also in the file).

At the stock 23 antelope the engine reaches **118 recoil**. Recoil rises monotonically with food
carried, and the modelled kill lands at **115 antelope** across 40/40 seeds.

The theoretical ceiling with stock supplies is **319** against the boss's 400, so the fight is not
winnable on the current plan without more consumables — that gap is the open problem, not a bug.

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
