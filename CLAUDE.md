# TUNNELS — Cooperative Blind-Digging Maze Game

A two-player asymmetric HTML5 game for one iPhone. The adult digs through underground tunnels with their **eyes closed**, steered only by the child's voice. The child reads the screen and the map. Built for Alma (child, Navigator) and a grown-up (Driver). One shared device, one shared laugh.

This file is the source of truth for Claude Code. Build exactly to this spec unless a constraint is physically impossible — flag conflicts, don't silently redesign.

**Changelog vs previous CLAUDE.md:**
- New **§6 v1.1** polish list from blindfolded-playtest review (press acknowledgement, per-button haptic signatures, wake-lock coverage, Do Not Disturb nudge).
- **The plan trace is cut.** Earlier revisions planned a v2 briefing finger-trace (the Navigator inking her intended route, compared against the actual path at the reveal). It was never built and has now been dropped from the game — the two buttons stay the *only* touch targets, in every phase, of every world. Its CONFIG flags and roadmap items are removed accordingly.
- New **§7: World Two — The Hat**, a top-down overworld mode (homage to *We Found a Hat*), slotted into the roadmap as v2.5. Same two buttons, same colour grammar, new perspective.
- New **§11 The Hat Rack (built)** — the invisible `hatUnlockRuns` gate is replaced by a visible, countable progression object driven by **days played** (see §11). Adds the golden-night bonus and a sealed playtest mode. **World Three — The Forest is built** (§12, roadmap v2.7) and **World Four — Deep Water is now built** (§13, roadmap v2.8) — the fourth peg fills from a silhouette to a blue bowler at 15 days played.
- New CONFIG keys in §8 to support all of the above.
- **Picker-at-boot, always.** The world picker now opens at the start of
  every session — even with only Tunnels' hard hat on the rack. The
  silhouettes of what's still to come are the point; a returning player
  should feel the rack every time, not just once a second hat exists.
  Supersedes the old day-5 gate on the picker's *appearance* only — the
  days-played unlock ladder itself (§11) is unchanged. See §7 and §11.
- **This file is now the single source of truth.** The former `WORLDS-THREE-AND-FOUR.md` (worlds 3 & 4, the rack, engine work, tuning) and `BUILD_PLAN.md` (the shipped v2→v2.5 build order) have been folded in and deleted — §11–§16 hold the worlds/rack detail, and a closing roadmap section lists everything still unbuilt.

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

1. **Briefing (eyes open):** Both players see the stage map. They talk through the route, agree on a plan, giggle. (From v2: the Navigator inks the plan — see roadmap.) Driver taps both buttons simultaneously to confirm "eyes closing now" — this is the ready signal and starts the stage.
2. **Digging (eyes closed):** The digger auto-digs forward at constant speed. At each junction the digger **pauses** and an audio cue plays (a soft "hm?" plus a haptic tick). The Driver presses GREEN (screen-left), RED (screen-right), or BOTH TOGETHER (straight on) as directed, and the digger waits until a press arrives (v1). With timed windows (v3+), no press = digger continues straight if straight exists, otherwise bumps. **Every accepted junction press gets an immediate acknowledgement tick + haptic (v1.1)** — an eyes-closed Driver must never wonder whether the game heard them.
3. **Bumps:** Hitting rock/roots/dead-end soil = a comedy event, not a failure. Screen shake, dust puff, a silly *thud-boing* sound, strong haptic buzz, the digger's helmet slips over its eyes. The digger **bounces back to the last junction** and re-pauses. Bumps are counted but never end a run.
4. **Breakthrough (stage complete):** The digger breaks through into a cavern or up through the surface — a soft, satisfying collapse of dirt. Fanfare, haptic celebration. On-screen banner: "OPEN YOUR EYES!"
5. **Reveal (eyes open):** The full maze is shown with the actual path traced in a contrasting line, **bump locations marked with little stars/bruise icons** — this is the laugh-together moment. Stats: time, bumps, personal best comparison. Both players tap their button together to continue.

A **run** = 3 stages (easy → medium → spicy). A run takes roughly 5–10 minutes total. Short is a feature.

---

## 3. Movement Model & Complexity Scaling (two buttons only, forever)

The digger auto-moves. The Driver only ever chooses at junctions. Complexity therefore scales through **maze topology and timing**, never through added inputs:

**Junction authoring rule (screen-direction invariant):** junctions may only occur while the character is travelling **vertically on screen** (heading up or down), so the possible answers at any pause are always screen-left (green), screen-right (red), or straight on (both). Horizontal corridors contain corners only — the character auto-turns, no choice, no pause. This keeps every call Alma makes a plain "green / red / both" with zero ambiguity. Generators (v2+) must respect this invariant. **This invariant applies identically in World Two (§7).**

**The digger is always drawn upright.** It faces its travel direction by horizontal mirroring only — never rotation. An upside-down digger reads as nonsense to a small Navigator.

| Lever | Easy | Harder |
|---|---|---|
| Junctions per stage | 2 | 8+ |
| Junction types | Side-to-side T's (green/red only) | Crossings where **straight on (the chord)** is the answer, or a tempting shaft |
| Dead ends | 1, shallow | Several, deep, with tempting gems inside |
| Junction pause window | Generous (digger waits indefinitely, v1) | Timed window (press within N seconds or go straight/bump) |
| Dig speed | Slow | Faster between junctions |
| Loops | None | Tunnels that circle back — Navigator must recognise "we've been here" |
| Soft-soil chutes | None | One-way drops the digger slides down; no takebacks, forces route planning |
| Gems | On the main path | Just off the path in dead ends — a deliberate detour risk/reward decision the pair makes together |

**Design homage:** like the picture book that inspires the art, gems can sit *agonisingly close* to the tunnel — visible to the Navigator, one wrong-feeling detour away. Whether to go for them is a conversation. Collected gems fill the journal but never gate progress.

**The Navigator's information also scales** (this is the real difficulty axis):
- v1: Navigator sees the digger's live position on the full maze. Builds trust and vocabulary.
- v2: "Map mode" stages — Navigator sees the maze map but the digger's position marker updates only at junctions.
- v3: "Memory mode" — position marker disappears after the briefing; Alma tracks the digger mentally. This is the deepest spatial-reasoning workout and is unlocked, never forced.
- v2.5: **Perspective scaling** — World Two shifts the whole world from cross-section (elevation view) to bird's-eye (plan view). Same grammar, new mental model. See §7.

**The spatial-learning ladder this game is climbing (design intent, keep in mind for every feature):**
1. Colour-direction vocabulary and screen left/right (v1)
2. Plan-then-execute: agree a route aloud at the briefing before moving, then see how it went at the reveal (the briefing conversation; the inked plan trace that once served this was cut)
3. Delayed and withheld position information (v2 map mode, v3 memory mode)
4. Perspective shift: plan view of a territory you're navigating (v2.5, World Two)
5. Landmark-based navigation language: "green at the cactus" (v2.5, World Two)

---

## 4. Maze Completion & Progression

- **Stage complete:** breakthrough animation → reveal → stats → next stage.
- **Run complete (3 stages):** the digger surfaces into the backyard at sunset. The reward scene: **a pitcher of chocolate milk and a plate of animal biscuits** rises onto the screen — the traditional after-digging snack, and the game's way of saying the ritual is over for today. Journal entry (v2) is stamped automatically: date, total time, total bumps, gems, a tiny thumbnail of the final maze with the path drawn. Streak counter increments.
- **Daily seed:** each calendar day generates its mazes from a date-based seed (same approach as CircleSquareTriangle) — everyone gets the same day's tunnels, replayable for personal best.
- **Difficulty over time:** difficulty is driven by **streak, gently, with a cap** — not by raw calendar. Streak 1–3: baseline. Each streak milestone nudges one lever (a junction here, a shorter pause window there) up to a ceiling at streak ~14. Breaking a streak drops difficulty back two notches, never to zero. Rationale: the game must stay winnable by a pair having an off day; frustration kills the ritual.
- Personal best is stored **per daily seed** and as an all-time "smoothest run" (fewest bumps) and "fastest run".
- Streaks, seeds, journal, and bests are **shared across worlds** — a Hat run and a Tunnels run both count toward the ritual. One streak, one journal, two worlds.

---

## 5. Art Direction — picture-book cross-section

Inspired by the flat, deadpan, earth-toned cross-section style of *Sam and Dave Dig a Hole* — an **homage in spirit, all original assets**. Do not copy characters, compositions, or trade dress.

- **View:** side-on cross-section of the earth, like the page of a picture book. A thin strip of pale sky at the very top with a small house, a bare tree, maybe washing on a line. Everything below is soil.
- **The cat:** from stage 2 onward a small cat sits on the roof of the house, watching. It never does anything except flick its tail. It is never mentioned. Navigators notice it around the second or third run; that moment belongs to them.
- **Spectacular buried treasure (decorative, v1):** clusters of gems and little hoards of gold coins sit in the solid earth *agonisingly close* to the tunnels — visible to the Navigator with a soft sparkle, never collectible, never acknowledged by the game. The digger walks straight past a fortune, every stage, forever. That's the joke. (Collectible gems remain a v2 mechanic; these decorative ones stay even then, because the digger never gets better at this.)
- **Palette (tokens, tune in settings):**
  - `soil-deep` #4A3728 (undug earth, matte)
  - `soil-warm` #6B4F35 (mid earth, subtle grain texture)
  - `tunnel` #C9A876 (dug path — visibly lighter, like exposed dry dirt)
  - `sky` #DCE3DD (pale, quiet, slightly grey-green)
  - `gem` #B23A48 (dusty red — the one saturated accent on screen)
  - `button-left` #5E7C4A (green), `button-right` #A6423A (red) — muted to sit inside the palette, still unmistakably green/red
- **Texture:** flat colour fields with a very subtle paper grain overlay. No gradients, no gloss, no outlines thicker than 1px. Rocks are soft blobs a shade darker than soil. Roots dangle from the surface. The occasional buried oddity (bone, old boot, teacup) as silent jokes in the dirt — decorative only.
- **Characters:** small, simple, deadpan. The digger is a little figure with a hard hat and a spade, drawn in 2–3 flat colours, always upright (mirrored, never rotated). **A small dog companion** (in from v1) trots along the dug tunnel a step behind the digger, following its exact path — including sliding backwards after a bump. When a gem is nearby (v2+) the dog's ear pricks up — a visual whisper only the Navigator sees. (The dog always knows.)
- **Motion:** minimal and dry. The dig is a steady rhythmic animation. Bumps are the biggest motion on screen. The breakthrough is dirt crumbling away in chunky flat particles. Respect `prefers-reduced-motion`.
- **Typography:** one rounded, friendly display face for banners ("OPEN YOUR EYES!") and a plain body face for stats. Sentence case everywhere. Words on screen are for the Navigator — keep them short enough for an early reader: "Ready?", "Go!", "Found a gem!", "You made it!"
- **Signature element:** the eyes-open **reveal page** — the whole maze rendered like a finished picture-book spread with the pair's wobbly path inked through it and bump-stars marking every thud. This is the screenshot-worthy moment; polish it hardest.

---

## 6. Version Roadmap

### v1 — Prove the ritual (built)
- One run = 3 stages, fixed hand-authored mazes ramping 2 → 4 → 6 junctions, full colour-button control scheme incl. the chord, untimed junction pauses, bumps with sound/haptics/shake, breakthrough, reveal with path trace and bump stars, stats, dog, cat, decorative treasure, snack scene, live digger position, ready/continue chord, localStorage bests, dev settings panel.

### v1.1 — Blindfold polish (from the eyes-closed playtest review; small, ship before v2)
- **Press acknowledgement:** every accepted junction press produces an immediate short tick sound + light haptic, before movement resumes. Silence after an input is intolerable with eyes closed. Bumps keep their own (louder, sillier) feedback; the ack fires on *any* accepted press, including ones that will bump — it confirms "heard you", not "correct".
- **Per-button haptic signatures:** on press-down, GREEN buzzes one short pulse, RED buzzes two. The Driver learns and verifies the buttons by feel, without ever opening her eyes to check her grip. Configurable in CONFIG (`hapticSigLeft`, `hapticSigRight`), degrade silently where `navigator.vibrate` is unavailable.
- **Wake-lock coverage:** request the wake lock at briefing (not just stage start) and re-request on visibility change in all non-terminal phases, so a slow chat over the briefing map never dims the screen.
- **Do Not Disturb nudge:** the first briefing of a session shows one quiet line: "Tip: turn on Do Not Disturb — the Driver can't see notifications coming." A banner mid-stage while blindfolded is catastrophic; we can't block it in a web page, so we say it once, politely.

### v2 — Make it a ritual (built, except the map panel — see below)
- **(built)** Procedural maze generation from daily date seed, 3-stage ramp. Validator-first, with the v1 hand-authored mazes kept permanently as the never-broken fallback.
- **(built)** Streak system + streak-driven difficulty (capped, as §4). One-day grace before a streak bends; two missed days start a new count.
- **(built)** Gems + dead-end temptations + the dog's gem-sense (the ear prick). World One only; the desert's treasure joke stays the second hat.
- **(built)** Journal: stamped entries per completed run (date, stats, maze thumbnail — footprints in the desert). Shared across worlds. localStorage array, capped at ~200 and quota-safe (trims oldest first). Minimal surface in v2: the fresh page shows on the run-complete card; a browsable book waits. (The referenced Creature Keepers file has since been removed from the repo, so the same journaling *shape* was implemented independently.)
- *pending* — Map panel for the Navigator alongside the live view.
- *cut* — **The plan trace.** Once planned as the key plan-then-execute mechanic (a briefing finger-trace inked in soft grey, drawn under the actual path at the reveal). Never built, now dropped: the plan-then-execute beat lives in the spoken briefing instead, and the two buttons remain the only touch targets in every phase.

### v2.5 — World Two: The Hat (built — see §7 for the full spec)
- **(built)** H1: three hand-authored overworld stages, the two tortoises, landmarks, shell-bonks, footprint reveal, sunset scene. World picker at boot.
- **(built)** H2: procedural desert generation on the shared daily seed; landmark-aware generation (every junction within one tile of a distinct nameable landmark; the nameable vocabulary grew to seven so big stages stay unambiguous); journal entries with footprint thumbnails.
- **Do not start v2.5 until the tunnel ritual has demonstrably stuck** (streaks happening without prompting) — this gate was met before the Hat was built.

### v2.6 — The Hat Rack (built — see §11 for the full spec)
- **(built)** Progression currency: `tunnels:daysPlayed` — distinct calendar days a run was completed, cumulative, monotonic, shared across worlds, migrated from the journal's `day` stamps. Worlds now unlock on **days played** (Tunnels 0, The Hat 5, The Forest 10, Deep Water 15), replacing the invisible `hatUnlockRuns` gate.
- **(built)** The golden night: a run that is bump-free across all three stages **and** finishes under a derived per-run par is worth two nights instead of one, capped at the first qualifying run each calendar day. **Par is never shown during a stage** and never as a number — only the golden outcome surfaces (gold line, rising two-note chime, distinct haptic, gold particles).
- **(built)** The hat rack: four pegs, one hat per world; unlocked pegs in full colour, locked pegs as pale silhouettes; countable moon pips for sleeps remaining beneath the next locked peg; a gold-moon tease beside it. Rendered on the run-complete card and in the picker. First appears at 5 days (backfilling pegs 1 & 2); an unlock ceremony precedes the stats when a new peg is earned.
- **(built)** Playtest mode (Marty only): three doors — URL `?unlockAll=1`, a dev-panel button ("Take all the hats down"), and the 4-3-2-1 rack gesture — unlock all worlds while sealing every write to the real save. A gold-moon tell in the rack corner and `PLAYTEST` in the debug overlay keep a test run unmistakable.
- **(built)** World Four (Deep Water) itself — see v2.8 below; its rack peg fills from a silhouette to a blue bowler at 15 days.

### v2.7 — World Three: The Forest (built — see §12 for the full spec)
- **(built)** One maze walked twice: **leg 1** the bear searches down the spine, **leg 2** the run back up to the rabbit's arm. No new maze on the return — the same maze with start and exit swapped. Forest stages are **chord-only** (`chordShare:1` — the load-bearing constraint that keeps every call a plain green / red / both in both legs); a straight vertical spine with horizontal dead-end arms, generated from the shared daily seed with hand-authored chord-only fallbacks.
- **(built)** The cast at the arm tips (rabbit, sleeper, deny/ask animals) and the deer at the foot of the spine. Meeting an animal is a **polite turnaround** — a bubble, a chime, a walk back to the junction, no bonk and **no bump counted**. Bare arms still bump. Bubbles are **pictograms** with a `forestSpeechMode:'words'` toggle (NO / OK). The rabbit's bubble is three slashed hats, protesting too much.
- **(built)** The **deer beat** (two identical hat silhouettes, the bear's filled red) → the **full-bleed red page** ("He knows where his hat is.", the longest haptic, non-strobing, reduced-motion-safe) → leg 2, the bear facing up. On the run back the animals are gone; a wrong arm bumps and, after `forestHintAfterBumps`, the rabbit flickers back.
- **(built)** The **two-weight reveal** (leg 1 soft grey, leg 2 charcoal ink, the rabbit redrawn hatless) and the deadpan run-complete scene: the bear sitting in his hat, a squirrel wandering in with a question nobody answers.
- **(built)** The **N-world picker** (§14.5): with three or more worlds unlocked the rack becomes a stepper — **green steps left, red steps right, the chord goes there**; locked pegs are skipped. Two unlocked worlds keep the original direct green/red choice. No new touch targets either way.
- *cut* — The plan trace on the return leg: dropped along with the plan trace everywhere (the finger-trace was never built).

### v2.8 — World Four: Deep Water (built — see §13 for the full spec)
- **(built)** The lantern and the seen-set — the ladder's seventh rung, **building a map from partial views**. Near-black water; the Navigator sees a soft circle of `deepFogRadiusTiles` around the little fish, and **everywhere it has swum stays lit permanently**, so the map draws itself. Rendered as a soft-edged fog mask over the static ocean (no hard vignette), erased along the swum path; a mid-stage relayout re-lights the known cells from the seen-set.
- **(built)** The **briefing flash** (`deepBriefingFlashMs`, default 4000, a dev-panel dial per §16.1): the whole ocean lit while the pair plan, then it fades to dark. The briefing stays untimed. In the dense **kelp** at the goal the fog shrinks to `deepFogPlantsRadiusTiles`.
- **(built)** The **big fish** — mechanically the crumb-follow dog, reskinned, at `deepPursuerLagTiles` behind; it backs up in unison when the little fish bounces, can never collide, and is **never a fail state**. Visible to the Navigator, invisible to the Driver — the correct distribution of dread. The **crab** (Deep Water's silent-joke slot) points once, lighting `crabHintTiles` down the correct branch, then goes back to sunning.
- **(built)** The **scripted ending** that cannot be avoided (`deepEndingMs`): both fish inside the plants, a held stillness, then the breakthrough — and in the run-complete scene the big fish sits, hatted, motionless, facing the reader. **World Four signs off with silence.**
- **(built)** The score is the **survey, not the chase** — `mapLitPct` ("You saw N% of the ocean", `deepMapLitStat`) with an all-time best, since the ending is fixed. The reveal inks a dotted bubble-trail and leaves **the unexplored dark exactly as it was**; the journal thumbnail draws only what was lit, a different shape every day. Mixed junctions (unlike the Forest's chord-only spine), own daily seed stream, tunnel fixtures as the never-broken fallback. Journal, streak, seed and bests stay shared — one streak, one journal, four worlds.
- *cut* — The plan trace live through the briefing flash: dropped along with the plan trace everywhere.

### v3 — Deepen the learning
- Map mode and Memory mode stages (Navigator information scaling, §3) — across the worlds.
- Timed junction windows as an unlockable "spicy" modifier.
- Loops and soft-soil chutes in tunnel generation.
- Sound design pass: each junction type gets its own audio cue so the Driver starts learning the maze by ear — a quiet second literacy.
- **Line-of-sight mode (World Two spicy variant):** the Navigator sees only what the watcher tortoise can see from its rock — paths behind big rocks and cacti are hidden and must be remembered from the briefing. The overworld's answer to memory mode.

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
- **Landmarks are first-class citizens.** Each stage places 3–5 distinct, nameable landmarks (a big saguaro-ish cactus, three stacked stones, a bleached log, a dark pool). Generation (H2) must place every junction within one tile of a landmark, so a call like "green at the cactus!" is always possible. Landmark callouts are the new literacy — the reveal can gently reinforce this by labelling landmarks on the reveal spread ("the cactus", "the three stones") in tiny sentence-case text.

### Rules (all inherited, restated for clarity)
- The Seeker auto-walks, tortoise-slow. Slow is the joke, and the pace suits a plan view where the Navigator is reading further ahead.
- **The screen-direction invariant holds unchanged:** junctions occur only while the Seeker travels vertically on screen (up-screen or down-screen). Green = turn toward screen-left, red = screen-right, chord = straight on. Horizontal path segments contain auto-turn corners only. Authoring and generation must respect this exactly as in World One.
- The Seeker is drawn from above and **rotates in 90° steps to face its heading** — in plan view rotation is how the world works and does not confuse a plan-reader (the upright-only rule is an elevation-view rule). Its walk is a tiny deadpan waddle.
- **Bumps become shell-bonks:** walking into a bush/boulder = *bonk*, the Seeker pulls into its shell (the helmet-slip equivalent), comedy pause, then backs up along its own footprints to the last junction. Same sounds/haptics family as tunnel bumps, slightly drier.
- Junction pause, chord grace window, re-cue, press acknowledgement, timers, stats: identical systems, shared code.

### Companions & silent jokes (the World Two equivalents)
- **A small lizard** suns itself on a rock somewhere in each stage. It never moves except to blink. Never mentioned. (The cat's cousin.)
- **A second hat**, half-buried in sand in a corner of the map, never acknowledged, never collectible. Readers of the book will understand. Alma will find it eventually and that moment belongs to her.
- The Watcher tortoise on its rock slowly turns its head to keep facing the Seeker throughout the stage — the only "camera" in the game, and it's a tortoise.

### Reveal & run completion
- The reveal spread shows the whole desert from above with the Seeker's **footprint trail** dotted through the sand and shell-bonk stars at every bonk (v2 reveal systems shared).
- **Run complete:** the Seeker reaches the hat and puts it on. Final scene (an homage in spirit to the book's last page — **all original art**, no reproduction of Klassen's illustration): a black sky full of stars, the two tortoises side by side, each wearing a hat, drifting off among them. Nobody says anything. World Two closes here — the **chocolate milk and animal biscuits** snack is World One's sign-off; the stars are the Hat's. The ritual is shared (one streak, one journal); only the closing image differs.

### Art direction (delta from §5)
- Palette shifts warm and pale: sand, sage-green scrub, long shadows. The hat is the focal object — a **tan cowboy hat** (`hatTan` #C2A06A, `hatCrown` #A88752, `hatShade` #8E6F42), matte and flat, all clearly darker than the sand so it reads instantly (wide flat brim, rounded dented crown, a centre crease — never a red disc). `gem` #B23A48 stays World One's one saturated accent.
- Same flat colour fields, paper grain, no gradients, 1px max outlines. Shadows are single flat shapes cast consistently to one side — in plan view, shadows are what make landmarks readable, so they matter more here.
- Run-complete star scene rendered side-on (the one elevation shot in this world), like the last page of a picture book: a night sky, two hatted tortoises adrift.

### World picker
- At boot, a simple picker card: **GREEN button = Tunnels, RED button = The Hat.** The buttons teach themselves. (Chord = replay whichever world you played last.) No menus, no scrolling, no third touch target.
- ~~World Two unlocks after `hatUnlockRuns` completed tunnel runs (default 5)~~ **Superseded by §11 (built):** worlds unlock on **days played** (`worldUnlockDays`, The Hat at 5), not run counts. `hatUnlockRuns` remains in CONFIG for back-compat but no longer gates the picker.
- ~~The picker appears once The Hat is earned (or immediately in sealed
  playtest mode); until then Tunnels boots directly.~~ **Superseded:** the
  picker now opens on **every boot**, from the very first session. With
  one hat unlocked it shows that one peg in full colour and the rest as
  pale silhouettes; either button (or the chord) begins Tunnels — there's
  no choice to make yet, only the rack to notice. The silhouettes do the
  talking; the game never says "more worlds are coming."

---

## 8. Dev Settings & Tuning Panel

All tunables live in a single `CONFIG` object at the top of the file, persisted to localStorage, editable at runtime via a hidden dev panel (**tap the sky 5 times**). Every value overridable by URL param (`?digSpeed=1.4`). Include a "reset to defaults" and an "export config as JSON" button.

```js
const CONFIG = {
  // movement
  digSpeed: 1.0,            // tiles per second between junctions (World One)
  walkSpeed: 0.8,           // tiles per second (World Two — tortoise-slow is the joke)
  junctionPauseMs: 0,       // 0 = wait forever (v1 default); >0 = timed window
  bumpBouncePx: 24,         // comedy bounce distance
  bumpStunMs: 900,          // pause after bump/bonk before re-offering the junction
  // maze generation (v2+)
  stageJunctions: [2, 4, 6],    // per stage in a run
  stageDeadEnds: [1, 2, 3],
  mazeCols: 9, mazeRows: 12,
  gemCount: 2, gemDeadEndBias: 0.7,  // fraction of gems placed in dead ends
  // overworld generation (v2.5+)
  landmarkCount: 4,         // nameable landmarks per Hat stage
  landmarkJunctionRadius: 1,// every junction within this many tiles of a landmark
  hatUnlockRuns: 5,         // completed tunnel runs before World Two appears
  // difficulty ramp
  streakDifficultyStep: 3,  // every N streak days, nudge one lever
  streakDifficultyCap: 14,
  // controls & accessibility
  colourLeft: '#5E7C4A', colourRight: '#A6423A',
  buttonHeightPct: 22,
  chordWindowMs: 250,       // grace period for the both-buttons straight-on chord
  hapticsEnabled: true, audioVolume: 0.8,
  pressAckEnabled: true,    // immediate tick+haptic on any accepted junction press (v1.1)
  hapticSigLeft: [20],      // press-down signature: green = one pulse (v1.1)
  hapticSigRight: [20, 60, 20], // red = two pulses (v1.1)
  // companions & silent jokes
  showDog: true, dogLagTiles: 0.85,   // how far behind the digger the dog trots
  showCat: true,            // roof cat, stage 2 onward (World One)
  showLizard: true,         // sunning lizard (World Two)
  treasureCount: 3,         // decorative gem/gold spots per stage (never collectible)
  buriedSecondHat: true,    // World Two's silent joke (never collectible)
  // navigator information
  showLiveDigger: true,     // v1 true; map/memory modes flip this
  positionUpdateAtJunctionsOnly: false,
  lineOfSight: false,       // Watcher sees only what's visible from the rock (v3, World Two)
  // progression & the hat rack (§11, built)
  worldUnlockDays: { tunnels:0, hat:5, forest:10, deep:15 }, // days-played thresholds
  rackPipThreshold: 7,      // show countable sleeps when this few remain
  rackShowsFromWorldTwo: true, // no rack until The Hat (5 days) is reached
  pickerAlwaysAtBoot: true,    // NEW — the picker opens every session, even with 1 hat unlocked
  rackEnabled: true,
  // the golden night (§11, built) — bump-free AND under par, once per calendar day
  goldenNightEnabled: true,
  parSlack: 1.25,           // multiplier on pure travel time
  junctionThinkMs: 12000,   // free deliberation per junction — deliberately generous
  goldenNightPerDay: 1,     // first qualifying run each calendar day only
  // playtest (Marty only) — URL ?unlockAll=1, dev panel, or the 4-3-2-1 rack gesture
  unlockAll: false,
  playtestSuppressesWrites: true,
  // World Three — The Forest (§12, built)
  forestWalkSpeed: 0.9,          // the bear's plod down the spine
  forestReturnSpeedMult: 1.4,    // faster on the run back
  forestRedPageMs: 1400,         // the full-bleed red beat
  forestAnimalCount: 4,          // animals seated at arm tips
  forestSpeechMode: 'pictures',  // 'pictures' | 'words' (NO / OK)
  speakerLineMs: 1800,           // how long an animal's bubble holds
  forestShowRabbitOnReturn: false, // keep the rabbit hidden on leg 2
  forestHintAfterBumps: 3,       // wrong arms before the rabbit flickers back
  // World Four — Deep Water (§13, built)
  swimSpeed: 0.85,               // the little fish drifts, unhurried
  deepFogRadiusTiles: 2.2,       // the lantern in open water
  deepFogPlantsRadiusTiles: 1.2, // the fog shrinks in the dense kelp at the goal
  deepBriefingFlashMs: 4000,     // the whole ocean lit, then dark (§16.1 dial)
  deepPursuerLagTiles: 3.5,      // the big fish's constant distance behind
  deepShowPursuer: true,         // the dread is the Navigator's; the Driver never sees it
  crabHintTiles: 4,              // the crab lights this far down the correct branch
  deepMapLitStat: true,          // "You saw N% of the ocean" — the survey, not the chase
  deepEndingMs: 2000,            // the scripted stillness before the big fish emerges
  // debug
  debugOverlay: false,      // show grid, seed, junction ids
  seedOverride: null,       // force a specific daily seed for testing
};
```

Rule: **no magic numbers in gameplay code** — if it affects feel, it goes in CONFIG.

## 9. Technical Constraints

- Single self-contained `index.html` — inline CSS/JS, no build step, no external network calls. Canvas rendering for the world; DOM for buttons and panels. Both worlds live in the same file and share every system that isn't world-specific (input, junction logic, chord, timers, reveal, journal, config, audio scaffolding).
- Target: iPhone Safari, portrait, one-handed-thumbs ergonomics. Must run offline once loaded.
- Persistence: `localStorage` only (journal, bests, streak, config). Namespace keys `tunnels:*` (shared across worlds — one streak, one journal).
- Audio: Web Audio API, synthesised or tiny embedded sounds — must work after a user gesture (the ready-signal press unlocks audio).
- Haptics: `navigator.vibrate()` where available; degrade silently.
- The screen must **never sleep mid-stage**: use the Screen Wake Lock API with graceful fallback; request at briefing and re-request on visibility change in all non-terminal phases (v1.1).
- Accidental exits are catastrophic mid-stage (eyes closed!): no touch targets other than the two buttons, in any phase; ignore multi-touch outside them. (The dev panel's sky-tap and the rack's peg hit-areas are the only non-button touch surfaces, and never during play.)

## 10. Tone Rules (for every string and sound in the game)

Bumps and bonks are funny. Nothing is ever a failure. The game never says "wrong", "oops" in a scolding way, or shows a red X. Stats celebrate ("smoothest run yet!") and never shame. The Driver being helpless is the joke; the Navigator being capable is the point. The tortoises never speak. The hat is never explained.

---

# Worlds Three & Four, the rack, and what's next (§11–§16)

*Folded in from the former `WORLDS-THREE-AND-FOUR.md` (now deleted) so this file is the single source of truth. Everything in §11–§14 is **built and shipped**; §16 is playtest tuning, not code to write.*

**Homage rule, non-negotiable:** all original assets, all original strings. No reproduction of Klassen's illustrations, characters, compositions, trade dress, or text. The debt we honour is the deadpan, the withheld information, and the flat colour. In-game names are ours: *The Forest*, *Deep Water* — never the book titles.

**Decisions locked:** thresholds 5 / 10 / 15 days · one currency (days played) · golden-night bonus = no bumps **and** under par, once per calendar day · rack appears only on reaching World Two · Forest speech is pictures with a words toggle · Forest trace charcoal · Forest built first · deadpan endings kept as the books leave them · **the plan trace is cut** (never built, dropped everywhere; the two buttons are the only touch targets in every phase).

## 11. The Hat Rack — progression made visible (**built**)

**Principle:** progress **never goes backwards**. Nothing earned is ever taken away, and the counter never mentions a day that wasn't played.

**The currency — days played:** distinct calendar days on which a run was completed, in any world. Cumulative, monotonic, shared across all four worlds — one ritual, one number. Stored as `tunnels:daysPlayed`, migrated from the journal's `day` stamps (`new Set(journal.map(e=>e.day)).size`). Streaks keep their own separate job (difficulty notches, the "Day 7 of digging together" line) and are celebrated, not spent.

**The unlock ladder:**

| Peg | World | Hat | Unlocks at |
|---|---|---|---|
| 1 | Tunnels | yellow hard hat | from the start |
| 2 | The Hat | tan cowboy hat | **5 days** |
| 3 | The Forest | red pointy hat | **10 days** |
| 4 | Deep Water | small blue bowler | **15 days** |

**The golden night (bonus):** a run that is **bump-free across all three stages and finishes under par** is worth **two nights instead of one**. Par is per run, not per stage, and derived (never authored) so it survives procedural generation and the streak speed lever:

```
parMs = Σ over the 3 stages of [ (routeTiles / effectiveSpeed) × 1000 × parSlack
                                 + junctionCount × junctionThinkMs ]
```

`parSlack` 1.25, `junctionThinkMs` 12000. The junction allowance is deliberately huge because `G.stageMs` runs during the `junction` phase — twelve seconds per junction means a careful, chatty pair beats par and only genuine stalling loses it. **The briefing is untimed and stays untimed.** Par measures time underground only, is **never shown during a stage** (a visible clock breaks §10), and surfaces only on the run-complete card. **Once per calendar day, first qualifying run only** — the seed is replayable, and a memorised maze would otherwise unlock everything in an afternoon. Celebration: two gold pips instead of one, a rising two-note chime, a distinct haptic, a scatter of gold particles, and the line *"A golden night."* A small gold-moon outline always sits beside the next peg so the pair know the bonus exists without ever being told a target time.

**The object:** four pegs, one hat per world (every world has a hat — the joke of the whole series). Unlocked pegs hold their hat in full colour; locked pegs hold a **pale silhouette** — the shape is the tease, never a padlock. Beneath the next locked peg, **sleeps remaining as countable moons** (shown when `rackPipThreshold` 7 or fewer remain).

**First appearance:** not before World Two is reached. The run that hits five days swings the rack into view and lands hats on pegs 1 & 2 together (backfilling), two silhouettes waiting. From then on it shows on every run-complete card.

**Where it appears:** the world picker (the rack *is* the picker, §14) on
**every boot, from day one** — a single lit peg with silhouettes waiting
is the whole suspense. The run-complete card's rack keeps its existing
gate, first appearing at World Two (5 days), since that's the unlock
ceremony's reveal moment and should stay a surprise the first time it
fills. Never mid-stage, never in the briefing, no percentages or bars.

**Picker vs run-complete gating:** `rackShowsFromWorldTwo` continues to
gate *only* the run-complete card's rack. The picker's rack ignores that
flag entirely and always renders — controlled instead by
`pickerAlwaysAtBoot` (default `true`).

**The unlock ceremony (worlds three & four):** the rack fills the screen (stats deferred); the new hat drops onto its peg with a distinct fanfare and a celebratory haptic; one line — *"A new hat."* / *"Somebody has lost this one."* (Deep Water: *"This one is not ours."*); chord to continue; then the stats card.

**Tone for the counter:** never reference absence. No "you missed yesterday", no streak-loss copy. It states only what is true now: this many hats, this many sleeps.

**Playtest mode (Marty only):** a sealed mode unlocking all four worlds immediately, via three doors — URL `?unlockAll=1`, the dev-panel button ("Take all the hats down"), or the 4-3-2-1 rack gesture within four seconds. While on, all writes (`journal`, `daysPlayed`, `streak`, `best`, `seedbest:*`) are suppressed; reads work so the rack renders truthfully. The tell: a gold moon in the rack corner and `PLAYTEST` in the debug overlay.

## 12. World Three — The Forest (**built**, roadmap v2.7)

Homage in spirit to *I Want My Hat Back*. All original assets. **Rung six: route reversal** — going back the way you came, the operation that turns a route into a map. It also teaches that **the answer was on screen the whole time**: the rabbit wears the stolen hat from the first briefing frame; the bear never notices; Alma will, instantly, and the eyes-closed Driver can do nothing until the bear works it out himself. That gap is the world.

**Structure — two legs and a red page (one maze, traversed twice):**
- **Leg 1, the search (down-screen):** the bear enters at the top and walks down the spine; animals sit at the arm tips. Chord to keep going down, green/red to turn into an arm.
- **The red page:** arriving at the deer (leg-1 exit) floods the screen full-bleed saturated red for `forestRedPageMs` with one line — *"He knows where his hat is."* — and the longest haptic in the game. Never a strobe at any setting; reduced-motion makes it a hold.
- **Leg 2, the run back (up-screen):** the goal becomes the rabbit's arm tip, speed × `forestReturnSpeedMult`. The trodden path stays drawn but **the animals and the rabbit are gone** — Alma must remember which arm, and which side. A wrong arm bumps, bounces back, try again.

**The load-bearing constraint:** Forest stages are **chord-only** (`stageChordShare:[1,1,1]`). A chord junction leaves vertically, so the return enters vertically and the calls stay exactly chord / green / red in both legs — green is still screen-left, the world never flips. Leg 2 needs no new maze: start = deer cell heading up, exit = rabbit's arm.

**Meeting an animal** is a conversation, not a bump: the bear stops, a bubble holds for `speakerLineMs`, a soft chime marks it, then a **polite turnaround** back to the junction — no bonk, no shake, **no bump counted**. Bare arms still bump (trees aren't conversational). Bubbles are **pictograms** (`forestSpeechMode:'pictures'`, a `'words'` toggle gives NO/OK): most animals a slashed hat, the asker a hat outline with a "?", **the rabbit three slashed hats crammed in** (protesting far too much), the sleeper a "z". **The deer beat** shows an empty hat outline; the bear's bubble fills the same shape red — two identical silhouettes, the whole story in shape and colour — then the red page. **The silent joke:** one animal is always asleep and never answers (the cat's/lizard's cousin). **Return-leg difficulty:** rabbit hidden (`forestShowRabbitOnReturn:false`) with a hint after `forestHintAfterBumps` (3) wrong arms.

**Reveal:** two weights — leg 1 soft grey, leg 2 charcoal ink (`#33291F`) — the whole journey at a glance; animals redrawn, the rabbit hatless. **Run complete:** the bear sitting in his hat, deadpan; a squirrel wanders in asking about a rabbit in a hat, the bear says no and asks why he's asking; nothing is explained. World Three signs off with a question nobody answers.

**Art:** elevation forest floor; tree-trunk walls in two greens and a grey-brown; warm ivory ground (`#EFE7D6`, paler than soil — air, not earth); one saturated red for the hat only (`#B23A48`). The bear is large, upright, mirrored not rotated, one dot eye, no expression ever.

## 13. World Four — Deep Water (**built**, roadmap v2.8)

Homage in spirit to *This Is Not My Hat*. All original assets. **Rung seven: building a map from partial views** — every earlier world hands the Navigator a complete map at briefing; Deep Water hands her a few seconds of one, then the light goes. What she holds in her head after that is cartography.

**The lantern and the flash:** near-black water; the Navigator sees a soft circle of `deepFogRadiusTiles` around the little fish, and **everywhere it has swum stays lit permanently** — the map draws itself, and that is the whole feature. The **briefing flash** lights the entire ocean for `deepBriefingFlashMs` (default 4000, a dev-panel dial — §16.1), then it goes dark; the briefing stays untimed. In the dense **kelp** at the goal the fog shrinks to `deepFogPlantsRadiusTiles`. Fog is a soft-edged mask over the static ocean, **no hard vignette**.

**The crab** (Deep Water's silent-joke slot): once per stage, beside one junction; swim within a tile and it points, lighting `crabHintTiles` down the correct branch, then goes back to sunning. Not required, not scored, easy to miss.

**The big fish:** mechanically the crumb-follow dog at `deepPursuerLagTiles` behind — so it backs up in unison when the little fish bounces, can never collide (no collision code), and is **never a fail state**. Visible to the Navigator, invisible to the Driver — the correct distribution of dread.

**The scripted ending (cannot be avoided):** the goal sits in the kelp; on arrival both fish hold inside the plants, a `deepEndingMs` stillness, then the breakthrough — and in the run-complete scene the big fish sits, hatted, motionless, facing the reader. **World Four signs off with silence.**

**The score is the survey, not the chase:** because the ending is fixed, the achievement is `mapLitPct` — lit cells over open cells — shown as *"You saw N% of the ocean"* with an all-time best (`deepMapLitStat`). **Reveal:** the route as a dotted bubble-trail with bump stars, and **the unexplored dark left exactly as it was** — the shape of what you missed, the reason to come back. The journal thumbnail draws only what was lit, a different shape every day. Mixed junctions (unlike the Forest's chord-only spine), its own daily-seed stream, tunnel fixtures as the never-broken fallback. Bumps are **tangles** (a wet muffled catch). Palette: `water` #0E1A24, `waterLit` #16303C, `kelp` #2F5147/#3F6A55, fish pale ivory (`#E8E0CE`), one accent blue for the hat only (`hatBlue` #3E6E9C, `hatBrim` #2E5578).

## 14. Engine work (all **built**)

The engine is world-agnostic behind the `WORLDS` hooks; nothing here changed the input model, the junction invariant, the chord, the bump economy, the streak system, the daily seed, the journal shape, or CONFIG discipline.

1. **Progression module** — `daysPlayed`, journal-derived migration, threshold table, unlock state.
2. **Par & the golden night** — per-run par from the optimal route, bump-free check, once-per-day cap; par never surfaces during play.
3. **The hat rack** — render, pip fill, first-appearance backfill at World Two, unlock ceremony; shared by the run-complete card and the picker.
4. **Playtest mode** — three doors, write suppression, the gold-moon tell.
5. **The N-world picker** — the rack *is* the picker. Two unlocked worlds keep the direct green/red pick; at three or more it becomes a stepper: **green steps left, red steps right, chord goes there**, locked pegs skipped. No new touch targets.

   One unlocked world — the common case for a brand-new player — is its own
   small case: the picker still opens, still shows the rack, but there's no
   choice to make. Either button, or the chord, starts Tunnels. Copy reads
   as an invitation ("Ready?") rather than a false either/or ("Green —
   Tunnels / Red — Tunnels").
6. **Multi-leg stages** (Forest) — leg 1 goal, cinematic beat, leg 2 start/heading/goal (the same maze, ends swapped).
7. **Full-screen beat** (the red page) — reduced-motion safe, non-strobing.
8. **Bubble overlay & polite turnaround** — pictogram bubbles with a words toggle; a bounce variant that skips the bonk and counts no bump.
9. **Two-weight reveal trace** (Forest).
10. **Fog mask & seen-set** (Deep Water) — per-stage lit-cell set, soft-edged mask, `mapLitPct` and its all-time best.
11. **Briefing flash** (Deep Water) — timed full reveal, then dark.
12. **Pursuer** (Deep Water) — the crumb-follow companion, longer lag, no collision logic.
13. **Scripted ending** (Deep Water) — hold, emerge, hat.
14. **Journal thumbnails** — the Forest draws both legs; Deep Water draws only what was lit. Journal, streak, seed and bests stay shared. One streak, one journal, four worlds.

## 15. CONFIG

All the keys these worlds need are already in the `CONFIG` object in §8 — the rack/golden-night/playtest block, the Forest block, and the Deep Water block. Existing rule stands: **no magic numbers in gameplay code.**

## 16. Open — playtest tuning (not code to write)

These are one-line CONFIG changes after a real session with Alma and a grown-up, not features to build:
1. **Deep Water's briefing flash** — 4000ms is a placeholder. Try 2000 / 4000 / 6000 from the dev panel; keep whichever makes Alma lean in rather than freeze.
2. **Par feel** — `parSlack` 1.25 and `junctionThinkMs` 12000. If a golden night ever feels like it rewarded rushing, `junctionThinkMs` goes **up**, not down.
3. **`forestHintAfterBumps`** — shipping at 3; revisit after the first run back.
4. **Whether the squirrel beat stays** — written in, easy to cut. You'll know when you see it drawn.

---

# What's left to build (index)

Everything through the four worlds and §11–§16 is shipped. The full picture lives in the §6 roadmap; this is the short list of what is *not* built, so it's clear at a glance:

- **The Map panel (v2, committed — the only committed-but-unbuilt feature).** Navigator sees the maze map with the digger's position marker updating only at junctions. Groundwork flag `positionUpdateAtJunctionsOnly` is in CONFIG but not yet read. This is the §3 information-scaling axis and the on-ramp to v3 memory mode. *(See §6 v2.)*
- **Picker-at-boot (spec change, not yet built).** Boot currently calls
  `showPicker()` only when `getDaysPlayed() >= worldUnlockDays.hat` (or
  playtest); otherwise it falls through to `setupStage(0)` straight into
  Tunnels. Change: call `showPicker()` unconditionally at boot (still
  after the initial `setupStage(0)` that preps the canvas/maze).
  `renderPicker()` needs a single-world branch (§14.5) with its own copy
  — e.g. "Ready?" / "Hold either button — or both — to dig in" — and
  `rackVisible()`'s gate must be bypassed specifically inside the
  picker's render path. The run-complete card's rack gate
  (`rackShowsFromWorldTwo`) is untouched.
- **v3 — Deepen the learning** *(future; see §6):* map/memory modes, timed junction windows as an unlockable "spicy" modifier (the raw `junctionPauseMs` mechanic already works — only the unlock wrapper is missing), loops + soft-soil chutes in generation, per-junction-type sound design, and World Two line-of-sight mode (`lineOfSight` is a declared-but-unread flag).
- **v4 — Stretch** *(only if the ritual sticks; see §6):* role swap, two-device mode, finger-drawn maze editor.
- **§16 tuning** is playtest calls, not code.
