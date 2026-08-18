# AMOXLIATL — START HERE

> ## ✅ sim is **v15**, marker `CUREATDUMP-V15` (`FORCEDOFF-V14`, `DUMPSEED-V13` also present)
>
> **At stock 23 antelope, across this session: 75 → 114 recoil, run t203 → t559.**
> cure 4 → 32 · **phoenix necklaces spent 0 → 22 of 22** · food unchanged at 77.
>
> ## ✅ THE PLATEAU IS FIXED — v13, marker `DUMPSEED-V13`
>
> Everything below the line "## The job this session" is the **v12 state** and is kept
> because the method in it is what found the bug. **The v12 diagnosis of the t723 death
> was wrong**; the corrected one is here. Read this box first, then the "v13" section
> further down for what actually changed.
>
> **The acceptance test now passes.** Recoil rises with every extra antelope until she dies:
>
> | carried | 23 | 40 | 60 | 80 | 100 | 110 | 115 | **120** |
> |---|---|---|---|---|---|---|---|---|
> | recoil | 81 | 156 | 207 | 264 | 338 | 377 | 394 | **402 — KILL** |
>
> 40/40 seeds win at 120 carried (117 actually spent), ends t945. `blockRisk` 0, `spikeLeak` 0.
> Below 120 every run now spends **every** antelope it carries — the runs are food-limited,
> not death-limited. That is the whole plateau, gone.

---

## The job this session: FIX THE SIM. Nothing else.

`osrs-amox-sim.html` is on **v12**. The mechanics in it are the best-measured they have ever been, but the
engine is **not converting them** — it dies for reasons that are modelling gaps rather than fight mechanics,
and it plateaus. Do not add features, do not chase the fourth coverage source, do not re-sweep parameters for
gain. **Diagnose the deaths.**

**Engine right now:** 75 / 400 at the stock 23 antelope, ends t203.
**And the tell that something is wrong:** carrying more food stops helping.

| antelope carried | actually SPENT | recoil | ends |
|---|---|---|---|
| 23 | 23 | 75 | t203 |
| 60 | 60 | 194 | t525 |
| 100 | **89** | 283 | t723 |
| 140 | **89** | 283 | t723 |
| 180 | **89** | 283 | t723 |
| 220 | **89** | 283 | t723 |

★ Rendi, correctly: *"this makes 0 sense to me, more delayed food should be more recoil affect, I don't see why
or how that's faltering."* **He is right, and the plateau is not a finding — it is a bug.** The run is not
running out of food; it is **dying at food #89 and never spending the rest**. Every plateau row is the same
death.

---

## The one thing to understand before touching anything

**The arena has TWO damage sources from the floor, not one, and they have different caps and different
timings.** Nearly every bug found so far is some check seeing one and not the other.

| | fires at | rolls | condition |
|---|---|---|---|
| **ICE SPIKE** | **auto + 3** | **1 – 8** | **only if the tile had NO floor pattern when the auto LANDED** |
| **FLOOR PATTERN** | auto + 7, then every tick | 1 – 5 | until auto + 104 |

★ Rendi: *"the ice spike doesn't happen if already standing on floor pattern during the standard auto attack."*
**Note which tick that check is on — the AUTO tick, not the spike tick.** It asks what was under you when the
auto landed, which is why the code tests `_live(c.formed)` and not `_live(now)`.

It is enforced **twice**, deliberately: `spikeOnFresh` does it directly, and the no-restack rule does it
upstream by preventing a second patch from ever forming on a patterned tile — so in practice there is never an
age-3 patch there to spike from. **Keep both.** If one is removed the other still holds. There is a
`spikeLeak` counter that trips if a spike ever bills on an already-patterned tile; audited at **0** across 20
runs, and it must stay there.

**Consequence worth carrying: standing on MATURE ice pays three ways** — it pays recoil, it does the descent
for you, and it suppresses the spike. A fresh tile does none of those and hits for up to 8.

A tile carrying a patch of **age 3** is about to hit you for up to **8**, and every pattern-only check reads it
as perfectly safe, because age 3 is below the pattern threshold of 7. **That single blind spot has now produced
two separate structural deaths.** When you write any survivability check, use the worst case for the tick —
there is a helper in the engine, `tileThreat(tile, tick)`, that returns it. Use it. Do not re-derive from
`iceAt()`, which is pattern-only and is what caused both bugs.

---

## How to find the next death — the method that worked; repeat it

The whole technique, about a minute per death:

1. Run at high food so the plateau shows: `E.P.meatSlots.v = 140`.
2. Across several seeds, print the death tick, hitpoints, tile, **the AGES of the ice patches on that tile**,
   and the stall state.
3. Read the last six ticks of the log.
4. The age tells you which source killed you. **Age 3 → spike (up to 8). Age 7–104 → pattern (up to 5).**
   Age below 3 or above 104 → neither; look elsewhere.
5. Find the check that should have prevented it and ask **which of the two sources it can see.**

**Worked example — the death that is still open.** Every seed at 140 food dies identically at **t723**:

```
t718  hp 2  stall 5
t719  hp 2  stall 4   floor 5 → 1 DEFERRED
t720  hp 2  stall 3   AUTO 8 → 1 DEFERRED ;; floor 2 → 1 DEFERRED
t721  hp 2  stall 2   floor 1 → 1 DEFERRED
t722  hp 2  stall 1   floor 3 → 1 DEFERRED
t723  hp 0  stall 0   delayed portion +8 — resolves AHEAD of the dump
                      stall ended — 8 landed from 8 queued (cap was frozen at 1)
                      DEATH — 2 landed against 2 hitpoints on tile 1
```

Death tile 1, ice age **11**, stall 0. Read it: the stall was armed at **hp 2, not 1**, so the delayed portion
healed only **+8** (capped at 10) instead of 9. Eight instances landed, leaving **2**. Then the dump tick's own
floor roll took the last 2.

Two candidate causes; establish which before fixing anything:

- **The planner assumed hp 1 at the arm.** The dump survivability check computes
  `postHp = 1 + capHere - instances`. If the arm actually happens at hp 2, the real heal is capped and that sum
  is wrong. **Why was it on 2 and not 1?** That is the first question — find what left it there. The arm gate
  is `st.hp <= 1`, so something healed between the descent and the arm, or the gate is being bypassed.
- **The auto at t720 seeded ice on the locked tile**, which is age 3 at t723 — a spike on the dump tick. The
  planner's dump check was upgraded to use `tileThreat` for exactly this case and **the numbers did not move**,
  so either that code path is not being reached or the real cause is the hp-2 arm above. **Verify the upgraded
  check is actually being hit** before assuming it works.

---

## v14 — THE CURE PHASE DEATH, AND THE 7-TICK-STALL QUESTION

★ Rendi, from his own run at stock 23:

```
t194 stall 7t armed, cap frozen at 1 hp (cure)
t197 ICICLE 6 → 1 DEFERRED       t199 poison 3 DEFERRED  · severity 12
t200 AUTO 3 → 1 DEFERRED         t201 CURE +5 vs 3 queued · 17 doses
```
…and dead two ticks later. Reproduced exactly, seed 1.

**It is not the dump tick.** The dump at t201 is fine: `1 + 5 − 3 = 3` hp, tile clean,
nothing lands. What kills him is what happens **after**:

| tick | | |
|---|---|---|
| t201 | dump, 3 hp, cure-entry nightshade halves 3 → **1** | tile 5, ice **age 1** |
| t202 | 1 hp, sits still | ice **age 2** |
| t203 | 1 hp, sits still | ice **age 3 → SPIKE** → dead |

**Nothing in the engine ever asked "can this tile kill me where I stand?"** Every tile
decision upstream is made for *recoil* — which tile pays the most instances. There was no
rule of the form *step off because you will die here*. He never moved because nothing told
him to.

And the reason no existing rule caught it: the router's step-off branch is gated on
`onLive`, which is `iceAt(pos) >= icePatAt` — **pattern-only**. At age 3 that reads
**false**. This is the **third** death from that one blind spot, and the second one *after*
the file already said in bold not to use `iceAt`. It is still there because it is not
spelled `iceAt` at the call site; it is spelled `onLive`.

**The fix — the survival override.** Unstalled, before any recoil consideration:

```js
if(st.stall===0 && want!=='reserve' && tileThreat(st.pos,st.tick) >= st.hp) want='off';
```

Stalled feet are exempt (they cannot move); `reserve` is exempt (the ice-block rule is the
one thing that outranks dying). **81 → 108 at stock 23, and nothing lost at any other food
count.** `blockRisk` 0, `spikeLeak` 0, `lostRecoil` 0 across 420 runs.

**MEASURED AND REVERTED — do not retry.** Making the planner's `stageOk()` use `tileThreat`
instead of `dmgAt` also fixes the t203 death (23:105) but costs 3–5 recoil at *every* other
food count. The override is strictly better and subsumes it. Also folded in and measured
**neutral**, kept only because it closes the same hole the food descent already had: a
`tileThreat` guard on the cure-phase descent.

### ★ "Is there no way to work in 7-tick stalls for more damage?" — yes, and the cure phase already does

The egg book is **not** being rejected by the attack rhythm. Event gaps repeat **8, 8, 13, 3**
(32-tick block), and a 7t stall armed at `E−4 … E−6` satisfies the `OVERHEAD+1` rule fine.
Measured stall usage in v14:

| phase | cap | what it arms | instances |
|---|---|---|---|
| **cure** | 5 | **L7 ×90, L17 ×19** | **3.1 / 4.0 of 5** |
| **food** | 9 | **L3 ×170** | **3.3 of 9** |

**The cure phase uses the egg book constantly. The food phase never does.** The reason is one
line of arithmetic — the dump gate:

> `postHp = 1 + cap − instances`, and it must survive the dump tile's roll: `postHp − thr ≥ 1`.

| phase | cap | on a tile still LIVE at the dump | on a tile DEAD at the dump |
|---|---|---|---|
| food | 9 | `inst ≤ 9−5 =` **4** (vs a spike, `9−8 =` **1**) | up to **9** |
| cure | 5 | `inst ≤ 5−5 =` **0** | up to **5** |

**The cure phase is forced onto dead-at-dump tiles — zero instances are affordable otherwise —
and that is exactly why it reaches for long stalls and gets 62–80% of its ceiling.** The food
phase has 9 points of capacity, so a mediocre live-tile 3-tick stall at 3.3 instances is
"good enough" and always available; a 7t only beats it on a tile whose patch **expires at the
dump**, and those are incidental rather than scheduled.

Measured directly (stock 23, seed 1): best instances available per food arm-decision was
**7.29 ignoring the dump gate**, but only **2.96 with it**. That ~4.3-instance-per-cycle gap
**is** the food-phase shortfall, and it **is** the kiting rotation — arm at ice age `105 − L`.
So: 7-tick stalls are the right instinct, the constraint is not stall length, it is
**dump-tile age**, and the rotation is what buys it.

---

## v15 — WHY THE NULL PHASE DIED WITH ALL 22 NECKLACES IN THE BAG

★ Rendi: *"phoenix necklace phase shouldn't be dying before neck consumption."* Correct, and
it was not a null-phase bug at all — **the cure phase killed it on the way in.**

```
t299  cure stall up, poison tick fires: severity 1 -> 0
t300  phase flips to NULL — mid-stall, with the cure's capacity still booked
t302  DUMP. cure is gated on doses>0 && poison>0. Poison is 0. It does not fire.
      4 queued instances vs 1 hitpoint. Dead, 2 ticks into the null phase,
      22 necklaces unspent.
```

`cureWillFire` is computed **once, before the stall search**, from `st.doses` / `st.poison`
**as they are now**. The poison pool drains on its own 30-tick metronome, so a poison tick
landing *inside* the stall empties it between the arm and the dump. **Same class as the v13
bug: the check reads the present when the thing it guards happens in the future.** At most one
poison tick can land inside a stall (it re-phases to stall-end + `poisonTick`), so the
projection is exact:

```js
if(st.phase==='cure'){
  const poisIn = (st.nextPoison>=lo && st.nextPoison<=dump) ? 1 : 0;
  if(!(st.doses>0 && (st.poison - poisIn) > 0)) return null;   // capacity is zero, arm nothing
}
```

**108 → 114, run t302 → t559, necklaces spent 0 → 22 of 22.** Nothing lost elsewhere.

### ★ CORRECTION FOLDED IN — the null ceiling of 22 stands, the SOURCE was wrong

★ Rendi, authoritative: *"No suffering recoil doesn't reflect self damage. But the tiles we
walk between the p-neck procs will if standing on active ice — which we before estimated
1 damage per attack on avg."*

So both halves are settled: the engine is **right** that the nest's self-hit recoils nothing
(`if(d.src!=='self') RECOIL(st)`), and the **22 ceiling is right** — it is paid by the **floor
ticks between procs**, not by the self-hit. My previous note zeroing the NULL row is
**withdrawn**; the row is restored at 22 with the correct attribution.

### …and the engine collects 2 of that 22. Here is exactly why, and it needs your call

Measured across 15 runs, 2529 null-phase ticks:

| hp | share of null-phase ticks |
|---|---|
| 3 | 3.2% |
| 4 | 8.3% |
| **5** | **79.8%** |
| 6+ | 7.5% |

**The phase parks at exactly 5 hp for four ticks in five, and bills ZERO floor ticks.** The
reason is a straight collision between two thresholds:

- **the nest needs hp ≤ 5** — `nightOk` requires `floor(hp/2) <= band`, and band is 2, so a
  nightshade nest is only legal at 5 or below (the cake only at exactly 3);
- **standing on ice needs hp ≥ 6** — `icePatMax` is 5, so at 5 hp a max floor roll is death.

There is no hitpoint value that satisfies both. The phase picks the nest, parks at 5, and the
floor is never billed. Everything I tried to bridge it **measured negative** (stock 23,
v15 = 114):

| attempt | total | null recoil | necklaces spent | run ends |
|---|---|---|---|---|
| **v15 as shipped** | **114** | 2 | **22 / 22** | **t559** |
| widen the nest window to 6 so a proc lands on the farm band | 112 | 3 | 1 | t337 |
| route onto ice in null above `icePatMax` | 114 | 2 | 22 | t559 |
| `nullFarm` ON, slack 3–6 / 8–13 | 112 / 113–114 | 3 / 2 | 1 / 22 | t337 / t557 |
| **stand on live ice at hp ≥ 5 and take the risk** | 113 | **4** | **2** | t337 |
| stand on live ice at hp ≥ 4 / ≥ 3 | 113 | 4 | 1 / 0 | t329 / t317 |

**Standing on ice at 5 does double the null recoil — and it costs 20 necklaces and 220 ticks,
because a 5-roll is death and the null phase has no heal to climb back with.**

> ⚠ **THE QUESTION, and it is a mechanics one so it is yours:** between the procs, **what
> keeps you above 5 hitpoints?** Your ~1-per-attack estimate requires standing on live ice,
> and at 5 hp the sim says a max roll kills you before you can bank twenty of them. Either
> (a) you sit at 6+ and something pays for the climb that is not the sweet — the sweet route
> is measured negative twice now; or (b) you accept 5 hp because the phoenix covers rolls of
> 2–4 and only a straight 5 kills; or (c) the nest fires from higher than 5 by some route the
> `nightOk` model does not have. **Tell me which and the 22 is reachable — the engine already
> bills the floor from your feet everywhere else, it is only the hp band that is blocking it.**

---

## ★ "WHY IS ANTELOPE DOING SO MUCH LESS DAMAGE ON THE NEWER VERSIONS?"

**First: nothing this session did it.** Food-phase recoil at stock 23, same seeds:

| | v12 | v13 | v14 |
|---|---|---|---|
| **food** | 71 | 77 | **77** |
| cure | 4 | 4 | **30** |
| total | 75 | 81 | **108** |

The drop is older than this session, and the honest answer is uncomfortable:

> **The 201 was never a verified number.** `amoxliatl-level3.md` line 26 flags it itself:
> *"⚠ The verification gate still does not pass as written. The rebuilt food phase reads
> **201** against the hand-walked **153** — over, not under."*

201 was an engine **over-reporting against Rendi's own hand-walk**, and it was recorded as
failing its own gate at the time. It was counting instances the run would not have survived
to collect. Every survivability rule added since has removed some of those phantom instances.

**I tried to reproduce 201 by restoring the withdrawn mechanics one at a time. None of them
bring it back** (23 antelope, 40 seeds, food-phase recoil):

| engine | food |
|---|---|
| v14, all corrections in | **77** |
| minus the ice-block hard constraint (XP-unsafe, diagnostic only) | 77 |
| minus the dump-tick initiation rule (feet free on the dump) | 77 |
| minus the corrected ice lifetime (back to `iceLife` 30) | 85 |
| patterns allowed to RESTACK / refresh on every auto | 79 |
| **all four withdrawn at once** | **84** |

So the loss is not attributable to any one mechanics correction, and it is not recoverable by
undoing them. **The 201 does not exist in any configuration I can build.**

### What the number should be compared against instead

**Rendi's hand-walk: 153.** That is the honest target, and it is 6.65 instances per antelope.

| | per antelope | at 23 |
|---|---|---|
| ceiling | 9.00 | 207 |
| **★ Rendi, hand-walked** | **6.65** | **153** |
| **v14 engine** | **3.35** | **77** |
| v9 engine's unverified claim | 8.74 | 201 |

**The engine gets about half of what Rendi gets by hand.** That is the real gap, it is a
factor of two rather than a factor of three, and its mechanism is now known exactly:

- On a tile still **live at the dump**, capacity allows `9 − 5 =` **4 instances** (1 against a
  spike). On a tile **dead at the dump**, up to **9**.
- Long stalls in the food phase are additionally rejected for **overflow**: measured, L17 is
  refused `tile_OVERFLOW_cap` **953 times** in one run — a 16-tick window on a live tile bills
  far more than the 9 capacity — plus the dump gate (444) and `nextBareTooSoon` (364).
- So the food phase settles on **L3 at 3.3 of 9**, forever, because that is the best thing
  incidental ice ages ever offer it.

**Every road leads back to the same place: the expiry rotation.** Arm at ice age `105 − L` and
the dump tile is dead, the overflow disappears (the patch melts mid-stall), and both the 4-cap
and the L17 overflow rejection stop binding at once.

---

## ★ "WHY AREN'T THE PHOENIX NECKLACES ALL BEING USED, AND ONLY RETURNING 1 DAMAGE?"

Two separate answers, and only the first is a problem.

### They ARE used — when the run lives long enough to reach the null phase

| carried | necklaces used (median / best seed) | seeds using any | dies in |
|---|---|---|---|
| 23 | **0 / 22** | 3/40 | null (39/40) — reaches it and dies immediately |
| 40 | 0 / 22 | 8/40 | null 32, rng 8 |
| 60 | **0 / 0** | 0/40 | **cure — never reaches null at all** |
| 80 | 6 / 22 | 40/40 | null |
| 100 | 9 / 22 | 36/40 | null |

At stock 23 the run now dies **just as the null phase opens** (t302). The necklaces are not
being skipped — the phase they belong to barely starts. **Necklace usage is a survival
symptom, not a necklace problem.** Fix the cure and food phases and they get spent.

### The "1 damage" is by design, and the ceiling table is WRONG about it

Three rules, all deliberate, that together make the null phase worth ~0 recoil:

1. **The nest's own self-hit does not recoil.** Engine: `if(d.src!=='self') RECOIL(st)`.
   Rock cake and nightshade are self-inflicted; the ring does not fire on them.
2. **★ Rendi confirmed the CCQ nulls the floor instances too** (`nullFarmNote`), so everything
   queued behind the self-hit recoils nothing either.
3. **`nullFarm` is set to 0 on measurement** — farming ON gives 197 recoil / 6 of 22 necklaces
   / ends t412; OFF gives 195 / 22 of 22 / ends t565. The climb does not pay.

**Therefore the `NULL | 22 | 1 per necklace` row in the §7 ceiling table is stale.** It assumed
the nest's self-hit recoils 1. The code says it does not, and standard ring behaviour agrees.
**The null phase is a survival phase worth ~0 recoil, not a 22-recoil phase.**

> ✅ **ANSWERED BY RENDI, folded in:** the ring does **not** reflect self damage — the engine
> was already right. The 22 comes from the floor ticks walked between procs. The ceiling
> table below is restored to 319 / 301.

---

## ⛔⛔ THE SHIP DRINKS ARE DEAD — ★ Rendi called it, and the reason is NOT the boat gate

**Read this box and skip the section below; it is kept only for the venom arithmetic, which
survives and is general.** Source: the `Sealed crate` page itself, read in full through Chrome.

**Two lines from the wiki settle it:**

> *"In order to sample these bottles, players must be **anywhere aboard a boat, including in the
> Shipyard**, and cannot be navigating the boat."*
>
> *"**Most bottles can only be sampled once**, and doing so grants the player 75 Sailing experience.
> A select number of drinks can be resampled, but will not award XP on subsequent sips."*

**I was wrong at ~75% that they were drinkable anywhere. They are boat-only. But that is the SMALL
problem.** The table has a **Repeatable** column, and every poison source in it reads **No**:

| drink | Sailing | poison effect | **Repeatable** | player safe |
|---|---|---|---|---|
| **Bottle of zul-rye beer** | 40 | Envenoms the player | **NO** | No |
| Bottle of zogre's sloppy kisses | 40 | poison + disease | **NO** | No |
| Bottle of spinner's last gasp | 12 | Poisons you for 4 minutes | **NO** | No |
| Bottle of slippery snake skins in possibly gravy | 12 | spawns a level-90 snakeling, 1 hp, **envenoms you instantly** | **NO** | No |

**Only 5 of the 65 crates are repeatable at all, and not one of them is a poison source.**

### Why that kills it even if the boat gate were beaten

**The poison counter caps at 100 and you can only hold one poisoning at a time.** So the entire
sealed-crate system is worth, across the whole account's lifetime, **one** counter-100 poisoning
(zul-rye) plus ~9 cures of scraps — **and Scorpia already gives counter 100, repeatably, on every
trip.** The drinks are strictly worse than what is already in the plan.

> **So chasing "make the game think I'm still at sea" is chasing the wrong constraint.** Even a
> perfect boat-state exploit buys ~29 extra cures **once, ever** — ~145 recoil, one time, and then
> the crates are spent. It is not a repeatable plan and it does not change the ceiling.

**The 319 ceiling stands. The cure pool stays 20 from Scorpia. The deferral hunt is still the only
route to this fight.**

### What survives, and it is worth keeping

**The venom→poison conversion is real, general, and still the best severity trick available:**
venom climbs 6→20 (+2 per hit, caps at 20), and *"will convert the venom to regular poison, which
starts at the same damage the venom had"* — **no reduction.** So **any repeatable venom source**,
converted at the 20 cap by one Guthix rest dose, yields **counter 100 = 20 cures**. That is the
game's hard maximum (the poison table also tops out at 20: Vespula, Scorpia, Yama). **There is
nothing stronger to find, and Scorpia already reaches it — so this is a convenience, not a gain.**

### And the 65-drink deferral sweep came back empty

Scanned all 65 effect strings for delayed, over-time, per-tick or timer-delivered healing.
**Nothing.** The only "over time" entries are cosmetic — camera rock, a white-screen flashbang, and
*captain clop's mango gin* ("move in a random direction 5 times over the next few seconds"). **No
drink in the batch has a heal that resolves off the drink tick.** Combined with the moid verb sweep
— all 65 are a plain `Drink`, no rare verb — **this batch is closed as a deferral surface.**

---

## ~~★★★★★ THE SHIP DRINKS SOLVE THE SELF-POISON PROBLEM~~ — WITHDRAWN, see the box above

★ Rendi surfaced the sealed-crate drinks. **One of them ends the open question the archive calls
*"the single highest-value unknown in the healing family."*** Verified on the wiki, all four:

| drink | effect | verdict |
|---|---|---|
| **Bottle of zul-rye beer** | *"Player is envenomed starting at 6 damage"* | **★ THE ONE.** See below |
| **Bottle of zogre's sloppy kisses** | poison **starting at 5 damage** + disease | works, 4× weaker, disease is solvable |
| Bottle of spinner's last gasp | poison **starting at 4 damage**, ~4 min | works, weakest |
| Bottle of elidinis's life water | full heal, cures disease, **6 min ANTIPOISON** | ✘ actively hostile — antipoison blocks the whole method |

All are **75 Sailing experience** on drink — a non-combat skill, so **zero combat experience**.
None are stackable. **The wiki states no boat restriction on any of them.**

### Why zul-rye beer is worth 100 counter — the venom conversion

**OSRS wiki, Venom:** starts at **6** damage, **+2 every hit**, **caps at 20** after ~144 seconds.
**And the conversion is lossless:**

> *"a dose of antipoison or **using another method of curing poison** will convert the venom to
> regular poison, **which starts at the same damage the venom had**"* — *"no damage reduction occurs
> during conversion."*

★ Rendi's own note already had the enabling half: *"a Guthix rest downgrades venom to poison."*
**So: drink → let the venom climb → convert with one rest dose → poison at that damage.** Counter =
5 × damage. **This is the "something that does 100 poison damage" Rendi asked for, exactly.**

### How far you can let it climb decides the yield

Venom is a **timer effect, not an NPC attack**, so it is **not capped to current hitpoints** — at
the 15 ceiling a hit of 15+ kills.

| route | convert at | counter | cures | **recoil / bottle** | slots | **per slot** |
|---|---|---|---|---|---|---|
| **A — ride it unstalled at 15 hp** | 14 dmg | 70 | 14 | **70** | 4.5 | **15.6** |
| **B — stall at 1 hp, every tick caps to 1** | 20 dmg | 100 | 20 | **100** | 6.0 | **16.7** |
| zogre bottle (no maturation needed) | 5 dmg | 25 | 5 | 25 | 2.25 | 11.1 |
| *hunter antelope, for comparison* | — | — | — | *9* | *1.45* | ***6.2*** |

**Route B is 2.7× the antelope's recoil per inventory slot, and it is renewable.**
**Do the maturation OUTSIDE the arena** — it is 8 venom hits over 240 ticks and none of the boss's
damage is landing during it, so the existing stall machinery covers it trivially.

### What kills her

| plan | slots |
|---|---|
| **4 × zul-rye beer + 80 rest doses (route B)** | **24** — and that is the whole 400, no antelope at all |
| 6 × zul-rye beer + 84 doses (route A, no stalling during maturation) | 27 |
| 3 bottles + 12 antelope | 30 |

**Free inventory is 18, plus a looting bag at 28 = 46 items.** **24 fits.**

### The quest gate — and the wiki summary's conclusion is WRONG

**Prying Times** (needed for the crowbar): **30 Smithing, 12 Sailing**, plus *Pandemonium* and
*The Knight's Sword*. A fetch summary concluded *"a level 3 combat account cannot complete it"* —
**that is wrong. Smithing and Sailing are non-combat skills and do not touch combat level.** The
only combat in the quest is a level-14 Drink troll, and the wiki guide says you can **skip it** by
returning to your boat before drinking. **Check *Pandemonium* for forced combat; that is the only
real gate left.**

### Two more things found in the same place

- **The disease on the zogre bottle is solvable.** At combat 3 disease is dangerous — the drained
  skill is already level 1, so *"the reduction applies to Hitpoints instead"*. But the
  **inoculation bracelet** *"reduces the damage normally taken from the disease to 0"*, 275 points,
  **no combat or Magic level required** (Zogre Flesh Eaters only). Worn slot, not inventory.
- **A direct venom→poison converter exists at the same crate.** The zul-rye page notes players with
  **66 Sailing and an adamant keel** can take an alternative drink from a nearby crate that
  *"converts the venom to poison instead"* — saving the rest dose per conversion. Non-combat grind.

### ★ VERIFIED VIA MOID (chisel.weirdgloop.org/moid, queried in-page against the `items` table)

Fetch is robots-blocked, so this was run through Rendi's Chrome against moid's own data object.

**Config family: `sailing_charting_drink_crate_*` — 69 items.** IDs confirmed:

| item | id | configName | invOps | stackable | tradeable |
|---|---|---|---|---|---|
| Bottle of zul-rye beer | **31853** | `..._zul_rye` | `Drink, Destroy` | **No** | **No** |
| Bottle of zogre's sloppy kisses | **31850** | `..._zogres_kiss` | `Drink, Destroy` | No | No |
| Bottle of spinner's last gasp | **31836** | `..._spinners_gasp` | `Drink, Destroy` | No | No |
| Bottle of elidinis's life water | 31886 | `..._elidinis` | `Drink, Destroy` | No | No |

**NONE of the 69 are stackable. NONE are tradeable.** Every bottle is one inventory slot and has to be
pried, not bought.

**Verb-space sweep of the family (the exhaustive complement, per §"sweep the VERB SPACE"):**

| verb | in this family | in the whole game |
|---|---|---|
| Drink | 68 | 821 |
| Destroy | 69 | 2516 |
| Eat | 1 — *Mystery fruit* (31903) | 368 |

**No rare verb anywhere in the family.** Every one of them is a plain `Drink`. By the rare-verb
signal that surfaced `Rub-together`, **this batch is NOT bespoke-script territory** — it lowers the
odds that a deferral oddity is hiding in these 69, and it is worth knowing before sweeping them by
hand. **No location-specific op exists on any of them either** — but per ★ Rendi's own correction an
op dump is a **floor** and cannot see a runtime location gate, so this does not settle the boat
question by itself.

### ★ THE CRATES ARE REPEATABLE — the "4 bottles" plan is not gated

**Destroy text, verbatim: *"You can get another from a sealed crate in the sea."*** Untradeable but
farmable. **4 bottles is a sailing trip, not a bottleneck.**

### ⛔ Bottle of crystal clear water — RULE IT OUT, and the wiki contradicts itself

The zul-rye page suggests it *"will reduce venom to poison when drunk."* **The item's own page and
transcript say otherwise: *"will provide six minutes of immunity to poison."*** That is an
**antipoison**, not a converter — **hostile to the method, exactly like elidinis's life water.**
Trust the item page. **The converter stays the Guthix rest dose**, which ★ Rendi's own note already
established.

### ★ AND COUNTER 100 IS THE GAME'S HARD MAXIMUM — there is no "insane" beyond it

Rendi asked for "some insane severity" as the fallback. **This is the ceiling and it is already
reached.** Venom caps at **20 damage**; the poison source table also tops out at **20** (Vespula,
Scorpia, Yama). Poison counter = 5 × damage, so **100 is the largest poison counter that exists in
OSRS**, and zul-rye reaches it for one bottle and one rest dose. **There is nothing stronger to
find — the search for a bigger poison is closed.**

### The one test that decides all of it

**Can these be drunk off a boat?** The wiki states no restriction, but Rendi suspects otherwise and
he is the practitioner. **Buy/pry one bottle and drink it standing on dry land.**

- **Drinkable anywhere** → renewable poisonings inside the arena → **the fight is solved on slots.**
- **Boat-only** → you can still arrive envenomed-at-cap and convert inside, because poison and venom
  are player timers and cross area boundaries. That is **one** 100-counter poisoning — equal to
  Scorpia, not better — and the problem reverts to the deferral hunt.

**~75% they are drinkable anywhere**, and the evidence is now three-sided: no restriction on any
item page, none in any transcript, and **no location-gated op in moid**. The destroy text —
*"you can get another from a sealed crate **in the sea**"* — describes where to **obtain** it, not
where to drink it. **But absence of documentation is not proof, and an op dump cannot see a runtime
gate.** It stays a one-click test.

**AND THE FALLBACK IS SURVIVABLE.** If they turn out boat-only: drink on the boat, mature the venom
there, convert, and walk in — **poison and venom are player timers and cross area boundaries**
(★ Rendi's own note, which is why the current 20 cures are reachable at all). That gives **one**
counter-100 poisoning per trip = **20 cures = 100 recoil** — exactly a Scorpia, but free, repeatable
out of a crate, and with no wilderness trip. **You lose the renewability, not the method.**

---

## On the deferral itself — the 65 drinks are the sweep surface your own file asked for

Rendi's ask stands: a **1-tick non-blocking deferral**. I have no new mechanism. But
`osrs-bug-principles.md` already names the right probe surface and it just got 65 entries bigger:

> *"**Newly-added consumables, especially Sailing-era ones.** New content is the weakest content and
> a fresh food is a freshly-written heal script. Any new drinkable is worth one timed eat against a
> tick counter before assuming it behaves normally."*
>
> *"**stop looking for foods and start looking for heals that run on timers**"* — and
> *"anything whose description mentions an effect **over time**"*.

**65 sealed-crate drinks, all released 19 Nov 2025, none of them swept against a tick counter.**
That is the largest untouched batch of freshly-written consumable scripts in the game, and the
requirement is one that **fails on exactly one axis** for purple sweets — point (1), resolves inline.
**Sweep the 65 for any drink whose heal does not land on the drink tick.** That is a cheap, safe,
zero-experience sweep and it is the best-odds deferral lead currently on the board.

---

## ⛔ ★ RENDI WAS RIGHT — it IS ~20 cures. My "80 cures" is WITHDRAWN.

★ Rendi: *"The cure button via Guthix rest lowers poison hits by 1 every use so this is still only
19 cures no?"* **Yes. I had the cure spending one counter point when it spends five.**

> **OSRS wiki, Guthix rest, verbatim: "Reduce venom to poison, or reduce poison DAMAGE by 1."**
> Damage = `ceil(counter/5)`, so **one damage step IS five counter points.** Scorpia's counter of
> 100 therefore buys **20 cures**, not 80. Exactly the number that was already in the file.

**The ceiling table is restored: cure 90, total 319.** The "607 / winnable" line is withdrawn, and
so is "you do not need a new deferral" — you still do. **The deferral hunt is back on.**

## ★★ BUT THE COUNTER REALLY IS 100, AND THE SIM WAS WRONG IN THE OTHER DIRECTION

The starting-value derivation survives, and it is worth keeping because it corrects two things:

> **Weapon poison(++), melee:** starts at **6 damage**, *"1 less damage every 5 hits"*, *"eventually
> totals **105 damage**"* = 5 hits at each of 6,5,4,3,2,1 over **30 hits**. The 105 matches exactly.
> **⇒ counter = 5 × starting damage.** Scorpia at 20 damage ⇒ **counter 100**.

**Correction 1 — the poison tick hits for 20, not 4.** The sim had counter 19 → `ceil(19/5)` = 4.
It is `ceil(100/5)` = **20**, falling by 1 per cure. Inside a stall it is capped at the frozen 1 and
nothing changes; **unstalled at 10 hitpoints it is instant death.** The sim was under-modelling the
cost of a leaked poison tick by **5×**.

**Correction 2 — a poison tick costs one FIFTH of a cure, not a whole one.** Cure = 5 counter
points, tick = 1. The engine charged both a single point, so every tick that fired burned a whole
cure. Cures actually available = `floor((100 − ticksFired) / 5)`:

| poison ticks fired | old model | **corrected** |
|---|---|---|
| 0 | 19 cures | **20** |
| 10 | 9 | **18** |
| 18 | 1 | **16** |
| 30 | 0 | **14** |

### Measured — v16, one change, `cureCost` 5 and `curePool` 100

| antelope | 23 | 40 | 60 | 80 | 100 | 110 | **115** | 120 |
|---|---|---|---|---|---|---|---|---|
| v15 | 114 | 156 | 207 | 264 | 338 | 377 | 394 | 402 |
| **v16** | **118** | **176** | **207** | **272** | **373** | **394** | **400 — 40/40 KILL** | 401 |

**The kill threshold drops from 120 antelope to 115.** Monotonic throughout; `blockRisk`,
`spikeLeak`, `lostRecoil` all 0 across 420 runs.

### ⚠ SAFETY — the poisonAnchor safe window MOVED, and 7 is now on its edge

A leaked tick costs 20 now, not 4, so the sweep that chose `poisonAnchor` 7 was run against the
wrong number. Re-measured at stock 23, 60 seeds:

| anchor | 5 | 6 | **7** | **8** | **9** | 10+ |
|---|---|---|---|---|---|---|
| recoil | 18 | 18 | **118** | **118** | **120** | 4 |

**The window was {6,7,8}; it is now {7,8,9}.** 7 still works but has **no margin on the low side** —
6 collapses the run to 18 recoil at t42. **8 is the centred choice and scores identically.**
**I have NOT changed it** — one change at a time, and the anchor is yours.

---

## ~~THE POISON POOL MAY BE 100, NOT 20~~ — half withdrawn above, kept for the derivation

**This is the biggest thing in this file and it is one arithmetic error, in one parameter.**

`osrs-amox-sim.html`: `curePool: {v:19, note:'Scorpia 20, decayed to 19 by travel.'}`
OSRS wiki, Scorpia: *"capable of inflicting poison, **starting at 20 damage**."*

**That 20 is the poison's DAMAGE PER HIT, not its severity.** The wiki's own numbers force it:

> **Weapon poison(++), melee:** *"deals **6 damage** … every 18 seconds"*, *"the poison deals 1 less
> damage every **5 hits** until the target is no longer poisoned"*, and *"eventually totals **105
> damage**."*
>
> 5 hits at each of 6,5,4,3,2,1 = **105 damage over 30 hits.** The 105 matches exactly.
> **⇒ number of poison hits = 5 × starting damage.**

**Scorpia's 20 starting damage ⇒ 5 × 20 = 100 poison hits.**

**And it cross-checks against ★ Rendi's own formula.** `damage = floor((severity+4)/5)`. At severity
**100**: `floor(104/5)` = **20** — precisely Scorpia's stated starting damage. At severity 20 the
damage would be **4**, and no source in the game is listed at 4-from-Scorpia.

### Why this is the whole fight

The cure button is **already** the thing being hunted: a **soft script**, so it *fires THROUGH a
stall* — the one heal that satisfies the deferral requirement without needing any new mechanism.
`osrs-bug-principles.md` states the only thing capping it:

> *"| Guthix rest | 4 doses — **but the pool caps total cures at 20**, so ~5 slots is the whole
> allowance |"*

**If the pool is 100, that allowance dissolves and the binder becomes inventory slots of Guthix
rest — which you can simply carry more of.** Rendi's own arithmetic, three pages earlier:
*"20 inventory slots of Guthix rest is 80 doses."*

**80 cures × 5 hp = 400 hp of through-stall healing.** That is the entire endurance requirement,
met with an item already in the kit, through a mechanism already verified.

| | cure ceiling | total ceiling | vs 400 boss hp |
|---|---|---|---|
| as modelled (pool 19, 18 doses) | 90 | **319** | 81 short — unwinnable |
| **pool 100, 20 rest slots × 4** | **400** | **607** | **winnable** |

### Measured in the sim right now

| curePool | rest slots × doses | recoil @ stock 23 | cure | ends |
|---|---|---|---|---|
| 19 (as shipped) | 6 × 3 = 18 | **114** | 32 | t559 |
| **100** | 6 × 3 = 18 | **118** | 40 | t336 |
| **100** | 6 × 4 = 24 | **132** | 54 | t368 |
| 100 | 20 × 4 = 80 | 132 | 54 | t368 |

**+18 recoil immediately, and then it stops scaling** — the cure phase has its own binder past ~24
doses, so the engine cannot yet spend the bigger pool. **That is an engine gap, not a ceiling gap.**
The ceiling moves from 319 to 607; the engine is the thing that now has to be built to reach it.

### ⚠ The catch, stated honestly

**At severity 100 a poison tick deals 20, not 4.** Unstalled, at 10 maximum hitpoints, that is
instant death — which is why the run ends *earlier* (t559 → t368) in the table above. Every poison
tick must land inside a stall, capped at the frozen 1. `poisonAnchor` 7 already does that ("t6, t7
and t8 all land inside the first stall and NEVER come out unstalled"), but **the cost of a single
leak goes from survivable to fatal.**

**And it explains why you would never have noticed.** With the anchor putting the first tick inside
the first stall, capped at 1, **you have never seen the uncapped poison number.**

### The test, and it is cheap and safe

**Do not test it on Scorpia.** Get poisoned by a **poison spider (starting damage 6)** somewhere
safe and **count the total poison hits before it wears off.**

- **30 hits** → the ×5 law holds → Scorpia is severity **100** → carry 20 slots of Guthix rest.
- **6 hits** → the law is wrong, your 20 stands, and this whole section is withdrawn.

One fight with a level-2 spider, no instance, no boss, no combat experience if you do not hit back.
**~75% that the pool is ~100.**

---

## The self-poison ruled-out list — two to add

Both look like poison sources and are not. **`poison karambwan` is the important one because it is
already in the kit** as a same-tick damage source, and its name invites exactly the wrong assumption:

| candidate | verdict |
|---|---|
| **Poison karambwan** | **NOT a poison source.** Wiki: deals 5 damage and *"applies a green poison hitsplat which is **purely visual**: you will not be poisoned or continue to take poison damage afterwards."* Also needs ≥6 hitpoints for the damage to land at all |
| **Poison chalice** | **NOT a poison source.** Seven random outcomes, none of them a poison status; the worst is 50% hp damage, and *"drinking the poison chalice cannot lower your Hitpoints past 4."* Not stackable, consumed |

**And "find a stronger poisoner" is closed:** the wiki's source table tops out at **20 starting
damage**, shared by **Vespula, Scorpia and Yama**. Scorpia is already the maximum. The lever is not
a better poisoner — it is whether 20-damage means severity 100.

---

## On the deferral hunt itself — one idea generated and killed by your own note

**The idea:** damage forcibly closes modal interfaces **two at a time**, and interface stacking from
the inventory can reach *"three, then four, and so on."* If the close is now inside the strong
script's processing step but still only eats two, then **4+ open interfaces should need a second
step, and the damage should slip to the next pass.**

**Dead, and your own observation kills it:** *"damage wipes 2 stackable interfaces but no longer
gets stalled by interfaces; **if you had 100 interfaces and damage rolled in you would now have
98**."* The damage lands anyway. The count is irrelevant. **The interface route is genuinely
exhausted, not under-explored.**

Which leaves the file's position unchanged and correct: **hard actions (~30% one turns up, ~50% it
restores tick-eating)** and **a soft script that creates a delay (~12%, contradiction-shaped)**.
I have nothing that moves either number.

> **But the poison arithmetic above may make the entire hunt unnecessary.** The cure button already
> *is* a non-blocking through-stall heal. Nobody needs to find a new one if the one in the bag has
> five times the uses everyone thought. **Run the spider test before spending another hour on
> deferral mechanics.**

---

## ⛔ RETRACTION — my tick-eat proposal is DEAD for this target, and the working folder said so

I cross-read `osrs-bug-principles.md` and `core-game-mechanics.md` after proposing the sweet
tick-eat loop. **The proposal is wrong, and the refutation is already written in Rendi's own
notes, measured.** `osrs-bug-principles.md` on Amoxliatl's standard attack:

> *"fires **from any distance**, including from inside its body — so no minimum-range trick, no
> safespot, no line-of-sight play; uses **instant damage calculation, 0 delay, mapped**; **cannot
> be styled away** — there is no positioning that forces a different attack; is **unavoidable —
> it cannot be dodged.** Tested."*

**There is no calculation-to-impact window on her auto. It is zero.** My model assumed 1–4 ticks
of projectile flight and produced a 100% kill; the flight does not exist, so the model produces
nothing. I reached "she is magic, therefore projectile, therefore tick-eatable" from the wiki
while the folder held a direct measurement saying otherwise — **exactly the documentation-over-
practitioner error the standing instructions warn about.** ~65% → **~0%.**

**Three more things I offered were already settled in the folder, two of them as mine to retract:**

| what I said | what the folder already had |
|---|---|
| "sweets stack — this demolishes the supply arithmetic" | Stated verbatim at 4868: *"purple sweets are stackable, so if a hit could be deferred WITHOUT blocking you, volume stops being a constraint at all."* Rendi's, not a finding |
| "the ring stores 100,000 charges — the slot wall does not exist" | His own corollary: *"when a charge-based mechanic has a storage item, the inventory stops being the constraint"* |
| **"the highest-leverage unknown is a stackable source above 15 hp — tell me if you know one"** | **CLOSED, and I should not have asked.** *"Raising the hitpoint ceiling: closed. 15 is the real maximum at base 10, with rests."* |
| "one safe test settles it: stand at 2 hp and take an auto" | **Already run.** That is what *mapped* means |

### What the folder's position actually is, and it is tighter than mine was

The requirement is **not** "find a delay". A stall is a delay and stalls are solved — *"stalls can
be enacted from inventory items even while in combat"*. The requirement is:

> **A NON-BLOCKING one-tick deferral.** Something that pushes the hit a tick while leaving an
> inventory click available. A stall is a *blocking* deferral, which is why only the soft cure
> button reaches through it, and that is where the ceiling comes from.

**Two stackable interfaces used to do exactly this. A 2021 engine change ended it** — the note
still pops two interfaces, damage still eats them, but it now applies in the *same* tick instead
of the next. Rendi's standing estimate: **~8–12% that a general repeatable non-blocking deferral
exists today.** I have nothing to add that moves that.

**And the four-point spec is the real object.** Points 1–4 (defers past a stall / no immediate
portion / stackable / does not overwrite while pending). Purple sweets fail on **exactly one**
axis — (1), they resolve inline — and it is the one axis that cannot be changed from outside.

### One of the folder's OPEN leads I can close, using the folder's own kit

`osrs-bug-principles.md` §5115:

> *"A shape worth watching for that nobody has named. An item that heals **when you are hit**
> would be deferred by construction… Whether an *item* with that behaviour exists **has never been
> checked**, and it would satisfy all four points at once."*

**It exists, it is already in the bag, and it is the phoenix necklace.** It fires off the damage
event rather than an action, has no immediate portion, and resolves out of the stall — points 1,
2 and 4, clean. **It fails on point 3 and only point 3:** one per inventory slot, consumed on
proc. This session measured what that is worth: **22 necklaces, ~1 recoil each, 22 against a 400
requirement.**

**So the lead is not unchecked — it is checked, and it fails on volume like everything else.**
That is now true of every route in the table: *volume is the only axis, on every route, every
time.* Worth stating plainly because it means a newly-found on-hit-heal item is only interesting
**if it is stackable** — otherwise it is another antelope meat.

### ⚠ A CONFLICT BETWEEN TWO OF YOUR OWN FILES — your call, worth ~1 recoil and a binder flip

| file | says |
|---|---|
| `osrs-amox-sim.html`, `restDoses` | `v:3` — *"★ Rendi: each cure spends ONE dose of a **3-dose** rest."* |
| `osrs-bug-principles.md`, three separate places | *"Guthix rest stacks **4 doses** to an inventory slot"* · *"the poison pool (20 doses, **4 to a slot**)"* · *"**20 inventory slots of Guthix rest is 80 doses**"* |

Both are attributed to you. One is stale. It decides which resource binds the cure phase:

- **at 3** → 6 slots × 3 = **18 doses** against 19 poison → **doses bind** (and `amoxliatl-level3.md`
  §7 says exactly that: *"doses bind by a hair"*). Cure ceiling **90**.
- **at 4** → 6 slots × 4 = **24 doses** against 19 poison → **the poison pool binds**. Cure ceiling **95**.

**Measured: +1 recoil at stock 23 (114 → 115).** Small, but it moves the ceiling table and it
changes which resource is worth buying more of. **I have NOT changed the sim** — it is a mechanics
value and you are authoritative on it. Say which and I will set it.

---

## ~~★★★ THE TICK-EAT ROUTE~~ — superseded by the retraction above, kept for the modelling only

## ★★★ THE TICK-EAT ROUTE — Rendi's delay idea, and it looks like the actual answer

★ Rendi: *"the only feasible route is to find a way to delay any possible attack of damage…
use the sweets, which are a stackable food, to have a clearance window to eat between the
delayed damage."*

**The mechanic he is describing is real, it is called tick eating, and the OSRS wiki states the
part that matters:**

> *"Almost all monster attacks are capped to the player's current hp when the damage is first
> calculated."*

**That is `capFreezes`.** The sim already models it — but as a property of the STALL. It is not.
It is a property of the attack's **calculation**, and it applies whether or not a stall item is
involved. **The stall's real contribution is DEFERRAL** — holding nine instances behind one
9-point heal — **not the cap.** The cap is free, and it is available every single attack.

### Two facts that demolish the supply arithmetic I gave earlier — both were wrong

| | what I said | what is actually true |
|---|---|---|
| sweets | ~118 items, needs slots | **the only stackable food in the game — ONE slot, unlimited** |
| ring | 400 recoil = 10 rings = 10 slots | **ring of suffering stores up to 100,000 charges** (2,500 rings) in the **equipped** slot — 250× a kill |

**Corrected loadout for the whole plan: equipped pre-charged ring of suffering, 1 slot of purple
sweets, 1 slot rock cake. That is it.** The 46-item wall I costed does not exist. My "short by
918 hp of healing" line is **withdrawn**.

### The loop, walked tick by tick

```
CALC   (impact − flight)   be on exactly 1 hp   -> her roll is capped to 1   -> 1 recoil
CALC+1 click a purple sweet (lands +1)          -> 2-4 hp before the hit lands
IMPACT the capped 1 lands                       -> survive
       rock-cake back down to 1 before the next CALC  (unlimited, 1 hp per click)
```

**Modelled, 20,000 runs: kills her 100% of the time at ~t3,400 (~34 min), spending ~425 sweets
and ~425 cake clicks.** No antelope. No Guthix rest. No cure. No necklaces.

### The one constraint that does bind: FLOOR SPACE

Every auto seeds a 104-tick patch on the tile you were standing on. At an auto every 8 ticks
that is **~14 live patches at all times**, and you must reach a clean tile for every one.

| tiles you can rotate through | kills |
|---|---|
| **9 (the entrance alone)** | **0%** |
| 13 | 0% |
| **16** | **100%** |
| 25 / 144 | 100% |

**So ★ Rendi's own "kite outside the 3×3 entrance between specials" stops being a preference and
becomes a requirement.** It was measured as worth nothing when the dump-tick instance cap was the
binder; on this route it is the binder. The arena is 144 tiles, so 16 is not a hard ask — but the
~106 special fire ticks each still demand you back on tiles 1–9 for that single tick, or an ice
block spawns and that is combat experience you cannot undo.

### Rendi's three candidates, priced

| idea | verdict | confidence |
|---|---|---|
| **stackable interfaces to defer damage** | **THIS IS IT.** It is tick eating, and it is already half-encoded in the sim as `capFreezes` | **85%** the route works |
| **ring of stone morph** | **CLOSED, and closed recently.** Purely cosmetic, zero stats, movement ends it, and per the wiki: *"As of June 2026, transformation rings, including the ring of stone, can no longer be used to avoid boss freezes."* Jagex shut exactly this class | **2%** |
| **firemaking-fail in a no-fire area to burn a tick** | A blocked action returns a message; it does not defer incoming damage. The fire route in the arena is separately tested closed | **3%** |

### The two things that decide it, and the single test that settles both

1. **Does her standard auto cap to your current hp at calculation?** — ~85%. The wiki says
   "almost all monster attacks"; she is a **magic** attacker; and you have already measured the
   cap holding inside stalls, which is the same mechanic.
2. **Does the auto have ≥2 ticks between calculation and impact?** — ~75%. You need 2 because
   a click lands a tick late (`actionInit` 1). Her icicle has flight 4, so she does use
   projectiles.

> **ONE TEST SETTLES BOTH, and it is safe: stand OFF the ice on exactly 2 hitpoints and let a
> single auto land.** If it deals **2**, the cap is live and the whole route opens. If it deals
> more than 2, the cap needs the stall and this is dead. Count the ticks between her attack
> animation and the hitsplat while you are there — that is the flight, and it tells you whether
> the sweet lands in time.

**Combined: ~65% that the sweet tick-eat loop kills her outright, and it needs three inventory
slots.** Against a 319 ceiling that cannot reach 400 no matter how much antelope you buy, this
is not an incremental improvement — **it is the only route found so far that finishes the kill.**

### And they compose

The two are not rivals. The stall route is **twice the rate** (0.25 recoil/tick vs 0.125) but
**hard-capped** at 207 + 90; the tick-eat route is half the rate and **unlimited**. Spend the
antelope and doses first at the fast rate, then fall back to sweets for the remainder. **Do not
build that scheduler until the 2-hp test comes back.**

---

## ★ THE TANK PLAN, PRICED — "200 doses, sit at 15, never let her land 14+, farm the floor"

Walked tick by tick, 400,000 Monte Carlo runs against the sim's own parameters, best-play
policy (only step on ice if you can still be back at 15 for the next event).

### The number

| plan | per-auto survival | autos a 400 kill needs | **P(kill)** |
|---|---|---|---|
| **ice ON, specials dodged, 200 doses** | 80.8% | ~66 | **1 in 1.3 million** (0.00007%) |
| ice OFF, autos only | 89.5% | ~300 | 1 in 3.2 × 10¹⁴ |
| *fantasy: exactly 15 at every auto AND 100% ice uptime* | 88.2% | 33 | *1.6%* |

**0 kills in 400,000 runs, and 0 in another 300,000 with infinite heals and infinite ring
charges.** The heals are not what is stopping it.

### Why — the three assumptions, each priced separately

**1. "always sit 15/10 hp on the auto" — you get it 59% of the time, with unlimited supplies.**
A Guthix rest overheals to 15 **only if it lands while you are already at 10**; below that it
caps at 10, and a sweet never passes 10 at all. So after an average 8-damage auto you must
climb 7 → 10 on sweets (1–3, one click per 3 ticks) *before* a rest can lift you to 15. That
is 7–9 ticks and the autos come every 8. Measured: **59% of autos land on 15, 40% land on 10.**
At 10 hp an auto kills **41.2%** of the time.

**2. "never have boss land more than 14" — this is the part that cannot be arranged.**
`autoMax` is **16** and the roll is uniform 0–16, so P(≤14) = 15/17 = **88.2% per auto**, and
the special is 0–17, P(≤14) = **83.3%**. There is no defensive lever at combat 3 — protection
prayers need 37 Prayer. Compounding:

| autos survived | 10 | 33 | 66 | 300 |
|---|---|---|---|---|
| P at a perfect 15 | 28.5% | 1.6% | 0.026% | 5 × 10⁻¹⁵ |

> **★ THIS IS THE WHOLE REASON THE PLAN IS BUILT ON STALLS.** Her max hit (16) is **larger than
> your maximum possible hitpoints (15)**. You cannot tank one auto with certainty, ever, at any
> supply level. Deferring damage into a stall and capping it at a frozen 1 is not an
> optimisation — it is the only thing that converts an unbounded roll into a bounded one.

**3. "sped-up kill via floor tiles" — this part is sound, and it is the only part that helps.**
The floor pays **1 recoil per tick** versus 0.125/tick from attacks alone, which is what drags
the auto count from 300 down to 66 and buys eight orders of magnitude. But it cannot run full
time:

```
heal    rest 5 + sweet 1-3 per 3 ticks         =  2.33 hp/tick
damage  floor 3.0 + auto 1.0 + spec 0.27       =  4.27 hp/tick on permanent ice
                                       deficit = -1.94 hp/tick
                    sustainable ice uptime     =  35% of ticks
```

### And the constraint that binds before the dice even get rolled

| | need | have |
|---|---|---|
| ring charges (1 per recoil) | 400 = **10 rings** | — |
| 200 doses of Guthix rest | **67 rests** | — |
| sweets over a ~708t run | **~118** | — |
| **total items** | **~195** | **46** (18 free inventory + 28 looting bag) |

⚠ **PARTLY WITHDRAWN — see the tick-eat section above.** Purple sweets are **stackable** (one
slot, unlimited) and the ring of suffering stores **100,000 charges** in the equipped slot, so
neither is a slot cost. The Guthix rests still are. The *survival* arithmetic below stands
unchanged — it was never the supplies that killed this plan, it was the 16 max hit.

⚠ **COMBAT XP FLAG.** ~22 specials in a 708-tick run, and each one requires you on tiles 1–9
**on the fire tick**. Farming ice out in the room when one fires spawns an ice block —
melee-only, always a max hit, **combat experience that cannot be undone**. That is 22 separate
one-tick windows to never miss, while also managing a 3-tick heal rotation.

### What would move these numbers

- **Anything that lifts the effective hp ceiling above 16** turns the auto from a death roll
  into a non-event and changes the arithmetic categorically. This is the single
  highest-leverage unknown in the whole fight. `hpCeil` 15 is the Guthix rest overheal; if you
  know a stackable source that beats it, say so.
- **Anything that lowers her max hit** — same effect, and I know of no lever at combat 3.
- **Floor uptime above 35%** is worth roughly one order of magnitude per 20 points of uptime,
  but it is heal-rate bound, not decision bound.

**Verdict: well under one in a million on the dice, and effectively zero in practice because
the inventory runs out first.** The stall plan is not the conservative option — it is the only
one that survives contact with a 16 max hit.

---

## THE CEILING — total possible recoil with this method

Every point of recoil needs one instance of coverage, and instances come only from
consumables. Re-confirmed against the v14 engine:

| phase | ceiling | working | v14 engine gets (stock 23) |
|---|---|---|---|
| **FOOD** | **207** | 23 antelope × 9 (10 hp max, no overheal) | **77** |
| **CURE** | **90** optimistic / **72** safe | 18 doses × 5, or × 4 keeping the 5th slot as poison clearance (`cureRecoil` 4) | **30** |
| **NULL** | **22** ★ confirmed | floor ticks walked between the procs, ~1 per attack — NOT the self-hit | **2** |
| | **319 / 301** | | **114** |

**Against 400 boss hitpoints, the stock kit cannot kill her — 81 short at perfect play, 199
short at the engine's current play.** This is not an execution problem; dodging, tick-saving
and stall-fitting improve *survival*, not *coverage*, because none of them add consumables.

**The only lever that adds instances is antelope, at 9 recoil each:**

- **32 antelope** — theoretical minimum. `400 − 90 cure − 22 null = 288 food ÷ 9`.
  (★ the null 22 is confirmed real, so this stands.)
- **~47 antelope** — at ★ Rendi's own hand-walked food rate of 6.65 per antelope.
- **120 antelope** — what the v14 engine actually needs today (3.55 of 9), 40/40 seeds.

**Carry the spread, not the point estimate.** The honest answer to "how many antelope" is
**between 32 and 120, and where it lands is entirely decided by whether the food phase gets
the expiry rotation.** Rendi's own hand-walk sits at 47 — that is the realistic target. Every 1 instance/cycle the food phase gains is worth ~23 recoil at
stock and removes ~8 antelope from the requirement.

---

## v13 — WHAT THE t723 DEATH ACTUALLY WAS (both v12 candidates were wrong)

The method above worked exactly as written. Both hypotheses it produced were wrong, and
they were wrong in an instructive way, so the reasoning is kept.

**Candidate 1 — "the arm happened at hp 2, so the heal capped at +8" — DISPROVEN, and it
was never a loss anyway.** The arm gate `st.hp<=1` was **not** bypassed. It armed at hp 1
and froze the cap at 1, exactly as the log said. A **natural regen** then landed *inside*
the stall and lifted 1 → 2, which is why the delayed portion read +8 instead of +9.
But **the arithmetic is identical**: `1 + 9 = 2 + 8 = 10`, so `postHp` was 2 either way.
The planner's number was right. The hp-2 reading was a real observation of a real regen
and a red herring as a cause.

**Candidate 2 — "the tileThreat dump check is not being reached" — DISPROVEN.** It was
reached, on **every** candidate tile, every arm tick. Instrumented at seed 1, 140 food,
the chosen plan was `arm 706, L 17, dump 723, tile 1, 8 instances, postHp 2`, and
`tileThreat(tile, 723)` returned **0 on all five candidate tiles.**

**The actual cause — the third thing, which neither candidate named:**

> `tileThreat()` reads `st.ice`. **The ice that killed him did not exist yet at plan time.**

It was seeded at **t712 by an auto the stall itself was about to swallow**, and was age 11
— a live pattern, worth 5 — by the dump at t723. At the arm tick t706 that patch was not in
the array at all, so every threat function that reads the array returned 0, and the gate
`postHp - thr < 1` never fired.

**This is why upgrading that line from `iceAt` to `tileThreat` moved the numbers by zero.**
A pattern-vs-spike upgrade cannot help when the patch is not in the array. Both previous
structural deaths were *"the check sees one damage source and not the other"*; this one is a
different class — **the check sees the present and not the future**. Add it to the list:

| blind spot | symptom |
|---|---|
| pattern-only (`iceAt`) | age-3 patch reads safe, hits for 8 |
| present-only (`tileThreat`) | ice the stall is about to seed reads as no tile at all |

**The fix, one line of logic.** The same scope already had the correct projection —
`patAt(tile, t, AU)`, which is what `tileCost` has always used to count the instances. The
dump gate simply was not using it. It now takes the **maximum of both views**:

```js
const thrNow  = tileThreat(tt,dump);                       // what is on the tile NOW
const thrSeed = patAt(tt,dump,AU) ? p('icePatMax')         // what THIS STALL will seed
              : (AU.some(x => dump-x===SPK && !patAt(tt,x,AU)) ? p('iceSpikeMax') : 0);
const thr = Math.max(thrNow, thrSeed);
```

Both terms are needed. `thrNow` alone is blind to seeded ice (the bug). `thrSeed` alone is
blind to a patch already down at age 3, which `AU` cannot see. **Keep both**, same as the
double enforcement of the spike suppression.

**Measured, one change, nothing else touched:** plateau 89/283 → gone; 120 antelope kills
her in 40/40 seeds. `blockRisk` 0 and `spikeLeak` 0 throughout.

### The next thing, which is NOT the plateau

At 110 carried the run now spends all 110, reaches **377 recoil, and dies 23 hp short with
21 phoenix necklaces unspent** in the null phase, poison pool empty. That is a genuine
resource-exhaustion end state, not an unmodelled death — recoil still rises monotonically
with food, so the acceptance test is unaffected. But **21 unspent necklaces is a lot of
unconverted capacity**, and the null phase is where it sits. Worth a look before the
rotation, not instead of it.

---

## What was fixed this session, so it is not re-done

**The plateau was 54 foods / 162 recoil. It is now 89 / 283.** One whole death class was found and closed:

**The descent was halving into a spike.** The step machine nightshades *before* the floor block resolves, so a
descent from 10 went 10 → 5 → (spike rolls 5+) → dead. The guard meant to prevent this checked only the floor
pattern, so a tile carrying an age-3 patch looked safe. Now guarded with `tileThreat`, in both the normal
descent and the re-descent branch. Worth **+121 recoil and 35 more foods spent**.

Related and still worth revisiting: ★ Rendi's own cycle line reads *"floor 1-5 **and** the halve"* — **roll
first, halve second.** The engine has the order the other way round; the guard works around that rather than
fixing it. Fixing the order properly may remove the need for the guard.

---

## The mechanics, as measured. These are settled — do not re-litigate them

★ = Rendi, measured in game, authoritative over the wiki and over anything derived.

**The ice timeline (★ definitive, re-tested end to end).** Counting the auto as +0:

- **+3** ice spike hits, *if no floor pattern is already on the tile*
- **+4** the floor pattern OBJECT spawns, *if not already there*
- **+7** the pattern starts damaging, then **every tick**
- **+104** last damaging tick
- **+105** gone

`icePatAt` = 7, `iceLife` = 104, `iceSpikeAt` = 3, `icePatSpawn` = 4. **The 30/31/32 reading is withdrawn** —
it was wrong; the original ~100 was close.

**★ Patterns do not re-spawn on a tile that already has one.** They do not stack and do not refresh, so a
tile's expiry is fixed by the **first** auto that landed on it. Re-standing on an old patch does not buy a
fresh 104 ticks. (The engine used to push a fresh patch on every auto, silently renewing exactly the tiles the
plan keeps returning to.)

**★ The dump tick is an INITIATION tick.** *"The tick after the stall I take floor damage unless ice subsides
and because it takes me 1 tick to eat a new food. Can't move that tick or eat that tick, but can initiate
both."* Both clicks go in; neither resolves until the tick after. So you are still on the stall's tile when the
floor rolls, on `1 + heal − instances`, with the new food not yet landed. **No stepping off, no refill.**
`frozenFeet` is `stall > 0`.

**★ You must be on tiles 1–9 on the tick the special FIRES** — that single tick, not the whole flight. Outside
it an **ice block** forms: melee-only to destroy, always a max hit, and **combat experience that cannot be
undone**. This is a hard constraint in the stall selector, not a router preference — it was being violated on
30% of fire ticks and every violation had the feet locked in a stall, which is why a router-level reservation
could never catch it. **`blockRisk` must read 0.** It is the one counter where non-zero is not a worse score
but an invalid run.

**★ The icicle never occurs if you are already standing on a live floor pattern**, unless the floor is gone.
Modelled across the flight: patterned at fire *and* still patterned at impact. Measured at **zero recoil** in
the current plan — the food phase is capacity-bound, so suppressing an icicle only frees a slot that a floor
tick refills at the same 1. It buys permission, not damage.

**★ Poison:** severity steps by one per *damage event*, once per 30 ticks; damage is `floor((severity+4)/5)`,
so the damage holds flat for five hits then steps, roughly one step per 90 seconds. **The cure is the BUTTON
and it needs poison to act on; a dose costs ONE.** Leftover doses are a consequence of the fight's length, not
a bug to optimise away.

**★ She is POISON-immune (100%), VENOM-immune (100%) and CANNON-immune.** Dynamite(p) lands nothing here, and
the cannon branch of the zero-XP toolkit is closed for this fight.

**★ The fire route is closed.** Tested in the arena: *"a magical force prevents you from lighting a fire in
this area."* Standard logs, bow, all of it. One live thread remains — inventory-lighting and ground-lighting
**check in a different order** (area-first vs level-first), which proves they are separate code. One action
settles whether that is a reordering or a missing check: use a tinderbox on a log you *do* have the level for,
**from the ground**, inside the arena.

---

## The two open strategy ideas — recorded, but NOT this session's job

**Kiting the floor.** ★ Rendi: *"use the same resources but spawn ice and use the expiring ones for each auto
attack to make sure they expire by the tick you come out of the stall"* and *"kite outside the 3×3 entrance
between special attacks to free up floor space."*

This is the right shape and follows directly from the corrected mechanics. With a 104-tick life and **no
refreshing**, the plan becomes a **rotation**: seed patches at known ticks and time each arm to the patch whose
expiry lands on that cycle's dump. The condition is exact — for a stall of length L armed at A on a patch
formed at F, being live throughout *and* dead at the dump requires `A + L − 1 = F + 104`, i.e. **arm at ice age
`105 − L`** (age 98 for a 7-tick stall). A **one-tick window per patch**, which makes it a scheduling problem
rather than a search problem.

Kiting out of the entrance is legal — the combat-experience constraint binds only on the special's fire tick.
Tested as a router preference and measured **identical**, because floor space is not the binding constraint
today; the dump-tick instance cap is. **Re-test once the deaths are fixed**, since it should matter a great
deal when long stalls become usable.

**⚠ Do not implement the rotation until the plateau is gone.** A scheduler layered on an engine that dies for
unmodelled reasons only produces another unfalsifiable number.

---

## The arithmetic, and why no antelope count is quoted

**Both previous answers are withdrawn.** v9 said +21 kills it (60/60) — that rested on `iceLife` 100 and a
movable dump tick. v10 said +89 (40/40) — that rested on `iceLife` 30. **Neither assumption survived
measurement, and the current build cannot answer the question at all while it plateaus.**

The honest status is **undetermined**. Do not plan a food count against this file.

**The acceptance test for this session: recoil must rise with every extra antelope carried, until the boss
dies.** Anything short of that means a death is still unmodelled.

**v13 UPDATE — the acceptance test PASSES, so the question is answerable again.** The sim's
current number is **120 antelope carried, 117 actually spent, 402 recoil, dead at t945, 40/40
seeds.** Two caveats before anyone shops for 120 antelope:

1. It is **the sim's** number, from a build that has been correct for about an hour. The two
   withdrawn answers (+21, +89) each looked this solid at the time.
2. It has **no margin** — 402 recoil against 400 hp is two points of slack, and it is 40/40
   only because the planner is deterministic. Carry more than 120.

What would move it: another death class surfacing at high food, or the null-phase necklace
capacity noted above being converted (which should *reduce* the count).

---

## Standing instructions

- When Rendi corrects a game mechanic, **take it as authoritative over the wiki** and write it in.
  Practitioners beat documentation. Twenty-plus corrections, twenty-plus times.
- **Identify before theorising** — `chisel.weirdgloop.org/moid` for IDs and menu options, `mejrs.github.io/osrs`
  for spawn counts. **But an `ops` dump is a FLOOR, not a complete list**: it is the static menu and cannot see
  `use X on Y`, nor options a script adds at runtime. Never conclude "nothing else in the game can do X" from an
  op sweep — say "at least these".
- **Use Rendi's Chrome** whenever a fetch fails or you need Reddit.
- **Flag anything that would gain combat experience.** That is the one thing that cannot be undone.
  ⚠ Superheat Item is a Magic spell; it surfaces in any "heat without a fire" search. Never use it.
- Give probability percentages on hypotheses, and say what would move them.
- **Price by walking the ticks**, not by multiplying per-item values.
- **Change one thing at a time and measure it.** If a change makes the number worse, revert before trying the
  next. A previous session stacked five fixes and produced 28, 19, 14, 139 against a 172 baseline.
- **Syntax-check the inline script before sending the sim back**, and load it in a browser to confirm no page
  errors. One command each; both have caught real breakage.
- ⚠ **The cloud container reverts the working copy without warning** — it happened five times in one session.
  **Before editing, grep the file for a known recent marker to confirm you have the current version**, and
  re-stage from Rendi's disk if not. His disk copy is the source of truth.

---

## ★ Generalised engine knowledge from this project has been WRITTEN OUT of this file

So the next RuneScape conversation starts with the mechanics rather than rediscovering them:

- **`core-game-mechanics.md` §14 — Player action timing.** The one-tick click tax, the 3-tick food
  cooldown and its fast-food exceptions, the food→potion combo being one-directional, the
  overheal-only-from-full rule, 2-tiles-per-tick with one allowance per tick, inventory actions
  sharing a tick with movement, **the initiation tick**, and attack-delay vs queue-delay.
- **`core-game-mechanics.md` §15 — Status effect arithmetic.** Poison as a counter (`damage =
  ceil(counter/5)`, counter = 5 x starting damage), **a damage-reducing cure spending 5 counter
  points and a poison tick spending 1**, the 20-damage/counter-100 game maximum, venom's 6→20 climb
  and its **lossless** conversion to poison, disease redirecting to Hitpoints on level-1 skills,
  recoil's 10%+1 step function, and purple sweets as the only stackable food.
- **`osrs-bug-principles.md` §10 — the five ways a survivability check lies.** The recurring shapes
  behind every death this rebuild found, plus the measurement discipline that made them findable.
- **`osrs-bug-principles.md` §11 — moid is queryable in-page.** The `window.items` technique, with
  the config-family and verb-space sweeps as one expression each.

## Files

| File | What |
|---|---|
| `AMOXLIATL-START-HERE.md` | This. Read first. |
| `AMOXLIATL-START-HERE.txt` | The short paste-prompt version of this. |
| `amoxliatl-level3.md` | Full investigation state — fight data, every ★ correction, phase order, cycles, the fire/cooking research, ceiling arithmetic. Standalone. |
| `osrs-amox-sim.html` | The sim. Single file, one inline script, no dependencies. **v12.** |
| `osrs-bug-principles.md` | The reasoning layer — probe design, the op/ap research corollary, the zero-XP toolkit. |
| `core-game-mechanics.md` | Engine-level: queues, delays, op vs ap, instancing. §6 carries the op-dump caveat. |
