# TUNNELS — Cooperative Blind-Digging Maze Game

A two-player asymmetric HTML5 game for one iPhone. The adult digs through underground tunnels with their **eyes closed**, steered only by the child's voice. The child reads the screen and the map. Built for Alma (child, Navigator) and a grown-up (Driver). One shared device, one shared laugh.

This file is the source of truth for Claude Code. Build exactly to this spec unless a constraint is physically impossible — flag conflicts, don't silently redesign.

---

## 1. The Two Roles

### Driver (adult — eyes closed)
- Holds the phone in portrait, thumbs resting on two large fixed buttons.
- **Never opens eyes during a stage.** Eyes open only at checkpoints and stage completion.
- Only inputs: LEFT button and RIGHT button. Nothing else, ever, in any version.
- Experiences the game through sound, haptics, and Alma's voice.

### Navigator (child — eyes open)
- Watches the screen: sees the maze cross-section, the digger character, and (from v2) a map panel.
- Gives verbal directions using the colour vocabulary (below).
- Cannot touch the screen during a stage. Her controller is her voice.

### Colour vocabulary (core mechanic — non-negotiable)
- LEFT = **GREEN**. RIGHT = **RED** ("Red" and "Right" share the R — the mnemonic is deliberate).
- Alma calls "green!" or "red!", not "left/right" (though both work socially — the game never punishes vocabulary).
- The two on-screen buttons are large solid blocks of these colours, fixed to the bottom-left and bottom-right corners so the Driver's thumbs never hunt. Minimum touch target: bottom 22% of screen height, each button 50% of screen width. They are the only touch targets during play.
- Colours must be a settings value, not hard-coded (see §7) — if green/red proves confusing or inaccessible we may swap to blue/orange.

---

## 2. Core Loop (one stage)

1. **Briefing (eyes open):** Both players see the stage map. They talk through the route, agree on a plan, giggle. Driver taps both buttons simultaneously to confirm "eyes closing now" — this is the ready signal and starts the stage.
2. **Digging (eyes closed):** The digger auto-digs forward at constant speed. At each junction the digger **pauses** and an audio cue plays (a soft "hm?" plus a haptic tick). The Driver presses GREEN or RIGHT/RED as directed. No press within the junction window = digger continues straight if straight exists, otherwise bumps.
3. **Bumps:** Hitting rock/roots/dead-end soil = a comedy event, not a failure. Screen shake, dust puff, a silly *thud-boing* sound, strong haptic buzz, the digger's helmet slips over its eyes. The digger **bounces back to the last junction** and re-pauses. Bumps are counted but never end a run.
4. **Breakthrough (stage complete):** The digger breaks through into a cavern or up through the surface — a soft, satisfying collapse of dirt. Fanfare, haptic celebration. On-screen banner: "OPEN YOUR EYES!"
5. **Reveal (eyes open):** The full maze is shown with the actual path traced in a contrasting line, **bump locations marked with little stars/bruise icons** — this is the laugh-together moment. Stats: time, bumps, personal best comparison. Both players tap their button together to continue.

A **run** = 3 stages (easy → medium → spicy). A run takes roughly 5–10 minutes total. Short is a feature.

---

## 3. Movement Model & Complexity Scaling (two buttons only, forever)

The digger auto-moves. The Driver only ever chooses at junctions. Complexity therefore scales through **maze topology and timing**, never through added inputs:

| Lever | Easy | Harder |
|---|---|---|
| Junctions per stage | 2 | 8+ |
| Junction types | T-junctions (forced choice) | 4-way crossings (straight is an option — *not pressing* becomes a move) |
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

---

## 4. Maze Completion & Progression

- **Stage complete:** breakthrough animation → reveal → stats → next stage.
- **Run complete (3 stages):** the digger surfaces into the backyard at sunset. Journal entry is stamped automatically: date, total time, total bumps, gems, a tiny thumbnail of the final maze with the path drawn. Streak counter increments.
- **Daily seed:** each calendar day generates its mazes from a date-based seed (same approach as CircleSquareTriangle) — everyone gets the same day's tunnels, replayable for personal best.
- **Difficulty over time:** difficulty is driven by **streak, gently, with a cap** — not by raw calendar. Streak 1–3: baseline. Each streak milestone nudges one lever (a junction here, a shorter pause window there) up to a ceiling at streak ~14. Breaking a streak drops difficulty back two notches, never to zero. Rationale: the game must stay winnable by a pair having an off day; frustration kills the ritual.
- Personal best is stored **per daily seed** and as an all-time "smoothest run" (fewest bumps) and "fastest run".

---

## 5. Art Direction — picture-book cross-section

Inspired by the flat, deadpan, earth-toned cross-section style of *Sam and Dave Dig a Hole* — an **homage in spirit, all original assets**. Do not copy characters, compositions, or trade dress.

- **View:** side-on cross-section of the earth, like the page of a picture book. A thin strip of pale sky at the very top with a small house, a bare tree, maybe washing on a line. Everything below is soil.
- **Palette (tokens, tune in settings):**
  - `soil-deep` #4A3728 (undug earth, matte)
  - `soil-warm` #6B4F35 (mid earth, subtle grain texture)
  - `tunnel` #C9A876 (dug path — visibly lighter, like exposed dry dirt)
  - `sky` #DCE3DD (pale, quiet, slightly grey-green)
  - `gem` #B23A48 (dusty red — the one saturated accent on screen)
  - `button-left` #5E7C4A (green), `button-right` #A6423A (red) — muted to sit inside the palette, still unmistakably green/red
- **Texture:** flat colour fields with a very subtle paper grain overlay. No gradients, no gloss, no outlines thicker than 1px. Rocks are soft blobs a shade darker than soil. Roots dangle from the surface. The occasional buried oddity (bone, old boot, teacup) as silent jokes in the dirt — decorative only.
- **Characters:** small, simple, deadpan. The digger is a little figure with a hard hat and a spade, drawn in 2–3 flat colours. **A small dog companion** trots along the tunnel behind the digger; when a gem is nearby the dog's ear pricks up — a visual whisper only the Navigator sees. (The dog always knows.)
- **Motion:** minimal and dry. The dig is a steady rhythmic animation. Bumps are the biggest motion on screen. The breakthrough is dirt crumbling away in chunky flat particles. Respect `prefers-reduced-motion`.
- **Typography:** one rounded, friendly display face for banners ("OPEN YOUR EYES!") and a plain body face for stats. Sentence case everywhere. Words on screen are for the Navigator — keep them short enough for an early reader: "Ready?", "Go!", "Found a gem!", "You made it!"
- **Signature element:** the eyes-open **reveal page** — the whole maze rendered like a finished picture-book spread with the pair's wobbly path inked through it and bump-stars marking every thud. This is the screenshot-worthy moment; polish it hardest.

---

## 6. Version Roadmap

### v1 — Prove the ritual (build this first, complete and polished)
- One run = 3 stages, fixed hand-authored mazes (not procedural yet): stage 1 = two junctions + one dead end; stages 2–3 slightly larger.
- Full colour-button control scheme, junction pause (untimed — digger waits), bumps with sound/haptics/shake, breakthrough, reveal screen with path trace and bump stars, stats.
- Navigator sees live digger position.
- Simultaneous-press ready signal and continue signal.
- localStorage: personal bests, total runs.
- Dev settings panel (§7) — in from day one.
- **No** daily seeds, streaks, gems, dog, or journal yet. Ship the loop.

### v2 — Make it a ritual
- Procedural maze generation from daily date seed, 3-stage ramp.
- Streak system + streak-driven difficulty (capped, as §4).
- Gems + the dog companion + dead-end temptations.
- Journal: stamped entries per completed run (date, stats, maze thumbnail). Same localStorage journaling pattern as Creature Keepers v3.
- Map panel for the Navigator alongside the live view.

### v3 — Deepen the learning
- Map mode and Memory mode stages (Navigator information scaling, §3).
- Timed junction windows as an unlockable "spicy" modifier.
- Loops and soft-soil chutes in generation.
- Sound design pass: each junction type gets its own audio cue so the Driver starts learning the maze by ear — a quiet second literacy.

### v4 — Stretch (only if the ritual has stuck)
- **Role swap:** ultra-simple mazes where Alma drives eyes-closed and the adult navigates using the colour words. The full-circle moment.
- Two-device mode (Navigator on a second screen with map only).
- Maze editor: draw a maze with your finger, save it for tomorrow's partner.

---

## 7. Dev Settings & Tuning Panel

All tunables live in a single `CONFIG` object at the top of the file, persisted to localStorage, editable at runtime via a hidden dev panel (**tap the sky 5 times**). Every value overridable by URL param (`?digSpeed=1.4`). Include a "reset to defaults" and an "export config as JSON" button.

```js
const CONFIG = {
  // movement
  digSpeed: 1.0,            // tiles per second between junctions
  junctionPauseMs: 0,       // 0 = wait forever (v1 default); >0 = timed window
  bumpBouncePx: 24,         // comedy bounce distance
  bumpStunMs: 900,          // pause after bump before re-offering the junction
  // maze generation (v2+)
  stageJunctions: [2, 4, 6],    // per stage in a run
  stageDeadEnds: [1, 2, 3],
  mazeCols: 9, mazeRows: 12,
  gemCount: 2, gemDeadEndBias: 0.7,  // fraction of gems placed in dead ends
  // difficulty ramp
  streakDifficultyStep: 3,  // every N streak days, nudge one lever
  streakDifficultyCap: 14,
  // controls & accessibility
  colourLeft: '#5E7C4A', colourRight: '#A6423A',
  buttonHeightPct: 22,
  hapticsEnabled: true, audioVolume: 0.8,
  // navigator information
  showLiveDigger: true,     // v1 true; map/memory modes flip this
  positionUpdateAtJunctionsOnly: false,
  // debug
  debugOverlay: false,      // show grid, seed, junction ids
  seedOverride: null,       // force a specific daily seed for testing
};
```

Rule: **no magic numbers in gameplay code** — if it affects feel, it goes in CONFIG.

## 8. Technical Constraints

- Single self-contained `index.html` — inline CSS/JS, no build step, no external network calls. Canvas rendering for the world; DOM for buttons and panels.
- Target: iPhone Safari, portrait, one-handed-thumbs ergonomics. Must run offline once loaded.
- Persistence: `localStorage` only (journal, bests, streak, config). Namespace keys `tunnels:*`.
- Audio: Web Audio API, synthesised or tiny embedded sounds — must work after a user gesture (the ready-signal press unlocks audio).
- Haptics: `navigator.vibrate()` where available; degrade silently.
- The screen must **never sleep mid-stage**: use the Screen Wake Lock API with graceful fallback.
- Accidental exits are catastrophic mid-stage (eyes closed!): no touch targets other than the two buttons during play; ignore multi-touch outside them.

## 9. Tone Rules (for every string and sound in the game)

Bumps are funny. Nothing is ever a failure. The game never says "wrong", "oops" in a scolding way, or shows a red X. Stats celebrate ("smoothest run yet!") and never shame. The Driver being helpless is the joke; the Navigator being capable is the point.
