# WORLDS THREE & FOUR — design spec (v2, decisions locked)

Companion to `CLAUDE.md`. Merge into it as **§11–§16**, or keep as a sibling file with a pointer from CLAUDE.md's roadmap. Claude Code should treat `CLAUDE.md` as authoritative for what is already built and this file as authoritative for what comes next.

**What this adds:**
- **§11 The Hat Rack** — a visible, countable progression object, plus the golden night bonus and a sealed playtest mode. Replaces the invisible `hatUnlockRuns` gate.
- **§12 World Three — The Forest** (homage in spirit to *I Want My Hat Back*). Ladder rung six: **route reversal**, and noticing what the map already showed you.
- **§13 World Four — Deep Water** (homage in spirit to *This Is Not My Hat*). Ladder rung seven: **building a map from partial views**.
- **§14** engine work. **§15** CONFIG additions. **§16** what's still open.

**Homage rule, unchanged and non-negotiable:** all original assets, all original strings. No reproduction of Klassen's illustrations, characters, compositions, trade dress, or text — not one line of dialogue, not one drawn animal. The debt we're honouring is the deadpan, the withheld information, and the flat colour. In-game names are ours: *The Forest*, *Deep Water* — never the book titles.

**Decisions locked in this revision:** thresholds 5 / 10 / 15 days · one currency (days played) · golden-night bonus = no bumps **and** under par, once per calendar day · rack appears only on reaching World Two · Forest speech is pictures with a words toggle · Forest trace charcoal · Forest built first · deadpan endings kept as the books leave them.

---

## §11 The Hat Rack — progression made visible

### Principle
Progress **never goes backwards**. The streak system already bends kindly because the ritual must survive an off week; the unlock counter must be gentler still. Nothing earned is ever taken away, and the counter never mentions a day that wasn't played.

### The currency: days played
Count **distinct calendar days on which a run was completed**, in any world. Cumulative, monotonic, shared across all four worlds — one ritual, one number.

- Store as `tunnels:daysPlayed` (integer) alongside the existing streak record.
- **Migration for existing saves:** derive from the journal, which already stamps `day` per entry — `daysPlayed = new Set(journal.map(e => e.day)).size`. Nothing is lost when the gate moves off `runs`.
- Streaks stay exactly as they are and keep their own job (difficulty notches, the "Day 7 of digging together" line). They are celebrated, not spent.

### The unlock ladder

| Peg | World | Hat | Unlocks at |
|---|---|---|---|
| 1 | Tunnels | yellow hard hat | from the start |
| 2 | The Hat | tan cowboy hat | **5 days** |
| 3 | The Forest | red pointy hat | **10 days** |
| 4 | Deep Water | small blue bowler | **15 days** |

### The golden night (bonus)
A run that is **bump-free across all three stages and finishes under par** is worth **two nights instead of one**.

- **Par is per run, not per stage** — a slow first stage can be made up later. Par is derived, never authored, so it survives procedural generation and the streak speed lever:

  ```
  parMs = Σ over the 3 stages of [ (routeTiles / effectiveSpeed) × 1000 × parSlack
                                   + junctionCount × junctionThinkMs ]
  ```

  `routeTiles` is the optimal route (the existing route-walk already computes it), `effectiveSpeed` is that stage's post-notch speed, `parSlack` 1.25, `junctionThinkMs` 12000.

- **Why the junction allowance is so large:** `G.stageMs` runs during the `junction` phase, so every second Alma spends deciding is on the clock. Twelve seconds per junction means a careful, chatty pair comfortably beats par and only genuine stalling loses it. **The briefing is untimed and stays untimed** — the planning conversation, which is the point of the whole game, costs nothing. Par measures time underground only.
- **Par is never shown during a stage.** No countdown, no timer bar, no colour change as it runs down. A visible clock would poison the eyes-closed ritual and breaks §10. It is revealed only afterwards, on the run-complete card.
- **Once per calendar day, first qualifying run only.** The daily seed is replayable all day; a memorised maze makes a flawless second run trivial, and without this cap a determined afternoon unlocks everything.
- **The celebration:** two pips fill instead of one, both **gold** rather than pale, with a rising two-note chime, a distinct haptic, and a scatter of gold flat-particles across the rack. One line: *"A golden night."* The pair should be able to tell a golden night from an ordinary one from across the room, with the sound alone.
- The rack shows a small **gold moon outline** beside the next peg at all times, so the pair know the bonus exists and what it looks like, without ever being told a target time.

### The object
A **hat rack**: four pegs, one hat per world. Every world has a hat — that's the joke of the whole series, so it's the right thing to collect.

- **Unlocked** pegs hold their hat in full colour.
- **Locked** pegs hold a **pale outline of the hat's silhouette** — not a padlock, not a question mark. The shape is the tease. A family that reads these books recognises a red pointy hat from its outline instantly, and that recognition is the reward for waiting. Alma will work out which book is coming and announce it. Let her.
- Beneath the **next** locked peg, **sleeps remaining as countable pips** — small moons, one per night, shown when `rackPipThreshold` (7) or fewer remain. With gaps of five, they're effectively always visible once the rack exists. A number is a wall; five moons is a countdown you can touch and count aloud.

### When the rack first appears
**Not before World Two is reached.** Days 1–4 look exactly as they do today. The rack's own arrival is therefore the first ceremony: on the run that reaches five days, the rack swings into view for the first time and **two hats land on pegs 1 and 2 together**, backfilling what they've already earned, with two empty silhouettes waiting to the right. From that run onward the rack appears on every run-complete card.

### Where the rack appears
- **Run-complete card** — always (once unlocked), above the stats. This is where pips fill.
- **The world picker** — the rack *is* the picker (§14.3).
- **Nowhere else.** Never mid-stage, never in the briefing. No percentages, no bars.

### The unlock ceremony (worlds three and four)
1. The rack fills the screen; the normal stats card is deferred.
2. The new hat **drops onto its peg** — one chunky flat-particle bounce, a fanfare distinct from the stage fanfare, a celebratory haptic.
3. One short line in the display face, then a smaller one: *"A new hat."* / *"Somebody has lost this one."* (Deep Water: *"This one is not ours."*)
4. Chord to continue. The picker opens with the new world highlighted.
5. Stats card follows.

### Tone rules for the counter
Never reference absence. No "you missed yesterday", no "come back tomorrow", no streak-loss copy. The rack states only what is true right now: this many hats, this many sleeps.

### Playtest mode (Marty only)
A sealed mode that unlocks all four worlds immediately, for testing ahead of the pair.

**Three doors, because the rack doesn't exist on day one:**
1. **URL param** — `?unlockAll=1`. Instant, works from the very first boot.
2. **Dev panel** — the existing five-taps-on-the-sky gesture gains a button: **"Take all the hats down."** Toggles the mode.
3. **The rack gesture** — once the rack exists, tap the four pegs in **reverse order (4, 3, 2, 1)** within four seconds. The hats lift off and land back on in colour, one at a time, right to left. The same gesture toggles it off. Deliberate enough that it is not stumbled into, and it looks like part of the game if anyone ever sees it happen.

**Sealed from the real save.** While playtest mode is on, all writes to `journal`, `daysPlayed`, `streak`, `best` and `seedbest:*` are suppressed. Reads work normally, so the rack renders truthfully underneath. Test runs never stamp the journal, never consume a night, never break or advance a streak.

**The tell:** a small gold moon sits in the top corner of the rack, and the debug overlay prints `PLAYTEST`. Marty must never be able to confuse a test run with a real one.

---

## §12 WORLD THREE — THE FOREST

Homage in spirit to *I Want My Hat Back*. All original assets.

### Why this world exists (design intent)
World One taught screen-direction vocabulary in elevation. World Two moved it to plan view with landmark language. World Three adds **rung six: route reversal** — going back the way you came, a genuinely harder spatial operation than going, and the one that turns a route into a map.

It also teaches something subtler and funnier: **the answer was on screen the whole time**. The rabbit wears the stolen hat from the first frame of the briefing. The bear never notices. Alma will notice within four seconds and lose her mind about it, and the Driver — eyes closed, being told urgently about a rabbit — cannot do a thing until the bear works it out for himself. That gap is the world.

### The fiction
A bear has lost his hat and walks the forest asking whoever he passes. The animals answer, deadpan and unhelpfully. One of them is wearing a red pointy hat and is extremely keen to communicate that it has not seen any hats anywhere. The bear asks it politely and walks on. At the bottom of the forest a deer asks what the hat actually looks like. And then the bear knows.

### Structure: two legs and a red page
One maze, traversed twice.

**Leg 1 — the search (down-screen).** The bear enters at the top and walks down the spine. Animals sit at the tips of the side arms. Junction pauses behave exactly as they always have: chord to keep going down, green or red to turn into an arm.

**The red page.** On arriving at the deer (the leg-1 exit, bottom of the spine), the screen floods **full-bleed saturated red** for `forestRedPageMs`. One large line, ours: *"He knows where his hat is."* The longest haptic in the game. Then the red clears and the forest returns, bear facing up.

**Leg 2 — the run back (up-screen).** The goal becomes the rabbit's arm tip. Speed × `forestReturnSpeedMult`. The trodden path stays drawn (the forest doesn't forget), but **the animals are gone and so is the rabbit** — Alma must remember which arm, and on which side. A wrong arm is a dead end, a bump, a bounce back to the junction: funny, free, try again.

### The load-bearing geometric constraint (do not skip)
**Forest stages use chord-type junctions only** — `stageChordShare: [1, 1, 1]`. The route always continues straight down the spine; every arm branches sideways.

This is not stylistic, it is what makes the world legal. The screen-direction invariant requires vertical travel at every pause. On the return leg the bear enters each junction from whichever direction the route *left* it. A T-junction leaves horizontally, so the return enters horizontally and the calls become "straight on" and "upward" — and there is no button for upward. A chord junction leaves vertically, so the return enters vertically and the calls are exactly: chord to keep climbing, green or red into an arm. Identical grammar, identical buttons, reversed journey. Green is still screen-left in both legs, because the world never flips.

The generator already supports this via `doChord()` and `stageChordShare`. Leg 2 needs no new maze: set start to the deer cell, heading up, exit to the rabbit's arm tip.

### Meeting an animal
Arriving at an arm tip holding an animal is **a conversation, not a bump**:
- The bear stops. A bubble appears for `speakerLineMs`. A soft chime marks it, so the Driver knows something happened.
- The bear turns and walks back to the junction along the existing bounce path, with **no bonk sound, no shake, and no bump counted**. A *polite turnaround*.
- Arm tips with **no** animal keep the normal bump. Trees are not conversational.

**The answers are pictures, not words** (`forestSpeechMode: 'pictures'`). Alma reads letters but not yet words, and pictograms are more Klassen anyway — the humour is in the flatness, not the phrasing. The vocabulary is tiny and consistent:

| Speaker | Bubble |
|---|---|
| Most animals | a red pointy hat silhouette with a diagonal slash |
| The one who asks back | a hat silhouette in outline, with a question mark |
| **The rabbit** | **three slashed hats, crammed in, far too many** — protesting much too much. The entire characterisation, no words needed, and Alma will read it instantly |
| The sleeper | a closed-eye line and one small z |
| The deer | see below |

`forestSpeechMode: 'words'` swaps the bubbles for large two-letter words (NO / OK) for the day she's ready. One CONFIG flip, no rebuild.

**The deer beat — the trigger for the red page.** The deer's bubble shows an **empty hat outline**. The bear's own bubble then draws the same shape and **fills it red and pointy**. Two identical silhouettes, side by side, one of which Alma has been shouting about for two minutes. Then the red page. The logic of the story is told entirely in shape and colour, with nothing to read — and it is the clearest possible statement of *the answer was already on the map*.

**The silent joke:** one animal is always asleep and never answers, in every stage, forever. The bear asks anyway. It is never remarked upon. (World One's cat, World Two's lizard, World Three's sleeper.)

### Plan trace
This is the world it was invented for. During the red page the game holds and lets Alma **ink the way back** with her finger before the bear moves. Plan in soft grey, actual in ink, compared at the reveal. `planTraceOnReturnLeg: true`. Skippable, as always.

### Return-leg difficulty
Ship with the rabbit hidden (`forestShowRabbitOnReturn: false`) and a hint after three wrong arms (`forestHintAfterBumps: 3`) — the rabbit reappears briefly, no comment, no penalty. Take the hint away when she stops needing it.

### Reveal
The full forest as a picture-book spread. **Two lines, two weights:** leg 1 in soft grey, leg 2 in charcoal ink — the shape of the whole journey visible at a glance. Bump stars as usual. Animals redrawn in place. The rabbit drawn without its hat.

### Run complete
The bear, sitting, wearing the red hat, facing the reader. Deadpan. A squirrel wanders in and asks whether he has seen a rabbit around here wearing a hat. The bear says he has not, and asks why the squirrel is asking him. Nothing else happens and nothing is explained. World One signs off with chocolate milk, World Two with stars; **World Three signs off with a question nobody answers.**

### Art direction (delta from §5 of CLAUDE.md)
- **View:** elevation, side-on, on the forest floor. Tree trunks are the impassable walls — flat vertical bands in two greens and a grey-brown, no outlines over 1px, occasional flat root wedges. Background a warm ivory (`#EFE7D6`), noticeably paler than World One's soil: the forest reads as air, not earth.
- **Palette:** ivory ground, `bark` #6B5A44, `canopy` #6C7A55, `moss` #8A9468, animals in two or three flat muted colours each, and **one saturated red for the hat and only the hat** (`#B23A48`).
- **The red belongs to the hat here.** The Forest's reveal trace goes **charcoal** `#33291F` at 85% alpha; the grey plan line is unchanged.
- **The bear:** large, upright, mirrored not rotated (elevation rules), one dot eye, no expression at any point including the end. He fills more of a cell than the digger does — his size against a small rabbit is the comedy.
- **Motion:** minimal. A slow two-frame plod. The red page is the only large motion event besides bumps. Under `prefers-reduced-motion` the red page becomes a hold, not a flash — **and it is never a strobe at any setting.**

### WORLDS hook table
| Hook | Value |
|---|---|
| `label` | The Forest |
| `companion` | false (the cast replaces the dog) |
| `hasGems` | false |
| `bumpNoun` | Bumps |
| `bumpDry` | false (a thud into a trunk) |
| `dust` / `burst` | `#8A9468` / `['#8A9468','#EFE7D6']` |
| `speed` | `speedWithNotch(CONFIG.forestWalkSpeed)`, × `forestReturnSpeedMult` on leg 2 |
| `stageFor` | generator with `chordShare: 1`, plus animal and rabbit placement |
| `topStrip` / `bottomStrip` | thin canopy band / 0 |
| `drawExtras` | animals, the sleeper, bubbles |
| `drawExitMarker` | leg 1: the deer. leg 2: nothing (that's the point) |
| `drawTrail` | two-weight double trace |
| `drawAvatar` | the bear |
| `rewardLine` | "He has his hat." |
| `reward` | the sitting-bear scene |

---

## §13 WORLD FOUR — DEEP WATER

Homage in spirit to *This Is Not My Hat*. All original assets.

### Why this world exists (design intent)
Rung seven: **building a map from partial views**. Every world so far hands the Navigator a complete map at briefing. Deep Water hands her a few seconds of one, then the light goes. What she holds in her head after that is cartography — the thing the whole ladder has been climbing toward.

### The fiction
A small fish wears a hat that is not its. It is quite sure it will not be caught. Behind it, at a constant unhurried distance, comes a much larger fish. Ahead are the big tall plants, where the plants grow close together and nobody can see anything at all.

### The lantern and the flash
- The water is near-black. The Navigator sees a soft circle of `deepFogRadiusTiles` around the little fish, and **everywhere it has already swum stays lit permanently**. The map draws itself as they explore. That is the whole feature.
- **The briefing flash:** the entire ocean is lit for `deepBriefingFlashMs`, then goes dark. Plan and talk fast. Default 4000ms — **left deliberately soft pending playtest** (§16.1); the dev panel dial is the point of it.
- **The crab:** once per stage, beside one junction, a crab is sunning itself. Swim within one tile and it silently points; the corridor beyond the correct branch lights for `crabHintTiles` tiles and stays lit. Then it goes back to what it was doing. Not required, not scored, easy to miss. Deep Water spends its silent-joke slot here deliberately — the crab is worth more than a background gag.

### The big fish
Mechanically this is the dog with a longer lag: it follows the little fish's exact pixel trail at `deepPursuerLagTiles` behind, which means **it also backs up when the little fish bumps and bounces**, exactly as the dog does. The two can never collide, no collision code is needed, and the funniest possible behaviour — both fish reversing in unison out of a dead end, neither acknowledging the other — comes free.

It is **visible to the Navigator** and invisible to the Driver, which is the correct distribution of dread. It is **never a fail state**, it cannot catch you early, and nothing about it is timed. Alma will narrate it as though the world is ending, and she should.

### The plants, and an ending that cannot be avoided
The last stretch of every stage is a dense kelp corridor where the fog shrinks to `deepFogPlantsRadiusTiles`. The goal sits inside it. On arrival the ending is **scripted and identical every time**:

1. Both fish are inside the plants. The camera holds. Nothing moves but a single bubble.
2. Two seconds of stillness — long enough to be uncomfortable, which is the joke.
3. The big fish swims out, unhurried, **wearing the hat**.
4. The stats card follows.

### The score is not the hat
Because the ending is fixed, the achievement is **how much of the ocean you lit up**. Add `mapLitPct` — lit cells over total open cells — shown as *"You saw 68% of the ocean"*, with an all-time best. This reframes the world from chase to survey, removes every trace of failure pressure, and rewards precisely the skill being taught. Protect this idea in review.

### Reveal
The full ocean at last — **except the parts never seen, which stay black**. The route inked as a dotted bubble-trail, bump stars as usual. The unexplored dark is the most informative thing on the page: it is the shape of what you missed, and the reason to come back tomorrow.

### Run complete
Still water. The plants. The big fish, hatted, motionless, facing the reader. One bubble rises and the scene holds. No fanfare beyond a low chime. **World Four signs off with silence.**

### Art direction (delta from §5)
- **View:** elevation, open water, portrait. Vertical swimming is natural here, which suits the vertical-junction invariant better than any world so far.
- **Palette:** `water` #0E1A24, `waterLit` #16303C, `kelp` #2F5147, `kelpLight` #3F6A55, fish in pale ivory outline (`#E8E0CE`) with a single dot eye, and **one accent blue for the hat only** (`hatBlue` #3E6E9C, `hatBrim` #2E5578). The hat must read against near-black at a glance.
- Fog is a soft-edged mask over the static layer, not darkened tiles. No hard vignette ring.
- **Motion:** drifting only. Fish move on a slow tail sine, kelp sways on a long period, bubbles rise continuously and are the only particles. Bumps become **tangles**: the fish stops dead in the weed, everything shakes gently, a wet muffled sound rather than a thud.

### WORLDS hook table
| Hook | Value |
|---|---|
| `label` | Deep Water |
| `companion` | true, reskinned as the pursuer |
| `hasGems` | false |
| `bumpNoun` | Tangles |
| `bumpDry` | true, wetter variant of the dry bonk |
| `dust` / `burst` | `#2F5147` / `['#2F5147','#3F6A55']` |
| `speed` | `speedWithNotch(CONFIG.swimSpeed)` |
| `stageFor` | standard generator, mixed junctions, plus kelp tail, crab and hat placement |
| `topStrip` / `bottomStrip` | 0 / 0 (open water to both edges) |
| `drawExtras` | kelp, bubbles, crab, pursuer, fog mask |
| `drawExitMarker` | the hat, faintly lit, visible only within fog range |
| `drawTrail` | dotted bubble trail plus the unexplored dark |
| `drawAvatar` | the little fish |
| `rewardLine` | "It was a good hat." |
| `reward` | the still-water scene |

---

## §14 Engine work required

Sequenced for Claude Code. Nothing here changes the input model, the junction invariant, the chord, the bump economy, the streak system, the daily seed, the journal shape, or the CONFIG discipline.

**Shared, build first**
1. **Progression module** — `daysPlayed` with journal-derived migration, threshold table, unlock state.
2. **Par and the golden night** — per-run par from the optimal route, bump-free check, once-per-day cap, gold pip celebration. Par never surfaces during play.
3. **The hat rack** — render, pip fill, first-appearance backfill at World Two, unlock ceremony. Shared by the run-complete card and the picker.
4. **Playtest mode** — three doors, write suppression, the gold-moon tell.
5. **N-world picker** — the two-way picker cannot express four worlds. **Green steps left along the rack, red steps right, chord goes there.** Locked pegs are skipped, not selectable. No new touch targets, no menus, and it teaches the chord to anyone who forgot it.

**World Three**
6. **Multi-leg stages** — a `legs` concept: leg 1 goal, cinematic beat, leg 2 start/heading/goal. Cheap, because leg 2 is the same maze with start and exit swapped.
7. **Full-screen beat** — the red page, reduced-motion safe, non-strobing.
8. **Bubble overlay and polite turnaround** — pictogram bubbles with a words toggle, and a bounce variant that skips the bonk and doesn't count a bump.
9. **Two-weight reveal trace.**

**World Four**
10. **Fog mask and seen-set** — per-stage lit-cell set, soft-edged mask, `mapLitPct` stat and its all-time best.
11. **Briefing flash** — timed full reveal, then dark, plan trace live throughout.
12. **Pursuer** — reskin of the crumb-follow dog, longer lag, no collision logic.
13. **Scripted ending** — hold, emerge, hat.

**Both**
14. **Journal thumbnails** — the Forest draws both legs; Deep Water draws only what was lit, so the keepsake is a different shape every day. Journal, streak, seed and bests stay shared. One streak, one journal, four worlds.

---

## §15 CONFIG additions

```js
// progression & the rack
worldUnlockDays: { tunnels:0, hat:5, forest:10, deep:15 },
rackPipThreshold: 7,          // show countable sleeps when this few remain
rackShowsFromWorldTwo: true,  // no rack until the desert is reached
rackEnabled: true,

// the golden night
goldenNightEnabled: true,
parSlack: 1.25,               // multiplier on pure travel time
junctionThinkMs: 12000,       // free deliberation per junction — deliberately generous
goldenNightPerDay: 1,         // first qualifying run each calendar day only

// playtest (Marty only)
unlockAll: false,             // URL ?unlockAll=1, dev panel, or the 4-3-2-1 rack gesture
playtestSuppressesWrites: true,

// World Three — The Forest
forestWalkSpeed: 0.9,
forestReturnSpeedMult: 1.4,
forestRedPageMs: 1400,
forestAnimalCount: 4,
forestSpeechMode: 'pictures', // 'pictures' | 'words'
speakerLineMs: 1800,
forestShowRabbitOnReturn: false,
forestHintAfterBumps: 3,
planTraceOnReturnLeg: true,

// World Four — Deep Water
swimSpeed: 0.85,
deepFogRadiusTiles: 2.2,
deepFogPlantsRadiusTiles: 1.2,
deepBriefingFlashMs: 4000,    // playtest this one
deepPursuerLagTiles: 3.5,
deepShowPursuer: true,
crabHintTiles: 4,
deepMapLitStat: true,
```

Existing rule stands: no magic numbers in gameplay code.

---

## §16 Still open

1. **Deep Water's briefing flash** — 4000ms is a placeholder for your playtest. Try 2000 / 4000 / 6000 from the dev panel and tell me which one made Alma lean in rather than freeze.
2. **Par feel** — `parSlack` 1.25 and `junctionThinkMs` 12000 are calculated to make a careful pair comfortably safe and a stalling pair not. Worth watching on a real run: if a golden night ever feels like it rewarded rushing, `junctionThinkMs` goes up, not down.
3. **`forestHintAfterBumps`** — shipping at 3. Revisit after the first run back.
4. **Whether the squirrel beat stays** — written in, easy to cut. You'll know when you see it drawn.
