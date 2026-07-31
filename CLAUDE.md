# TUNNELS — Cooperative Blind-Digging Maze Game

A two-player asymmetric HTML5 game for one iPhone. The adult digs through underground tunnels with their **eyes closed**, steered only by the child's voice. The child reads the screen and the map. Built for Alma (child, Navigator) and a grown-up (Driver). One shared device, one shared laugh.

This file is the source of truth for Claude Code. Build exactly to this spec unless a constraint is physically impossible — flag conflicts, don't silently redesign.

**Changelog vs previous CLAUDE.md (this revision — NOT yet built, see §17):**
- **Three worlds are unlocked from the very first boot.** Tunnels, The Hat and The Forest all sit on the rack in full colour on day one. Only **Deep Water** must be earned. The staged 5/10/15 ladder is retired. Rationale and consequences in §11.
- **Deep Water is gated on continuous nights, not cumulative ones.** New currency: `tunnels:streakBest` — the longest unbroken run of nights the pair has ever managed. Monotonic (a broken streak never pushes the hat further away), so §11's "progress never goes backwards" principle survives intact. Default threshold **7**. `deepUnlockMode` flips back to cumulative days played if wanted.
- **World One is substantially harder — and only World One.** Junctions per stage 2/4/6 → **4/6/8**, deep arms 1/2/3 → **3/5/7**, width 9 → **11 columns**, plus the real new branching lever: **double-armed crossings** (`doubleArmShare`), chord junctions that grow a dead arm on *both* sides. `digSpeed` rises to 1.3 to hold the run inside its 5–10 minute envelope. See §3.1 and §8.
- **Generation parameters become per-world (`genProfiles`).** Today `effectiveGenParams()` is shared by all four worlds, so raising `stageJunctions` would silently harden the desert, the forest and the ocean too. The profile table isolates them. This is a prerequisite for the difficulty change, not an optional tidy-up.
- **Braided loops are NOT in this revision.** They stay a v3 item. The reason is in §3.1 — the current generator guarantees the screen-direction invariant by building a tree, and loops break that guarantee in a way that can put a junction in front of a horizontally-travelling digger, where green stops meaning screen-left.
- **Picker simplification.** With three worlds lit at boot the picker is *always* the N-world stepper. The single-world branch (specced, never built) and the two-world direct green/red pick are both unreachable and are **cut**. The picker still opens at every boot.
- **The rack shows from the first run-complete card.** `rackShowsFromWorldTwo` is retired; the rack is visible everywhere it appears, from day one. Three hats and one silhouette is the picture on night one.
- **The unlock ceremony now fires exactly once** in the life of the game — Deep Water, *"This one is not ours."*
- Carried forward unchanged: the two-button input model, the screen-direction invariant, the chord, the bump economy, the shared streak/journal/seed, the homage rules, and the tone rules. **The plan trace stays cut.**

---

## 1. The Two Roles

### Driver (adult — eyes closed)
- Holds the phone in portrait, thumbs resting on two large fixed buttons.
- **Never opens eyes during a stage.** Eyes open only at checkpoints and stage completion.
- Only inputs: LEFT button and RIGHT button. Nothing else, ever, in any version, **in any world**.
- Experiences the game through sound, haptics, and Alma's voice.

### Navigator (child — eyes open)
- Watches the screen: sees the world, the character, and (from v2) a map panel.
- Gives verbal directions using the colour vocabulary (below).
- Cannot touch the screen at any point. Her controller is her voice, always — there is no sanctioned Navigator touch in any phase.

### Colour vocabulary (core mechanic — non-negotiable, shared by every world)
- Directions are **screen directions** — exactly what the Navigator sees on the map, never relative to the character. A 4.5-year-old must never have to mentally rotate ("his left or my left?"). What Alma sees is what the buttons do.
- LEFT = **GREEN** = move toward the screen's left. RIGHT = **RED** = move toward the screen's right ("Red" and "Right" share the R — the mnemonic is deliberate).
- **BOTH BUTTONS TOGETHER = STRAIGHT ON** — keep going the way the character is already heading. Alma calls "both!" (or "straight!", or "keep going!" — all socially valid; the game never punishes vocabulary). The chord is also the ready/continue signal between stages, so the pair learns it before they ever need it underground. A single press waits a short grace window (`chordWindowMs`) for its partner before committing, so chords never misfire as turns.
- **Driver playtest note:** with eyes closed, "keep going!" is a common child phrasing for the chord — the briefing card should teach the Driver this equivalence.
- The two on-screen buttons are large solid blocks of these colours, fixed to the bottom-left and bottom-right corners so the Driver's thumbs never hunt. Minimum touch target: bottom 22% of screen height, each button 50% of screen width. They are the only touch targets during play.
- Colours must be a settings value, not hard-coded (see §8) — if green/red proves confusing or inaccessible we may swap to blue/orange.

---

## 2. Core Loop (one stage)

1. **Briefing (eyes open):** Both players see the stage map. They talk through the route, agree on a plan, giggle. Driver taps both buttons simultaneously to confirm "eyes closing now" — this is the ready signal and starts the stage.
2. **Digging (eyes closed):** The digger auto-digs forward at constant speed. At each junction the digger **pauses** and an audio cue plays (a soft "hm?" plus a haptic tick). The Driver presses GREEN (screen-left), RED (screen-right), or BOTH TOGETHER (straight on) as directed, and the digger waits until a press arrives (v1). With timed windows (v3+), no press = digger continues straight if straight exists, otherwise bumps. **Every accepted junction press gets an immediate acknowledgement tick + haptic (v1.1)** — an eyes-closed Driver must never wonder whether the game heard them.
3. **Bumps:** Hitting rock/roots/dead-end soil = a comedy event, not a failure. Screen shake, dust puff, a silly *thud-boing* sound, strong haptic buzz, the digger's helmet slips over its eyes. The digger **bounces back to the last junction** and re-pauses. Bumps are counted but never end a run.
4. **Breakthrough (stage complete):** The digger breaks through into a cavern or up through the surface — a soft, satisfying collapse of dirt. Fanfare, haptic celebration. On-screen banner: "OPEN YOUR EYES!"
5. **Reveal (eyes open):** The full maze is shown with the actual path traced in a contrasting line, **bump locations marked with little stars/bruise icons** — this is the laugh-together moment. Stats: time, bumps, personal best comparison. Both players tap their button together to continue.

A **run** = 3 stages (easy → medium → spicy). A run takes roughly 5–10 minutes total. **Short is a feature, and it is a hard constraint** — every difficulty change must be checked against it (see §3).

---

## 3. Movement Model & Complexity Scaling (two buttons only, forever)

The digger auto-moves. The Driver only ever chooses at junctions. Complexity therefore scales through **maze topology and timing**, never through added inputs:

**Junction authoring rule (screen-direction invariant):** junctions may only occur while the character is travelling **vertically on screen** (heading up or down), so the possible answers at any pause are always screen-left (green), screen-right (red), or straight on (both). Horizontal corridors contain corners only — the character auto-turns, no choice, no pause. This keeps every call Alma makes a plain "green / red / both" with zero ambiguity. Generators must respect this invariant. **This invariant applies identically in every world.**

**The digger is always drawn upright.** It faces its travel direction by horizontal mirroring only — never rotation. An upside-down digger reads as nonsense to a small Navigator.

| Lever | Easy | Harder |
|---|---|---|
| Junctions per stage | 4 | 14 (hard ceiling) |
| Junction types | Side-to-side T's (green/red only) | Crossings where **straight on (the chord)** is the answer, or a tempting shaft |
| Dead ends | 3, at least 2 tiles deep | Several, deep, with tempting gems inside |
| Junction pause window | Generous (digger waits indefinitely, v1) | Timed window (press within N seconds or go straight/bump) |
| Dig speed | Steady | Faster between junctions |
| Loops | None | Tunnels that circle back — Navigator must recognise "we've been here" |
| Soft-soil chutes | None | One-way drops the digger slides down; no takebacks, forces route planning |
| Gems | On the main path | Just off the path in dead ends — a deliberate detour risk/reward decision the pair makes together |

### 3.1 World One's difficulty baseline (**this revision — raised**)

World One was the gentlest world in the game because it was the only one available on night one. It isn't any more — the Hat and the Forest now sit beside it from the first boot, and variety carries the novelty that the unlock ladder used to. Tunnels can therefore be the world that *bites*.

**How the existing generator actually works (read this before changing numbers).** `tryCarve()` builds the maze as a descending spine. Each junction grows **exactly one** dead-end arm, and `canCarve()`'s adjacency rule — a new cell may touch only its predecessor — guarantees the corridor graph is a **tree**. `validateStage()` then re-checks that independently: it rejects any loop, rejects any cell of degree > 3, and requires `dead-end count === junction count`. Every difficulty change below is expressed in those terms, because a change the validator rejects is a change that silently falls back to the hand-authored fixtures every single day.

**New baseline (see §8 for the values):**
- **Junctions per stage: 4 / 6 / 8** (was 2 / 4 / 6). Not higher — see the legibility ceiling below.
- **Deep arms: 3 / 5 / 7** (was 1 / 2 / 3). Note the semantics: `stageDeadEnds` is *not* a count of dead ends (that always equals the junction count). It is how many of those arms are **deep** — 2–3 tiles rather than a one-cell stub. Raising it is the cheapest real difficulty gain in the file: a stub bumps you immediately, a deep arm makes you commit before it does. The generator clamps it with `Math.min(deepTarget, J)`, so it can never exceed the junction count.
- **Width: 11 columns** (was 9). T-junctions fail mostly from column crowding — `doT()` already tries the roomier side first — so extra width raises the generator's success rate as well as the branching.
- **Double-armed crossings (`doubleArmShare`, new — the real "more branches" lever).** A chord junction currently grows an arm on one side only, giving a degree-3 T-shape. Growing an arm on **both** sides makes a genuine 4-way crossing: degree 4, three live answers, and an extra dead end — **at no cost in rows**. This is the change that makes the maze feel branchier rather than just longer. It needs two small edits: `doChord()` grows both arms when the share calls for it, and `validateStage()` stops rejecting degree 4 (details in §17.4).
- **`stageChordShare` is left alone at `[0, 0.4, 0.5]`.** It already exists and is already tuned. Stage 1's zero is deliberate — the teaching stage stays green/red-only by construction — and raising stages 2 and 3 above 0.5 starts to make the spine monotonous. Do not touch it.
- **Gems: 3, with an 0.8 dead-end bias.** More temptation off the path, so the detour conversation happens most stages.

**Rows are the constraint, and rows are the screen.** The generator needs roughly `4 + 1.75 × J` rows to place its junctions, so junction count and grid height are the same dial. But `cs` (cell size) is `min(W/cols, (H − strips)/rows)`, so more rows means smaller cells — and on a phone the maze must stay legible to a pre-reader sitting beside an adult. At J=8 the row budget lands near 22, which is about a 26px cell on a modern iPhone. That is the floor. **Legibility, not the generator, is what caps this world's difficulty** — which is exactly why the branching comes from double-armed crossings and deeper arms rather than from ever-longer spines.

Replace the fixed `mazeRows` with a per-stage budget: `rows = max(profile.rows, 6 + 2 × J)`. This also fixes a latent bug — `effectiveGenParams()` currently adds rows only for junctions the *streak ramp* added, not for a raised baseline, so raising `stageJunctions` alone would starve the generator and fall back to fixtures every day.

**Difficulty changes must not leak into the other three worlds.** `hatStageFor()`, `deepStageFor()` and (via `effectiveForestParams()`) `forestStageFor()` all call the same `effectiveGenParams()`. Raising `stageJunctions` today raises it everywhere. Introduce `genProfiles` (§8) keyed by world, with Tunnels holding the new values and the other three holding today's. `effectiveGenParams(stageIdx, world)` reads the profile; the Forest keeps its `chordShare: 1` override.

**The time budget is a constraint, not a consequence.** Doubling the junction count without touching speed pushes a run past ten minutes and breaks §2. `digSpeed` rises **1.0 → 1.3** in the same change. Difficulty comes from topology; duration stays put. After the first real session, measure a full run: if it lands outside 5–10 minutes, `digSpeed` is the dial, not `stageJunctions`.

**Par scales automatically.** `stageParMs()` derives par from route tiles and junction count, and reads `G.digSpeed`, so a harder, faster maze earns a correctly-scaled par and the golden night stays as reachable as it was. Do not hand-tune par to compensate.

**Braided loops are deferred to v3, and here is why.** In a tree there is exactly one way to arrive at any cell, which is what lets the generator and validator guarantee that every junction is entered while travelling vertically. Add a loop and a cell can be reached from several directions — including horizontally. `arriveAt()` doesn't care how you got there: it counts open neighbours and calls `enterJunction()`. If the digger enters a junction travelling right, `turnLeft()` returns *up*, so GREEN would move the digger up-screen. That breaks §1's non-negotiable rule in the one place a blindfolded Driver can't detect it. Loops are still worth building, but they need a real arrival-direction validator, not a post-pass — a separate piece of work, not a config value.

**The streak ramp needs a ceiling.** `effectiveGenParams()` nudges junction count up from the new, higher baseline. `maxJunctionsPerStage: 12` clamps the result after every ramp calculation, and the row budget scales with it. If the generator still can't place them within `genAttempts` it falls back — never crashes, never silently drops the invariant.

**The hand-authored fallback mazes stay exactly as they are.** They are now noticeably easier than a generated stage. That is correct and deliberate: the fallback exists so a bad night still produces a finished run, and a bad night is precisely when you want the easy maze.

**The Navigator's information also scales** (this is the real difficulty axis):
- v1: Navigator sees the digger's live position on the full maze. Builds trust and vocabulary.
- v2: "Map mode" stages — Navigator sees the maze map but the digger's position marker updates only at junctions.
- v3: "Memory mode" — position marker disappears after the briefing; Alma tracks the digger mentally. This is the deepest spatial-reasoning workout and is unlocked, never forced.
- **Perspective scaling** — World Two shifts the whole world from cross-section (elevation view) to bird's-eye (plan view). Same grammar, new mental model. See §7.

**The spatial-learning ladder this game is climbing (design intent, keep in mind for every feature):**
1. Colour-direction vocabulary and screen left/right (World One)
2. Plan-then-execute: agree a route aloud at the briefing before moving, then see how it went at the reveal
3. Delayed and withheld position information (v2 map mode, v3 memory mode)
4. Perspective shift: plan view of a territory you're navigating (World Two)
5. Landmark-based navigation language: "green at the cactus" (World Two)
6. Route reversal: going back the way you came (World Three)
7. Building a map from partial views (World Four)

**Note on the ladder after this revision:** rungs 1, 4, 5 and 6 are now all available on the first night. The ladder is therefore a **suggested reading order, not a gate** — it still describes what each world teaches, and it still governs what each world's difficulty ramp assumes, but it no longer controls what the pair can reach. Two safeguards keep this honest: the picker highlights **Tunnels** by default on the very first boot (§14.5), and the Forest's own ramp keys off `daysPlayed` as well as streak so a day-one Forest run is its gentlest possible version (§12).

---

## 4. Maze Completion & Progression

- **Stage complete:** breakthrough animation → reveal → stats → next stage.
- **Run complete (3 stages):** the digger surfaces into the backyard at sunset. The reward scene: **a pitcher of chocolate milk and a plate of animal biscuits** rises onto the screen — the traditional after-digging snack, and the game's way of saying the ritual is over for today. Journal entry is stamped automatically: date, total time, total bumps, gems, a tiny thumbnail of the final maze with the path drawn. Streak counter increments.
- **Daily seed:** each calendar day generates its mazes from a date-based seed — everyone gets the same day's tunnels, replayable for personal best.
- **Difficulty over time:** difficulty is driven by **streak, gently, with a cap** — not by raw calendar. Streak 1–3: baseline. Each streak milestone nudges one lever (a junction here, a shorter pause window there) up to a ceiling at streak ~14, and in every case clamped by `maxJunctionsPerStage` (12). Breaking a streak drops difficulty back two notches, never to zero. Rationale: the game must stay winnable by a pair having an off day; frustration kills the ritual.
- Personal best is stored **per daily seed** and as an all-time "smoothest run" (fewest bumps) and "fastest run".
- Streaks, seeds, journal, and bests are **shared across worlds** — every world's run counts toward the ritual. One streak, one journal, four worlds.

---

## 5. Art Direction — picture-book cross-section

Inspired by the flat, deadpan, earth-toned cross-section style of *Sam and Dave Dig a Hole* — an **homage in spirit, all original assets**. Do not copy characters, compositions, or trade dress.

- **View:** side-on cross-section of the earth, like the page of a picture book. A thin strip of pale sky at the very top with a small house, a bare tree, maybe washing on a line. Everything below is soil.
- **The cat:** from stage 2 onward a small cat sits on the roof of the house, watching. It never does anything except flick its tail. It is never mentioned. Navigators notice it around the second or third run; that moment belongs to them.
- **Spectacular buried treasure (decorative):** clusters of gems and little hoards of gold coins sit in the solid earth *agonisingly close* to the tunnels — visible to the Navigator with a soft sparkle, never collectible, never acknowledged by the game. The digger walks straight past a fortune, every stage, forever. That's the joke. (Collectible gems are a separate mechanic; these decorative ones stay regardless, because the digger never gets better at this.)
- **Palette (tokens, tune in settings):**
  - `soil-deep` #4A3728 (undug earth, matte)
  - `soil-warm` #6B4F35 (mid earth, subtle grain texture)
  - `tunnel` #C9A876 (dug path — visibly lighter, like exposed dry dirt)
  - `sky` #DCE3DD (pale, quiet, slightly grey-green)
  - `gem` #B23A48 (dusty red — the one saturated accent on screen)
  - `button-left` #5E7C4A (green), `button-right` #A6423A (red) — muted to sit inside the palette, still unmistakably green/red
- **Texture:** flat colour fields with a very subtle paper grain overlay. No gradients, no gloss, no outlines thicker than 1px. Rocks are soft blobs a shade darker than soil. Roots dangle from the surface. The occasional buried oddity (bone, old boot, teacup) as silent jokes in the dirt — decorative only.
- **A note on the denser maze:** a wider grid with more arms leaves less solid soil per screen. Keep `treasureCount` and the buried oddities where they are rather than scaling them up — `buildStaticTunnels()` already spaces treasure at a Manhattan distance of 3 and the joke depends on it being *rare and close*, not wallpaper. If the reveal starts to look busy, thin the decorative sparkle before you thin the tunnels.
- **Characters:** small, simple, deadpan. The digger is a little figure with a hard hat and a spade, drawn in 2–3 flat colours, always upright (mirrored, never rotated). **A small dog companion** trots along the dug tunnel a step behind the digger, following its exact path — including sliding backwards after a bump. When a gem is nearby the dog's ear pricks up — a visual whisper only the Navigator sees. (The dog always knows.)
- **Motion:** minimal and dry. The dig is a steady rhythmic animation. Bumps are the biggest motion on screen. The breakthrough is dirt crumbling away in chunky flat particles. Respect `prefers-reduced-motion`.
- **Typography:** one rounded, friendly display face for banners ("OPEN YOUR EYES!") and a plain body face for stats. Sentence case everywhere. Words on screen are for the Navigator — keep them short enough for an early reader: "Ready?", "Go!", "Found a gem!", "You made it!"
- **Signature element:** the eyes-open **reveal page** — the whole maze rendered like a finished picture-book spread with the pair's wobbly path inked through it and bump-stars marking every thud. This is the screenshot-worthy moment; polish it hardest.

---

## 6. Version Roadmap

### v1 — Prove the ritual (built)
- One run = 3 stages, fixed hand-authored mazes, full colour-button control scheme incl. the chord, untimed junction pauses, bumps with sound/haptics/shake, breakthrough, reveal with path trace and bump stars, stats, dog, cat, decorative treasure, snack scene, live digger position, ready/continue chord, localStorage bests, dev settings panel.

### v1.1 — Blindfold polish (built)
- **Press acknowledgement:** every accepted junction press produces an immediate short tick sound + light haptic, before movement resumes. Silence after an input is intolerable with eyes closed. Bumps keep their own (louder, sillier) feedback; the ack fires on *any* accepted press, including ones that will bump — it confirms "heard you", not "correct".
- **Per-button haptic signatures:** on press-down, GREEN buzzes one short pulse, RED buzzes two. The Driver learns and verifies the buttons by feel, without ever opening her eyes to check her grip. Configurable in CONFIG (`hapticSigLeft`, `hapticSigRight`), degrade silently where `navigator.vibrate` is unavailable.
- **Wake-lock coverage:** request the wake lock at briefing (not just stage start) and re-request on visibility change in all non-terminal phases.
- **Do Not Disturb nudge:** the first briefing of a session shows one quiet line: "Tip: turn on Do Not Disturb — the Driver can't see notifications coming."

### v2 — Make it a ritual (built, except the map panel)
- **(built)** Procedural maze generation from daily date seed, 3-stage ramp. Validator-first, with the v1 hand-authored mazes kept permanently as the never-broken fallback.
- **(built)** Streak system + streak-driven difficulty (capped, as §4).
- **(built)** Gems + dead-end temptations + the dog's gem-sense (the ear prick). World One only.
- **(built)** Journal: stamped entries per completed run. Shared across worlds. localStorage array, capped at ~200 and quota-safe (trims oldest first).
- *pending* — Map panel for the Navigator alongside the live view.
- *cut* — **The plan trace.** The plan-then-execute beat lives in the spoken briefing instead, and the two buttons remain the only touch targets in every phase.

### v2.5 — World Two: The Hat (built — see §7)
### v2.6 — The Hat Rack (built — see §11)
### v2.7 — World Three: The Forest (built — see §12)
### v2.8 — World Four: Deep Water (built — see §13)

### v2.9 — Open the rack, harden the tunnels (**this revision, not yet built — see §17**)
- **Three worlds unlocked at boot.** Tunnels, The Hat, The Forest all available from the first session; Deep Water alone is earned.
- **Deep Water gated on best-ever streak** (`streakBest`, default 7), with `deepUnlockMode:'days'` as the cumulative alternative.
- **World One difficulty raised** (§3.1): junctions 4/6/8, deep arms 3/5/7, 11 columns, double-armed crossings, a per-stage row budget, `digSpeed` 1.3, `maxJunctionsPerStage` clamp.
- **Generation parameters split per world** (`genProfiles`) so the raise touches Tunnels only.
- **Picker reduced to the stepper only**; single-world and two-world branches cut.
- **Rack visible everywhere from day one**; `rackShowsFromWorldTwo` retired.

### v3 — Deepen the learning
- Map mode and Memory mode stages (Navigator information scaling, §3) — across the worlds.
- Timed junction windows as an unlockable "spicy" modifier.
- Soft-soil chutes in tunnel generation. (**Loops have moved forward into v2.9** — see §3.1.)
- Sound design pass: each junction type gets its own audio cue so the Driver starts learning the maze by ear — a quiet second literacy.
- **Line-of-sight mode (World Two spicy variant):** the Navigator sees only what the watcher tortoise can see from its rock.

### v4 — Stretch (only if the ritual has stuck)
- **Role swap:** ultra-simple mazes where Alma drives eyes-closed and the adult navigates using the colour words. The full-circle moment.
- Two-device mode (Navigator on a second screen with map only).
- Maze editor: draw a maze with your finger, save it for tomorrow's partner.

---

## 7. World Two — THE HAT (top-down overworld)

An homage in spirit to Jon Klassen's *We Found a Hat* — **all original assets**. Do not copy Klassen's tortoise designs, compositions, or trade dress. Our tortoises are their own tortoises. The debt we're honouring is the deadpan, the patience, and the desert light.

### Why this world exists (design intent)
World One teaches directions in a **cross-section (elevation) view**. World Two moves the same grammar to a **bird's-eye (plan) view** — the perspective real maps use. The Navigator learns that a flat picture seen from above corresponds to a territory a character is inside. This, plus landmark language, is the second rung of map literacy. Nothing else changes: same two buttons, same colours, same chord, same roles, same ritual.

### The fiction
Two tortoises in a scrubby desert. The **Watcher** (Alma's avatar) sits on a tall rock at the bottom of the screen, looking out over the land. The **Seeker** (the Driver's avatar) has its eyes closed — tortoises are very good at walking with their eyes closed, everyone knows this — and plods through the scrub. At the far end of the maze, in plain view the whole time, sits **a hat**. The Watcher calls directions; the Seeker walks. The Seeker never sees the hat until it's wearing it.

### View & world
- **Bird's-eye, portrait.** The Watcher's rock at the bottom of the play area (just above the buttons), the hat near the top. The maze is walkable sand paths between impassable scrub: bushes, boulders, prickly pear, dry logs, dune humps.
- The **hat is visible from the very first frame** of the briefing. A visible goal changes the planning conversation: Alma routes *toward something*, not just *through something*.
- **Landmarks are first-class citizens.** Each stage places 3–5 distinct, nameable landmarks. Generation must place every junction within one tile of a landmark, so a call like "green at the cactus!" is always possible. The reveal labels landmarks in tiny sentence-case text.

### Rules (all inherited, restated for clarity)
- The Seeker auto-walks, tortoise-slow. Slow is the joke, and the pace suits a plan view where the Navigator is reading further ahead. **World Two keeps `walkSpeed` at 0.8 — the World One speed rise does not apply here.** The tortoise is slow on purpose and its mazes did not get bigger.
- **The screen-direction invariant holds unchanged.**
- The Seeker is drawn from above and **rotates in 90° steps to face its heading** — in plan view rotation is how the world works (the upright-only rule is an elevation-view rule). Its walk is a tiny deadpan waddle.
- **Bumps become shell-bonks:** walking into a bush/boulder = *bonk*, the Seeker pulls into its shell, comedy pause, then backs up along its own footprints to the last junction.
- Junction pause, chord grace window, re-cue, press acknowledgement, timers, stats: identical systems, shared code.

### Companions & silent jokes
- **A small lizard** suns itself on a rock somewhere in each stage. It never moves except to blink. Never mentioned. (The cat's cousin.)
- **A second hat**, half-buried in sand in a corner of the map, never acknowledged, never collectible.
- The Watcher tortoise on its rock slowly turns its head to keep facing the Seeker throughout the stage — the only "camera" in the game, and it's a tortoise.

### Reveal & run completion
- The reveal spread shows the whole desert from above with the Seeker's **footprint trail** dotted through the sand and shell-bonk stars at every bonk.
- **Run complete:** the Seeker reaches the hat and puts it on. Final scene (**all original art**): a black sky full of stars, the two tortoises side by side, each wearing a hat, drifting off among them. Nobody says anything. The **chocolate milk and animal biscuits** snack is World One's sign-off; the stars are the Hat's.

### Art direction (delta from §5)
- Palette shifts warm and pale: sand, sage-green scrub, long shadows. The hat is the focal object — a **tan cowboy hat** (`hatTan` #C2A06A, `hatCrown` #A88752, `hatShade` #8E6F42), matte and flat, all clearly darker than the sand so it reads instantly (wide flat brim, rounded dented crown, a centre crease — never a red disc). `gem` #B23A48 stays World One's one saturated accent.
- Same flat colour fields, paper grain, no gradients, 1px max outlines. Shadows are single flat shapes cast consistently to one side.
- Run-complete star scene rendered side-on (the one elevation shot in this world).

### World picker
- The picker opens at **every boot**. It is the hat rack (§11, §14.5).
- **World Two is unlocked from the first boot** (`worldUnlockDays.hat: 0`). The old `hatUnlockRuns` run-count gate and the later 5-days gate are both retired; `hatUnlockRuns` may be deleted from CONFIG entirely.
- Navigation is the stepper in every case: **green steps left, red steps right, the chord goes there.** Locked pegs are skipped.

---

## 8. Dev Settings & Tuning Panel

All tunables live in a single `CONFIG` object at the top of the file, persisted to localStorage, editable at runtime via a hidden dev panel (**tap the sky 5 times**). Every value overridable by URL param (`?digSpeed=1.4`). Include a "reset to defaults" and an "export config as JSON" button.

```js
const CONFIG = {
  // movement
  digSpeed: 1.3,            // CHANGED 1.0 → 1.3 — buys back the time the harder mazes cost (§3.1)
  walkSpeed: 0.8,           // World Two — tortoise-slow is the joke, unchanged
  junctionPauseMs: 0,       // 0 = wait forever (default); >0 = timed window
  bumpBouncePx: 24,         // comedy bounce distance
  bumpStunMs: 900,          // pause after bump/bonk before re-offering the junction
  // maze generation — NOW PER WORLD (§3.1). stageJunctions / stageDeadEnds /
  // stageChordShare / mazeCols / mazeRows stay as the shared defaults for
  // back-compat and the dev panel, but effectiveGenParams(stageIdx, world)
  // reads genProfiles first. Note stageDeadEnds is how many arms are DEEP
  // (2-3 tiles) — the dead-end COUNT always equals the junction count.
  genProfiles: {
    // World One: raised. Everything else holds today's values.
    tunnels: { junctions:[4,6,8], deepArms:[3,5,7], chordShare:[0,0.4,0.5],
               doubleArmShare:[0,0.30,0.45], cols:11, rows:14 },
    hat:     { junctions:[2,4,6], deepArms:[1,2,3], chordShare:[0,0.4,0.5],
               doubleArmShare:[0,0,0],       cols:9,  rows:12 },
    forest:  { junctions:[2,4,6], deepArms:[1,2,3], chordShare:[1,1,1],
               doubleArmShare:[0,0,0],       cols:9,  rows:12 },
    deep:    { junctions:[2,4,6], deepArms:[1,2,3], chordShare:[0,0.4,0.5],
               doubleArmShare:[0,0,0],       cols:9,  rows:12 },
  },
  stageJunctions: [2, 4, 6],    // shared default / fallback if a profile is missing
  stageDeadEnds: [1, 2, 3],
  stageChordShare: [0, 0.4, 0.5], // UNCHANGED — already tuned; stage 1's zero is deliberate
  mazeCols: 9, mazeRows: 12,
  rowBudgetBase: 6,             // NEW — rows = max(profile.rows, rowBudgetBase + 2×J)
  maxJunctionsPerStage: 12,     // NEW — hard clamp applied after the streak ramp
  genAttempts: 120,             // NEW — extracted from the hard-coded retry loop in generateStageRows
  gemCount: 3, gemDeadEndBias: 0.8, // CHANGED from 2 / 0.7 — more temptation off the path
  // overworld generation (World Two)
  landmarkCount: 4,
  landmarkJunctionRadius: 1,
  // difficulty ramp
  streakDifficultyStep: 3,  // every N streak days, nudge one lever
  streakDifficultyCap: 14,
  // controls & accessibility
  colourLeft: '#5E7C4A', colourRight: '#A6423A',
  buttonHeightPct: 22,
  chordWindowMs: 250,
  hapticsEnabled: true, audioVolume: 0.8,
  pressAckEnabled: true,
  hapticSigLeft: [20],
  hapticSigRight: [20, 60, 20],
  // companions & silent jokes
  showDog: true, dogLagTiles: 0.85,
  showCat: true,
  showLizard: true,
  treasureCount: 3,         // hold at 3 despite the bigger grid — rare and close is the joke (§5)
  buriedSecondHat: true,
  // navigator information
  showLiveDigger: true,
  positionUpdateAtJunctionsOnly: false,
  lineOfSight: false,
  // progression & the hat rack (§11) — LADDER REWRITTEN
  worldUnlockDays: { tunnels:0, hat:0, forest:0, deep:0 }, // CHANGED — days-played thresholds, now all 0
  deepUnlockMode: 'streak', // NEW — 'streak' (best-ever unbroken nights) | 'days' (cumulative)
  deepUnlockStreak: 7,      // NEW — best-ever streak required when deepUnlockMode === 'streak'
  deepUnlockDays: 10,       // NEW — cumulative days required when deepUnlockMode === 'days'
  rackPipThreshold: 10,     // CHANGED from 7 — the count is the whole progression now, show it early
  rackEnabled: true,
  pickerAlwaysAtBoot: true,
  // REMOVED: hatUnlockRuns, rackShowsFromWorldTwo — both retired (§11)
  // the golden night (§11) — bump-free AND under par, once per calendar day
  goldenNightEnabled: true,
  parSlack: 1.25,           // multiplier on pure travel time
  junctionThinkMs: 12000,   // free deliberation per junction — deliberately generous
  goldenNightPerDay: 1,
  // playtest (Marty only) — URL ?unlockAll=1, dev panel, or the 4-3-2-1 rack gesture
  unlockAll: false,
  playtestSuppressesWrites: true,
  // World Three — The Forest (§12)
  forestWalkSpeed: 0.9,
  forestReturnSpeedMult: 1.4,
  forestRedPageMs: 1400,
  forestAnimalCount: 4,
  forestSpeechMode: 'pictures',  // 'pictures' | 'words' (NO / OK)
  speakerLineMs: 1800,
  forestShowRabbitOnReturn: false,
  forestHintAfterBumps: 3,
  forestEasyUntilDays: 3,        // NEW — day-one Forest runs use the gentlest spine (§12)
  // World Four — Deep Water (§13)
  swimSpeed: 0.85,
  deepFogRadiusTiles: 2.2,
  deepFogPlantsRadiusTiles: 1.2,
  deepBriefingFlashMs: 4000,
  deepPursuerLagTiles: 3.5,
  deepShowPursuer: true,
  crabHintTiles: 4,
  deepMapLitStat: true,
  deepEndingMs: 2000,
  // debug
  debugOverlay: false,
  seedOverride: null,
};
```

Rule: **no magic numbers in gameplay code** — if it affects feel, it goes in CONFIG.

## 9. Technical Constraints

- Single self-contained `index.html` — inline CSS/JS, no build step, no external network calls. Canvas rendering for the world; DOM for buttons and panels. All four worlds live in the same file and share every system that isn't world-specific (input, junction logic, chord, timers, reveal, journal, config, audio scaffolding).
- Target: iPhone Safari, portrait, one-handed-thumbs ergonomics. Must run offline once loaded.
- Persistence: `localStorage` only (journal, bests, streak, config). Namespace keys `tunnels:*` (shared across worlds — one streak, one journal).
- Audio: Web Audio API, synthesised or tiny embedded sounds — must work after a user gesture (the ready-signal press unlocks audio).
- Haptics: `navigator.vibrate()` where available; degrade silently.
- The screen must **never sleep mid-stage**: use the Screen Wake Lock API with graceful fallback; request at briefing and re-request on visibility change in all non-terminal phases.
- Accidental exits are catastrophic mid-stage (eyes closed!): no touch targets other than the two buttons, in any phase; ignore multi-touch outside them. (The dev panel's sky-tap and the rack's peg hit-areas are the only non-button touch surfaces, and never during play.)
- **Rendering budget:** the raised World One grid is roughly double the cells of the old 9×12. The existing `staticLayer` offscreen canvas already handles this — the per-frame cost is the dug set, the avatar, the dog and particles. No change needed; just don't move static drawing into the frame loop.

## 10. Tone Rules (for every string and sound in the game)

Bumps and bonks are funny. Nothing is ever a failure. The game never says "wrong", "oops" in a scolding way, or shows a red X. Stats celebrate ("smoothest run yet!") and never shame. The Driver being helpless is the joke; the Navigator being capable is the point. The tortoises never speak. The hat is never explained.

**Tone note for the harder tunnels:** more branches means more bumps, and bumps must not accumulate into a feeling of failure. The bump counter stays a punchline on the reveal spread and never appears mid-stage. If a stage produces a lot of bumps, the copy still celebrates finishing — never "that was a rough one."

---

# The rack, the worlds, and what's next (§11–§17)

**Homage rule, non-negotiable:** all original assets, all original strings. No reproduction of Klassen's illustrations, characters, compositions, trade dress, or text. The debt we honour is the deadpan, the withheld information, and the flat colour. In-game names are ours: *The Forest*, *Deep Water* — never the book titles.

**Decisions locked (this revision):** three worlds open at boot · Deep Water gated on best-ever streak (7) · one golden-night bonus rule, unchanged · World One difficulty raised with `digSpeed` compensating · picker is the stepper only · rack visible from day one · Forest speech is pictures with a words toggle · deadpan endings kept as the books leave them · **the plan trace stays cut** (the two buttons are the only touch targets in every phase).

## 11. The Hat Rack — progression made visible (**rewritten this revision**)

**Principle (unchanged, and the reason for every decision below):** progress **never goes backwards**. Nothing earned is ever taken away, and the counter never mentions a day that wasn't played.

### 11.1 What changed and why

The rack was built as a four-rung ladder — a hat every five days — because Tunnels was the only world on night one and the other three had to be *waited for*. Opening three worlds at boot removes that job. Novelty now comes from variety on the first night, and the rack's remaining job is smaller and clearer: **there is one hat left, and here is exactly how far away it is.**

Consequences, all of them deliberate:
- The rack shows **three hats and one silhouette** from the very first boot. That is a different, better first impression than one hat and three silhouettes: it reads as a collection with a gap, not a locked door.
- The **unlock ceremony fires exactly once** in the life of the game. It should therefore be the best one — Deep Water's, unchanged: *"This one is not ours."*
- The **first-appearance backfill** logic (the day-5 run that swung the rack into view and landed two hats at once) is dead. Delete it.
- The **pip count is the whole progression**, so it shows early: `rackPipThreshold` rises 7 → 10, meaning the moons are countable from essentially the first run.

### 11.2 The currency

Two currencies exist; only one gates anything.

**`tunnels:daysPlayed`** — distinct calendar days on which a run was completed, in any world. Cumulative, monotonic, shared across all four worlds. Retained: it drives the "Day 7 of digging together" line, the Forest's gentle-start window (`forestEasyUntilDays`), and journal statistics. It no longer gates a world unless `deepUnlockMode` is set to `'days'`.

**`tunnels:streakBest`** — **NEW.** The longest unbroken run of nights the pair has ever managed. Written on every run completion as `max(streakBest, currentStreak)`, so it is **monotonic by construction**: breaking a streak never reduces it, and the last hat never moves further away. This is what "continuous days of play" means in a game whose founding principle forbids taking things back.

Migration on first load after this update: `streakBest = max(existing streakBest || 0, currentStreak || 0)`. If a longer historical streak can be reconstructed from the journal's `day` stamps, do so — a pair who already earned it should not have to earn it twice. Never write a value lower than the one already stored.

### 11.3 The unlock ladder

| Peg | World | Hat | Unlocks at |
|---|---|---|---|
| 1 | Tunnels | yellow hard hat | from the start |
| 2 | The Hat | tan cowboy hat | **from the start** |
| 3 | The Forest | red pointy hat | **from the start** |
| 4 | Deep Water | small blue bowler | **best-ever streak of 7 nights** (`deepUnlockMode:'streak'`, `deepUnlockStreak:7`) |

`deepUnlockMode:'days'` switches peg 4 to `deepUnlockDays` (10) cumulative days played instead. Both paths must be implemented; the mode is a one-line CONFIG flip after playtest.

**Counting copy for the moons under peg 4** must stay in the §10 tone — it states what is true now, never what was missed:
- streak mode: *"three more nights in a row"* → as moons, no number needed.
- days mode: *"three more sleeps"*.
- Never: "you broke your streak", "you missed yesterday", or any restatement of a lost count. If the current streak resets, the moons under peg 4 simply refill from the current streak again — silently, with no comment anywhere in the game.

### 11.4 The golden night (unchanged)

A run that is **bump-free across all three stages and finishes under par** is worth **two nights instead of one**. Par is per run, not per stage, and derived (never authored) so it survives procedural generation and the streak speed lever:

```
parMs = Σ over the 3 stages of [ (routeTiles / effectiveSpeed) × 1000 × parSlack
                                 + junctionCount × junctionThinkMs ]
```

`parSlack` 1.25, `junctionThinkMs` 12000. **This formula needs no adjustment for the harder World One** — more junctions mean a longer par automatically, and the raised `digSpeed` feeds `effectiveSpeed`. Do not compensate by hand.

**In streak mode, the golden night keeps its existing behaviour and nothing more.** It still awards the bonus day of `daysPlayed` in `updateProgression()`, still shows its gold pips and its line, and still caps at once per calendar day. It does **not** touch the streak. Two reasons. First, adding to a streak would let one good evening fake continuity, which is the only thing a streak measures. Second — and this is the part that's easy to miss — `advanceStreak()` **already forgives a missed day**: its condition is `today - lastDay <= 2`, so the streak only breaks on two consecutive misses. A "streak shield" bonus would stack forgiveness on forgiveness, and seven continuous nights could quietly stretch across a fortnight. Leave it alone; no new state, no new mechanic.

If the golden night should count toward the last hat, that is what `deepUnlockMode: 'days'` is for — there it already does, because the bonus adds a day played.

Everything else about the golden night is unchanged: the briefing is untimed; par is **never shown during a stage** and surfaces only on the run-complete card; once per calendar day, first qualifying run only. Celebration: two gold pips, a rising two-note chime, a distinct haptic, a scatter of gold particles, and the line *"A golden night."* A small gold-moon outline always sits beside peg 4 so the pair know the bonus exists without ever being told a target time.

### 11.5 The object

Four pegs, one hat per world (every world has a hat — the joke of the whole series). Unlocked pegs hold their hat in full colour; the locked peg holds a **pale silhouette** — the shape is the tease, never a padlock. Beneath peg 4, **nights remaining as countable moons** (shown when `rackPipThreshold` 10 or fewer remain, which in practice is always).

**Where it appears:** the world picker on **every boot**, and the run-complete card on **every completed run**, from day one. `rackShowsFromWorldTwo` is retired — there is no longer a day at which the rack "arrives", so there is no gate to keep. Never mid-stage, never in the briefing, no percentages or bars.

**The unlock ceremony (Deep Water only):** the rack fills the screen (stats deferred); the blue bowler drops onto peg 4 with a distinct fanfare and a celebratory haptic; one line — *"This one is not ours."*; chord to continue; then the stats card.

**Playtest mode (Marty only):** a sealed mode unlocking all four worlds immediately, via three doors — URL `?unlockAll=1`, the dev-panel button ("Take all the hats down"), or the 4-3-2-1 rack gesture within four seconds. While on, all writes (`journal`, `daysPlayed`, `streakBest`, `streak`, `best`, `seedbest:*`, `deepBestLit`) are suppressed; reads work so the rack renders truthfully. The tell: a gold moon in the rack corner and `PLAYTEST` in the debug overlay.

## 12. World Three — The Forest (built; one addition this revision)

Homage in spirit to *I Want My Hat Back*. All original assets. **Rung six: route reversal** — going back the way you came, the operation that turns a route into a map. It also teaches that **the answer was on screen the whole time**: the rabbit wears the stolen hat from the first briefing frame; the bear never notices; Alma will, instantly, and the eyes-closed Driver can do nothing until the bear works it out himself. That gap is the world.

**Structure — two legs and a red page (one maze, traversed twice):**
- **Leg 1, the search (down-screen):** the bear enters at the top and walks down the spine; animals sit at the arm tips. Chord to keep going down, green/red to turn into an arm.
- **The red page:** arriving at the deer (leg-1 exit) floods the screen full-bleed saturated red for `forestRedPageMs` with one line — *"He knows where his hat is."* — and the longest haptic in the game. Never a strobe at any setting; reduced-motion makes it a hold.
- **Leg 2, the run back (up-screen):** the goal becomes the rabbit's arm tip, speed × `forestReturnSpeedMult`. The trodden path stays drawn but **the animals and the rabbit are gone** — Alma must remember which arm, and which side. A wrong arm bumps, bounces back, try again.

**NEW this revision — the gentle-start window.** The Forest is now reachable on night one, by a pair who may not yet have the colour vocabulary solid. For the first `forestEasyUntilDays` (3) days played, Forest stages use their **shortest spine and fewest arms** regardless of streak, and `forestHintAfterBumps` is halved. After that window the normal ramp applies. This is a floor, not a ceiling — it never makes the Forest harder, only guarantees a survivable first encounter.

**The load-bearing constraint:** Forest stages are **chord-only**. `effectiveForestParams()` sets `chordShare = 1` on top of the shared params today; under `genProfiles` (§8) the Forest profile carries `chordShare:[1,1,1]` and `doubleArmShare:[0,0,0]` — a double-armed crossing in the Forest would put an arm on both sides of the spine and break the return leg's unambiguous which-arm memory, so it must stay zero there. A chord junction leaves vertically, so the return enters vertically and the calls stay exactly chord / green / red in both legs — green is still screen-left, the world never flips. Leg 2 needs no new maze: start = deer cell heading up, exit = rabbit's arm.

**Meeting an animal** is a conversation, not a bump: the bear stops, a bubble holds for `speakerLineMs`, a soft chime marks it, then a **polite turnaround** back to the junction — no bonk, no shake, **no bump counted**. Bare arms still bump (trees aren't conversational). Bubbles are **pictograms** (`forestSpeechMode:'pictures'`, a `'words'` toggle gives NO/OK): most animals a slashed hat, the asker a hat outline with a "?", **the rabbit three slashed hats crammed in** (protesting far too much), the sleeper a "z". **The deer beat** shows an empty hat outline; the bear's bubble fills the same shape red — two identical silhouettes, the whole story in shape and colour — then the red page. **The silent joke:** one animal is always asleep and never answers.

**Reveal:** two weights — leg 1 soft grey, leg 2 charcoal ink (`#33291F`). **Run complete:** the bear sitting in his hat, deadpan; a squirrel wanders in asking about a rabbit in a hat, the bear says no and asks why he's asking; nothing is explained.

**Art:** elevation forest floor; tree-trunk walls in two greens and a grey-brown; warm ivory ground (`#EFE7D6`); one saturated red for the hat only (`#B23A48`). The bear is large, upright, mirrored not rotated, one dot eye, no expression ever.

## 13. World Four — Deep Water (built; unchanged this revision except its gate)

Homage in spirit to *This Is Not My Hat*. All original assets. **Rung seven: building a map from partial views** — every earlier world hands the Navigator a complete map at briefing; Deep Water hands her a few seconds of one, then the light goes. What she holds in her head after that is cartography.

**It is now the only earned world**, which raises the stakes on its arrival and is the reason the ceremony copy stays exactly as written.

**The lantern and the flash:** near-black water; the Navigator sees a soft circle of `deepFogRadiusTiles` around the little fish, and **everywhere it has swum stays lit permanently** — the map draws itself. The **briefing flash** lights the entire ocean for `deepBriefingFlashMs` (default 4000, a dev-panel dial — §16.1), then it goes dark; the briefing stays untimed. In the dense **kelp** at the goal the fog shrinks to `deepFogPlantsRadiusTiles`. Fog is a soft-edged mask over the static ocean, **no hard vignette**.

**The crab:** once per stage, beside one junction; swim within a tile and it points, lighting `crabHintTiles` down the correct branch, then goes back to sunning. Not required, not scored, easy to miss.

**The big fish:** mechanically the crumb-follow dog at `deepPursuerLagTiles` behind — it backs up in unison when the little fish bounces, can never collide (no collision code), and is **never a fail state**. Visible to the Navigator, invisible to the Driver — the correct distribution of dread.

**The scripted ending (cannot be avoided):** the goal sits in the kelp; on arrival both fish hold inside the plants, a `deepEndingMs` stillness, then the breakthrough — and in the run-complete scene the big fish sits, hatted, motionless, facing the reader. **World Four signs off with silence.**

**The score is the survey, not the chase:** `mapLitPct` — lit cells over open cells — shown as *"You saw N% of the ocean"* with an all-time best. **Reveal:** the route as a dotted bubble-trail with bump stars, and **the unexplored dark left exactly as it was**. The journal thumbnail draws only what was lit. Mixed junctions, its own daily-seed stream, tunnel fixtures as the never-broken fallback. Bumps are **tangles**. Palette: `water` #0E1A24, `waterLit` #16303C, `kelp` #2F5147/#3F6A55, fish pale ivory (`#E8E0CE`), one accent blue for the hat only (`hatBlue` #3E6E9C, `hatBrim` #2E5578).

## 14. Engine (built; §14.5 revised)

The engine is world-agnostic behind the `WORLDS` hooks. Nothing in this revision changes the input model, the junction invariant, the chord, the bump economy, the daily seed, the journal shape, or CONFIG discipline.

1. **Progression module** — `daysPlayed`, `streakBest` (new), journal-derived migration, threshold table, unlock state.
2. **Par & the golden night** — per-run par from the optimal route, bump-free check, once-per-day cap; par never surfaces during play. Unchanged this revision (§11.4).
3. **The hat rack** — render, pip fill, unlock ceremony; shared by the run-complete card and the picker. First-appearance backfill removed.
4. **Playtest mode** — three doors, write suppression, the gold-moon tell.
5. **The picker** — see §14.5.
6. **Multi-leg stages** (Forest).
7. **Full-screen beat** (the red page) — reduced-motion safe, non-strobing.
8. **Bubble overlay & polite turnaround.**
9. **Two-weight reveal trace** (Forest).
10. **Fog mask & seen-set** (Deep Water).
11. **Briefing flash** (Deep Water).
12. **Pursuer** (Deep Water).
13. **Scripted ending** (Deep Water).
14. **Journal thumbnails.** Journal, streak, seed and bests stay shared. One streak, one journal, four worlds.

### 14.5 The picker (**unchanged in code, reinterpreted here**)

The rack *is* the picker, and it opens at every boot. `pickerAlwaysAtBoot` already does this — picker-at-boot was listed as unbuilt in the previous CLAUDE.md but is in fact shipped.

`renderPicker()` has three branches: single-world, two-world direct green/red, and the three-or-more stepper (**green steps left, red steps right, the chord goes there**, locked pegs skipped). With three worlds unlocked from boot, **only the stepper is reachable** — `showPicker()` builds `G.pickerWorlds` from `isUnlocked(w) && WORLDS[w]`, which is now three or four entries in every non-pathological case.

**Leave the other two branches in place.** An earlier draft of this spec said to delete them. That was wrong on inspection: the branch conditions are duplicated across four call sites (`renderPicker`, `press()`, the picker case in `update()`, and the `keydown` handler), so removing them means four coordinated edits for no behavioural gain, and they are the correct fallback if a future world ever ships locked or if `genProfiles`/`worldUnlockDays` is edited in the dev panel. Dead but load-bearing as a guard.

**Default highlight:** `showPicker()` currently seeds `G.pickerIdx` from `LS.get('lastWorld')`. Add one condition — when `getDaysPlayed() === 0`, start on Tunnels regardless. On the first night, three lit hats is an invitation; landing the highlight on Tunnels is the quiet suggestion of where to start.

**No new touch targets, in any case.** The two buttons and the chord do everything. The rack's `.pegHit` areas remain the only exception, and only outside play.

## 15. CONFIG

All keys are in §8. Existing rule stands: **no magic numbers in gameplay code** — `generateStageRows()`'s hard-coded `attempt<120` and `tryCarve()`'s arm lengths are the two current violations, and `genAttempts` fixes the first.

Keys removed this revision (`hatUnlockRuns`, `rackShowsFromWorldTwo`) must be deleted from `CONFIG_DEFAULTS` *and* from every read site — no orphaned flags. Note `hatUnlockRuns` is already dead: it appears in `CONFIG_DEFAULTS` and nowhere else.

## 16. Open — playtest tuning (not code to write)

One-line CONFIG changes after a real session, not features to build:
1. **World One junction counts.** 4/6/8 is a judgement, not a measurement. Watch stage 3 on the actual phone — at J=8 the row budget puts the cell near 26px, and if the digger and dog look cramped, 4/6/7 before anything else.
2. **`doubleArmShare` 0/0.30/0.45.** The branchiness dial, and the one to move first if the maze feels too easy or too mean. It costs no rows either way.
3. **`digSpeed` 1.3.** Time a full run. Target 5–10 minutes. If it runs long, the speed goes up before the junctions come down; if the Driver feels rushed between junctions, the junctions come down instead.
4. **`stageDeadEnds` 3/5/7.** Deep arms are the cheapest difficulty in the file. If bumps start feeling punishing rather than funny, this comes down before the junction count does.
5. **`deepUnlockStreak` 7.** Seven continuous nights is ambitious for a family — and note `advanceStreak()` already forgives single missed days, so it is more reachable than it sounds. If it stalls, 5, or flip `deepUnlockMode` to `'days'`.
6. **Deep Water's briefing flash** — try 2000 / 4000 / 6000; keep whichever makes Alma lean in rather than freeze.
7. **Par feel.** If a golden night ever feels like it rewarded rushing, `junctionThinkMs` goes **up**, not down.
8. **`forestEasyUntilDays` 3** — revisit after a day-one Forest run.
9. **Whether the squirrel beat stays.** You'll know when you see it drawn.

---

## 17. Build order for Claude Code (this revision)

The whole game is one self-contained `index.html`. Work in this order; after each numbered block the game must still boot and complete a run, with no half-migrated saves. Function names below are the real ones in the file.

### 17.1 Unlock three worlds
1. `CONFIG_DEFAULTS.worldUnlockDays` → `{ tunnels:0, hat:0, forest:0, deep:0 }`. Add `deepUnlockMode: 'streak'`, `deepUnlockStreak: 7`, `deepUnlockDays: 10`.
2. Rewrite `isUnlocked(world)`: return true for `playtest`; return true for anything except `'deep'`; for `'deep'` return `deepUnlockMode==='streak' ? getStreakBest() >= CONFIG.deepUnlockStreak : getDaysPlayed() >= CONFIG.deepUnlockDays`. Keep it a pure read — it is called from `buildRackSVG()` on every render.
3. Delete `hatUnlockRuns` from `CONFIG_DEFAULTS` (it has no read sites).
4. `worldUnlockDay(world)` is still used by `buildRackSVG()` for the pip count and by `newlyUnlocked()`. Both are rewritten in 17.3 and 17.2 respectively; keep the helper until then.

### 17.2 `streakBest`, and rewriting the unlock detection
5. Add `getStreakBest()` reading `tunnels:streakBest`, falling back on first read to `max(currentStreak().count, longest run of consecutive `day` values in the journal)` — the same migration shape `getDaysPlayed()` already uses for `daysPlayed`. Never return less than the stored value.
6. In `finishRun()`, immediately after `LS.set('streak', streak)`, write `if(shouldPersist()) LS.set('streakBest', Math.max(getStreakBest(), streak.count))`. Capture the pre-write value first — `newlyUnlocked` needs the before/after pair.
7. Rewrite `newlyUnlocked(prevDays, nowDays)` as `newlyUnlocked(before, after)` taking `{days, streakBest}` objects, and have it return `['deep']` only when `isUnlocked('deep')` is newly true across that transition. The three free worlds can never appear in its output — with thresholds at 0, the existing `d > prevDays` test already returns nothing for them, so a brand-new player gets no spurious ceremony.
8. Add `streakBest` to the playtest write-suppression path (it goes through `shouldPersist()`, so this is free if you follow 6).
9. Do **not** add a streak shield or any golden-night streak interaction. See §11.4 — `advanceStreak()` already forgives one missed day, and stacking forgiveness would make "continuous" meaningless.

### 17.3 Rack and picker
10. Delete `CONFIG.rackShowsFromWorldTwo`. Simplify `rackVisible()` to `CONFIG.rackEnabled` (it is read in `finishRun()` and as the `renderPicker()` fallback; both then behave correctly from day one).
11. In `finishRun()`, delete the `firstRack` backfill block — the day-5 two-hats-at-once path and its `highlight` special case. The ceremony now only ever fires for Deep Water; its `sub` copy collapses to the single line *"This one is not ours."*
12. `rackPipThreshold` → 10. In `buildRackSVG()`, replace `remaining = worldUnlockDay(...) - days` with a mode-aware count: streak mode → `CONFIG.deepUnlockStreak - getStreakBest()`; days mode → `CONFIG.deepUnlockDays - getDaysPlayed()`. Clamp at zero. The `nextIdx` scan already resolves to peg 4 once the first three are unlocked.
13. In `showPicker()`, when `getDaysPlayed() === 0`, force `G.pickerIdx` to the Tunnels index instead of using `lastWorld`.
14. Add `streakBest` to the `drawDebug()` line beside `days:`.
15. Leave `renderPicker()`'s single-world and two-world branches alone (§14.5).

### 17.4 World One difficulty
16. Add `CONFIG.genProfiles` exactly as in §8, plus `rowBudgetBase`, `maxJunctionsPerStage`, `genAttempts`. Bump `gemCount` to 3, `gemDeadEndBias` to 0.8, `digSpeed` to 1.3.
17. Change `effectiveGenParams(stageIdx)` → `effectiveGenParams(stageIdx, world)`, defaulting `world` to `G.world`. Read `junctions`, `deepArms`, `chordShare`, `doubleArmShare`, `cols` from `CONFIG.genProfiles[world]`, falling back to the flat `stageJunctions`/`stageDeadEnds`/`stageChordShare`/`mazeCols` keys if a profile is absent. Apply the streak notches as today, then clamp `J` to `maxJunctionsPerStage`, then compute `rows = Math.max(profile.rows, CONFIG.rowBudgetBase + 2*J)`. Return `cols` in the params object.
18. Update the three call sites that pass generation params: `stageRowsFor()`, `hatStageFor()`, `deepStageFor()` and `effectiveForestParams()`. Each must pass its own world id. `effectiveForestParams()` keeps forcing `chordShare = 1`.
19. `tryCarve(cols, rows, ...)` already takes `cols` as an argument but `generateStageRows()` passes `CONFIG.mazeCols` — pass `params.cols` instead so the profile's width applies.
20. **Double-armed crossings.** In `tryCarve()`'s `doChord()`, when the junction is selected for a double arm (draw from `doubleArmShare` the same way the chord/T quota is drawn), grow arms on **both** sides: call `growArm(c, r, s, armWant)` and `growArm(c, r, -s, armWant)`, and succeed if **both** return ≥ 1, restoring the snapshot and falling back to the single-arm path if either fails. `canCarve()`'s adjacency rule already permits this — the second arm's first cell touches only the junction, which is its predecessor.
21. **Teach `validateStage()` about degree 4.** Two edits, both of which must stay independent of how the maze was built:
    - Replace `if(deg > 3) issues.push('4-way junction at ...')` with a rejection of `deg > 4` only. The vertical-entry check on the BFS parent stays exactly as it is and still applies to degree-4 cells — it is the invariant that matters.
    - Replace `if(leaves !== jCount)` with `if(leaves !== expectedLeaves)` where `expectedLeaves` is the sum of `(deg - 2)` over every junction cell. For an all-degree-3 maze that equals `jCount`, so today's fixtures and the other three worlds validate exactly as before; a degree-4 crossing correctly expects two arms.
    - Leave the loop check (`edges !== cells.length-1`) untouched. The maze stays a tree.
22. Replace the hard-coded `attempt<120` in `generateStageRows()` with `CONFIG.genAttempts`.
23. Confirm `solveMaze()` still returns the right press list at a degree-4 crossing: its `opts.length > 1` test and its `need.dx` mapping already cover three options, so this should need no change — but assert it, because `stageParMs()` uses `solveMaze().length` for the junction allowance and a wrong count skews par.
24. Leave `FALLBACK_STAGES`, `HAT_STAGES` and `FOREST_STAGES` untouched. They are 9 columns wide while generated Tunnels mazes are now 11 — that is fine, `parseStage()` reads the width from the grid and `layout()` sizes from `G.maze.cols`.

### 17.5 Verification before you call it done
25. Fresh save (clear all `tunnels:*`): boot shows the picker with **three coloured hats and one silhouette**, moons counting under peg 4, highlight on Tunnels.
26. Generated Tunnels stages pass `validateStage()` on the raised profile — run `window.TUNNELS.generateStageRows` across ~50 seeds for all three stages and report the fallback rate. **It should be near zero.** A high rate means the row budget is wrong, and every day would silently serve the old easy fixtures.
27. The other three worlds generate with **identical junction counts to before** the change. This is the leak test; verify explicitly rather than by eye.
28. A full Tunnels run completes in 5–10 minutes with the new values. Report the measured time and the stage-3 cell size in px.
29. `?unlockAll=1` still unlocks all four and suppresses every write including `streakBest`.
30. An existing save with journal history migrates without losing days, streak, bests or entries, and never shows fewer hats than before.
31. Grep for `hatUnlockRuns` and `rackShowsFromWorldTwo` — zero hits.

**Flag, don't redesign.** If any step conflicts with the invariants in §1, §3 or §9 — particularly the vertical-junction rule — stop and say so rather than working around it.
