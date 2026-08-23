# PRAYER-ONLY PURE, START HERE · 10 HITPOINTS · ONE FILE, TWO PLANS

---

# ★★★ 21 AUGUST, THE POST-QUEST FIGHT IS SOLVED, AND IT WAS ONE PARAMETER

**`exitGrace` 2 → 4. Nothing else changed.** Same route, same room loop, same prayer level, same protection,
same 7-10 floor, same boss. **27/160 → 100%.**

| seeds | block | exitGrace 2 | exitGrace 4 |
|---|---|---|---|
| 201-260 | disjoint | 10/60 | **60/60** |
| 401-420 | disjoint | 3/20 | **20/20** |
| 501-540 | disjoint | 6/40 | **40/40** |
| 601-640 | disjoint | none | **40/40** |
| 801-840 | disjoint | none | **40/40** |

`blockRisk` **0** in every one of those 200 runs.

It is a **cliff, not a dial**, and that is what makes it trustworthy rather than suspicious:

| exitGrace | 2 | 3 | **4** | 5 | 6 |
|---|---|---|---|---|---|
| kills / 40 | 6 | 3 | **40** | 40 | 0 |
| median end | t1234 | t1611 | **t3577** | t3939 | t1854 |

Grace 5 also kills everything and costs 360 ticks. Grace 6 collapses because almost no plan qualifies any more.

### Why the cliff sits at exactly 4

**Tally how the deaths happen at grace 2 and grace 3 and every single one is `stall=0`, phase `redempt`.**
Nobody ever dies inside a stall. They die *in the open*, on the walk back down to 1 hitpoint.

That walk is **four ticks long**: one eat tick, then belladonna 10 → 5 → 2 → 1, three clicks. Buy four clear
ticks after the dump and the entire re-arm happens on floor that cannot roll. Buy three and the last click of
the descent eats a 7-10.

And it does **nothing at all on the quest floor**, grace 0 and grace 4 are identical there, 60/60 either way.
Which fits: at 3-5 the descent can afford to eat a roll, at 7-10 it cannot.

### ⛔ TWO EXPLANATIONS I GOT WRONG FIRST, written down so they are not retried

1. **"It is the natural regen landing inside the stall."** Every trace I pulled at grace 3 died at t1610-1611
   with `⚠ REGEN +1 LANDED INSIDE A LIVE STALL, cap was frozen at 1`. It looked conclusive. Setting
   `regenTicks` to 99999, regen effectively off, left the cliff **exactly where it was**: 4/20, 5/20, 20/20,
   20/20. The regen was the proximate killer in those particular traces and **not the cause**.
2. **"The cliff tracks `cycleOverhead`."** Sweeping `cycleOverhead` 3 / 4 / 5 moves it **not one tick**, the
   rows come back identical. The 4 is the length of the real belladonna descent, which is fixed by the halving
   sums, not by that parameter.

---

# ★★ AND THE HARD RING HANDLING IS GONE WITH IT

The 27/160 build needed **noted rings on a ring of suffering**, a two-tick dialogue that cannot happen inside a
stall and aborts if anything finishes on you, because twelve inventory slots had to go to Guthix rest. I measured
that trade and wrote down that the easier unnoted handling *"costs seven kills"*.

**That conclusion is dead.** Under grace 4 the rests stop mattering:

| flasks | 12 | 8 | 6 | 4 | 2 |
|---|---|---|---|---|---|
| kills / 40 | 40 | 40 | 40 | **40** | 40 |
| median kill | 37.3 min | 35.5 | 35.9 | **35.9** | 37.3 |

So the slots go to rings instead and the refresh becomes **one click on one tick**: the ring crumbles, you equip
the next. The trade I measured was real, it was a property of a plan that was dying, not of the ring handling.

### ★ THE POST-QUEST BAG, 22 of 28

| slots | item |
|---|---|
| 14 | unnoted rings of recoil (560 charges against 520 boss hitpoints) |
| 4 | four-dose Guthix rest |
| 2 | prayer regeneration potion |
| 1 | purple sweets, stacked (~610) |
| 1 | belladonna |
| **6** | **spare** |

Per kill: **16 rest doses · ~610 sweets · ~1200 belladonna clicks · 520 recoil · ~36 minutes.**

---

# ✅ THE ICE BLOCK QUESTION IS ANSWERED, 21 August, confirmed in game

**Standing in the 3×3 entrance never triggers the ice block.** I confirmed it.

This was carried in this file for weeks as *"~50/50, and it is the most valuable observation on that boss"*, and
the losing branch closed the post-quest fight completely: 1-4 blocks per special healing her 16-25 each,
melee-only to destroy, and a zero-XP account can neither destroy them nor walk away from them, she would simply
out-heal recoil. **It went the right way.**

Consequence for the simulator: **`blockRisk` is now a structural invalidity check rather than a hedge.** A run
that reads non-zero left the entrance on a fire tick and is thrown out, not scored lower.

---

# ⛔ CORRECTION, THE ROCK CAKE SCALES. EVERY CAKE CONCLUSION BELOW WAS BUILT ON A WRONG NUMBER

**I, 21 Aug:** *"at higher health the rock cake does more damage, I believe at 10 it starts at 2 or 3 with
guzzle eventually scaling to 1 at lower hp levels. Definitely not 9 ticks."*

I had it as a **flat 1 at every hitpoint.** Withdrawn:

| I claimed | actually |
|---|---|
| the cake descends in **9 clicks** | **5**, 10 → 7 → 5 → 3 → 2 → 1 |
| the prayer point drains before the arm | a symptom of the 9-tick descent, not of the cake |
| 16-38 cake clicks per arm | an artefact of the same wrong number |

Re-measured with a banded cake (`cakeScale`, `cakeT1`/`cakeT2`, `cakeHi`/`cakeMid`), fresh block 3001-3030:

| cakeBelow | nightshade | kills | kill time |
|---|---|---|---|
| 4 | 559 | 30/30 | 36.5 min |
| 7 | 385 | 30/30 | 44.4 min |
| 8 | 366 | 30/30 | 45.4 min |
| **9 ★ SHIPPED** | **338** | **30/30** | **45.4 min** |
| 10 | 10 | **10/30** | none |
| 99 (pure cake, zero nightshade) | 0 | **14/30** | none |

### The whole fight turns on ONE CLICK of the descent

`cakeBelow 9` keeps nightshade for the **first step only**, 10 → 5 in one click, and gives the cake
everything below. `cakeBelow 10` hands that step to the cake as well, which costs **10 → 7 → 5** instead of
**10 → 5**. One extra tick. **30/30 → 10/30.**

Pure cake tops out at 14/30, and raising the tick cap to five hours shows it **stalling at 508 recoil of 520**, it survives and cannot finish. Zero nightshade is not available at any speed.

### The shipped setting does not depend on the exact curve

At `cakeBelow 9` the top band is never used, so `cakeHi 2` and `cakeHi 3` both read **30/30 at 338-339
nightshade.** If the real numbers get measured the shipped route does not move, correct `cakeT1`, `cakeHi`,
`cakeMid` and nothing else in the engine hardcodes the cake.

---

# ★★★ THE NUMBER THAT ACTUALLY DECIDES THE SUPPLY, MISCLICKS, NOT THE DESCENT

**ME:** *"one fail is -500 nightshade which is months of farming."* Exactly right, and it dominates
everything else. Expected nightshade per **successful** kill, counting failed attempts:

| misclick rate | kills / 24 | nightshade per kill, all-in |
|---|---|---|
| perfect | 24/24 | **338** |
| 0.25% | 7/24 | **774** |
| 0.5% | 2/24 | **1735** |
| 1% | 0/24 | **no kills at all** |

Tuning the descent from `cakeBelow` 4 to 9 saves **220** nightshade. **A quarter of one percent of misclicks
costs 390.** Click accuracy is worth more than every descent setting on this page combined, and it is the one
thing no parameter here can buy.

---

# ★★ THE NIGHTSHADE SUPPLY, the v6 working (superseded above, kept for the trail)

I, 21 Aug: *the quest fight only happens once; post-quest kill count happens over and over, so the supply
that matters is nightshade.* Correct, and the shipped route was spending **904 cave nightshade a kill.**

### Where they go, measured

| route | self-damage clicks | nightshade | arms | floor ticks taken |
|---|---|---|---|---|
| **QUEST, floor tank** | **0** | **0** | 0 | 400 → **400 recoil** |
| POST-QUEST, Redemption | 1218 | 904 | 307 | 414 |

### The mix that makes it cheaper, `cakeBelow`

Nightshade **halves**: 10 → 5 → 2 → 1. The rock cake hits a **flat 1**. So the two are *numerically identical*
wherever the halving is already 1 or 0, and the cake is strictly better there because it never runs out.

- The **arm click** deals 0 on either item (it is clipped to zero at the arm hitpoints). **Always free.**
- The **2 → 1 click** deals 1 on either item. **Always free.**

Measured on two disjoint blocks, 1201-1230 and 1601-1630, 30 seeds each, shipped route otherwise untouched:

| cakeBelow | nightshade | kills | median kill | verdict |
|---|---|---|---|---|
| 0 (was shipped) | 904 | 30/30 | 35.9 min | none |
| 2 | 647 | 30/30 | 35.9 min | free |
| 3 | 603 | 30/30 | 36.3 min | free |
| **4** | **556** | **30/30** | **36.3 min** | **★ shipped, −38% at +0.4 min** |
| 5 | 477 | 30/30 | 44.6 min | the floor, and it costs 9 minutes |
| 6 | 479 | 30/30 | 46.1 min | no better, slower |
| 7 | 480 | 28/30 | 52.1 min | starts losing kills |
| 99 (pure cake) | 0 | **0/30** | dies t602 | the 9-click descent, see below |

### ⚠ THE HONEST NUMBER: 556 is still a lot, and the descent is not where the rest of it is

The sums floor is about **116**, 520 recoil at the full **9** instances an arm, two nightshade an arm.
This plan converts **1.7** instances an arm, not 9. **The entire gap is the exit-clearance problem**, because
every instance banked lowers `1 + heal − instances`, which is exactly the number the exit roll has to be
survived on. If supply is the thing stopping post-quest kill count, **that** is the thing to pull, not the descent.

And the capacity 9 is hard: the Redemption heal is `⌊prayer÷4⌋` = 12 at 50 Prayer, clipped by the 10-hitpoint
bar to 9. More Prayer does not raise it. The bar is the cap.

---

# ★ CAN THE ZERO-NIGHTSHADE ROUTE RUN POST-QUEST? No, measured, not assumed

The quest floor tank uses **no self-damage item at all**, and the reason is worth stating because it is the
cleanest thing in this whole file:

> **Recoil is 10% + 1, rounded down. Every hit of 1-9 returns exactly 1.** The size of the roll is irrelevant to
> output. So the floor tank never needs to be at any particular hitpoint, it needs to be *hit*. It stands on
> live ice at 15, takes 3-5 a tick, returns exactly 1 a tick whatever the roll was, and eats back up.
> **400 floor ticks taken → 400 recoil dealt. One-to-one.** The randomness never touches the output; it only
> changes how often you eat.

That does not carry to the 7-10 floor, and the failure is food throughput, now measured (seeds 1401-1424):

| floor tank, post-quest | recoil of 520 | kills |
|---|---|---|
| 15 rest slots | 115 | 0/24 |
| 25 rest slots | 145 | 0/24 |
| **53 rest slots** (does not even fit an inventory) | 231 | **0/24** |
| control, same tank on the **quest** 3-5 floor | **400** | **22/24** |

At 3-5 you can take three ticks between heals. At 7-10 you can take **one**, so every heal buys one recoil, and
520 recoil needs 520 heals. **Post-quest must be the Redemption route, and the Redemption route must descend.**

---

# ★ THE ROCK CAKE, asked and answered, and the sums behind the question was right

The question: *can a longer stall bank more damage after the Redemption proc, so the descent is shorter and a
reusable cooled rock cake replaces month-farmed nightshade?*

**The sums is exactly right.** `postHp = 10 − instances`. At the full capacity of **9 instances the dump
lands you on 1 with zero descent clicks needed.** The routes currently convert **1.7, 2.1 instances per arm of
a possible 9**, which is why each cycle costs about four self-damage clicks.

**But the longer stall cannot get there, and the reason is that the thing to pull is self-limiting.** Measured:

| config | kills | instances/arm | stalls actually armed |
|---|---|---|---|
| quest, menu 1/3/7 | 160/160 | 1.81 | 1t×2 · **3t×217** · 7t×2 |
| quest, forced 7t only | 0/160 | 6.63 | 7t×1 |
| post-quest, menu 1/3/7 | none | 2.06 | 1t×5 · 3t×92 · 7t×37 |

The 7-tick stall is **already in the menu and already legal**, the planner simply does not pick it. Force it and
conversion does climb to 6.6-6.9 of a possible 9, near the ceiling, and the arm count collapses from 221 to 1 and
the run to zero kills.

The binding constraint is not capacity and not stall length. It is **the exit**. The dense instance source is
live ice under locked feet, and arming on live ice is rejected **66,778 times per run** (`deadZone_liveTile`):
every extra instance you bank lowers `1 + heal − instances`, which is exactly the number the exit roll has to be
survived on. **More damage banked buys the exit less clearance.**

### Why the cake specifically fails, traced, quest, seed 401, t81-t95

Belladonna halves: 10 → 5 → 2 → 1, **three clicks**. The cake hits 1: 10 → 9 → 8 → … → 1, **nine clicks.** Nine
ticks of standing still, and two independent things kill it:

```
 t81-t87  rock cake 9→8→7→6→5→4→3→2      seven clicks
 t83      ⚠ FORCED OFF, tile 6 can do 8 this tick and you are on 7
 t88      ⚠ PRAYER DRAINED TO 0, Redemption switched itself off before it could fire
 t89-t90  sweet + rest → back to 7 hp     the descent is undone
 t91-t94  rock cake 7→6→5→4              starting again
 t95      → tile 5 ⚠ nowhere safe to stop · floor 3 · ☠ DEATH at 3 hitpoints
```

1. **Prayer drain outruns the descent.** At bonus 0 a point lasts 10 ticks and the regen potion gives one every
   12. A 3-tick belladonna descent fits inside one point's life. A 9-tick cake descent does not.
2. **You are not allowed to stand still for nine ticks.** The floor forces you off mid-descent, forced movement
   forces a heal, and every heal undoes cake clicks. That is why the cake costs **16-38 self-clicks per arm** for
   a nine-click descent, against belladonna's 3.9 for a three-click one.

More `exitGrace` does not rescue it either, swept 4 / 6 / 8 / 10 / 12 on both floors, it is 0 kills at every
one, and the arm count falls as grace rises.

### ★ THE PART THAT MATTERS: the quest route needs no self-damage item at all

**The shipped 100% quest route is the floor tank. It carries no nightshade, no belladonna and no rock cake**, the floor *is* the descent. The nightshade farm is a **post-quest-only** problem, and post-quest it is ~1200
clicks off one stackable slot per kill.


> ## ⛔ CORRECTIONS TAKEN THIS ROUND, three of mine were wrong
>
> 1. **★ "These accounts need to be ten hit points. They're prayer-only pures."** I modelled method A at 13 and
>    method B at 20. **Both of those number sets are WITHDRAWN.** Everything below is rebuilt at **hpBase 10**,
>    ceiling 15 via the Guthix-rest overheal.
> 2. **★ "You could have put them together with a drop-down."** Done, one file,
>    `osrs-prayer-pure-sim.html`, **Route** dropdown picks Redemption or Floor-tank.
>    `osrs-protmagic-floor-sim.html` is now a stub that says so.
> 3. **★ "There is free floor, it's just not in the first 3×3 tiles, run out between specials and back for the
>    special."** Already how the sim routes: the stall selector picks any iced tile in the arena and the
>    reservation pulls you back to 1-9 for the fire tick. What was *wrong* was the constraint's span, see the
>    bug below, which was worth 48 ice blocks a run.
>
> ## ✅ THE CONTROL, Method A + the rectangle, quest variant, 160 seeds, page defaults
>
> | recoil | ends | kills | procs | prayer spent | blockRisk | misfire | starved |
> |---|---|---|---|---|---|---|---|
> | **400** | **t2641** (~26 min) | **160/160** | 220 | **220 of 264** | **0** | 0 | **0** |
>
> **One 4-dose prayer regeneration potion covers the kill with 44 points spare.** One inventory slot.
> Ring-free the same route is **142/160**.

---

# ★★★ THE CIRCULAR PATTERN, you asked whether one exists, and it BEATS the free router

**Yes. Four tiles, walked as a rectangle, one station per boss cycle.** And it is not a convenience tax paid for
memorability, it is the best plan on the page.

```
      x=6  x=7  x=8
 y=0 [  1    2    3  ]  ┐
 y=1 [  4    5    6  ]  ├─ TILES 1-9, the entrance. Inside it on every icicle FIRE tick.
 y=2 [  7    8    9  ]  ┘   Bolt-holes: 7 sits under 10 and 13, 9 sits under 12 and 15.
       ────────────────
 y=3    10   11   12       ← the ring's top two corners
 y=4    13   14   15       ← the ring's bottom two corners
 y=5  16  17  18  19  20     (the room opens out from here)

  ★ THE RECTANGLE:  10 → 13 → 15 → 12 → back to 10
                    down · across · up · across
  one station per boss cycle (32 ticks), then step up into 1-9 for the icicle
  11 and 14 are never trodden, a running step covers two tiles and skips the middle
```

**Quest variant, 10 hitpoints, 160 seeds (four disjoint blocks of 40), 11 unnoted rings, `blockRisk 0` throughout:**

| route | no ring | ★ RECTANGLE `10-13-15-12` | ★ SHUTTLE `13-14` |
|---|---|---|---|
| **A, Redemption** | 142/160 · t2749 | **160/160 · t2641** | 132/160 |
| **B, Floor tank** | 130/160 · t1931 | 155/160 · t2275 | **157/160 · t2115** |
| **H, Hybrid** | **95/160 · t1107** | 48/160, worse | worse |

Recoil is **400 on every row**. The ring costs nothing and buys 18/160 on the Redemption route and 27/160 on the floor-tank route.

### ⛔ The hybrid will not take a ring, and that is structural

Every ring costs it recoil outright, and several produced `blockRisk > 0`, invalid runs. Its speed comes from
arming beside whatever patch happens to be live at the moment the prayer point lands; pinning it to a fixed
route destroys exactly that. **Run the hybrid ring-free.**

### ★ WHY the ring works, traced, not guessed

Every death on the Redemption route across 80 seeds finishes on **t63 or t97**, the second or third icicle. Past t97 the run
never dies. The shape is identical every time:

> the dump leaves **FIVE hitpoints** (`1 + heal − instances`) → the reservation for the special fires →
> the walk into tiles 1-9 has to stop on an **iced entrance square** → the floor rolls 5 → dead.

**Five is inside the floor's 3-5 band, so any live patch is lethal at exactly the hitpoints the dump most often
leaves you on.** The entrance ices because the character is standing in it when the autos land. A corridor ring
keeps every auto one square *outside* tiles 1-9, so the reservation always has somewhere clean to stand.

**The opening is the fight.** Survive the first three icicles and the model never kills you again.

### ⛔ MEASURED AND REVERTED: fixing it with timing instead of geography

I tried a `specHold` guard, refuse to descend inside a window before the fire tick, eat back above the floor
roll instead. The diagnosis is right; what fixes it is not. At a 4-tick window the Redemption route goes **142/160 → 0/160** and
**401 recoil → 8**, because the cycle is 11-12 ticks long and the special comes every 32, so a guard wide enough
to matter eats the only window the cycle has. At 2 ticks it is 146/160, inside the noise. **Reverted, and left
written into the sim as a comment so it is not tried a third time.** what fixes it is spatial.

---

# ★★★ WHICH ROUTE TO ACTUALLY RUN, and it is not the same answer for the two bosses

| | most cyclical | easiest to execute | quest variant | post-quest |
|---|---|---|---|---|
| **A, Redemption + rectangle** | 4 tiles, 4 clicks/cycle | hardest, the whole fight at 1 hp | 160/160 · 27 min | **the only route that survives a floor of 6** |
| **B, Floor tank + shuttle** | **2 tiles, on/off** | **easiest, never below the floor roll** | **157/160 · 22 min** | 40/40 at floor 5, **6/40 at floor 6** |
| **H, Hybrid** | none, takes no ring | hardest to hold in your head | 95/160 · **16 min** | 20/40 at floor 5 |

### ✅ Quest variant → **run B**, the floor tank on the two-tile shuttle

Two tiles. No stall, no arm click, and you are **never below the floor roll**, which is the whole difference:
on the Redemption route a slipped click costs the run, on the floor-tank route it costs hitpoints you eat back. It is 157/160 perfect and
**the best of the three at every non-zero slip rate**. The Redemption route only wins at literally zero misclicks, by three
runs in 160, and it is nine minutes slower per attempt.

### ✅ Post-quest → **run A**, Redemption on the rectangle

**520 hitpoints · two 4-dose regeneration potions · 13 rings · 40 seeds:**

| floor | A + rectangle | A ring-free | B + shuttle | B ring-free | H |
|---|---|---|---|---|---|
| **3-5** | **520 · 40/40 · t3457** | 520 · 35/40 | **520 · 40/40 · t3059** | 520 · 25/40 | 520 · 20/40 · t1417 |
| **3-6** | **520 · 40/40 · t3986** | 133 · 15/40 | 267 · 6/40 | 274 · 2/40 | 490 · 15/40 |
| 3-7 | 25 · 0/40 | 18 · 0/40 | 217 · 0/40 | 213 · 0/40 | 287 · 4/40 |
| 3-8 | 24 · 0/40 | none | 188 · 0/40 | none | 88 · 0/40 |
| **6-10** (wiki) | 1 · 0/40 | none | 146 · 0/40 | none | 2 · 0/40 |

At **floor 3-5 they tie**, both 40/40, and B is 400 ticks faster. At **floor 3-6 the Redemption route is 40/40 and the floor-tank route
is 6/40**, and you will not know the floor's real maximum until you are standing on it. **A is the only route
solid to the answer.** Above a maximum of 6 neither works, the cliff between 6 and 7 is unchanged.

⚠ **And B's misclick advantage does not survive the trip across.** Post-quest at floor 3-5, kills out of 40:
A 40 / 23 / 12 / 2 against B 40 / 19 / 11 / 3 at 0 / 0.25 / 0.5 / 1 % slip, **a tie**. The bigger hits punish a
missed flick harder, which is exactly what B was leaning on. At floor 3-6 A leads 40/18/12/0 against 6/3/2/1.

### ⚠ THE HONEST CASE FOR THE HYBRID, it wins the metric I did not lead with

Counting **every** attempt, kills and deaths, with a minute to restock after a kill and five to re-gear after a
death, **expected wall clock per kill, quest variant:**

| slip per click | 0% | 0.25% | 0.5% | 1% |
|---|---|---|---|---|
| A, Redemption + rectangle | 27 min | 35 | 49 | 123 |
| B, Floor tank + shuttle | 23 min | 27 | 33 | 52 |
| **H, Hybrid** | **19 min** | **19** | **23** | **33** |

**The hybrid is the fastest plan per kill at every slip rate**, because its failures are cheap: it dies early and
you go again. It loses on everything you asked about, it takes no ring so there is no pattern to learn, it
completes barely half its trips, and it burns ~1.7 trips of supplies per kill. Post-quest at floor 3-6 it is
15/40, behind A.

**Pick the hybrid if you are optimising hours and do not mind dying. Pick B if you want to finish the trip you
started. Pick A when the floor might roll a 6.**

---

# ★★★ AND WHEN YOU PUT A HUMAN ON IT, THE PLAN CHANGES

`slipChance` is a per-click miss probability on the clicks that can end the run. **Kills out of 160:**

| slip per click | 0% | 0.25% | 0.5% | 1% | 2% |
|---|---|---|---|---|---|
| **A, Redemption + rectangle** | **160** | 107 | 70 | 21 | 5 |
| **B, Floor tank + shuttle** | 157 | **116** | **85** | **48** | **22** |
| **H, Hybrid, ring-free** | 95 | 90 | 73 | 47 | 16 |

**The Redemption route wins only at zero.** From one slip in four hundred onward the floor tank is the better plan, and by
one in a hundred it is **more than twice as good**, because A spends the entire fight sitting on 1 hitpoint,
where a missed click has nothing behind it, and B spends it above the floor roll.

> ## ⚠ A HOLE IN MY OWN MODEL, FOUND WHILE MEASURING THIS
>
> The **conditional** flick (`protMagic 3`), which is exactly what the floor-tank route runs, was never reading
> `slipChance`. Only the held-flick branch (`protMagic 2`) and the arm click paid the tax. The floor-tank route therefore
> read **157/160 at every slip rate up to 2%**, i.e. misclick-proof. That was a branch that forgot to pay the
> tax, not a property of the plan. **Fixed; the row above is the corrected one.** Any earlier statement from me
> that method B is insensitive to misclicks is withdrawn.

---

# ★ RING SUPPLY, carry them unnoted, and it is the right call

★ ME: *"I can use unnoted rings and not refill for the hybrid method, it will just take like 10 inventory
slots, maybe 11 of rings."*

Modelled as `ringMode 1` and it is now the page default. The comparison is not really about ticks:

| | noted refresh (`ringMode 0`) | ★ unnoted swap (`ringMode 1`) |
|---|---|---|
| clicks | **two-tick dialogue**, use noted ring, then `1` + Enter | **one click** |
| failure mode | **lost outright if any damage lands inside the two ticks** | cannot be aborted |
| needs | a damage-free, unstalled, off-ice window | any unstalled tick |
| ring ticks per run | A 21 · B 38 · H 21 | **A 9 · B 9 · H 7** |
| price | none | **~4 recoil a run on the Redemption route** (≈11 ticks) |

40 charges a ring × **10 rings = exactly the 400 hitpoints of the quest variant**; the run never went dry on 10,
so the 11th is margin, and your sums was right. The ~4 lost recoil is a ring crumbling **mid-stall**: the
swap can only resolve on the next unstalled tick, so the rest of that dump reflects nothing. That is what
`lostRecoil` counts.

**Given the slip table above, one unabortable click beats two fragile ones every time.**

⚠ **Post-quest wants 13 rings, not 10**, 520 hitpoints needs 520 charges.

---

## ⛔⛔ THE ONE THING THAT DECIDES WHETHER ANY OF THIS EXISTS

**Redemption fires when hitpoints fall *below* 10% of maximum. At 10 maximum that is below 1.0, and no integer
hitpoint value is below 1.0 and above zero. On the wiki's wording the band is EMPTY and method A is impossible
on a ten-hitpoint account.**

| band | recoil | kills |
|---|---|---|
| **at or below 10%** (1 hp = exactly 10% of 10) | **401** | **40/40** |
| strictly below 10% | **3** | 0/40 |

**The precedent is in your own file, for the analogous item:** *"the Phoenix necklace procs at exactly 20% or
below, **not strictly below as the wiki states**."* Same wording, same wrongness. **~70% inclusive on that alone.**

> ★ **THE TEST, ONE HIT.** 10 maximum hitpoints, sit at 1, Redemption on, take any hit, the wiki says it fires
> "including 0 damage". **Watch the hitpoints NUMBER, not the hitsplat** (`core-game-mechanics.md` §10, only
> the first four hitsplats render per tick, so a heal can be completely invisible). Nothing else on this page is
> worth doing before this.

---

## ★ METHOD A vs METHOD B AT TEN HITPOINTS, and it is not close

| | **A, Redemption into a stall** | **B, Protect Magic + floor recoil + sweets** |
|---|---|---|
| Prayer | 50 (49 is the requirement) | 37 |
| combat level | **9** | 8 |
| **quest variant @ 10 hp** | **401 recoil · 40/40 · t2713** | **229 recoil · 0/40** |
| *(for reference, at 20 hp)* | none | 400 recoil · 38/40 · t885 |

**B does not survive the ten-hitpoint constraint, and the reason is structural.** B heals the floor back rather
than freezing it, so it needs a working band between *"high enough that a floor roll cannot kill"*
(`icePatMax+1` = 6) and its maximum. At 20 hitpoints that band is 14 wide and B is the faster method. **At 10 it
is four wide and there is nothing to work with.**

**A is indifferent to maximum hitpoints, because the frozen cap makes every instance cost exactly 1**, the
stall converts a 1-5 roll into a 1, so the only thing that scales is the *number* of instances the heal can
absorb, which is `min(⌊Prayer÷4⌋, maxHp − 1)` = **9** at 10 hitpoints. That single property is why A survives
the cut and why it is the only one of the two that could ever reach the post-quest boss.

**★ At 10 hitpoints take exactly 49-50 Prayer and no more.** The heal is `⌊Prayer÷4⌋` = 12 but only **9** can
land, so `overheal` reads **678 across a kill**, three points thrown away on every one of 226 procs, and it
cannot be avoided because 49 is the requirement. **Every Prayer level above 50 is combat level for nothing.**
50 Prayer and 49 Prayer are both combat 9, so take 50 for the margin.

---

## ★ THE BUG THAT WAS WORTH 48 ICE BLOCKS A RUN

`blockRisk` sat at 41-48 at ten hitpoints no matter how far the router's reservation lead was pushed, **because
the router was never the offender.** The ice-block constraint on the stall selector tested the span
`[arm … arm+L−1]`. **The feet are locked from the arm through the DUMP inclusive**, because the dump is an
initiation tick and the step out only resolves the tick after. A special firing exactly on the dump tick found
the feet outside tiles 1-9. Measured at t193, seed 1.

Fixed to `[arm … dump]`, and `blockRisk` went **48 → 0** with the recoil unchanged. **Same off-by-one as the
dump-threat check, and the same lesson: the dump tick is not the last tick you are on that tile.** This is the
one counter where a non-zero value is not a worse score.

With it fixed, `reserveLead` goes back to **2**, the long lead was compensating for the wrong thing, and the
short one is 300 ticks faster.

---

## THE PATTERN, method A at ten hitpoints, lifted from the log

**Kit:** 50 Prayer · 10 hitpoints · ring of suffering · belladonna (stackable) · one 4-dose prayer regeneration
potion · a 3-tick stall (crate ring) · Protect from Magic · purple sweets and Guthix rest for the wait.
**⚠ Prayer bonus ZERO and the prayer HELD ON**, see the caveat below.

```
   hp 5    belladonna → 5                    ← descending, OFF the ice
   hp 2    belladonna → 2
   hp 1    belladonna → 1                      (1 prayer point in the pool)
   hp 1    ARM: belladonna + crate ring, SAME CLICK, stepping onto the chosen patch
           · the self-hit queues FIRST and deals 0, self-damage cannot kill
           · cap frozen at 1. Every floor tick now costs 1 and returns 1.
   hp 1    feet locked. floor → 1 DEFERRED
   hp 1    feet locked. AUTO → 1 DEFERRED · floor → 1 DEFERRED
   hp 10   DUMP. ✚ REDEMPTION procs off the 0-damage self-hit, heals +9 (3 wasted)
           · the queue is NOT nulled, every instance lands and every one recoils
   ...     wait off the ice until the next prayer point, then descend and repeat
```

**Five rules, each earned by a death:**

1. **Arm at ONE hitpoint, always**, the arm hitpoints ARE the frozen cap and therefore the cost of every
   instance in the stall.
2. **Self-damage on the same click as the stall**, at 1 hitpoint belladonna deals 0, and that zero-hit is what
   procs the prayer out of the stall. Phoenix nest shape, null removed.
3. **The dump may land only on 1 with a point in hand, or above the maximum floor roll.** Check the tile at
   **dump AND dump+1**.
4. **Never step onto a tile you cannot finish on**, the mover checks where it lands, never what it crosses.
5. **Stay in tiles 1-9 for the whole arm-to-dump span**, not just arm-to-hi.

**⚠ The prayer-bonus-0 result is configuration-dependent and I over-generalised it last time.** Held at bonus 0
the drain keeps the pool pinned at 1 so nothing is wasted to Redemption's drain-to-zero, and on the quest
variant that is the fastest configuration, with `prayerStarved` reading 0. **But the drain is 10 ticks a point
against a 12-tick regen, so the moment the cycle stretches, the point burns off before it can be spent.** On the
post-quest numbers the log shows exactly that: *"PRAYER DRAINED TO 0, Redemption switched itself off before it
could fire."* **The real thing to pull is spending the point on the tick it arrives; bonus 0 only achieves that while
the cycle is short.** Sweep it rather than fixing it.

---

## ✅ THE PROC ORDER, ★ I described it exactly, and the engine already matches

★ ME: *"rock cake into an inventory action stall… the rock cake could hit even a zero on my one HP and still
fire redemption. But it fires the redemption after all the other ones come in and allows me to survive, which is
specific to the redemption prayer. So all the damage calculated on me comes in after the heal. Say I redemption
from one, trigger it with a zero out of the stall to nine, and it has five attacks on me, it'll hit five
1 1 1 1 1 to recoil back to the boss with. Same works with auto attacks and all the other stuff."*

**Verified against the log, seed 3, t5-t12, this is the engine's own output, not a restatement:**

```
t5    ARM, belladonna + stall on the SAME click from 1 hp
      self-hit 0 queued FIRST · cap frozen at 1
t8    AUTO 1 → 1 DEFERRED (cap frozen 1)
t11   floor 4 → 1 DEFERRED (cap frozen 1)
t12   ✚ REDEMPTION at 1 hp (0-damage self-hit), heals +9 → 10 hp
      stall ended, 2 landed from 3 queued, 2 recoil
      → 10 − 2 = 8 hp
```

Your example is the same shape one size up: 1 → 9, five instances, 9 − 5 = 4. **Every queued instance lands as a
1 against the frozen cap and every one of them recoils, autos, floor ticks, poison, all of it.**

**One thing your own file settles, and it bounds the whole method.** *"Healing from zero is not itself a
survival, you will die with HP remaining. It is CCQ that saves you, and nothing else."* Redemption is not CCQ.
So the heal has to land while you are still above zero, and **capacity is exactly the heal**:

> **instances per proc ≤ ⌊Prayer ÷ 4⌋, capped by maxHp − 1. At 10 hitpoints that is 9.**

**★ AND THAT IS WHERE THE WHOLE REMAINING GAP IS. The sim converts 3.4 of those 9.** Your hand-walked example
converts 5. Nine is the ceiling. **Closing it is worth roughly 2.5×, t2713 → ~t1100.**

### Why the long stalls are refused, mechanism found, not solved

A 9-instance stall needs a 7- or 17-tick window. Dumping the candidate list mid-fight, **every 7- and 17-tick
candidate is rejected at every arm offset**; only 1/2/3-tick stalls survive. The reason is the prayer drain:

**held at prayer bonus 0, Redemption drains a point every 10 ticks, and the accumulator is always mid-cycle.**
Any stall long enough to hold 9 instances has a high chance of crossing the drain threshold before it dumps,
which empties the pool, switches the prayer off, and the planner correctly refuses to arm into a proc that
cannot fire. **So the configuration that wins on the quest variant is the same one capping stall length at
three ticks.**

⚠ **Raising the bonus does not simply fix it.** Bonus 12 / 24 / 40, held or toggled, all collapse to 9 recoil at
t95, the long stalls become legal and the run dies for a different reason. **That is the open work**, and it is
the highest-value thing left on the quest variant.

### ✅ One real conflict found and fixed on the way

Two safety rules were fighting: *don't cross live ice at 1 hitpoint* (the path guard, which exists because the
mover checks the tile it finishes on and not the ones it crosses) and *be on tiles 1-9 when the special fires*.
Traced at t97, unstalled at 1 hitpoint outside the entrance, walking back for the special, path guard refusing
every step for three ticks until the special fired with the feet outside.

**Your rule decides it: the ice-block constraint outranks dying.** A floor roll costs the attempt; an ice block
is melee-only and unrecoverable experience. The survival override already carried that exemption; the mover did
not. **`blockRisk` is now 0 across every configuration swept, and the control is unchanged at 402 / 40-40.**

---

## ★★★ THE HYBRID, built, and it is the fast one

**No, we had never built it.** Both methods lived in one file but you picked one; I declined the composite twice
on the grounds that it would be two engines producing an unfalsifiable number. That reason has expired, both
routes now run clean (`blockRisk 0`, `lostRecoil 0`, `redempMisfire 0`) and both kill the quest variant, so the
composite is testable rather than hand-waved.

**The idea is forced by the two plans' own shapes.** Method A converts ~1.6 instances per proc and then sits
idle for about eight ticks waiting for the next prayer point. Method B spends exactly those ticks farming the
floor for a recoil apiece. **So do both:** above the maximum floor roll with an empty pool, stand on MATURE ice
and take the ticks; the moment a point lands, descend and spend it on a Redemption stall where the frozen cap
converts hitpoints at 1:1 instead of 1:3.

### Quest variant, 10 hitpoints, 40 seeds

| route | recoil | ends | kills | kit |
|---|---|---|---|---|
| **A, Redemption stall** | 401 | t2713 (~27 min) | **35/40** | one 4-dose regen potion |
| **B, Floor tank** | 400 | t1915 (~19 min) | 29/40 | sweets, rests optional |
| **★ H, HYBRID** | 400 | **t1105 (~11 min)** | 24/40 | both |

**2.5× faster than A and 1.7× faster than B**, at some cost in reliability, and against the slip table, a
shorter fight is itself worth reliability, because 226 cycles of exposure become about 90.

⚠ **The farm branch inside the hybrid is the same one measured and reverted TWICE on the pure Redemption
route.** Both of those tests predate the ice-block span fix, the path-guard removal and the entrance-ranking
fix, three router bugs that were poisoning exactly this behaviour. It was re-tested from scratch here rather
than assumed either way, and this time it works.

---

## ⛔ A CORRECTION TO MY OWN NUMBER, method A is 35/40, not 40/40

Your *"seems like it's only modelling the level 3 fight"* was right about the file and it turned up something
worse than cosmetic.

**`HAVE`, the list of stall lengths, was still hardcoded `[1,2,3,7,17]`.** I had parameterised it earlier and
**the edit was lost in a later rewrite**, so the sweep I reported (*"adding the egg book and the guzzle changes
nothing"*) was measured against a list that already contained them. **The parameters did nothing and the
identical rows meant nothing. That test is withdrawn.**

Re-wired and re-measured honestly:

| stall kit | recoil | kills |
|---|---|---|
| **1 + 3 + 7 (your kit)** | **401** | **35/40** |
| 1 + 3 only | 400 | 36/40 |
| **3 only, no easter ring** | **6** | **0/40** |
| 1 + 3 + 7 + 17 (guzzle) | 401 | 35/40 |

**Method A is 35/40, not the 40/40 I had been quoting**, the extra was a **2-tick stall that is not in your
kit**. Two things survive: **the 1-tick easter ring is structural** (crate ring alone is 6 recoil), and adding
the 17-tick guzzle still changes nothing.

## ✅ AND THE PAGE NO LONGER LOOKS LIKE THE LEVEL 3 SIM

You were seeing dead parameters. The engine was prayer-only but the tables still displayed the whole combat-3
kit. Audited by checking which parameters the code actually reads, and **removed 42 that nothing reads**:

> hunter food (`meatSlots`, `meatImmediate`, `meatDelayed`, `meatDelay`) · phoenix necklaces (`pnecks`,
> `neckBand`, `neckHeal`, `pneckThresh`, `nestMin`, `nestNight`, `nullFarm`, `nullFarmSlack`, `nullClimbOver`,
> `neckClimb`) · the nightshade interface (`nsInterface`, `nsSafeTile`, `nsDefer`) · the cure (`softThroughStall`)
> · **the entire 14-parameter Logistics group**, looting bag, herb withdraws, necklace pickups, inventory
> slots · and the level 3 descent toggle (`descentMode`, `selfMode`, `nightshadeTicks`, `selfDmg`).

**118 parameters → 81, and the Logistics table is gone from the page entirely.** What is left is the fight
(boss, ice), the kit this account actually carries (heal, stall), the routing, and the prayer model.

---

## ✅ AUDITED: THE STALL STILL LOCKS THE FEET, the movement work did not leak

★ I asked for this check directly, and it was the right thing to ask: three rounds of movement corrections
all touched the mover, and the one rule that must never bend is that **a stall takes your feet as well as your
hands.** Audited across both plans, 40 runs:

| check | result |
|---|---|
| mid-stall tick pairs examined | 9,104 |
| **positions changed mid-stall** | **0** ✅ |
| dump ticks examined | 4,522 |
| **positions changed on the dump tick** | **0** ✅ |
| arm-tick steps (legal) | 2,265 |
| nightshade-interface stalls (the only exception) | 0 |

**Feet locked from the arm through the dump inclusive**, which is the same span the ice-block constraint now
uses. The 2,265 arm-tick steps are the one legal case and they are your own correction: the step onto the chosen
tile **shares** the arm tick, because that is the only tick on which position can be set at all, from the arm
onward the feet belong to the stall. And the nightshade-interface exception reads 0 because that belonged to the
phoenix null phase, which left with the level 3 plan.

**What the movement corrections actually changed was the UNSTALLED ticks only**, which tile you run to, and
whether a tile in between blocks the run. Nothing in them touched `frozenFeet`.

---

## ⚠ EAT-AND-MOVE SHARE A TICK, I knew it, the engine models it, and my ROUTER does not use it

★ ME: *"You can eat and move at the same time, so that cooldown between the sweets doesn't impede movement
really at all."*

**Correct, and it is already in `core-game-mechanics.md` §14**, an inventory action and a movement are
different registers and share a tick. **The engine models it properly:** a sweet and a step both resolve on the
same tick whenever both are wanted.

**What does NOT use it is my router, and the waste is large.** Measured on the sweets-only tank run:

| ticks | moved | **stood still** | ate | **ate AND moved** | ticks earning |
|---|---|---|---|---|---|
| 3,000 | 1,345 | **1,655** | 1,848 | **694** | **13.3%** |

**Over half the fight is spent standing still while eating.** Those are free movement ticks being thrown away, the climb back from a floor instance ought to double as the walk to the next patch or into the entrance, and it
does not. Recovering them is worth up to about **2×** on the tank route.

⛔ **Two attempts, both measured and both REVERTED.** Staging next to the oldest mature patch during the climb
took the quest route from **400 / 29-40 to 246 / 2-40**, and sweets-only from **400 / 22-40 to 27 / 0-40**, because a tile adjacent to live ice is precisely where the next auto seeds you, and at climbing hitpoints that
is fatal. Pre-positioning into the entrance had the same shape.

**So this is logged as the open improvement rather than shipped worse.** The mechanic is yours and correct; the
routing that would exploit it is not written yet, and my two guesses at it both made the number worse. **It is
the single biggest known inefficiency left in method B**, and it does not touch method A, whose idle ticks are
spent waiting for a prayer point it cannot hurry.

---

## ✅ SWEETS ALONE CARRY THE QUEST VARIANT, confirmed, and it removes the rest bank from the plan

★ ME: *"Only if post-quest floor damage hits 10 or more would we run out of rests. Sweets can supplement all
health except for the +5 and they are stackable."*

**Confirmed on the quest variant, and it is the cleanest result on this page.** Method B, 40 seeds:

| kit | recoil | ends | kills |
|---|---|---|---|
| 25 four-dose + 28 three-dose (184 doses) | 400 | t1915 | 29/40 |
| **ZERO doses, purple sweets only** | **400** | t2837 | **22/40** |

**The rests buy 30% off the clock and about seven percentage points of survival. They do not buy the kill.**
Sweets are stackable, so the healing bank is effectively one inventory slot and the constraint is the **3-tick
eat cooldown**, not volume: 1-3 per 3 ticks is **0.67 hitpoints per tick, forever**. A Guthix rest drunk one
tick after a sweet adds 5 without touching that cooldown, which is what the extra 1.67/tick is.

**So B's sustainable rate is `0.67 ÷ (damage per floor instance)` instances per tick, indefinitely.** At the
quest floor of 3-5 that is one instance per 6 ticks with no supply cost at all.

### ⚠ But post-quest it stops being about volume before the floor reaches 10

Your "10 or more" is optimistic by my model, the wall arrives at **7**, and it is not a supply wall, it is a
survival one. Every single death at a floor maximum of 7 reads **`doses 0`** followed by a floor roll landing
on exactly the hitpoints left. **520 hitpoints, floor 3-7, 40 seeds:**

| rest slots | doses | recoil | kills |
|---|---|---|---|
| 25 (your kit) | 184 | 218 | 0/40 |
| 50 | 284 | 314 | 0/40 |
| 100 | 484 | 498 | 6/40 |
| **200** | **884** | **520** | **40/40** |

**It does close on supply, but it needs about 880 doses, which is 200 inventory slots.** That is not a kit, it
is a wish.

**The mechanism, and it is the special reservation rather than the floor itself.** On sweets alone you recover
0.67/tick, so after a 7-damage instance you spend ~10 ticks climbing from 3 back to 10. **You cannot choose
where you are for all of those ticks**, every 32 ticks the special forces you into tiles 1-9, and the entrance
carries ice because her autos seed wherever you stand, including there. At the quest floor of 3-5 you survive an
iced entrance tile from 6. **At a floor maximum of 7 you need 8+, and on sweets alone you are frequently below
it.** That is what kills it, and no amount of rests changes it once they run out, it only postpones it.

**Which is the same conclusion the stalled route reaches from the other side, and worth stating together:**
method A's frozen cap makes it indifferent to the floor's size, method B's whole cycle is denominated in it.

---

## ✅ YOU WERE RIGHT, BOTH METHODS WORK ON THE QUEST VARIANT. Three router bugs, all mine.

★ ME: *"Both methods should work for the quest variant because the floor only damages 3-5 during quest."*

**Correct, and my 0/40 for method B was three stacked bugs in my routing, not the fight:**

1. **The false path guard**, retracted last round; running skips the tile in between.
2. **The step-off ranking was inverted.** I had it prefer the ENTRANCE, on the theory that tiles 1-9 are always
   safer. It is the opposite, and your own level 3 file already says so: *"take autos out in the room, and be on
   a CLEAN entrance tile only for the special."* Ice seeds wherever you stand when an auto lands, so parking in
   the entrance between farm trips **ices the entrance**, and then the special's reservation walks you onto an
   iced entrance tile at 6 hitpoints. Traced at t895: `want=reserve`, no clean entrance tile left, forced onto a
   patch of age 17, dead in three ticks. **Flipped to prefer out in the room.**
3. **`stepOn` was 14 on a 10-hitpoint account**, a default carried over from the 20-hitpoint model, so the
   "climb back to here before returning to the ice" threshold was unreachable and the plan almost never went
   back. **Set to 10.**

### The quest variant, 40 seeds, both methods kill

| | recoil | ends | kills | kit |
|---|---|---|---|---|
| **A, Redemption stall** | **401** | **t2713** (~27 min) | **40/40** | one 4-dose regen potion |
| **B, Protect Magic + floor** | **400** | **t1915** (~19 min) | **29/40** | 25 rests + 28 bag rests + sweets |

**A is slower and never dies. B is 30% faster and dies about a quarter of the time.** At 10 hitpoints B's whole
band is 8→10, so it takes one floor instance per trip and climbs back, it works, but it has no margin.

⚠ **The Guthix-rest overheal to 15 makes B WORSE, not better** (365 recoil, 0/40, runs out of clock): climbing
to 15 costs more ticks than the extra floor tick returns. Sit at 10 and cycle faster.

### The post-quest variant, the floor is still the only question

★ ME: *"unsure what it is post quest."* The wiki says the icy pool deals **6-10 per tick**, verbatim, and you
are authoritative over the wiki. So here is the answer for every value. **520 hitpoints, two regeneration
potions for A, 25 rests for B, 40 seeds:**

| floor damage | **A, recoil / kills** | **B, recoil / kills** |
|---|---|---|
| **3-5** (the quest value) | **522 · 40/40** | **520 · 23/40** |
| **3-6** | **520 · 23/40** | 262 · 1/40 |
| 3-7 | 18 · 0/40 | 213 · 0/40 |
| 4-8 | 25 · 0/40 | none |
| 6-10 (the wiki figure) | 1 · 0/40 | 140 · 0/40 |

**Both methods have a cliff between a floor maximum of 6 and 7, and B's is slightly earlier than A's.** So:

- **If the post-quest floor is 3-5 like the quest one, both methods already kill it**, A on two potions,
  B on the same rest bank. Nothing new is needed.
- **If it is 6-10 as the wiki says, neither works at 10 hitpoints** and the frozen cap is the only mechanism
  with a plan through, because it turns any roll into a 1.

**Measure the floor's MAXIMUM roll once, post-quest, and read the row.** It is one number and it decides the
entire fight.

---

## ★★ SURVIVABILITY, the number you asked for, and it is the most important one on the page

Every kill rate here is measured against a planner that never misclicks and never hesitates. Over a 27-minute
fight that is not a person. So the sim now carries a **`slipChance`** dial, a per-action chance that either of
the two clicks that can end the run does not go in: **the ARM** (cake + stall on the same tick) and **the
PROTECT FROM MAGIC FLICK**. Method A, quest variant, 40 seeds:

| slip per click | 0% | 0.25% | 0.5% | 1% | 2% |
|---|---|---|---|---|---|
| **kills** | **40/40** | **24/40** | **19/40** | **10/40** | **0/40** |
| recoil | 401 | 400 | 372 | 195 | 59 |

**One click in two hundred takes you from certain to a coin flip. One in a hundred halves it again. One in
fifty is zero.** The reason is volume, not fragility: 226 cycles × 2 critical clicks ≈ **450 chances to die**,
and a slipped arm leaves you sitting at 1 hitpoint with no cover at all.

**So the routine is not a convenience, it is the plan.** Throughput is worth nothing next to click reliability
here, the difference between 0.25% and 1% slip is bigger than every optimisation in this document combined.

# ✅ WHICH RUNS KILL, every preset, exactly as the page ships it

You remembered the sim working every time. **You were right, and I muddled it by quoting failure numbers from
one hostile scenario without saying which one.** 80 seeds per preset:

| preset | recoil | ends | kills |
|---|---|---|---|
| ★ QUEST, Floor tank on the shuttle | 400 | t2110 | **80/80 · 100%** |
| QUEST, Redemption on the rectangle | 400 | t2641 | **80/80 · 100%** |
| QUEST, Hybrid, ring-free | 400 | t1127 | 49/80 · 61% |
| QUEST, Floor tank on sweets alone | 400 | t3108 | 76/80 · 95% |
| ⛔ **POST-QUEST, THE REAL FIGHT** *(520 hp, floor 7-10)* | 1 | t95 | **0/80** |
| ⚠ HYPOTHETICAL, post-quest boss with a 3-5 floor | 520 | t3457 | 80/80 · 100% |
| CONTROL, no ring | 400 | t2749 | 70/80 · 88% |

**Every quest-variant row kills. `blockRisk 0` on all of them.**

⛔ **And I had the post-quest presets mislabelled.** ★ ME: *"post quest is 7-10, pre quest is 3-5."* I had two
presets named POST-QUEST sitting at 100% that were actually running the **quest** floor against the post-quest
boss, a hypothetical dressed up as a plan. There is now exactly **one** post-quest preset, at the real 7-10
floor, and it scores **0/80**. The 3-5 version survives only as a clearly-labelled what-if, because it does earn
its place: it isolates the floor as the thing that breaks that fight. Her 520 hitpoints and her 22/34 max hits
are **not** the problem, the same setup with a quest-like floor is 80/80.

**The only failing row is the one built to fail**, the post-quest boss at your 7-10 floor figure. Every `0/80`
and every "unsolved" I have written is about that row and nothing else. The hybrid at 61% is the other soft
number, and that is by design: it is the fast, reckless plan.

# ★★ THE TWO PLANS, everything else on this page is workings

| | kills | time | slots | the loop |
|---|---|---|---|---|
| **QUEST**, floor tank | **160/160** | ~30 min | 28 of 28 | step 13 ↔ 14, bolt-hole 7 / 8 |
| **POST-QUEST**, Redemption | **27/160** | ~16 min | 25 of 28 | room loop 16 → 20 → 26 → 22 |

They are the first two entries in the **Preset** dropdown and the page boots on the quest one. Everything below
the separator in that dropdown is an alternate or a disproof.

### ⚠ "No ring" meant NO LOOP, my terminology, and it was terrible

★ ME: *"What does 'no ring' specifically mean in your responses?"* Fair. I had been calling the fixed tile
route a **ring**, on a page whose whole premise is a **ring of recoil** and a **ring of suffering**. Two
completely different things sharing one word, and I never noticed. **It is called the LOOP now**, everywhere, the dropdown, the parameter, the notes. When I said "ring-free" I meant "the planner picks a tile every tick
instead of walking a fixed route."

## ✅ QUEST, floor tank, loop 13 ↔ 14

**160/160 · 400 recoil · ~30 min · `blockRisk 0` · exactly 28 slots.**

| item | slots |
|---|---|
| rings of recoil, unnoted (10 used + 1 spare) | 11 |
| prayer regeneration potion | 1 |
| purple sweets, stackable, ~410 | **1** |
| Guthix rest, 15 four-dose flasks | 15 |
| **total** | **28 of 28** |

```
  STEP ON   13     → it ticks you 3-5, each tick recoils 1
  EAT       while you stand, sweet, or sweet then rest one tick behind it
  STEP OFF  to 7   → one square, always the same square
  STEP ON   14     → the other station
  STEP OFF  to 8
  ⛔ NEVER step onto ice inside 6 ticks of the icicle, that is the whole fix
  HOME      tiles 1-9 on every icicle FIRE tick
```

**The rests are a speed dial, not a survival one.** It is 160/160 at *every* bank size including zero, and the
whole range from a full bag to none is **30 → 35 minutes**. Drop a flask whenever you want the slot, about
twenty seconds each.

## ✅ POST-QUEST, Redemption, room loop 16 → 20 → 26 → 22

**27/160 · ~16 min · 25 of 28 slots.** Up from **zero**.

| item | slots |
|---|---|
| ring of suffering + noted rings of recoil | **1** |
| prayer regeneration potions × 2 | 2 |
| purple sweets | 1 |
| Guthix rest, 20 four-dose flasks | 20 |
| belladonna | 1 |
| **total** | **25 of 28** |

★ **The noted refresh earns its place here and only here.** It is the harder click, a two-tick dialogue that
aborts if anything lands inside it, but it frees **twelve slots** for rest, and at this floor that is worth
**seven kills** (27/160 against 20/160 on unnoted rings). On the quest variant it is worth nothing, so use
unnoted rings there and keep the easy click.

⚠ **17% is not a guarantee. It is the best that exists**, and the descent scheduler is still untried.

---

# ⛔ COOLED ROCK CAKE AS THE DESCENT, measured, and it does not work

★ ME: *"look into seeing if this is possible with cooled rock cake guzzle descents rather than nightshade, as
that's harder to get."*

| descent tool | the Redemption route | hybrid |
|---|---|---|
| belladonna (halves) | **160/160 · t2641** | **160/160** |
| cooled rock cake (1 damage) | **0/160 · dead t255** | 45/160 |

**The halving is what makes the descent fit.** 10 → 5 → 2 → 1 is three clicks; 10 → 1 on a flat 1 damage is
**nine**, and the window between the prayer point landing and the next attack is not nine ticks long. The cycle
never completes and the run dies at two and a half minutes.

★ Your other instinct is the right one though: *"maybe standing on subsequent floor tiles might be better to
help lower hp quick."* **That is exactly what the floor tank does**, it uses the floor itself as the descent,
which is free and needs no consumable at all. It is also why the floor-tank route is the one that does not need belladonna,
nightshade or a rock cake in the bag. **The quest route above carries none of them.**

---

# ★★★★ THE 100% ROUTE, and it is also the easiest one

★ ME: *"If there are always free tiles now on the descent I'm not sure why it's dying in any of these. Still
need easiest most optimized 100% route."*

**He was right that the remaining deaths were avoidable, and what fixed it was two ticks per block.**

Every surviving death on the floor-tank route was one shape. Traced, seed 54:

```
t670  hp 10  steps ONTO ice age 38, takes 5           → hp 5
t671  hp  5  the icicle cycles · the reserve drags him into tiles 1-9
             every entrance square is iced · "nowhere safe to stop"
             floor rolls 5                             → ☠ DEAD
```

**It spent its entire buffer on a floor tick one tick before it needed that buffer to walk home.** The trip
earned **one** recoil and cost the run.

`holdBeforeSpec 6`, do not step ONTO live ice inside six ticks of the fire tick. Two ticks of a 32-tick block.

| hold | kills | ends |
|---|---|---|
| 0 | 157/160 | t1852 |
| 2 | 157/160 | t1852 |
| 4 | 157/160 | t1911 |
| **6** | **160/160** | **t2082** |

**230 ticks, under two and a half minutes, for the last three kills.**

## ★★ And it does not trade away the thing that made the floor-tank route worth running

| route | 0% | 0.25% | 0.5% | 1% | 2% |
|---|---|---|---|---|---|
| **★ B, floor tank, shuttle, hold 6** | **60/60** | **48/60** | **43/60** | **24/60** | **5/60** |
| B, no hold | 59/60 | 44/60 | 37/60 | 25/60 | 6/60 |
| H, hybrid + room ring | 60/60 | 43/60 | 29/60 | 13/60 | 3/60 |
| A, Redemption + rectangle | 60/60 | 42/60 | 29/60 | 9/60 | 2/60 |

**It is simultaneously the perfect-play route AND the most misclick-tolerant one**, 43/60 against 29/60 at one
slip in two hundred, half again better than either Redemption plan. Those two things usually trade against each
other and here they do not, because **you are never at 1 hitpoint on this plan.** A missed click costs
hitpoints you eat back, not the run.

## ⛔ AND THE LOADOUT I SHIPPED DID NOT FIT IN AN INVENTORY

★ ME: *"the less rests the better as long as it doesn't kill an insane amount of time."* Sweeping that
turned up an error I should have caught much earlier. The preset asked for **25 four-dose flasks plus 28
half-made rests = 53 slots of Guthix rest.** Add 11 rings, a regeneration potion and the sweets and it wants
**66 slots. An inventory is 28.** It was never carryable.

**`holdBeforeSpec 6` also makes the floor-tank route 160/160 at every rest count, including zero**, so the bank is purely a
speed dial now. 160 seeds:

| carried | slots | doses | kills | ends |
|---|---|---|---|---|
| 25×4 + 28×3 | **53, will not fit** | 184 | 160/160 | 20.9 min |
| 25×4 | 25 | 100 | 160/160 | 27.4 min |
| **15×4** | **15** | **60** | **160/160** | **30.4 min** |
| 10×4 | 10 | 40 | 160/160 | 31.8 min |
| 8×4 | 8 | 32 | 160/160 | 32.1 min |
| 4×4 | 4 | 16 | 160/160 | 33.7 min |
| none | 0 | 0 | **160/160** | 35.0 min |

**The whole range from a full bank to nothing is 21 → 35 minutes, and every row kills 160 out of 160.**

Below about 8 slots the rests stop earning: 8 slots buys **2.9 minutes** over carrying none at all. The value
is all in the first ten or so.

### ✅ The loadout that actually fits, and it is exactly 28

| item | slots |
|---|---|
| rings of recoil (unnoted, 10 used + 1 spare) | 11 |
| prayer regeneration potion | 1 |
| purple sweets, stackable, ~410 of them | **1** |
| **Guthix rest, 15 four-dose flasks = 60 doses** | **15** |
| **total** | **28 of 28** |

**~30 minutes, 160/160.** Want it faster, you cannot, there is no slot to put the rests in. Want more room in
the bag, every flask you drop costs about twenty seconds and nothing else.

⚠ **The sim now prints this budget under every run** and says **DOES NOT FIT** in red when it exceeds 28. That
check should have existed from the first handback.

> ### ✅ THE ANSWER
>
> **Floor tank · shuttle 13-14 · bolt-holes 7 and 8 · never step onto ice within six ticks of the icicle.**
> **400 recoil · 160/160 · ~30 minutes · `blockRisk 0` · exactly 28 inventory slots.**
>
> It is the first preset on the page and the page boots on it.

### ★ And the hybrid reaches 100% too, on the room ring

The hybrid ring-free was 95/160. On the **room ring** it is **160/160 at t2545**, because what was killing it
was icing its own refuge, the same disease as everything else. It is a little faster than Redemption and
markedly less forgiving than the floor tank. **Three routes now hit 160/160 on the quest variant.**

---

# ★★★ POST-QUEST AT THE REAL 7-10 FLOOR NOW KILLS, 31/160, up from ZERO

**Every piece of this came from my corrections, and the last one was the big one.**

★ ME: *"Tiles should be freeing up and I never should have to run over a floor tile."* **He was right, and
the measurement is embarrassing:**

| | quest variant | post-quest 7-10 |
|---|---|---|
| live patches in the **arena** (144 tiles) | 7.1 → **4.9% of the floor** | 4.8 → 3.4% |
| live patches in the **entrance** (9 tiles) | 6.9 → **77% of the box** | 4.8 → 54% |
| fewest clean entrance tiles seen | **0 of 9** | **0 of 9** |
| ticks spent inside the entrance | **98%** | 96% |

**basically every patch in the arena was in the entrance, because that is where it stood.** 135 clean tiles
sat unused while the character iced itself into a 9-tile box. **And my own ring was doing it**, the rectangle's
bolt-holes were 7, 7, 9, 9, all entrance squares.

### The fix: move the ring out into the room

```
 y=5    16  17  18  19  20
 y=6  21  22  23  24  25  26  27

  ★ THE ROOM RING   farm  16 → 20 → 26 → 22    (the outer corners)
                    rest  17 → 19 → 25 → 23    (the inner square)
  come back into tiles 1-9 for the icicle FIRE tick, and only for that
```

**Post-quest 7-10, four disjoint blocks of 40 seeds:**

| config | b1 | b2 | b3 | b4 | total |
|---|---|---|---|---|---|
| entrance bolt-holes (what I shipped) | 0 | 0 | 0 | 0 | **0/160** · 12 recoil |
| room ring, bolt-holes still in the corridor | 0 | 0 | 0 | 0 | 0/160 · 230 recoil |
| room ring, **no** `redempBoost` | 3 | 1 | 0 | 2 | 6/160 |
| **room ring + `exitGrace 2` + `redempBoost 1`** | **11** | **10** | **5** | **5** | **31/160 · 219-308 recoil** |
| same at `exitGrace 3` | 7 | 11 | 6 | 8 | 32/160 |

**All three parts are structural**, remove any one and it falls to 0-6 in 160. And it is consistent across
disjoint seed blocks, so it is not the seed luck that caught me earlier.

### ⚠ And sweets beat pizza, exactly as I predicted

★ ME: *"optimise for lowest rest count though, as pizza will take some inventory at 2 bites per slot."*

My first pass had the climb eating **88 pizza bites a run, 44 inventory slots, not carryable.** Switched the
climb to stackable purple sweets and kept the big food as an opt-in:

| climb | recoil | kills | sweets | pizza |
|---|---|---|---|---|
| pizza + overheal rest | 227 | 0/40 | 0 | **88** |
| **sweets + overheal rest** | **353** | **9/40** | 210 | 0 |

Not just cheaper, **better**, because the eat cooldown gets spent on something that costs nothing.

> **⚠ 22% is not a plan.** Median recoil is 291-374 of 520 and most runs still end at 1 hitpoint. But it has
> gone from *provably stuck at zero* to *a fight that sometimes ends*, and the remaining deaths are no longer
> the planner refusing to act.

---

# ★ A MECHANIC CORRECTION FROM ME, the rest overheals from ANY hitpoints

★ ME: *"chocolate cake has 3 bites (more per slot) and does 5 hp, combo eaten into a rest for +10 overheal
with the rest sitting on top, at least from 1 hp to 11."*

That is a rest landing at **6** and taking him to **11**. The engine forbade it: it only let a rest exceed the
10 bar when the rest finished on a *full* bar. **His rule is now the default** (`restOverhealFrom 1`), because he
is authoritative on how the game works and that outranks the score.

**It turned out to help anyway**, the floor-tank route got materially faster:

| | before | after |
|---|---|---|
| B, floor tank + shuttle | 400 · t2115 · 157/160 | 400 · **t1852** · 157/160 |
| B, ring-free | 400 · t1931 · 130/160 | 400 · **t1680** · **133/160** |

**Four and a half minutes off the floor-tank route for free**, because a rest drunk mid-climb now lands its full value instead
of clipping at the bar. The Redemption route and the hybrid are unchanged.

### ⛔ And the burst combo, built, and it never fires

★ ME: *"I was thinking you needed an instant heal to 15 in a descent to run over an unavoidable pattern for a
tick."* Built as `burstHeal`: cake with a rest one tick behind it, triggered only when the tile under you can
already reach your hitpoints **and** no neighbour is clean.

**Post-quest it fires zero times a run.** Once the ring is out in the room there is basically always a clean
neighbour to step to, **the trapped case it was designed for is one the room ring already removed.** So the
answer to *"will I run over a live tile on the descent?"* is: **out in the room, basically never.** In the
entrance, constantly, which is what the old ring was doing to you.

⛔ My first pass at the trigger was far too loose (`hp <= icePatMax`, which is nearly always true at a 10-floor):
it fired 52-281 times a run and cost **36/160 → 5/160**. Tightened to the trapped case it costs nothing and does
nothing. **Left in, defaulted off, as insurance if the floor ever does saturate.**

★ **And you were right to reject the pizza version.** Using a big food as a general climb replacement scores
**0/160** post-quest against 31/160 on sweets. Sweets are stackable and therefore free; every bite of anything
else is inventory you cannot spare. The chocolate cake is in the model at your 5-per-bite, three-to-a-slot, but
only as the burst food.

---

# ⛔ A CORRECTION: THE REST BANK IS NOT A BINDER ON THE QUEST VARIANT

I told you the floor-tank route spends *"184 of 184 doses, zero margin"* and called it the binder. **The first half is true
and the conclusion is wrong.** It spends everything it has because it is opportunistic, not because it needs it.

★ ME: *"can we extend the fight and not use as many rests, subbing in sweets as much as possible?"* Swept:

| rest bank | recoil | ends | kills | doses used |
|---|---|---|---|---|
| 184 | 400 | t2119 | **40/40** | 184 |
| 105 | 400 | t2542 | 39/40 | 105 |
| 70 | 400 | t2733 | **40/40** | 70 |
| 35 | 400 | t2902 | 38/40 | 35 |
| 30 | 400 | t2926 | 38/40 | 30 |
| **0, sweets only** | 400 | t3115 | **39/40** | **0** |

**The rests buy speed, not survival.** The full bank saves about 1,000 ticks, ten minutes, over sweets alone,
and costs 25 inventory slots to do it. At 70 doses you keep 40/40 and get ten of those slots back. **Carry what
you want the trip to be worth; nothing about the kill depends on it.**

⚠ Post-quest is the opposite and stranger: *more* rests made it **worse** (36/160 at a 184 bank, 0/160 at 224+),
because the extra doses change the router's decisions and it dies earlier. That is a behavioural cliff, not a
supply curve, and it is unexplained.

---

# ★ THE EXIT sums, the one thing that IS proven impossible

Redemption heals `⌊prayer÷4⌋` = 12 at 50 prayer, clipped by the 10-hitpoint bar to **9**. So the exit out of a
1-hitpoint stall is **`10 − instances`**, and it can never be more than 10.

| instances | exit hp | max 5 (quest) | max 6 | max 7 | **max 10 (post-quest)** |
|---|---|---|---|---|---|
| 0 | 10 | LIVES (5) | LIVES (4) | LIVES (3) | **DIES** |
| 1 | 9 | LIVES (4) | LIVES (3) | LIVES (2) | **DIES** |
| 2 | 8 | LIVES (3) | LIVES (2) | LIVES (1) | **DIES** |
| 3 | 7 | LIVES (2) | LIVES (1) | DIES | **DIES** |
| 4 | 6 | LIVES (1) | DIES | DIES | **DIES** |
| 5 | 5 | DIES | DIES | DIES | **DIES** |

★ ME: *"coming out the stall, if the pattern hasn't expired won't it hit after the stall for my full hp?"*
**Yes, and post-quest every row dies, including zero instances, where you walk out at full health.** 1 + 9 = 10,
the floor rolls up to 10. Not a tuning problem, not a filter artefact: **sums.**

So dumping onto live ice post-quest is not an optimisation to avoid, it is **impossible**. The tile under your
feet at the dump must be expired, never-seeded, or inside the grace window. And the quest fight works for the
reason I gave: at 3 instances you exit on 7, and 7 − 5 = 2.

---

# ★★★ ME'S GRACE-WINDOW IDEA, it is right, and it moved the blockage

★ ME: *"Couldn't we just use a shorter stall at the beginning of an auto attack? And combine that with the
floor spike, but before the floor pattern does damage, since there's a delay between the spike damage and the
floor pattern damaging you on an initial spawn."*

**That delay is real and it is in the engine already.** The seeded-tile timeline, from your own measurements:

```
+0  auto lands, seeds the tile
+3  SPIKE (1-8)
+4  the pattern OBJECT spawns  ── visible, and completely harmless
+5  …still harmless
+6  …still harmless
+7  it starts DAMAGING, every tick, until +104
```

**Ticks +4, +5 and +6 are free floor.** The pattern is sitting there and cannot touch you. So a short stall armed
on the auto tick swallows the auto and the spike as capped 1s, and dumps into a window where **no roll is
coming**, which means the exit never has to clear a 7-10 roll at all.

That is exactly the case `deadZone_deadTile` was throwing away. Built as `exitGrace`, *"the exit needs N clear
ticks to act in"* replacing *"the exit must beat the biggest roll"*. **Counted, floor 7-10, 40 seeds:**

| rejection | exitGrace 0 | exitGrace 2 |
|---|---|---|
| **stalls armed per run** | **0** | **7** |
| `deadZone_deadTile` | **5,269** | **247** |
| `dumpKillsYou` | 3,861 | 991 |
| total rejections/run | ~11,700 | ~1,500 |

**The blockage is gone.** And the cycle demonstrably fires at a 7-10 floor, traced, seed 1:

```
t81  hp 1   REDEMPTION ARM, belladonna + stall, cap frozen at 1
t83  hp 1   floor 4 → 1 DEFERRED (cap frozen 1)
t84  hp 9   ✚ REDEMPTION, 1 landed from 2 queued
```

### ⛔ But it still does not kill, and the reason has moved

Recoil goes 1 → 11 and the run still dies around t91. The new top rejection is `dumpKillsYou` (66%), which is
genuine physics, not a filter artefact. And the trace says why:

```
t87  hp 10  ⚠ FORCED OFF, tile 6 can do 10 this tick and you are on 10
t90  hp  5  belladonna → 5
t91  hp  0  ⚠ FORCED OFF, tile 3 can do 8 and you are on 5
            → tile 7 ⚠ nowhere safe to stop · floor 2 · ☠ DEATH
```

**At a 7-10 floor with a 10-hitpoint ceiling, every live tile is lethal at FULL health.** The frozen cap
protects you inside the stall, that part works. What has no cover is the **descent**: three unstalled
belladonna ticks at 10 → 5 → 2 → 1, and at 5 or 2 hitpoints any live tile ends you. The arena saturates because
an auto seeds a tile every 8 ticks, and eventually there is nowhere to stand while you drop.

> ### ★ SO THE NEXT thing to pull IS YOUR OWN IDEA, APPLIED ONE STEP EARLIER
>
> The grace window is **three ticks**, ages +4, +5, +6. The descent is **three belladonna clicks**.
>
> **They are the same length.** Descend across the window on a freshly-seeded tile and arm on the tick the
> pattern comes alive at +7, at which point you are stalled and it is capped at 1. Every unstalled tick of the
> cycle would then be spent on ground that physically cannot damage you.
>
> That is a scheduler, not a filter: it has to *aim* the descent at a tile of a known age rather than reject bad
> plans after the fact. **Still not built**, it is the one untried thing to pull, and everything it needs now exists.

### ★ AND THE CLIMB COULD NOT REACH 15 EITHER, my pizza idea, built as `redempBoost`

★ ME: *"could we pizza + rest dose on these rare descents to go from 1 to 15 in one tick?"* **The reason that
matters is one number: at 10 hitpoints a max floor roll of 10 kills you; at 15 it leaves you on 5.** Fifteen is
the only state where live-floor contact is survivable post-quest.

**And the redemption route could not get there.** Two more lines that go unsatisfiable at a big floor:

- its climb eats **sweets, capped at `hpBase` = 10**, only a Guthix rest overheals, and only when it lands
  while you are already at full;
- its climb **target** is `icePatMax + 1`, which post-quest is **11 on a 10-hitpoint bar.**

Built: a big food to the bar, then an overheal rest on top, targeting `hpCeil`. **Measured, 40 seeds:**

| | quest | post-quest 7-10 | post-quest 3-7 |
|---|---|---|---|
| control | 400 · **40/40** · t2641 | 1 · 0/40 · t95 | 25 · 0/40 · t231 |
| `exitGrace 2` | 400 · **40/40** | 11 · 0/40 · arms **7** | 119 · 0/40 · arms 95 |
| `redempBoost 1` + `exitGrace 2` | 400 · **40/40** | 12 · 0/40 · arms 8 | 119 · **1/40** · arms 112 |

**Quest is untouched at every setting**, identical recoil, identical tick, it just spends 57 pizza bites where
it used to spend 57 sweets. At 3-7 the pair produces the **first kill this configuration has ever scored**. At
7-10 it moves 11 → 12 recoil and nothing else.

**Both ship defaulted to 0.**

> ### ⛔ WHERE POST-QUEST AT 7-10 ACTUALLY STANDS
>
> Four things have now been tried. Here is what each one settled:
>
> | | result |
> |---|---|
> | `rotateArm`, aim the stall at an expiring patch | no effect on quest, moves one post-quest row |
> | `deadTileExit`, exempt dead tiles from the clearance rule | **breaks** floor 3-6, rescues nothing |
> | `exitGrace`, my +4/+5/+6 window | **unblocked the planner**, 0 → 7 arms a run at 7-10 |
> | `redempBoost`, my climb to 15 | safe and free on quest, +1 kill at 3-7, nothing at 7-10 |
>
> **What is proven:** dumping onto live ice post-quest is impossible (the table above). **What is fixed:** the
> planner no longer refuses to arm, it now arms and then dies. **What is left:** the runs die around t96, which
> means supplies are not even the binder yet, surviving the first hundred ticks is.
>
> **What is NOT proven is that the fight is impossible.** The untried thing to pull is the descent scheduler, and it is
> untried, not rejected.

**`exitGrace` ships defaulted to 0.** At grace 2-4 it breaks the floor-3-6 case (40/40 → 1/40, because it lets
through exits that then die on the descent); at grace 6 it is a no-op everywhere. The quest variant is
**identical at every setting**, 80/80, 400 recoil, t2641, even though grace 2 accepts 19,285 plans a run the
old rule refused. So it is safe, it is off, and it is there for the descent work.

---

### ★ WHY the planner refuses to arm, counted, not asserted

I claimed this from reading the code, which is how I got the cliff half-right the first time. So every rejection
in the stall search is now counted by name and shown in the page under the verdict. **40 seeds, per run:**

| rule | quest 3-5 | post-quest 7-10 |
|---|---|---|
| **stalls actually armed** | **221** | **0** |
| `deadZone_deadTile` | 237 · 0.3% | **5,269 · 45%** |
| `dumpKillsYou` | 4,499 · 6% | 3,861 · 33% |
| `noLegalTile` | 1,166 · 1.6% | 1,336 · 11% |
| `deadZone_liveTile` | 66,682 · 89% | 834 · 7% |

**`deadZone_deadTile` is the answer.** Those are plans where the dump tile is **already dead**, no roll is
coming, and the rule rejected them anyway, because it says *"the exit must clear the biggest roll the floor can
make."*

At a 3-5 floor that means **beat 5**, and Redemption heals 9 from 1 hitpoint, so the exit can be anything up to
10. Plans pass, 221 stalls arm, the fight works.

At a 7-10 floor it means **beat 10**, and the exit tops out at *exactly* 10. **Not one plan can ever satisfy it.**
Zero stalls arm all run, the character parks at 1 hitpoint, Redemption drains itself off at t88, and he dies at
t95 to a floor roll of one.

⚠ **And the obvious repair still fails**, `deadTileExit`, which exempts dead tiles, breaks the 3-6 case
(80/80 → 3/80). The reason is that a dead tile does not stay dead: the next auto seeds it and it spikes for up
to 8 three ticks later, so sitting at 2 hitpoints on it is not safe either. **The rule is guarding something
real; it is just expressed in a quantity that goes unsatisfiable.** That is the open problem, stated precisely.

### So, plainly: does it live by standing on ice that expires, or does it die?

**Both, on different fights, and the distinction is the floor:**

- **Quest variant, floor 3-5.** It lives. Standing on ice that expires inside the stall is *how it is supposed
  to work* and the engine allows it, but it is a **bonus, not a requirement**, because a 3-5 roll against the
  hitpoints a 9-point heal leaves you on is survivable even when you get the timing wrong. That is why the quest
  presets are 100%.
- **Post-quest at 7-10, the real fight.** The expiry timing stops being a bonus and becomes **the only thing
  that could work**, and the planner never gets the chance, because `deadZone_deadTile` rejects every plan
  before one is ever armed. It dies of my filter, not of the floor.

---

# ✅ THE EXPIRY MECHANIC, checked, and the engine already had it

★ ME: *"we have to stand on the ice floor as it expires out of the stall, or else the tick after Redemption
heals us we do not have clearance and the floor will do 7-10 non-quest and 3-5 quest, we come out of the stall
with 1 1 1 1 1 and that lowers our healed hitpoints."*

**Both halves of that are already in the engine, and here are the lines.** The dump-tile threat is the maximum
across the dump tick *and the tick after it*, and only on this plan:

```js
const thr = Math.max(_thrAt(dump), (st.phase==='redempt' ? _thrAt(dump+1) : 0));
const postHp = 1 + healHere - dmgPerInst*(fixed0+n);
if(thr>0){ if(postHp - thr < 1) return; }
```

`postHp` is exactly *"the row of ones lowers our healed hitpoints"*, the heal minus every instance the stall
swallowed. And `_thrAt(dump+1)` is exactly *"the tick after Redemption heals us"*. The comment above it records
the run that put it there: *"the dump at t142 left 6 hitpoints on a tile whose patch was age 6, one short of
damaging, and the floor rolled 4 at t143. The planner read t142 and the engine billed t143."*

**What was NOT built is aiming for the expiry**, the engine filtered out plans that would die; it never
scheduled arms so the patch dies inside the lock. So I built it.

### ⚠ `rotateArm`, built, measured, and it is not the thing to pull

Ranks plans whose patch's last damaging tick falls inside `[arm … dump]` above everything else, so you bank a
capped 1-damage instance every tick of the stall and step out onto dead ground.

| | quest variant | post-quest 3-7 |
|---|---|---|
| off | 400 · 80/80 · t2641 | 25 · 0/80 · t231 |
| rank | 400 · 80/80 · t2641 | 25 · 0/80 · t231 |
| require | 400 · 80/80 · t2641 | **127 · 0/80 · t1887** |

**Identical to the tick on the quest variant**, because the old third-level tie-break already preferred the
oldest patch. Post-quest it moves one row and kills nothing. **Default 0.**

---

# ⛔ THE POST-QUEST CLIFF IS MY PLANNER, I found the line, and my fix was wrong too

★ my floor figure for the real boss is **7-10**, which supersedes the wiki's 6-10. Both are past the cliff,
so I traced a 7-10 run tick by tick instead of re-reading the table:

```
t80  hp 1  want=off  tile 6   stall 0   belladonna → 1 hp
t81  hp 1  want=off  tile 6   stall 0
…    fifteen straight ticks at ONE hitpoint, unstalled …
t88  hp 1  want=off  tile 11  stall 0   ⚠ PRAYER DRAINED TO 0, Redemption switched itself off
t95  hp 0  want=reserve  tile 5         floor 1 · ☠ DEATH, 1 landed against 1 hitpoints
```

**He did not die to a 7-10 floor. He died to a floor roll of ONE, at 1 hitpoint, because no stall was ever
legal.** And the reason no stall was legal is one line of mine:

```js
if(!(_after > p('icePatMax') || (postHp<=1 && _readyNow))) return;
```

`_after > icePatMax` says *"come out of the dump above the biggest roll the floor can make."* At `icePatMax` 5
that is a sane demand on a 9-point heal. **At `icePatMax` 10 it demands more than 10 out of a heal of 9. It is
arithmetically unsatisfiable.** So the cycle never starts, the character parks at 1 hitpoint, and Redemption
drains itself off while he waits.

**The cliff between a floor maximum of 6 and 7 is at least partly this line, not the fight.** That was suspected
in the level-3 write-up; it is now traced.

### ⛔ And the obvious fix is measured and rejected

`deadTileExit`, if the dump tile is dead at the dump *and* at dump+1, the exit only has to leave you alive,
because there is no roll coming. That is precisely the mechanic I describes, and it is narrower than the
blanket relaxation that was rejected before.

| | quest A | quest hybrid | post-quest 3-6 | 3-7 | 7-10 |
|---|---|---|---|---|---|
| off | 80/80 | 49/80 | **80/80 · 520** | 0/80 · 25 | 0/80 · 1 |
| on | 80/80 | **31/80** | **3/80 · 45** | 0/80 · 119 | 0/80 · 9 |

**It breaks the one post-quest floor that works**, 3-6 goes from 80/80 to 3/80, costs the hybrid 18 kills, and
rescues nothing. **Default 0.** Left in the file because the trace is the valuable part.

> ### ⛔⛔ SO: POST-QUEST AT A FLOOR OF 7-10 IS UNSOLVED
>
> Not "hard", unsolved. I know the planner is part of the problem and I know exactly which line, and the two
> fixes I could think of both measured worse. **Everything this document says about the post-quest boss assumes
> a floor that behaves like the quest one.** At your 7-10 figure, none of the three plans work as modelled.

---

# ⛔⛔ A CORRECTION I OWE THIS DOCUMENT, THE WALK IS NOT WHAT IS WORKING

I sold you the rectangle as *"down the left, across the bottom, up the right, back across the top, it keeps the
autos out of the entrance."* **The measurement is right and the explanation is wrong.** Isolated properly on
the Redemption route, 160 seeds each:

| what is pinned | kills |
|---|---|
| nothing (control) | 142/160 |
| **farm tile only**, ring picks where you stand, step-off left free | **142/160, identical to no ring at all** |
| farm tiles swapped for `17-18-19`, bolt-holes unchanged | **160/160** |
| farm tiles swapped for `10-10-10-10`, bolt-holes unchanged | **160/160** |
| bolt-hole pinned to ONE square (`8,8,8,8` or `7,7,7,7`) | 142/160 |
| **bolt-holes `7,7,9,9`** (the shipped ring) | **160/160** |
| bolt-holes `7,8,9` | 160/160 |
| bolt-holes `7,8` | 131/160 |
| bolt-holes `7,9` | **10/160** |

**The farm tiles are decoration.** And the Redemption route spends **97% of its ticks inside tiles 1-9**, so it was never
walking a rectangle in the first place. What the ring actually sets is **which entrance square you step off
onto, cycle by cycle.**

### ★★ And the real headline is bigger than the ring

Across 640 runs and every configuration above, **every plan-A death lands at t63 or t97, and not one lands
after.**

| | deaths | latest |
|---|---|---|
| control | 18 | **t97** |
| rectangle | **0** | none |
| bad rotation (`7,9`) | 150 | **t97** |
| no rotation (`8,8,8,8`) | 18 | **t97** |

**The Redemption route's kill rate is not a survival rate. It is the probability you survive the first ninety-seven ticks**, the second and third icicle, and the "ring" is a way of scripting that opening. After the third icicle the run
has never died, in any configuration I have measured.

⚠ **What I cannot explain yet:** `7,8,9` is perfect, `7,7,9,9` is perfect, `7,8` is 131 and **`7,9` is 10**. It
is not "rotate more is better", and I am not going to invent a story for it. It is left measured, and the
shipped ring is the one verified over four disjoint blocks of forty seeds.

### The floor-tank route is different, and I can't explain that either

On the floor tank the two halves are **both** structural, which is the opposite of the Redemption route:

| | kills |
|---|---|
| control | 130/160 |
| farm tiles only (`patrolRest 0`) | 113/160, **worse than the control** |
| rotation only (farm pinned to one tile) | 106/160, **worse than the control** |
| farm `13,14` + bolt-holes `8,8` (walk, no rotation) | 134/160 |
| **farm `13,14` + bolt-holes `7,8`** (the shipped shuttle) | **157/160** |
| farm `13,14,15` + bolt-holes `7,8,9` (same idea, three stations) | 107/160 |

Removing either half puts it below the control, and the obvious three-station generalisation is much worse.
**That is a tuned configuration, not a principle**, and it should be described that way until somebody explains it.

**What survives all of this unchanged:** the numbers. 160/160 and 157/160 are real, reproduced over four
disjoint seed blocks, at the same 400 recoil and `blockRisk 0`. What changed is *why*, and "walk the rectangle"
was my story, not the data's.

---

# ⛔ THREE THINGS ME CAUGHT IN THE BUILD, AND ALL THREE WERE REAL

### 1. "The sim is using Protect from Magic on the Redemption route too, not just the hybrid, is that correct?"

**Yes, and it is structural on every plan.** Swept it, 160 seeds:

| `protMagic` | the Redemption route recoil | ends | kills |
|---|---|---|---|
| **0, off** | 2 | t16 | **0/160** |
| **1, held** | 1 | t16 | **0/160** |
| **2, flicked** | **400** | **t2641** | **160/160** |
| 3, conditional | 6 | t107 | 0/160 |

With it off, **every** route dies at t16 with single-digit recoil. Her auto rolls to 16 against your 10, one
unstalled auto ends the run inside the first block. Held is unaffordable (drains 5 ticks per point against a
12-tick regen, and Redemption needs the pool). So it is not a hybrid-only feature and it is not a leak: **the
flick is what makes any of the three plans exist.**

⚠ What it *costs* is worth saying out loud, because it is counter-intuitive: **a protected auto returns no
recoil.** The Redemption route flicks it **218 times a run** and gets 112 more flicks blocked for want of a free tick. That is
218 hits' worth of output traded for survival, the plans still reach 400 because their damage comes out of the
stall dumps and the floor, not out of raw autos.

### 2. "The hybrid dies around tick 92, that doesn't sound right"

**It dies at t97, it is reproducible, and it is not a coding error, it is the same opening failure the ring
cures on the Redemption route.** Seed 1 traced tick by tick: the dump leaves **5 hitpoints** → the reservation for the special
fires → the walk into tiles 1-9 has to stop on an **iced entrance square** → the floor rolls 5 → dead.

The hybrid inherits it *because* it runs ring-free: it parks in the entrance between cycles, so the autos ice the
entrance. Seed 1 is what the page loads with, which is why it looks worse than it is, the median is still
**t1107 and 95/160**.

**Two narrower fixes tried, both measured, both reverted:**

| attempt | hybrid kills |
|---|---|
| control (ring-free) | **95/160** |
| ring supplies the bolt-hole only, farm tile still free (`patrolScope 1`) | 78-87/160, and seed 1 *still* dies at t97 |
| "never take an auto inside tiles 1-9" as a standalone rule (`cleanEntrance 1`) | **50/160** |
| full ring | 48/160, but seed 1 survives to t447 |

The bolt-hole version fails for a reason worth keeping: **the death is about where the AUTO lands, not where you
rest.** Only pinning the farm tile out of the entrance helps, and pinning the farm tile is exactly what costs the
hybrid its recoil. **So the hybrid's t97 death is not fixable without turning it into the Redemption route.** That is the
finding, not a bug, it is fast because it is reckless, and the recklessness is what kills seed 1.

### 3. "The tick sheet isn't producing a complete log"

**It was producing nothing at all, and it was not alone.** Five cards in the page were dead markup carried over
from the level-3 fork with no renderer behind them: `sheet`, `gaps`, `logi`, `triggers`, `drill`. My miss, and I
should have caught it when I loaded the page for the handback.

- **The tick sheet is now the full log you asked for.** ★ ME: *"the tick sheet is supposed to include the
  entire logged tick by tick breakdown of the entire fight simulated."* It does. The engine has always recorded a
  frame per tick; nothing was reading them except the scrubber. Every tick of the loaded run is now a row,   **hitpoints, her hitpoints, which tile, whether you were inside 1-9, whether the prayer was up, prayer points,
  stall ticks remaining**, and everything the engine said happened. Filters for *every tick* / *only ticks where
  something happened* / *damage and recoil* / *your clicks* / *icicle fire ticks*, a tick-range box, and a button
  that downloads the **complete** log as plain text (~2,170 lines for a kill run) so you can grep or diff it.
  Above it, the 32-tick block reference and your route's click phrase, both generated from the parameters so they
  cannot drift from the engine.
- **`gaps` / `logi` / `triggers` / `drill` are removed** rather than faked. They belonged to the level-3 plan's
  food/cure/null phases, which this account does not have.
- In their place: **a Supplies card.**

---

# ★ SUPPLIES, the diagnostic you asked for

The Supplies card reads the run currently in the timeline and shows what it actually spent, against what the
parameters say you carried. Across 160 seeds (median / max used):

| plan | Guthix rest doses | purple sweets | prayer points | rings |
|---|---|---|---|---|
| **A, Redemption + rectangle** | 58 / 62 of 184 | 58 / 62 | 220 / 226 of 264 | 10 of 11 |
| **B, Floor tank + shuttle** | **184 / 184 of 184** | 498 / 525 | 176 / 186 of 264 | 10 of 11 |
| **B, sweets only** | 0 | **908 / 953** | 260 / 264 of 264 | 10 of 11 |
| **H, Hybrid, ring-free** | 144 / 158 of 184 | 168 / 186 | 92 / 100 of 264 | 10 of 11 |

**Two things fall out of this that were not visible before:**

1. **The Redemption route is cheap.** 58 rest doses of a 184 bank and 58 sweets. It is the *reliability* that is expensive,
   not the supplies.
2. ⚠ **The floor-tank route drinks the entire rest bank, every single run, 184 of 184, on 160 seeds out of 160.** It is not
   that it needs exactly 184; it is that it drinks everything it has and then finishes on sweets. That is zero
   margin, and it is why the sweets-only variant takes t3115 instead of t2115: **the rests are what make the floor-tank route
   fast, and it is already using all of them.** If the post-quest fight runs longer, that bank is the first thing
   to raise.

⚠ **And one counter was under-reporting while I built this**: purple sweets were only being counted on the
stall routes, so the floor-tank route read **0 sweets** on its first pass. The floor-tank route heals through a different branch. Fixed, the 498 and 908 above are the corrected figures, and any earlier sweet count from me is withdrawn.

---

# ★ THE PRESET DROPDOWN, the streamlined process in one click

You asked where the dropdown for the recommended process was. It is now the **first control on the page**, above
Plan and Ring, and **the page boots on it** rather than on whatever the parameter defaults happen to be.

Each row sets the plan, the ring, the prayer mode, the ring supply and the boss's stats **together**, because
picking them independently is how you end up measuring a combination nothing was measured at:

```
  Preset  [ ★ QUEST, recommended · Floor tank on the two-tile shuttle  ▾ ]
          ★ QUEST, recommended · Floor tank on the two-tile shuttle
            QUEST, Redemption on the rectangle
            QUEST, Hybrid, ring-free (fastest per kill, dies often)
            QUEST, Floor tank on sweets alone, no potions
          ★ POST-QUEST, recommended · Redemption on the rectangle
            POST-QUEST, Floor tank on the shuttle
            POST-QUEST, the wiki floor (6-10 per tick)
            CONTROL, no ring, router free (the before picture), custom (you have changed something), ```

Touch any parameter, the Plan dropdown or the Ring dropdown and it drops to **custom**, with a warning that the
page's numbers were not taken at those settings. The last two rows exist so the two claims that matter most can
be *checked* rather than believed: the control shows the ring's 142→160, and the wiki-floor row shows every
route collapsing to single digits if the icy pool really does deal 6-10.

⚠ **One consequence, recorded because it bit me:** the page now mutates `P` at boot, so any offline test rig that
snapshotted `P` at load was capturing the *preset* instead of the file's declared defaults, which made the Redemption route
read 0/160 in one sweep until I spotted it. The sim now freezes `P_DEFAULTS` before any preset runs, and the
test rig reads that.

---

## ✅ AND YES, THE RING IS A DROPDOWN IN THE SIM NOW

The **Plan** dropdown already carried all three plans (Redemption · Hybrid · Floor-tank). The patrol was a
number field in the parameter table, which is not the same as being selectable, so there is now a second
**Ring** dropdown directly under it:

```
  Plan  [ Redemption stall ▾ ]
  Ring  [ 1, ★ THE RECTANGLE, the four corners of the corridor block (10-13-15-12) ▾ ]
        0, no ring, the router picks every tick
        1, ★ THE RECTANGLE (10-13-15-12)
        2, ★ THE SHUTTLE (13-14)
        3, the near row (10-11-12)
        4, the far row (13-14-15)
        5, the whole corridor block, six stations
        6, ⛔ boss orbit, measured and rejected, kept as the disproof
```

**Changing the plan moves the ring with it** to the pairing that was measured best, Redemption→1,
Floor-tank→2, **Hybrid→0**, and the note underneath says why. Override it freely; if you pick a pairing that
was not the measured one the note leads with **⚠ not the measured pairing** rather than silently letting you
run a worse plan. Picking the hybrid tells you outright that it takes no ring.

**So: there is no hybrid mapping to select, and that is the finding rather than a gap.** Every ring made the
hybrid worse and several produced `blockRisk > 0`.

## ★ THE ROUTINE, one phrase, keyed to the prayer orb, for both variants

**Do not count ticks. Count prayer points.** The whole cycle is anchored to one visible event: the prayer orb
ticking up. It fires every 12 ticks like a metronome and the cycle is 11-12 ticks long, so the loop and the
clock are the same thing.

```
   ┌─ WAIT for the prayer orb to tick to 1
   │     · eat a sweet if you are under 6
   │     · flick Protect from Magic on the tick BEFORE any auto
   │
   ├─ 1  REDEMPTION ON                     (the orb just moved)
   ├─ 2  BELLADONNA                        10 → 5
   ├─ 3  BELLADONNA                         5 → 2
   ├─ 4  BELLADONNA + CRATE RING            2 → 1, SAME TICK, stepping onto the ice
   │        the cake/belladonna deals 0 at 1 hp and queues FIRST
   ├─ 5  ── stalled, hands off ──
   ├─ 6  ── stalled, hands off ──
   ├─ 7  DUMP: the 0-hit fires Redemption, the heal lands, the row of ones lands
   │           and every one of them recoils
   └─ back to WAIT
```

**Four clicks in the phrase, and they are always the same four in the same order.** Two of them can kill you if
missed, click 4 (the arm) and the flick, and those are the two the slip table is about.

### ★ AND NOW IT HAS A SPATIAL HALF AS WELL AS A TEMPORAL ONE

The phrase above says *when*. The rectangle says *where*, and the two lock together because the ring advances
exactly once per boss cycle:

```
  icicle FIRES  → you are in 1-9                    (tile 7 or 9, whichever is under your station)
  ...           → step DOWN one square to your station
  three autos   → you are on the station, seeding / farming it
  icicle FIRES  → step UP one square, back into 1-9
  next cycle    → the NEXT corner of the rectangle
```

**"Down, three autos, up, then the next corner."** The bolt-hole is always the tile directly above the station
(7 above 10 and 13, 9 above 12 and 15), so the shuttle is one motion repeated, never a decision. And the two
long legs of the rectangle, 13→15 and 12→10, are a **single running step each**, because running covers two
tiles and skips the middle, which is why 11 and 14 are never trodden and never iced.

**Three rules that never change and are worth memorising as words rather than ticks:**

1. **"Orb, three cakes, ring."** The whole cycle.
2. **"Never stand on new ice."** A patch that was already there cannot spike; a fresh one hits for 1-8 three
   ticks after the auto that made it. Old ice is safe ground, new ice is the trap.
3. **"Entrance for the special."** Tiles 1-9 on the fire tick, every time, no exceptions, that is the one
   mistake the account cannot recover from.

**★ The one simplification worth making by hand.** The dump leaves you on `1 + heal − instances`, which varies,
so the descent is sometimes two belladonna and sometimes four, and a phrase whose length changes every
repetition is not a phrase. **Top up to full before every descent and it is always exactly three** (10 → 5 → 2
→ 1). ⚠ Stated as a human instruction, not a measured one: I built it as a switch, it stopped Redemption firing
altogether, and I removed it rather than ship a broken control. It costs healing ticks and some doses, of which
you have 184, and given the slip table it is almost certainly the right trade.

**For the post-quest variant the phrase is identical.** The frozen cap means the cycle does not care that the
floor rolls 6-10 instead of 3-5, and the only changes are outside the phrase: two regeneration potions instead
of one, and the step-off and flick thresholds move with her bigger hits.

## ★ THE POST-QUEST FLOOR, SEPARATED PROPERLY, you asked twice and I had merged them

There are **two** ground effects on that boss and they are not the same thing:

| | what the wiki says, verbatim | damage | type | Protect from Magic |
|---|---|---|---|---|
| **ice spikes** | *"Her standard magic attack summons ice spikes on the player's current tile and around the arena."* | **no number given** | **typeless** | protects the **initial attack only**, not the spike |
| **icy pool** (= the floor pattern) | *"Finally, an icy pool will be left on the tile where the player was originally standing on that will deal **6-10 damage per tick**."* | **6-10 per tick** | unstated | not stated as protected |

**So the floor pattern is 6-10 per tick, and the spike's number on that boss is really unpublished.** The sim
still carries the quest variant's 1-8 for the spike, flagged. And the wiki gives no duration for the pool, so
the 104-tick life is carried over from your own quest-variant measurement and is an assumption there.

⚠ **Both of these are wiki figures and you are authoritative over them**, which is why the threshold table below
exists rather than a single answer.

---

## ⛔⛔ A MECHANIC I INVENTED, RETRACTED, AND IT WAS COSTING 2.4×

★ ME: *"First step crossing a live tile doesn't actually hit you. You SKIP tiles when you run two tiles at a
time. You can walk one tile at a time or you can run two tiles at a time, and it skips that floor pattern that
might be between you and the clear tile."*

**I had this backwards and I wrote it into `osrs-bug-principles.md` as a general engine lesson.** It said *"the
mover checks the tile it finishes on, never the ones it crosses, at 1 hitpoint every crossed tile is lethal."*
**Withdrawn from all three files.**

**Only the tile you END THE TICK ON bills you.** The engine was already right about this, it reads position
once, after the move. **The "path guard" I built on the false version was itself the bug:** it refused legal
escapes and stood the character still on the tile that then killed him, which is the death I traced at t189 and
then blamed on the fight.

**Removed, along with the one-step escape rule that rested on the same idea.** Measured on the post-quest floor:

| | recoil | ends |
|---|---|---|
| with my guard | 58 | t272 |
| **without it (correct)** | **142** | **t736** |

**2.4×.** And the general form is worth carrying: **running is strictly better than walking for escaping a
ground hazard**, two tiles of reach and whatever lies between is skipped rather than survived. So the
reachable-safe set is everything within `2 × unstalled ticks`, not everything joined by a clean path.

### What that changes about method B, and it is now a supply answer

B is no longer structurally stuck, it scales cleanly with doses on both bosses:

| | 25 rest slots | 60 | 150 |
|---|---|---|---|
| **post-quest, floor 6-10** | 142 | 247 | **515 · 13/40 kills** |

| | 16 rest slots | 40 | 40 + stepOff 6 |
|---|---|---|---|
| **quest variant, 10 hp** | 185 | 298 | 342 |

**So the short version is inventory, and the inventory does not stretch.** Your stated kit, ~25 four-dose plus ~28
three-dose, 184 doses, is the "25 rests" row: **142 of 520 on the post-quest boss and 185 of 400 on the quest
one.** Reaching 515 needs 150 slots of rests, which does not exist.

**The ten-hitpoint constraint is still what kills method B**, and now for a measured reason rather than a
structural claim: the usable band is 6 to 10 (15 with the overheal), so each trip onto the floor buys one or two
instances and costs a full climb back. **Method A is unaffected by all of this**, 401 recoil, 40/40, t2713, because the frozen cap means it never needs the band at all.

---

## ⛔ TWO THINGS I GOT WRONG, BOTH CAUGHT BY ME

### 1. "An arena where every live tile deals 6-10", WRONG, and it was doing real work in my conclusion

★ ME: *"Won't tiles be freeing up to eat sweets on? Patterns go away per the rules in the engine and are free
to go to every time except the special attack tick."*

**Counted in the log: five tiles live out of 144. The arena is 96% clean.** A patch lasts 104 ticks, one forms
per auto, so at steady state there are only about a dozen and usually fewer. My sentence was technically true
and completely misleading, and it was carrying the weight of the "method B cannot do the post-quest boss"
conclusion.

**What was actually killing the tank route was CORNERING, and it was two bugs in my router, not the fight:**

- It would step onto a live patch inside a cluster, take its roll, drop below the step-off line, and then find
  every neighbour also live. **Fixed:** a farm tile must now have an *exit*, at least one neighbour clean this
  tick and next. With 139 clean tiles that is nearly always satisfiable, and where it is not, that patch is a
  trap.
- The escape search allowed a target up to a full movement tick away (2 squares) but **checked only the
  destination**, so the first step could finish on a live patch. Traced at t189: 5 hitpoints, target two squares
  off, first step onto a tile dealing 10, the path guard correctly refused it, and the character stood still on
  the tile that killed him. **Fixed:** at or below the maximum floor roll the escape must be **one step**, so
  the destination *is* the step.

**Worth +45% on the post-quest tank route**, 40 recoil → 58, t183 → t272. Gated to the tank route: applying it
to the stalled route as well cost 2 recoil for no benefit.

**The conclusion survives, but for the right reason now.** Method B still does not do the post-quest boss, 58 of 520, and it is not because there is nowhere to stand. It is that **at 15 hitpoints one 6-10 roll drops
you to 5-9, and climbing back to 15 costs more ticks than the single instance was worth.** Clean tiles are
plentiful; the hitpoint budget is what runs out.

### 2. "3.4 of 9 instances, worth 2.5× if longer stalls were used", WRONG mechanism, RETRACTED

★ ME: *"we have access to 1 and 3 tick stalls."*

**The stall set is now a parameter rather than a hardcoded list**, defaulting to your kit, the Easter ring (or
a book opened after a move) at 1 and the crate ring at 3. And measuring it retracts my claim outright:

| stall kit | recoil | ends | kills |
|---|---|---|---|
| **1 + 3 (your kit)** | **402** | t2713 | 40/40 |
| + egg book (7) | 402 | t2713 | 40/40 |
| + egg book (7) + guzzle (17) | 402 | t2713 | 40/40 |

**Identical. The planner never takes them.** The arm tally says why: **8,640 of 9,160 arms are 3-tick stalls**,
averaging 1.59 instances, and the 17 is used zero times.

**So the ceiling is not 9, it is about 3-4.** A 3-tick stall holds damage on three ticks, so three floor
instances plus whatever auto it swallows is the structural maximum, and Redemption's heal of 9 is roughly
double what a 3-tick stall can ever spend. **The surplus is unavoidable** (Redemption needs 49 Prayer, which
gives a heal of 12), and it means Prayer level above the minimum buys nothing here at all.

**The real headroom is ~2×, inside the 3-tick stall, not 2.5× across longer ones**, 1.59 of a possible ~3.5.
And what limits it is the tile condition: live for all three ticks and **dead at the dump**. That is a four-tick
window on a 104-tick patch, which is exactly the rotation, now much smaller than I had been describing it.

### 3. Rock cake at 1 hitpoint, confirmed, and it is what the whole cycle rides on

★ ME: *"rock cake doesn't kill at 1 hp, it hits 0, just confirming."* **Yes, and that is exactly how it is
modelled:** `selfAmt = max(0, min(raw, hp − 1))`, so at 1 hitpoint it deals **0** and cannot kill. That
zero-damage hit is the first thing in the queue and it is what fires Redemption out of the stall. Belladonna
behaves the same way at 1 and is preferred only because it is stackable and is not interrupted by an incoming
projectile.

---

## ✅ POISON REMOVED FROM THE MODEL ENTIRELY

★ ME: *"for both of these methods we don't need poison. We're not going to be taking any poison damage,
we're not going to be using any cures, these are irrelevant with prayer, and the poison timer is just going to
randomly complicate things inside the processes. Remove that entirely out of the factors."*

**Done, and you were right that defaulting it to zero was not the same thing.** It was already inert at
`curePool 0`, but an inert 30-tick metronome is still a metronome, it sat in three places where a single stray
non-zero would have silently changed the shape of every cycle:

- the stall selector's **must-cover event list**, where a poison tick counts as an event to size a stall around;
- the **next-event sums** that decides how much slack a cycle has;
- the **capacity projection**, which had a whole branch for "will the cure still fire at the dump".

**What was removed:** the poison metronome and its re-phasing · the poison entry in the event list · the cure at
the dump and its projection · `cureWillFire` and the cure branch of the capacity model · `st.poison` and
`st.nextPoison` from state, frames, the death record and the HUD · and the six parameters
`curePool` `cureCost` `cureRecoil` `cureHeal` `poisonAnchor` `poisonTick`.

**And six level 3 render panels went with them**, because every one read those parameters: the antelope gap
table, the looting-bag logistics chain, the poison-anchor lock, the food/cure/null drill sheets and the
319-point ceiling. **The ceiling panel is rebuilt for this model instead:**

> ceiling = prayer points carried × min(⌊Prayer ÷ 4⌋, maxHp − 1)
> = **264 × 9 = 2,376** against a 400-hitpoint boss.

Which states the position in one line: **supply is not the binder and never was.** The binder is how many of
those instances a stall can collect before it has to dump, the instances-per-arm figure, currently 3.4 of 9.

**Both routes are unchanged by the removal:** A **402 recoil, 40/40, t2713**; B 185, 0/40. `osrs-amox-sim.html`
still carries the poison model, correctly, because the combat-3 plan still uses it.

---

## ★ THE RECOIL STEP, you caught a real gap in my engine

★ ME: *"if the boss somehow does manage to hit over ten in a singular hitsplat, the recoil sends TWO damage
back onto the boss instead of one."*

**`core-game-mechanics.md` §15 has the rule and `RECOIL()` was ignoring it**, it took no damage argument and
returned a flat 1 on every call. It went unnoticed because on the combat-3 account the step is unreachable by
construction: you can never take a 10 and live, so 1 is always right there. **It is not right here.** Fixed:
`floor(dmg/10) + 1`, and the ring pays a charge per point reflected, so a 2 costs two charges and the refresh
rhythm moves with it.

**Method A's control did not move, 402 recoil, 40/40, identical.** That is not luck, it is the point below.

### ⚠ AND THIS IS THE THING TO TAKE AWAY: THE FROZEN CAP CAPS THE RECOIL TOO

The frozen cap caps the **hitsplat**, so it also caps what the ring sees. **Inside a stall armed at 1 hitpoint
every splat is a 1 and every recoil is a 1, you can never earn the 2-damage step there, no matter how hard she
hits.** The 2 is only reachable *outside* a stall, or with a cap frozen at 10 or more, which a ten-hitpoint
account cannot do.

**So the two methods sit on opposite sides of the step.** A maximises the *number* of instances at 1 each; B is
the only one that can ever collect 2s. That is a real trade rather than a preference, and it is why B gets
better as the boss hits harder while A does not care.

## ★ ARMING AT 2 INSTEAD OF 1, it works, and it costs you 44%

★ ME: *"going from two to one HP if I somehow regenerate an HP, with a one proccing instead of a zero, and
lining up two two two two if the stall is short enough to survive a redemption heal."*

**Correct on the mechanic.** The cake takes 2 → 1, that hit fires Redemption, and the cap is frozen at 2 so the
queue lands as 2s. And your instinct about the stall length is exactly right, capacity halves, so the stall
must be about half as long. Here is the sums at 10 hitpoints with a heal of 9:

| arm at | frozen cap | each instance costs | splat | recoil each | instances per proc | **recoil per proc** |
|---|---|---|---|---|---|---|
| **1 hp** | 1 | 1 hp | 1 | 1 | **9** | **9** |
| 2 hp | 2 | 2 hp | 2 | 1 | 5 | 5 |

**A 2-damage splat still recoils only 1**, 10%+1 rounds down and anything under 10 returns 1, so the 2s buy
you nothing back. Measured across a whole run: arm-at-1 **402 recoil, 40/40**; arm-at-2 **62 recoil, 0/40**.

**Treat it as a contingency, not an option.** If a natural regen strands you at 2 and there is time to
belladonna back to 1 before the arm, do that. Arm at 2 only when the alternative is a wasted cycle, it is
worth 5 recoil instead of 9, not nothing.

## ⛔ "DOES METHOD B STILL WORK IF THE FLOOR SCALES OVER TEN?", no, and more rests do not fix it

Your proposal: stay above 10 hitpoints via the Guthix-rest overheal so the floor cannot kill you, keep Protect
from Magic up, and recoil off every floor tick. **520 hitpoints, floor 6-10, base 10 with the overheal ceiling
at 15, step off below 11, 40 seeds:**

| Guthix rest slots | recoil | ends | kills |
|---|---|---|---|
| 25 | 40 | t181 | 0/40 |
| 60 | 40 | t181 | 0/40 |
| **200** | **40** | **t181** | **0/40** |

**Identical at every dose count, so it is not a supply problem and rests cannot buy it.** The reason is the
ceiling, not the volume: the overheal takes you to 15, **one** floor tick of 6-10 puts you on 5-9, and you are
now below the line in an arena where every live tile deals 6-10. Getting back to 15 means climbing to full and
then overhealing again, two rests, several ticks, for **one** floor instance worth 1 recoil (2 only on a
maximum roll). And every one of those healing ticks is spent beside an arena where one misstep is death.

**The overheal buys one tick on the floor, and you need hundreds.**

**★ Which makes the frozen cap not an optimisation on that boss but the only mechanism that works**, it turns
a 6-10 roll into a 1 regardless of how hard she hits. Method B has no answer to a floor that exceeds your
maximum hitpoints; method A does not care what the floor rolls. That is the cleanest statement of why A is the
one to take forward, and it is the opposite of the answer at 20 hitpoints on the quest variant.

---

## ★ THE POST-QUEST AMOXLIATL, it is a THRESHOLD, so the wiki does not have to be right

★ ME: *"real boss numbers are the non-quest variant only, and not sure if it mentions floor damage numbers.
Wiki sometimes wrong."*

**It does mention it, and here it is verbatim**, `Amoxliatl/Strategies`, section *Fight overview*:
*"Finally, an icy pool will be left on the tile where the player was originally standing on that will deal
**6-10 damage per tick**."* **But you are authoritative over the wiki, so it is swept rather than trusted.**

**520 hitpoints · two 4-dose regeneration potions · method A · 40 seeds:**

| floor damage | recoil | kills | ends |
|---|---|---|---|
| **3-5** (the quest variant's measured value) | **520** | **40/40** | t3517 |
| **3-6** | **520** | **40/40** | t4093 |
| 3-7 | 15 | 0/40 | t224 |
| 3-8 | 14 | 0/40 | t224 |
| 3-9 | 8 | 0/40 | t256 |
| **6-10** (the wiki figure) | 1 | 0/40 | t43 |

**The cliff is between a maximum roll of 6 and a maximum of 7, and it is sharp.** So the thing to measure in
game is the floor's **maximum**, once, and read the answer off this table. If it is 6 or under the fight is
already solved; if it is 7 or over this engine cannot route it.

**★ AND 520 HITPOINTS IS NOT THE PROBLEM, that was worth checking and it came back clean.** At the quest
variant's floor damage the post-quest boss dies to **two** 4-dose potions rather than one: **520 recoil, 40/40,
t3517.** One potion runs out at 475 of 520, a supply failure, not a death. **Two inventory slots, not a wall.**
The 22/34 max hits change nothing either: the frozen cap turns any roll into a 1, and Protect from Magic is
flicked for the ones that land outside a stall.

**What I could not do, stated plainly.** Above a floor maximum of 6 the cycle collapses and I do not know
whether that is the fight or my planner. Two pieces of evidence, both pointing at the planner:

- **Raising maximum hitpoints does not rescue it**, 15 and 20 both score 15 recoil, 0/40, identical to 10.
  If it were survivability, hitpoints would help.
- The binding rule is my own: the dump may only land above the maximum floor roll, or on exactly 1 with a
  prayer point in hand and a dead tile underneath. Once the floor maxes at 7, the first branch needs
  `postHp ≥ 8` and the heal only delivers 9, so almost nothing fits.
- **⛔ Relaxing that rule was measured and REJECTED**, allowing any dump that merely survives, on the grounds
  that sweets and rests can climb out of the dead zone afterwards, costs the quest control **402 → 55 recoil**
  and 40/40 → 0/40. The rule is structural, not conservative.

**The thing to pull that is left is the rotation**, `arm at ice age 105 − L`, so the patch expires exactly at the dump
and the dump tile is dead. On the quest variant that is an optimisation worth t2713 → ~t900. **At a floor
maximum of 7 or more it is the only thing that can work**, because a dead dump tile is the only branch the rule
leaves open. Still not built, for the same reason as before: build it after the floor is measured, not before.

**One thing still unmeasured on that boss, it used to be two.**

1. ~~**Does staying in tiles 1-9 on the fire tick prevent the unstable ice blocks?**~~
   **✅ ANSWERED, 21 August, confirmed in game: standing in the 3×3 entrance never triggers the block.**
   This was the single most valuable unmeasured fact on the boss and it was written down here at ~50/50. The
   losing branch closed the post-quest fight completely: 1-4 blocks per special healing her 16-25 each, melee-only
   to destroy, and a zero-XP account can neither destroy them nor walk away from them, she would simply
   out-heal recoil. It went the right way. Consequence for the sim: **`blockRisk` is now a structural
   invalidity check rather than a hedge.** A run that reads non-zero left the entrance on a fire tick, and it is
   thrown out rather than scored lower.
2. **The spike's damage.** *"Spikes formed afterwards deal typeless damage"*, typeless cannot be protected, and
   the sim is still carrying the quest variant's 1-8.

---

## ⚠ A BUG I FOUND IN MY OWN ENGINE, AND IT IS IN `osrs-amox-sim.html` TOO

`icePatMin` has existed as a parameter since the beginning, is documented ★ measured, *"3, not 1, capped by
current hitpoints"*, is printed in the reference table as *"3-5 per tick standing"*, and **was never read**.
The roll was `ri(1, icePatMax)`. **Both sims have been billing a 1-5 floor against a measured 3-5 one, every
tick.**

It survived this long because **inside a stall it changes nothing**, the frozen cap is 1 either way, and a
stall plan spends almost no unstalled ticks on live ice. Measured impact here: method A **401 → 402 recoil**,
same kill, same tick. Method B (unstalled) **229 → 183**.

**Fixed in this file. The parent is left untouched**, fixing it there would move the level 3 acceptance sweep,
and that is your call, not mine. Given the above the move should be small, and in the safe direction: the level
3 plan has been modelled against a slightly *softer* floor than you measured.

---

# ★★★ TWO ENGINE BUGS FOUND WHILE BUILDING THE PATROL, AND BOTH WERE COSTING KILLS

### 1. The tile you END a movement tick on was never checked

★ I, and this is the rule the engine was built on: *"you SKIP tiles when you run two at a time, the first
step crossing a live tile doesn't actually hit you."* True, and the engine was right about it. What it was
**not** right about is that a walk longer than two squares does not arrive this tick, **it stops somewhere**,
and that somewhere was whatever the greedy step happened to finish on.

Every *destination* in the file is carefully ranked: the reservation picks the least dangerous entrance tile,
the step-off filters on `tileThreat`, the survival override reads the tile under your feet. Nothing read the
**intermediate landing tile**.

> **Traced, seed 2, t63.** The dump left 5 hitpoints. The reservation for the special at t65 fired from tile 11.
> The greedy walk toward tile 1 stopped on tile 4. Tile 4 was carrying a 47-tick-old patch. The floor rolled 5.
> Dead.

Fixed by enumerating every tile a running tick can *end* on and taking the closest one to the target that is
survivable. **Measured, 160 seeds: the floor-tank route 120/160 → 130/160, hybrid 92/160 → 100/160, the Redemption route unchanged.**

★ **And the reservation is exempt**, exactly as the survival override is, your constraint outranks dying. On a
reserving tick the ranking flips to closest-first, and the walk into tiles 1-9 will step on a live patch if that
is what it takes. Without that exemption the fix takes the floor-tank route to 40/40 and `blockRisk` **0 → 4**, which is not
a better run, it is an invalid one.

### 2. The arm fired even when the arm-tick step could not reach the planned tile

The capacity planner picks a tile **and** a length together, and the whole ice-block guarantee rests on that
tile: when a special cycles inside the lock, the plan is forced into tiles 1-9. But the arm-tick step draws from
the **same movement allowance the router already spent this tick**, so on a tick the router already ran two
squares, the step moves **zero** and the stall goes up wherever the character is standing.

> **Traced, hybrid, seed 120, t667.** Router forced off two squares to tile 10. Plan said *"7t stall on tile 7"*
>, tile 7 is inside the entrance and the icicle cycles at t673, mid-lock. The step logged `tile 10 → 10`. The
> stall armed anyway. Feet locked outside 1-9 through the special. **One ice block.**

⛔ **First fix was too broad and was reverted:** refusing *every* arm that cannot reach its planned tile is
correct-sounding and cost the Redemption route **160/160 → 112/160**, because most unreachable plans are merely *worse*, not
*invalid*, the selector would rather arm on the wrong tile than not arm at all. Narrowed to the one case where
the tile is a constraint rather than a preference: a special cycling inside the lock with the feet outside the
entrance. **`blockRisk` back to 0 with no cost.**

### 3. And a third, same family: the stall selector's span stopped at the dump

`mustBeSafe` asked whether a special cycles while the feet are **locked**, `[arm … dump]`. That is not the whole
exposure, the dump is an initiation tick, so the first step out resolves at `dump+1`, and a tile three squares
from the entrance needs **two** movement ticks to get home.

> **Traced, hybrid, seed 131, t961.** A 3-tick stall armed at t957 on tile 17, dumped at t960, icicle fired at
> t961. No special inside the lock, so the selector allowed it, and the walk home could only reach tile 10.

Fixed with a multi-source BFS from the entrance: a tile outside 1-9 is legal only if no special fires before you
could physically be back inside it.


## Do not build

- **The rotation scheduler**, even though the post-quest boss needs it. Build it only after the block question
  above comes back, because if she heals unconditionally the whole thing is moot.
- **A quoted potion count as a plan.** One 4-dose covers the quest kill with 38 points spare. Carry two.

## Files

| File | What |
|---|---|
| `osrs-prayer-pure-sim.html` | **`PRAYERPURE-V2`**, 10 hitpoints, both plans on the Route dropdown |
| `osrs-protmagic-floor-sim.html` | stub, superseded by the merge |
| `osrs-amox-sim.html` | the combat-3 sim, untouched; its acceptance sweep still reproduces |

## Sources

- [Amoxliatl/Strategies, OSRS Wiki](https://oldschool.runescape.wiki/w/Amoxliatl/Strategies)
- [Amoxliatl, OSRS Wiki](https://oldschool.runescape.wiki/w/Amoxliatl)
- [Prayer regeneration potion, OSRS Wiki](https://oldschool.runescape.wiki/w/Prayer_regeneration_potion)
