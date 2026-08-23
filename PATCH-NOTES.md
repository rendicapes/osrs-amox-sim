# PATCH NOTES, AMOXLIATL PRAYER-PURE SIM

**For the video timeline.** Newest first. Every number here is measured on disjoint seed blocks with
`blockRisk 0`, and every reversal names what it reverses.

---

## 23 AUG · v18, ★★★ THE 82% WAS A MIS-FITTED PLAN, NOT THE PHYSICS. BACK TO 100% FOR TWO SLOTS.

Rendi, on the v16 result: *"why did it go down, the free tick should have helped not hurt."* Right
instinct, and chasing it found the actual answer.

### ★ WHY REMOVING THE TAX HURT: IT WAS NEVER A TIMING OFFSET

`st.stall = len + p('stallClickTax')`. The tax did not *shift* the window — it **added a tick of stall
duration to every single arm**, and the planner's `hi = A+L-1+_TAX` / `dump = A+L+_TAX` made every
candidate stall *look* a tick longer too, so the choice set was bigger. Taking it away removes a real
tick of protection from ~376 arms a kill.

**And it changes which plan gets picked, not just its length.** Same seed, same everything else:

| | tax 1 | tax 0 |
|---|---|---|
| arms at | t95, **3-tick stall on tile 4** | t94, **1-tick stall on tile 14** |
| Redemption fires | t99 | t95 |
| t96 | still stalled, feet locked, safe | **"nowhere safe to stop" → forced to tile 7, live ice** |
| outcome | kill at t3937 | **dead at t96** |

### ★★ THE AUTOPSY: ALL 33 DEATHS ARE ONE EVENT

Not a survival tail. **Every death in all 180 runs is at tick 96**, and the traces are byte-identical
across seeds. The kill:

> `t96  → tile 7 ⚠ nowhere safe to stop | floor 10 · recoil 1 | ☠ DEATH, 10 landed against 10 hitpoints`

**Post-quest floor damage rolls `icePatMin 7` to `icePatMax 10`, against `hpBase 10`.** The maximum
floor roll is *exactly* maximum hitpoints, so **one unprotected tick on live ice is a 1-in-4 death from
full health.** The plan has zero margin by construction; the phantom tick was covering the one tick
where that margin was needed.

### ★★★ THE FIX: `restSlots` 0 → 2. NOTHING ELSE.

A Guthix rest overheals to `hpCeil 15`, so a floor 10 stops being lethal.

| `restSlots` | kills (tuning block 201-240) |
|---|---|
| **0** (shipped) | 34/40 · 85.0% |
| **2** | **40/40 · 100%** |
| 4 | 40/40 · 100%, +7 nightshade |
| 8 | 40/40 · 100%, +15 nightshade |

**Validated on a holdout of 140 seeds never used to pick it** (241-260, 401-420, 501-540, 1201-1230,
1601-1630):

| | kills | nightshade | median kill | arms | blockRisk |
|---|---|---|---|---|---|
| `restSlots 0` | 113/140 · 80.7% | 470 | 44.6 min | 376 | 0 |
| **`restSlots 2`** | **140/140 · 100%** | **473** | 44.6 min | 376 | **0** |

**Cost: two inventory slots and three nightshade.** No time cost, no arm-count change, `blockRisk 0`.
Bag goes 22 → **24 of 28**, four spare. Two is the cheapest value that works; more rests only cost
nightshade.

### ⛔ AND `graceMobile 1` IS CATASTROPHIC. SWEPT, REVERTED.

The walk-aware descent check looked like the obvious answer to *"nowhere safe to stop"*. It is not:
**0/40, median arm count collapses 376 → 8.** Reverted, untouched at 0.

### The lesson, and it is the useful one

**The rests were dropped at v5 because a sweep said they were unnecessary — on the engine with the
phantom tick.** The free tick had been standing in for the Guthix rest all along, and nobody could see
it because both were load-bearing for the same single tick.

**A parameter that sweeps flat is not proven unnecessary; it may be masked by a bug.** When an engine
fault is fixed, every parameter that was swept *under* the fault has to be re-swept, not inherited —
the v16 conclusion ("the cost of the fix is reliability and time") was wrong for exactly this reason,
and it was wrong because I re-measured the route without re-tuning it.

---

## 23 AUG · v17, ★★ v16'S OPEN QUESTION CLOSED IN ONE LINE. NO NUMBER MOVES.

v16 left one thing open: does an **auto** also defer when the stall is armed on its tick, or only the
floor? I put **~55% on "it lands"**, reasoning that damage is computed when an attack is *fired*, so an
auto already in flight was never deferrable.

> **Rendi, authoritative, 23 Aug: "it defers yes like every other stall."**
> **And: "the stalls initiate on the window they're open."**

Wrong by 55 points, and that is **23 out of 23** for the practitioner over the model. There is **no
asymmetry**. Floor and attacks share **one** window:

| | window | ticks for an L-tick stall |
|---|---|---|
| floor **and** attacks | `[T … T+L−1]`, arm tick inclusive | L |

So **a 1-tick stall does swallow the next auto** — Rendi's original 22 Aug intuition, the one the click
tax was invented to satisfy. The intuition was right and the fix was wrong: **the engine was already
delivering it**, and the tax bought nothing except a tick of floor it was not entitled to.

### ☑ AND IT CHANGES NOTHING ON THE PAGE — CHECKED, NOT ASSUMED

Rendi's read on hearing the plan: *"pretty sure this is already worked in... don't think it changes
anything on our current sim."* Correct on both halves.

**The engine already defers the arm tick's auto.** In one tick iteration the order is: decrement
`st.stall` → **ARM** → resolve the boss. The boss block reads `stalled = st.stall > 0`, so a stall armed
on T is live for T's own auto.

**The only `A+1` left is a conservative *planner* bound** (`const lo = A+1` in `consider`), which merely
declines to *count* an arm-tick auto as covered, so the planner arms a tick earlier than it strictly
must. Relaxing it to `A` and re-running:

| | kills | median nightshade | median kill | median arms |
|---|---|---|---|---|
| shipped, `lo = A+1` | 43/50 · 86.0% | 470 | 44.6 min | 376 |
| scratch, `lo = A` | 43/50 · 86.0% | 470 | 44.6 min | 376 |

**0 of 50 runs differ**, run for run, on kill / ticks / nightshade / arm count — 50 seeds across two
disjoint blocks (401-420, 1601-1630). The branch is unreachable: the planner **never places an auto on an
arm tick anyway** — 0 of 376 arms in seed 401.

**Left as is deliberately.** The change is provably a no-op, and a no-op that costs a re-validation of
every shipped number is a worse trade than a comment. The bound is now documented in place so nobody
reads `A+1` as a claim about the engine again. **v16's 81.7% / 44.6 min / 470 nightshade all stand.**

### The lesson worth keeping

The planner's window formula and the engine's actual behaviour were two different models of the same
thing, and only the planner's was written down anywhere. **A bound in a chooser is not a statement about
the machine** — v15 read one as the other and built a whole "known bug" on top of it.

---

## 23 AUG · v16, ★★★ SETTLED IN GAME. THE DOUBLE-COUNT IS FIXED, AND POST-QUEST IS NOT A 100% ROUTE.

v15 ended on a question only Rendi could answer. He answered it.

> **Rendi, authoritative, 23 Aug: "stall defers floor damage."**

Under v15's own branch that is decisive. *If it DEFERS, the stall IS live on tick T and the click tax
should not apply to the stall at all.* So **`stallClickTax` 1 → 0**, and the legacy asymmetry was right
all along:

| | window | ticks for an L-tick stall |
|---|---|---|
| floor | `[T … T+L−1]`, **arm tick inclusive** | L |
| attacks | `[T+1 … T+L−1]` | L−1 |

It also now has a *reason* rather than being an accident: the arm goes in during the **input** phase of
T, ahead of queue processing, and the floor roll resolves in that same tick's queue processing, so it is
swept in.

### ★ RE-MEASURED: 180 runs, five disjoint seed blocks, `blockRisk 0` in every one

Blocks 201-260, 401-420, 501-540, 1201-1230, 1601-1630. Same engine, one parameter, nothing else touched.

| | tax 1 (the bug) | **tax 0 (correct)** |
|---|---|---|
| kills | 180/180, **100%** | **147/180, 81.7%** |
| median kill, winning runs | 38.8 min | **44.6 min** |
| arms per kill | 324 | **376** |
| rock cake clicks | 1,331 | **1,683** |
| nightshade, winning runs | 464 | **470** |
| ice-block risk | 0 | **0** |

> **SUPERSEDED BY v18 above. The 81.7% was a mis-fitted plan, not a verdict on the corrected physics.
> `restSlots` 0 → 2 puts it back to 100% for two inventory slots. Read on for how the number was reached.**

**The headline is the kill rate, not the supply.** v15 warned that the 465 nightshade was optimistic; it
was not — **nightshade per winning kill moved 464 → 470, which is nothing**, and the 28-slot bag still
fits. What the double-count was actually buying was **reliability and time**. The route loses ~18 points
of kill rate and gains ~6 minutes.

*(The mean nightshade looks like it falls, 465 → 386. That is a truncation artefact — a run that dies
spends less. Read the winning-runs row; it is the only one that means anything for supply.)*

### ★ AND `exitGrace` 4 CANNOT RESCUE IT. SWEPT, WORSE, REVERTED.

`exitGrace` 2 → 4 is what took this route from 27/160 to 100% at v5, so it was the obvious knob to try
again. Swept at tax 0 on block 201-260, 60 seeds each:

| exitGrace | kills | median arms |
|---|---|---|
| **4** | **50/60, 83.3%** | 376 |
| 5 | 23/60, 38.3% | 153 |
| 6 | 0/60, 0.0% | 81 |

Raising the grace **rejects arm plans** — the median arm count collapses 376 → 153 → 81, so you arm too
rarely to land the recoil the boss needs. The knob that rescued this route once cannot rescue it twice.
**Reverted to 4**, which remains optimal. One change, measured, reverted when it made the number worse.

### ⚠⚠ WHAT THIS DOES **NOT** SETTLE, AND IT IS NOW THE OPEN QUESTION

> **CLOSED THE SAME DAY — see v17 above. The answer was "it defers", my ~55% was backwards, and it
> changes no number on the page. Everything below in this section is superseded; kept as the record.**

The tax went in on Rendi's 22 Aug statement that *"any inventory action takes a tick to register"* —
which is §14, and is not in doubt for a **prayer** click. Both cannot be literally true of the same tick
unless the two things are different: the click **registers** on T+1 for effects you initiate, while the
**delay the item creates** is placed on T and catches whatever resolves in that tick's queue. **That
reading is mine, not measured.**

Its live consequence is that the **attack window still starts at T+1**, so **a 1-tick stall swallows zero
autos** — which is exactly the thing the tax was added to fix, and exactly Rendi's original question
("arming a 1-tick stall the tick before should cover the next auto attack, no?").

**Cheapest test that settles it:** arm a 1-tick stall on the tick before an auto is due, and watch whether
the auto is deferred.

- **Deferred** → the attack window starts on T too, and this engine is still a tick out on attacks.
  Post-quest would then get *cheaper* again, so this is worth an in-game minute before trusting the 82%.
- **Lands** → floor and attacks genuinely have different windows and the model is complete.

**~55% it lands**, on the reasoning that damage is computed when an attack is *fired* rather than when it
resolves, so an auto already in flight was never deferrable. What would move it up: any trace where a
projectile's damage is fixed before the stall. What would move it down: Rendi's own 22 Aug intuition,
which is a practitioner's and has beaten the model 21 times out of 21.

### ⚠ SEPARATELY UNTESTED, AND DO NOT LET THIS RESULT LEAK INTO IT

**Rendi, 23 Aug: whether an *interface* defers damage as a low-priority hold is still untested.** That is
a different mechanism from the item stall measured here. The archive runs against it — damage is a
**STRONG** script and force-closes modals, which is why interfaces stopped holding damage and get eaten
two at a time. The interface-nest branch is unreachable in every shipped route (`stallVia` is set to
`'item'`), so nothing on the page depends on the answer, but it is now flagged in the code so it cannot
be inherited silently if that branch is ever re-enabled.

### Not affected

The **quest floor-tank route** (never arms a stall at all — re-run as a control, unchanged), everything
measured before v10, and the **level 3 sim**, which never had the tax.

---

## 22 AUG · v15, ⛔⛔ KNOWN BUG: THE CLICK TAX DOUBLE-COUNTS. POST-QUEST NUMBERS ARE OPTIMISTIC.

Rendi asked whether the level 3 sim already accounted for the one-tick delay, said he'd seen it in the tick log.
Chasing that exposed a bug **in the prayer sim, introduced by me in v10.**

### What the level 3 sim actually does, it has no tax, but it isn't symmetric either

Traced, seed 1, arm at t15 with a 3-tick stall. Deferrals finish on **t15, t16, t16, t17**, spanning three ticks,
when the untaxed attack window is only `[t16 … t17]`.

**The arm tick itself is being harvested for floor damage.** So the legacy model is:

| | window |
|---|---|
| attacks | `[T+1 … T+L−1]` |
| **floor ticks** | **`[T … T+L−1]`**, arm tick inclusive |

That asymmetry was already there, in both sims, long before this week.

### ⛔ THE BUG: the tax stacks on top of it

Same seed, prayer sim, 3-tick stall:

| | floor deferrals | instances |
|---|---|---|
| `stallClickTax 0` | t142, t143, t144, **3 ticks** | 3 floor + 1 auto = 4 |
| `stallClickTax 1` | t141, t142, t143, t144, **4 ticks** | 4 floor + 1 auto = **5** |

**A 3-tick stall is harvesting four ticks of floor.** That is roughly **one extra instance per arm**, and
instances are the output, so it inflates everything downstream.

### ⚠ WHAT THIS PUTS IN DOUBT

Every post-quest number taken since v10: the **100% kill rate**, the **465 nightshade**, the **~39 minutes**,
the `cakeBelow 5` tuning and the `exitGrace 4` re-sweep. All measured on an engine collecting one instance per
arm too many. **Treat them as optimistic until this is settled.**

Not affected: the **quest floor-tank route** (never arms a stall at all), everything measured before v10, and
the level 3 sim (reverted, `md5 3956ddc7…`, untouched).

### ★ THE FIX DEPENDS ON A GAME QUESTION ONLY RENDI CAN SETTLE

**Click a stall item on tick T while standing on live ice. Does that tick's floor damage get deferred into the
stall, or does it land normally?**

- **If it LANDS**, the stall isn't live yet. Floor window becomes `[T+1 … T+L]`: exclude the arm tick, keep
  the tax.
- **If it DEFERS**, the stall *is* live on T, and the click tax should not apply to the stall at all.

**Right now the engine does both, which cannot be right under either answer.** One observation settles it and
decides which way the shipped post-quest route gets re-measured.

---

## 22 AUG · v14, ⛔ MY HAZARD WAS WRONG. THE STALL-FREE CYCLE MAY BE THE BEST ROUTE ON THE PAGE.

### ⛔ THE CORRECTION

I wrote in v13 that standing on the tile means the floor rolls 7-10 against your healed 10 and kills you a
quarter of the time.

**Rendi:** *"It will proc the Redemption off the rock cake on the same tick, and the auto attack will calculate
the ONE and the floor will calculate the ONE past the Redemption."*

**He's right, and I mis-attributed the mechanic.** "Everything lands as a 1" is **not** the stall's frozen cap.
It's **same-tick calculation order**, and this file already had it written down, for the queued case:

> *a 0-damage self-hit at 1 hp fires Redemption; the heal lands; **every queued instance still computes against
> pre-heal hitpoints**, lands as a 1, and recoils.*

**That property needs no stall.** The cake puts you on 1, the proc heals you to 10, and the auto and floor in
that same tick still compute against the 1. The stall was only ever a way to get *more* instances into that
window, it was never what made them land small.

### ★ THE sums, sums, not simulation. Nothing below has been run.

Per cycle: auto 1 + floor 1 = **2 damage taken, 2 recoil dealt**, ending on 8 hitpoints.

| | |
|---|---|
| recoil needed | 520 |
| recoil per cycle | 2 |
| **cycles** | **260** |
| Redemption drains the pool to zero every proc | none |
| regeneration potion returns 1 point per | **12 ticks** |
| **cycle rate is PRAYER-BOUND at** | **1 per 12 ticks** |
| **total** | **3120 ticks ≈ 31 min** (shipped route: 39) |

### ★ AND THE DESCENT FITS, which was the actual question

From 8, the cake is a flat 1 all the way down: **8 → 7 → 6 → 5 → 4 → 3 → 2 → 1 is seven clicks, inside a
twelve-tick prayer window.** Five ticks of slack per cycle, and with no stall, no feet lock during any of it.

### ★ WHAT IT WOULD COST

- **Zero nightshade**, the whole descent is flat-1 cake
- **Zero food**, Redemption is the only heal it needs
- **No Protect from Magic**, an unprotected auto computing against 1 hitpoint deals 1, and protecting it would
  zero the damage and therefore the recoil
- **Bag ≈ 16 slots**: 14 rings + 1 rock cake + 1 regeneration potion

Against the shipped post-quest route: **465 nightshade → 0, 21 slots → 16, 39 min → 31.**

### ⚠ ONE structural ASSUMPTION, AND EVERYTHING ABOVE RESTS ON IT

**That an unstalled same-tick hit really does compute against pre-heal hitpoints.** It is the same rule this
project already asserts for the queued case, but it has **not been observed without a stall.**

Two smaller ones: the icicle still demands you be inside tiles 1-9 on its fire tick every 32, which eats into
the 5-tick slack; and on the **quest** floor (3-5) Rendi notes you could stay a second tick for extra recoil,
which post-quest you cannot, the next tick computes against 8, and a 7-10 lands in full.

---

## ★ THE TEST RENDI IS RUNNING, recorded so it isn't lost

**Open an interface while standing on a floor pad, and see whether the damage rolls come in queued when he
clicks off.** If they queue and release on the click-off, floor damage is deferrable with **no stall at all**.
One observation, no risk, at comfortable hitpoints.

It decides two things at once: whether the open-field route needs building, and whether that plan's descent can
cross live floor safely by holding the tick and releasing it after a heal.

---

## 22 AUG · v13, THE STALL-FREE CYCLE: SPEC'D, NOT BUILT. AND ONE FAILED EXPERIMENT.

**Rendi:** *"Running between four tiles back to a singular tile every auto attack that's already got the ice
pattern on it, and using the cooled rock cake with the Redemption on the same tick, the rock cake will queue
before the auto attack plus the floor damage… and then run off the tile, all in a single tick set, with no
stall, and give a lot more clearance for the descent."*

### ⛔ FIRST, A FAILED EXPERIMENT, reported because the result looks like an answer and is not

I tried to price his claim cheaply by isolating the feet lock: keep the cap freeze and the swallow window, just
stop locking the feet.

| | clean | 0.15% slip | 0.3% slip |
|---|---|---|---|
| feet LOCKED (shipped) | 20/20 | 14/20 | 8/20 |
| feet FREE | **0/20** | 0/20 | 0/20 |

**This does NOT mean mobility is worthless.** The capacity planner books its instances as *the damaging ticks of
the tile your feet are locked to*, and every survival check downstream is written against that tile. Open up the
feet and the router walks you off mid-stall: you collect none of the instances the plan was bought for, and you
cross tiles the plan never checked. **The flag breaks the planner's assumptions instead of isolating the
variable.** The feet lock cannot be priced cheaply in this engine, and the stall-free route cannot be
approximated by switching it off.

### ★ WHY IT ISN'T MODELLED, and it's my code, not the game

**Redemption only fires inside the queue-resolution loop, which runs at a stall dump. There is no open-field
proc path in this engine at all.** Unstalled damage can never proc Redemption here. So the plan reads as
impossible for a reason that is an artefact of the simulator.

### ★ AND HIS DESIGN WITHDRAWS HALF OF MY OLD OBJECTION

The standing note against this (written for the level 3 build) argued *the proc band is one hitpoint wide and an
open-field floor roll steps straight over it.* **Answered.** He isn't using the floor roll to enter the band, he's using the **rock cake clicked on the same tick**, which queues first and lands him on exactly 1. Entry to
the band is controlled, not rolled. That objection is withdrawn.

### ⚠ THE HAZARD THAT REMAINS, sums, not simulation. Check it before building.

The Redemption heal **caps at max hitpoints**, so the proc leaves you on **10**. The post-quest floor rolls
**7-10**.

- **Stay** on the tile to collect the recoil → you're on 0-3, and a roll of 10 kills outright. Roughly a quarter
  of cycles, across ~380 cycles a kill.
- **Step off** to dodge it → the tile deals no damage, so it recoils nothing and the cycle earns zero.

**The tile that pays you is the tile that can kill you**, and without the frozen cap it pays 1-2 recoil for 7-10
hitpoints. That is what the build has to answer, and it is far harsher post-quest than at the level 3 floor.

---

## ★★ THE OPEN QUESTION THAT WOULD DISSOLVE IT, and it's Rendi's own

**Rendi, same session:** *"I want to test the idea of possibly seeing if the floor damage, the ice pattern, is a
low priority action. Maybe you can hold that behind the interface, for instance, after a delayed heal, and then
without even a stall, click off to enact the damage."*

**These two ideas are connected, and the second is the missing piece of the first.** If floor damage can be
deferred *without a stall*, held behind an open interface and released on a click of your choosing, the
hazard above disappears. You would no longer choose between standing there eating a 7-10 and stepping off
earning nothing. You'd collect the floor tick, hold it, heal, and release it when you can afford it.

**It is plausible rather than wishful, because this engine already models the opposite behaviour for a different
interface:** the nightshade-interface stall collapses on a damaging tile, *"holding the interface only works on
a non-damaging tile; any damage rolling in while you wait collapses it."* Both cannot be right in general, and
which one governs the **ice pattern specifically** is exactly what's unmeasured.

### ★ CHEAPEST SETTLING TEST

**Stand on a live pattern tile at comfortable hitpoints with an interface open, and watch whether the floor tick
finishes on schedule or waits.** One observation, no risk, and it decides whether the stall-free route is worth
building at all.

Both are now written into the simulator as `redempOpenField` (the spec) and `icePriorityQ` (the question), so
neither gets lost.

---

## 22 AUG · v12, RE-OPTIMISED FOR SURVIVABILITY. RESTS DROPPED, STALL ITEMS FINALLY COUNTED.

**Rendi:** *"multiple tick stalls are needed each attempt not just 1 standardized one? would be much smoother
with just 1 consistent stall, also if more stall variants are needed more inventory would be taken up, example:
egg book, egg ring, crate ring."* Then: *"don't focus so much on speed just survivability, so guthix rests kinda
meh and 1 stall item is better, 2 max than mixing them up."*

### ⛔ AN ACCOUNTING ERROR OF MINE, THE STALL ITEMS WERE NEVER IN THE SLOT BUDGET

He is right and it was missing entirely. **Every distinct stall LENGTH is a distinct ITEM in the bag**, an
easter ring is not a crate ring, and the slot budget counted none of them. Every post-quest bag figure in
v5-v11 was under-counted by two. Now fixed: the budget reads the lengths the run actually armed and bills one
slot each. The quest floor tank arms nothing, so it pays nothing here and its 28 was always correct.

### ★ SHIPPED: THE GUTHIX REST IS GONE, `restSlots` 4 → 0

At a 0.15% misclick rate, two disjoint blocks:

| | with 4 flasks | with none |
|---|---|---|
| block 10601-10616 | 9/16 | 12/16 |
| block 11001-11020 | 13/20 | 12/20 |

**The rests make no measurable difference to survivability either way.** They are dropped because they cost four
slots and about sixty extra clicks and buy nothing, not because they hurt.

*(⛔ I briefly read the first block as rests being actively worse and said so. That was sampling noise across 16
seeds. Retracted.)*

### ★ CAN IT RUN ON ONE STALL? Yes, and it costs the best number on the page

| post-quest config | clean | at 0.15% slip | nightshade | time | stall slots |
|---|---|---|---|---|---|
| **1-tick + 3-tick, no rests ★ SHIPPED** | **20/20** | **12/20** | **465** | 39 min | 2 |
| 1-tick only, **with** 4 rest flasks | 20/20 | 8/20 | 541 | 57 min | 1 |
| 1-tick only, no rests | **5/20** | 2/20 | none |, | 1 |
| 3-tick only, any setting | **1/20** | 1/20 | none |, | 1 |

**One consistent stall is possible, but only the 1-tick and only if you put the rests back**, that plan
depends on rest throughput to survive its own 57-minute length. It trades the best survivability number on the
page (12/20 → 8/20) and 18 extra minutes of exposure for one inventory slot and a simpler rhythm.

**The 3-tick can never be the single stall.** 1/20 at every setting.

### ⚠ THE ONE THING THE SIM CANNOT MEASURE, AND IT MAY DECIDE THIS

Every row above assumes **the same misclick rate for both rhythms.** A single repeated stall should in reality
have a *lower* slip rate than alternating two, and misclicks dominate everything (v7: at 0.25% slip the all-in
nightshade roughly doubles). The break-even is roughly this: **if standardising on one stall takes your real
slip rate from 0.15% down to about 0.08%, the one-stall route wins.** Only Rendi can judge that.

### ★ THE POST-QUEST BAG, 21 of 28, and no food flasks at all

| slots | item |
|---|---|
| 14 | unnoted rings of recoil |
| **2** | **stall items, 1-tick and 3-tick** *(newly counted)* |
| 2 | prayer regeneration potions |
| 1 | purple sweets, stacked (~600) |
| 1 | cave nightshade, stacked |
| 1 | rock cake |
| **7** | **spare** |

Healing is now **entirely** Redemption plus sweets: 318 procs heal 2862 hitpoints free, 599 sweets, **zero rest
doses.**

---

## 22 AUG · v11, WHICH STALLS ARE ACTUALLY NEEDED, AND WHY THE HARDER FIGHT PACKS LIGHTER

Two questions from Rendi, both measured rather than reasoned.

### ★ WHAT LENGTH OF STALL DOES THE PLAN ACTUALLY NEED?

Shipped post-quest, 16 seeds, counting every stall armed in a whole kill:

| stall table available | kills | nightshade | kill time | lengths actually armed |
|---|---|---|---|---|
| **1-tick and 3-tick (his kit) ★** | **16/16** | **471** | **38.7 min** | 1t ×56 · 3t ×269 |
| 1-tick only | 16/16 | 539 | 56.2 min | 1t ×470 |
| **3-tick only** | **1/16** | none |, | 3t ×184 |
| + the 7-tick egg book | 16/16 | 470 | 38.7 min | 1t ×56 · 3t ×269 · **7t ×1** |
| + a 2-tick, if one exists | 16/16 | 474 | 38.7 min | 1t ×46 · 2t ×9 · 3t ×270 |

**The 3-tick is the workhorse, 269 arms of 325, but the 1-tick is the one you cannot drop.** Three-tick alone
is 1/16. One-tick alone still kills every time, but it costs 17 more minutes and 68 more nightshade.

**The 7-tick egg book fires ONCE in an entire kill. The 2-tick fires nine times and changes nothing.**
Rendi has a 1-tick and a 3-tick. That is exactly the kit, nothing else in the stall table is worth acquiring.

### ★ WHY POST-QUEST PACKS 23 SLOTS AND QUEST PACKS 28

It looks backwards for the harder fight. **The two plans pay for recoil in different currencies.** Measured
side by side:

| | QUEST, floor tank | POST-QUEST, Redemption |
|---|---|---|
| recoil | 400 | 520 |
| floor ticks absorbed | 400 | 416 |
| **Redemption procs** | **0** | **318** |
| **hitpoints healed free** | **0** | **2862** |
| rest doses drunk | **60** | **16** |
| sweets eaten | 745 | 583 |
| **Guthix rest flasks carried** | **15** | **4** |

**The tank gets no free healing.** It absorbs 400 floor ticks and has to eat every one of them back, 60 doses,
15 flasks. Redemption heals itself 2862 hitpoints across 318 procs, so it drinks 16 doses and carries 4 flasks.
**That eleven-flask gap is the entire slot difference**, and three of the slots that come back go into rings,
because 520 recoil needs 13 rings where 400 needed 10.

Put another way: **quest pays in food, post-quest pays in nightshade.**

| | QUEST 28 of 28 | POST-QUEST 23 of 28 |
|---|---|---|
| rings | 11 | 14 |
| Guthix rest flasks | 15 | 4 |
| regeneration potions | 1 | 2 |
| purple sweets (stack) | 1 | 1 |
| nightshade (stack) | none | 1 |
| rock cake | none | 1 |

**And the quest 28 is a speed choice, not a requirement.** The sweets-only preset runs the identical route on
**13 slots** at 100%, five minutes slower. Quest fills its bag because it has the room to. Post-quest cannot,
and does not need to.

*(Correction: the post-quest bag was listed as 22 in v9-v10. It is 23, the rock cake became a real slot when
`cakeBelow` went above 0.)*

---

## 22 AUG · v10, TWO CORRECTIONS FROM RENDI THAT RE-PRICED THE WHOLE PAGE

Both of these are **his mechanics over my model**, and between them they killed one headline route, revived
another, and moved the shipped post-quest numbers.

### ⛔ CORRECTION 1, THE STALL ARM PAYS THE CLICK TAX

**Rendi:** *"arming a stall of 1 tick length pressed the tick before to arm with something like the easter ring
should cover the next auto attack by a 1 tick delay if timed correctly no? Still confused why 2 ticks over 1."*
Then, confirming: *"yeah that needs to be applied to the stall arm / any inventory action takes a tick to
register."*

**He was pointing at an inconsistency in my own engine, not asking for a tuning change.** §14 is already in this
project and this engine already obeys it *for prayers*, the Protect from Magic flick clicks on T and is live
for damage calculated on T+1. **The stall arm was not paying it.** I armed on T and had the stall live on T, so
the swallow window came out as `[T+1 … T+L−1]`, **empty at L = 1.**

Under the tax the arm on T goes live on T+1, the window is `[T+1 … T+L]`, and a **one-tick stall swallows
exactly one attack**, the auto he described. Measured, same seeds, nothing else changed:

| forced 1-tick stall | instances/arm | recoil of 520 | kills |
|---|---|---|---|
| no tax (my old model) | 0.37 | 195 | 0/16 |
| **with tax** | **1.08** | **520** | **16/16** |

**`stallClickTax` now defaults to 1.** It is kept switchable only to reproduce every measurement taken before
this date.

### ⛔ CORRECTION 2, THE REAL ROCK CAKE CURVE

**Rendi:** *"on the rock cake it starts 10-2 = 8 hp, then 1 from there to 7 6 5 4 3 2 1 etc, so probably the
entire rock cake route is scrapped from the sim."*

**Two at 10 hitpoints, one at every hitpoint below.** That is `10 → 8 → 7 → 6 → 5 → 4 → 3 → 2 → 1`, an
**eight-click descent.** Both of my models were wrong, in opposite directions:

| model | descent | source |
|---|---|---|
| flat 1 (v6 and earlier) | 9 clicks | my guess |
| banded 3/2/1 (v7-v9) | 5 clicks | my guess from his "2 or 3" |
| **2 then 1** | **8 clicks** | **Rendi, measured** |

Written in as `cakeT1 9 · cakeHi 2 · cakeT2 0 · cakeMid 1`.

### ⛔ CASUALTY: THE ZERO-NIGHTSHADE ROUTE IS DEAD

The v9 headline, 2-tick stall, pure cake, **66/66 at zero nightshade**, was real against the *five*-click
descent I had invented. Against the real eight-click curve it is **7/16 without a 2-tick stall and 8/16 with
one.** The plan was never real. It is left in the dropdown, renamed `⛔ DISPROVED`, so the mistake stays
visible instead of being without fuss deleted.

### ⛔ CASUALTY: EVERY REDEMPTION-BASED **QUEST** ROUTE

`quest, Redemption on the rectangle` was 160/160. Under the click tax it reads **1/12**, and retuning
`exitGrace` 3/4 against `cakeBelow` 4/5 tops out at 3/12. The reason is structural, not tuning: the quest
variant's margin came from packing a stall cycle into a 3-5 floor, and **one extra tick per arm is more than
that margin.**

**★ The shipped QUEST route is completely unaffected, the floor tank never arms a stall at all.** No stall, no
click tax. Still 20/20 at 400 recoil, 0 nightshade, ~30 min.

### ★ RE-TUNED AND RE-SHIPPED: POST-QUEST

`exitGrace` was tuned against the untaxed model, so it had to be re-swept. New optimum, on his **actual kit**
(1-tick and 3-tick stalls only, no egg book, no 2-tick):

| grace 4 | nightshade | kills | kill time |
|---|---|---|---|
| cakeBelow 4 | 491 | 14/20 | 32.2 min |
| **cakeBelow 5 ★ SHIPPED** | **467** | **20/20** | **38.7 min** |
| cakeBelow 6 | 472 | 20/20 | 43.6 min |
| cakeBelow 7 | 550 | 20/20 | 57.9 min |

**Shipped: `stallClickTax 1` · `exitGrace 4` · `cakeBelow 5` · stalls 1 and 3 only.**
**20/20 · 467 nightshade · ~39 min · blockRisk 0.**

Six minutes faster than the v9 route, and the nightshade number is honest for the first time, 338 was measured
against a cake curve that does not exist.

### WHERE THAT LEAVES THE SUPPLY PROBLEM

Zero nightshade is **not available.** 467 a kill is the real number, and it is worse than the 338 I reported
yesterday because that figure was built on my invented curve. The rock cake still earns its slot, it does 1321
of the ~1790 self-damage clicks per kill for free, but it cannot take the top of the descent.

And the v7 finding stands and now matters more: **misclicks dominate.** At 0.25% slip the all-in cost roughly
doubles. Click accuracy is still worth more than every descent setting on the page.

---

## 22 AUG · v9, ★★★ THE 2-TICK STALL. ZERO NIGHTSHADE.

**Rendi, after the 1-tick result:** *"2 tick stall then?"*

**Yes.** Adding a two-tick stall to the menu makes the **pure rock cake** descent work, and the nightshade bill
goes to **literally zero.**

| | shipped post-quest | ★ with a 2-tick stall |
|---|---|---|
| kills | 100% | **66/66** across four disjoint blocks |
| **nightshade per kill** | **338** | **0** |
| kill time | 45.4 min | **49.2 min** |
| blockRisk | 0 | **0**, nobody died, nobody ran out of clock |

Blocks: 3601-3616, 4001-4020, 5001-5015, 6001-6015.

### Why 2 works and 1 doesn't, one tick of swallow window

Instances are what the stall **swallows**, over the window `[T+1 … T+L−1]`.

- **L = 1** → the window is `[T+1 … T]`, **empty**. It swallows no attack at all. Conversion **0.37** instances
  an arm, which is why the 1-tick experiment ground out at 195 recoil of 520 *even with nightshade*.
- **L = 2** → the window is one tick wide. Conversion **1.10, 1.30**. That is enough.

**The entire difference between a dead route and a farmable one is one tick of swallow window.**

### The menu 2/3 is all you need

Dropping the 1-tick and the 7-tick entirely reads an identical 66/66, half a minute faster.

### ★ THE BAG, 21 of 28

14 unnoted rings · 4 four-dose Guthix rest · 2 prayer regeneration potions · 1 stack of purple sweets ·
**1 rock cake**. Seven spare, and **not one consumable self-damage item.**

### ⚠⚠ TWO THINGS MUST BE TRUE, AND NEITHER IS CONFIRMED

Shipped as a **separate preset**, not as the default, because of these:

1. **A two-tick stall must exist.** Rendi's own earlier note says *"we have access to 1 and 3 tick stalls"*, so
   this may simply not be in the kit.
2. **The rock cake must deal 3 at 10 hitpoints.** Rendi said *"2 or 3"* and wasn't sure. Set `cakeHi` to 2 and
   this plan reads **0/50**, it dies in every block, because the descent becomes 10 → 8 → 6 → 4 → 3 → 2 → 1,
   **six clicks instead of five**, and one tick of descent is the entire margin.

### ★ CHEAPEST SETTLING TEST ON THE PAGE

**Guzzle a rock cake once at 10 hitpoints and read the splat.** That single number decides whether post-quest
kill count costs 338 nightshade a kill or none at all.

---

## 21 AUG · v8, THE SHORT STALL: TESTED, AND IT FAILS FOR A REASON WORTH KNOWING

**Rendi:** *"Is there no different length of stall that would allow for more descending time between the
attacks? Like, if I did a shorter stall, the Easter ring, a one-tick stall, and then descended longer, maybe
then I could rock cake the whole way rather than use any belladonna."*

Right question, and it was **untested**: every pure-cake run before this let the planner choose its own stall
length, and the planner ranks candidates by *most instances first*, which biases it away from the short stall.
So I forced the menu to 1-tick only (`stallCrate 0`, `stallEgg 0`) and raised the tick cap to 30000 so a slow
route could not be mistaken for a dead one.

| forced 1-tick stall | kills | recoil of 520 | arms | outcome |
|---|---|---|---|---|
| pure cake, grace 4 | 0/16 | 195 | 523 | all 16 **ran out of clock**, none died |
| pure cake, grace 6 | 0/16 | 9 | 75 | none |
| pure cake, grace 8 | 0/16 | 5 | 30 | none |
| **CONTROL, same 1-tick menu, NIGHTSHADE** | **0/16** | **195** | **542** | all 16 ran out of clock |
| shipped (menu 1/3/7, cakeBelow 9) | **16/16** | **520** | 384 | t4538 |

### The control is what settles it

**The 1-tick stall fails with the cheap three-click nightshade descent too**, same 195 recoil, same zero kills.
So the descent was never what it was failing on, and no amount of extra descent time was going to rescue it.

### Why, one line of sums

> **The stall is not overhead. The stall is where the damage happens.**

Instances are what the stall *swallows*: scheduled attacks over the window `[T+1 … T+L-1]`, plus the floor ticks
under your locked feet over `[T … T+L-1]`. **At L=1 that attack window is empty**, a one-tick stall swallows no
attack at all, and the only instance available is the single floor tick you happen to be standing on.

Measured conversion: **0.37 instances per arm**, against the shipped route's **1.35**.

Shortening the stall shortens the **earning** window one-for-one. It buys descent time by selling the very thing
the descent exists to set up. 520 recoil at 0.37 an arm is about **1400 cycles and roughly two hours**, and it
never gets there, it is not dying; it is grinding.

### And it confirms where the real wall is

Pure cake is blocked by **one tick of descent**, not by stall length. `cakeBelow 9` (nightshade for 10 → 5 in
one click) is 30/30; `cakeBelow 10` (cake doing 10 → 7 → 5, one tick longer) is 10/30. That single click is the
whole difference, and no stall length moves it.

---

## 21 AUG · v7, THE ROCK CAKE SCALES, AND THE SUPPLY PROBLEM MOVES

**RENDI CORRECTED ME:** *"at higher health the rock cake does more damage, I believe at 10 it starts at 2 or 3
with guzzle eventually scaling to 1 at lower hp levels. Definitely not 9 ticks."*

I had modelled it as a **flat 1 at every hitpoint.** Everything I concluded from that is withdrawn:

| I claimed | actually |
|---|---|
| the cake descends in **9 clicks** | **5**, 10 → 7 → 5 → 3 → 2 → 1 |
| the prayer point drains before the arm | that was a symptom of the 9-tick descent, not the cake |
| 16-38 cake clicks per arm | an artefact of the same wrong number |

**NEW:** `cakeScale` (banded damage), `cakeT1` / `cakeT2` (band edges), `cakeHi` / `cakeMid` (band damage),
`maxTicks` (the tick cap is now visible, so a slow route can't be mistaken for a dead one).

### ★ SHIPPED: post-quest `cakeBelow` 4 → **9**. Nightshade **557 → 338 per kill.**

| cakeBelow | nightshade | kills | kill time |
|---|---|---|---|
| 4 (was shipped) | 559 | 30/30 | 36.5 min |
| 7 | 385 | 30/30 | 44.4 min |
| 8 | 366 | 30/30 | 45.4 min |
| **9** | **338** | **30/30** | **45.4 min** |
| 10 | 10 | **10/30** | none |
| 99 (pure cake) | 0 | **14/30** | none |

### The whole fight turns on ONE CLICK of the descent

`cakeBelow 9` keeps nightshade for the **first step only**, 10 → 5 in one click, and gives the cake
everything below it. `cakeBelow 10` hands that first step to the cake too, which costs **10 → 7 → 5** instead
of **10 → 5**: one extra tick. That one tick takes it from **30/30 to 10/30**.

Pure cake, zero nightshade, tops out at 14/30. Raising the tick cap to five hours shows it **stalling at 508
recoil of 520**, not dying, it survives and cannot finish.

### This setting does not depend on the exact curve

At `cakeBelow 9` the top band is never used, so `cakeHi 2` and `cakeHi 3` both read **30/30 at 338-339
nightshade.** If Rendi measures the real numbers, the shipped route does not move. The three numbers to correct
if he does are `cakeT1`, `cakeHi`, `cakeMid`, nothing else in the engine hardcodes the cake.

### ★★★ AND THE SUPPLY COST IS DOMINATED BY MISCLICKS, NOT BY THE DESCENT

Rendi: *"one fail is -500 nightshade which is months of farming."* Priced it, expected nightshade per
**successful** kill, counting the failed attempts:

| misclick rate | kills / 24 | nightshade per kill, all-in |
|---|---|---|
| perfect | 24/24 | **338** |
| 0.25% | 7/24 | **774** |
| 0.5% | 2/24 | **1735** |
| 1% | 0/24 | **no kills at all** |

Tuning the descent from `cakeBelow` 4 to 9 saves **220** nightshade. **A quarter of one percent of misclicks
costs 390.** Click accuracy is worth more than every descent setting on this page combined.

---

## 21 AUG · v6, THE MIXED DESCENT

**NEW:** `cakeBelow`, use the reusable rock cake at or below N hitpoints, nightshade above it.

Nightshade **halves** (10 → 5 → 2 → 1); the cake is flat-ish at the bottom. So the two are *numerically
identical* wherever the halving is already 1 or 0, and there the cake is strictly better because it never runs
out:

- the **arm click** deals 0 on either item, always free
- the **2 → 1 click** deals 1 on either item, always free

Post-quest went **904 → 556 nightshade** at +0.4 min. Superseded by v7 above.

**Also v6:** the supplies card now reports **cave nightshade consumed** and **rock cake clicks** as separate
lines, and the slot budget counts them as separate items.

---

## 21 AUG · v5, THE POST-QUEST FIGHT IS SOLVED, AND IT WAS ONE PARAMETER

### ★★★ `exitGrace` 2 → 4. Nothing else changed. **27/160 → 100%.**

Same route, same room loop, same prayer, same protection, same 7-10 floor, same boss.

| seeds | exitGrace 2 | exitGrace 4 |
|---|---|---|
| 201-260 | 10/60 | **60/60** |
| 401-420 | 3/20 | **20/20** |
| 501-540 | 6/40 | **40/40** |
| 601-640 | none | **40/40** |
| 801-840 | none | **40/40** |

`blockRisk` **0** in all 200 runs. It is a **cliff, not a dial**, 6 / 3 / **40** / 40 / 0 kills out of 40 at
grace 2 / 3 / 4 / 5 / 6.

**Why 4:** tally the deaths at grace 2 and 3 and every one is `stall=0`, phase `redempt`. Nobody dies inside a
stall, they die in the open on the walk back down to 1. That walk is four ticks: one eat, then nightshade
10 → 5 → 2 → 1. Buy four clear ticks and the whole re-arm happens on floor that cannot roll.

**It does nothing on the quest floor**, grace 0 and grace 4 are identical there, 60/60 either way.

### ⛔ TWO EXPLANATIONS I GOT WRONG FIRST

1. **"It's the natural regen landing inside the stall."** Every trace at grace 3 died at t1610-1611 with
   `⚠ REGEN +1 LANDED INSIDE A LIVE STALL`. Looked conclusive. `regenTicks 99999` left the cliff exactly where
   it was, 4/20, 5/20, 20/20, 20/20. Proximate killer, not the cause.
2. **"The cliff tracks `cycleOverhead`."** Swept 3 / 4 / 5, moved it **not one tick**, rows identical.

### ★★ AND THE HARD RING HANDLING WENT WITH IT

The 27/160 build needed **noted rings on a ring of suffering**, a two-tick dialogue that can't happen inside a
stall, because twelve slots went to Guthix rest. Under grace 4 the rests stop mattering: **40/40 at 12, 8, 6, 4
and 2 flasks**, median kill 37.3 / 35.5 / 35.9 / 35.9 / 37.3 min. So the slots go to rings and the refresh is
**one click on one tick**.

⛔ **REVERSES** the v4 note *"the easier unnoted handling costs seven kills."* That was a property of a plan
that was dying, not of the ring handling.

### ★ POST-QUEST BAG, 22 of 28

14 unnoted rings of recoil · 4 four-dose Guthix rest · 2 prayer regeneration potions · 1 stack of purple sweets
(~610) · 1 nightshade · **6 spare**

---

## 21 AUG · v4, THE ICE BLOCK QUESTION IS ANSWERED

**Rendi, confirmed in game: standing in the 3×3 entrance never triggers the ice block.**

Carried for weeks at *"~50/50, and the most valuable observation on that boss."* The losing branch closed the
post-quest fight completely, 1-4 blocks per special healing her 16-25 each, melee-only to destroy, and a
zero-XP account can neither destroy them nor walk away. **It went the right way.**

`blockRisk` is now a **structural invalidity check** rather than a hedge: non-zero means the run left the
entrance on a fire tick, and it is thrown out, not scored lower.

---

## 21 AUG · v3, WHY THE LONGER STALL CANNOT PAY FOR ITSELF

Rendi asked whether a longer stall could bank more instances after the Redemption proc, shortening the descent.

**The sums is right**, `postHp = 10 − instances`, and at the full 9 the dump lands you on 1 with zero
descent clicks. **The thing to pull is self-limiting.**

| config | kills | instances/arm | stalls actually armed |
|---|---|---|---|
| quest, menu 1/3/7 | 160/160 | 1.81 | 1t×2 · **3t×217** · 7t×2 |
| quest, forced 7t only | 0/160 | 6.63 | 7t×1 |
| post-quest, menu 1/3/7 | none | 2.06 | 1t×5 · 3t×92 · 7t×37 |

The 7-tick stall is **already in the menu and already legal**, the planner just doesn't pick it. Force it and
conversion reaches 6.6-6.9 of a possible 9, and the arm count collapses 221 → 1.

The binding constraint is **the exit**, not capacity and not stall length. The dense instance source is live ice
under locked feet, and live-ice arms are rejected **66,778 times per run**, every instance banked lowers
`1 + heal − instances`, the number the exit roll must be survived on.

**Capacity 9 is hard:** the heal is `⌊prayer÷4⌋` = 12 at 50 Prayer, clipped by the 10-hitpoint bar to 9. More
Prayer does not raise it.

---

## 21 AUG · v2, WHY THE QUEST ROUTE NEEDS NO SELF-DAMAGE ITEM

Rendi asked how a randomised floor can reliably work with no nightshade.

> **Recoil is 10% + 1, rounded down. Every hit of 1-9 returns exactly 1.** The size of the roll is irrelevant to
> output. The floor tank never needs to be at any particular hitpoint, it needs to be *hit*. It stands on live
> ice at 15, takes 3-5 a tick, returns exactly 1 a tick whatever the roll was, and eats back up.

Measured on the shipped quest route: **400 floor ticks taken → 400 recoil dealt, one-to-one, 0 self-damage
clicks.** The randomness only changes how often you eat.

**It does not carry to the 7-10 floor**, and the failure is food throughput:

| floor tank, post-quest | recoil of 520 | kills |
|---|---|---|
| 15 rest slots | 115 | 0/24 |
| 25 rest slots | 145 | 0/24 |
| 53 rest slots (does not fit an inventory) | 231 | 0/24 |
| control, same tank on the **quest** 3-5 floor | **400** | **22/24** |

At 3-5 you take three ticks between heals; at 7-10 you take **one**, so every heal buys one recoil and 520
recoil needs 520 heals. **Post-quest must be Redemption, and Redemption must descend.**

---

## THE TWO SHIPPED ROUTES, AS THEY STAND

| | QUEST | POST-QUEST |
|---|---|---|
| route | floor tank | Redemption |
| loop | 13 ↔ 14 shuttle | room loop 16 → 20 → 26 → 22 |
| floor | 3-5 | 7-10 |
| boss hp | 400 | 520 |
| kills | **100%** | **100%** |
| time | ~30 min | ~45 min |
| slots | 28 of 28 | 22 of 28 |
| **nightshade** | **0** | **338** |
| self-damage item | **none, the floor is the descent** | nightshade for 10→5, rock cake below |

---

## STILL OPEN

1. **The spike's damage.** *"Spikes formed afterwards deal typeless damage"*, typeless cannot be protected, and
   the sim still carries the quest variant's 1-8 for it.
2. **The real rock cake curve.** Shipped route is insensitive to it, but `cakeT1` / `cakeHi` / `cakeMid` are
   there to correct if it's measured.
3. **Instances per arm.** 1.7 of a possible 9. The sums floor for nightshade is ~116 a kill and the whole
   gap is exit clearance. This is the thing to pull if supply is what blocks kill count.
