# TUNNELS — Cooperative Blind-Digging Maze Game

A two-player asymmetric HTML5 game for one iPhone. The adult digs through underground tunnels with their **eyes closed**, steered only by the child's voice. The child reads the screen and the map. Built for Alma (child, Navigator) and a grown-up (Driver). One shared device, one shared laugh.

This file is the source of truth for Claude Code. Build exactly to this spec unless a constraint is physically impossible — flag conflicts, don't silently redesign.

**Changelog vs previous CLAUDE.md:**
- New **§6 v1.1** polish list from blindfolded-playtest review (press acknowledgement, per-button haptic signatures, wake-lock coverage, Do Not Disturb nudge).
- New **v2 feature: the plan trace** — Navigator inks her intended route during the briefing; the reveal compares plan vs actual. This is the core map-reading teaching mechanic.
- New **§7: World Two — The Hat**, a top-down overworld mode (homage to *We Found a Hat*), slotted into the roadmap as v2.5. Same two buttons, same colour grammar, new perspective.
- New CONFIG keys in §8 to support all of the above.

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
- Cannot touch the screen during a stage. Her controller is her voice. (Exception from v2: the briefing plan trace, which happens eyes-open before the stage starts.)

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
5. **Reveal (eyes open):** The full maze is shown with the actual path traced in a contrasting line, **bump locations marked with little stars/bruise icons** — this is the laugh-together moment. (From v2: the plan line is shown underneath in soft grey, so plan vs actual is the conversation.) Stats: time, bumps, personal best comparison. Both players tap their button together to continue.

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
- v2: "Map mode" stages — Navigator sees the maze map but the digger's position marker updates only at junctions. Plus the plan trace at briefing.
- v3: "Memory mode" — position marker disappears after the briefing; Alma tracks the digger mentally. This is the deepest spatial-reasoning workout and is unlocked, never forced.
- v2.5: **Perspective scaling** — World Two shifts the whole world from cross-section (elevation view) to bird's-eye (plan view). Same grammar, new mental model. See §7.

**The spatial-learning ladder this game is climbing (design intent, keep in mind for every feature):**
1. Colour-direction vocabulary and screen left/right (v1)
2. Plan-then-execute: commit to a route before moving, compare after (v2 plan trace)
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

### v2 — Make it a ritual (built, except the map panel + plan trace — see below)
- **(built)** Procedural maze generation from daily date seed, 3-stage ramp. Validator-first, with the v1 hand-authored mazes kept permanently as the never-broken fallback.
- **(built)** Streak system + streak-driven difficulty (capped, as §4). One-day grace before a streak bends; two missed days start a new count.
- **(built)** Gems + dead-end temptations + the dog's gem-sense (the ear prick). World One only; the desert's treasure joke stays the second hat.
- **(built)** Journal: stamped entries per completed run (date, stats, maze thumbnail — footprints in the desert). Shared across worlds. localStorage array, capped at ~200 and quota-safe (trims oldest first). Minimal surface in v2: the fresh page shows on the run-complete card; a browsable book waits. (The referenced Creature Keepers file has since been removed from the repo, so the same journaling *shape* was implemented independently.)
- *pending* — Map panel for the Navigator alongside the live view.
- *pending* — **The plan trace (key spatial-learning feature):** during the briefing, the Navigator traces her intended route on the maze with her finger; the game inks it as a soft grey line. (This is the one sanctioned Navigator touch, and only during briefing.) At the reveal, the plan line is drawn underneath the actual red path. Where they diverge is the teaching moment — "we planned green here but went red!" No scoring, no judgement, ever: the plan is a conversation piece, not a target. Skippable — some days you just dig. Config: `planTraceEnabled`.

### v2.5 — World Two: The Hat (built — see §7 for the full spec)
- **(built)** H1: three hand-authored overworld stages, the two tortoises, landmarks, shell-bonks, footprint reveal, sunset scene. World picker at boot.
- **(built)** H2: procedural desert generation on the shared daily seed; landmark-aware generation (every junction within one tile of a distinct nameable landmark; the nameable vocabulary grew to seven so big stages stay unambiguous); journal entries with footprint thumbnails.
- **Do not start v2.5 until the tunnel ritual has demonstrably stuck** (streaks happening without prompting) — this gate was met before the Hat was built.

### v3 — Deepen the learning
- Map mode and Memory mode stages (Navigator information scaling, §3) — in both worlds.
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
- The reveal spread shows the whole desert from above with the Seeker's **footprint trail** dotted through the sand, shell-bonk stars at every bonk, the plan-trace line underneath (v2 systems shared).
- **Run complete:** the Seeker reaches the hat and puts it on. Final scene: both tortoises side by side on the big rock, one hat between them, watching the sunset. Nobody says anything. After a beat, the snack rises: **chocolate milk and animal biscuits** — same snack, both worlds, because the ritual is the ritual.

### Art direction (delta from §5)
- Palette shifts warm and pale: sand, sage-green scrub, long shadows. Keep the same one-saturated-accent rule — the hat is the accent (a dusty red hat, `gem` #B23A48 earns its keep here).
- Same flat colour fields, paper grain, no gradients, 1px max outlines. Shadows are single flat shapes cast consistently to one side — in plan view, shadows are what make landmarks readable, so they matter more here.
- Sunset run-complete scene rendered side-on (the one elevation shot in this world), like the last page of a picture book.

### World picker
- At boot, a simple picker card: **GREEN button = Tunnels, RED button = The Hat.** The buttons teach themselves. (Chord = replay whichever world you played last.) No menus, no scrolling, no third touch target.
- World Two unlocks after `hatUnlockRuns` completed tunnel runs (default 5) — enough to prove the ritual first. Until then, the picker doesn't exist and Tunnels boots directly.

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
  planTraceEnabled: true,   // briefing finger-trace + plan-vs-actual reveal (v2)
  lineOfSight: false,       // Watcher sees only what's visible from the rock (v3, World Two)
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
- Accidental exits are catastrophic mid-stage (eyes closed!): no touch targets other than the two buttons during play; ignore multi-touch outside them. The plan trace (v2) accepts canvas touches **only during the briefing phase**.

## 10. Tone Rules (for every string and sound in the game)

Bumps and bonks are funny. Nothing is ever a failure. The game never says "wrong", "oops" in a scolding way, or shows a red X. Stats celebrate ("smoothest run yet!") and never shame. The plan trace is never scored against — divergence is a story, not an error. The Driver being helpless is the joke; the Navigator being capable is the point. The tortoises never speak. The hat is never explained.
