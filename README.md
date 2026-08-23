# Amoxliatl fight simulators — combat level 3 and prayer-pure

Two single-file, dependency-free tick simulators for the **Amoxliatl** fight in Old School RuneScape,
both built for accounts that cannot attack and therefore deal all damage through **ring of recoil**.

| file | account | boss | state |
|---|---|---|---|
| `osrs-amox-sim.html` | combat level 3, 10 hitpoints | quest variant, 400 hp | kill modelled at 115 antelope; stock supplies fall short |
| `osrs-prayer-pure-sim.html` | prayer-only pure, 10 hitpoints | quest 400 hp and post-quest 520 hp | **both routes 100%** |

They are separate engines and share no code. A finding in one does **not** transfer to the other
without re-measuring — see the prayer-pure section below for a case where exactly that assumption
produced a phantom bug.

---

## The level 3 simulator — `osrs-amox-sim.html`

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

`AMOXLIATL-START-HERE.md` carries the level 3 investigation state — the settled mechanics, the deaths
that were found and closed, what has been measured and reverted, and the open questions.
`PATCH-NOTES.md` is the prayer-pure sim's versioned log, newest first, and every reversal in it names
what it reverses.

---

## The prayer-pure simulator — `osrs-prayer-pure-sim.html`

A prayer-only pure at **10 hitpoints**, which is a different fight: Redemption becomes available, and
so does the **post-quest boss at 520 hitpoints** that the level 3 account cannot approach at all.

Two routes ship, both at 100%:

| | QUEST | POST-QUEST |
|---|---|---|
| route | floor tank | Redemption |
| loop | 13 ↔ 14 shuttle | room loop 16 → 20 → 26 → 22 |
| boss hp | 400 | 520 |
| kills | **160/160** | **100%** |
| time | ~30 min | ~45 min |
| slots | 28 of 28 | 23 of 28 |
| nightshade | **0** | 473 |

`blockRisk 0` on every run in every block quoted here. That counter is not a score — a non-zero value
means the player left tiles 1-9 on a special's fire tick, which spawns an ice block that is melee-only
to destroy, and melee is combat experience this account cannot take. Any run reading non-zero is
**invalid**, not ranked lower.

### The acceptance sweep — post-quest, and it is the interesting one

Five disjoint seed blocks, 180 runs (201-260, 401-420, 501-540, 1201-1230, 1601-1630):

| `stallClickTax` | `restSlots` | kills | median kill | nightshade |
|---|---|---|---|---|
| 1 *(the bug)* | 0 | 180/180 · 100% | 38.8 min | 464 |
| 0 *(correct)* | 0 | 147/180 · 81.7% | 44.6 min | 470 |
| **0** | **2** *(shipped)* | **100%** | **44.6 min** | **473** |

The bottom row is validated on a **holdout of 140 seeds never used to tune it**: 140/140.

### Why that middle row exists, because it is the whole lesson

The engine carried a parameter that added **one tick of stall duration to every arm**, introduced on a
misreading of how the stall window worked. It was caught, and removing it dropped the route from 100%
to 82%. Every one of the 33 deaths was at the **same tick**, byte-identical across seeds:

> `t96  → tile 7 ⚠ nowhere safe to stop | floor 10 · recoil 1 | ☠ DEATH, 10 landed against 10 hitpoints`

Post-quest floor damage rolls **7-10 against 10 maximum hitpoints**. Maximum floor damage is *exactly*
maximum health, so one unprotected tick on live ice is a one-in-four death from full. The plan has no
margin by construction, and the phantom tick had been covering the single tick where margin was needed.

The fix was two slots of Guthix rest — a rest overheals the ceiling to 15, so a 10 stops being lethal.
**The rests had been dropped earlier because a sweep of 12/8/6/4/2 came back 40/40 at every value** —
run on the engine with the phantom tick. Each was load-bearing for the same tick, so each masked the
other and neither looked required.

Two things worth carrying out of that:

- **A bound in a chooser is not a statement about the machine.** The claim that a stall covered attacks
  only from `T+1` came from an eligibility bound inside the stall *planner*, not from the engine, which
  had been deferring arm-tick attacks all along. Relaxing the bound produces bit-identical runs — the
  branch is unreachable, because the planner never schedules an arm on an attack tick.
- **A parameter that sweeps flat is not proven unnecessary; it may be masked by a bug.** After fixing an
  engine fault, re-sweep everything that was tuned under it, and be most suspicious of whatever swept
  flat.

Both stall questions were settled by in-game observation rather than from the desk: **a stall defers the
floor damage *and* the attack of the tick it is armed on** — one window, `[T … T+L−1]`, L ticks for an
L-tick stall. Full history in `PATCH-NOTES.md` under v15 through v18.

### Reproducing the numbers

The engine is pure and runs headlessly. Load the page under `jsdom` with `runScripts: 'dangerously'`,
then inject a second `<script>` to expose the bindings — they are top-level `const`/`let` in a classic
script, so they are not on `window`:

```js
const s = doc.createElement('script');
s.textContent = `window.__H = {P, simulate, mulberry32, applyPreset, setRNG: f => { RNG = f; }};`;
doc.body.appendChild(s);
```

Then per run: `applyPreset('postquest')` → override parameters **after** the preset (it resets them all)
→ `setRNG(mulberry32(seed))` → `simulate(false)`. Use `simulate(true)` only for a tick sheet; it builds
thousands of frames. Roughly 1.5 s a run.

---

## Caveat

The numbers are the **simulators'**, not a promise. Every antelope count and every nightshade count
quoted in this project's history has been withdrawn at least once when a mechanic turned out to be
measured wrong — the 82% row above is one of them, and it survived a version release. Treat any figure
here as the current best model, and re-measure before spending anything on it.
