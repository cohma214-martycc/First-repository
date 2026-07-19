# TUNNELS — Build Plan: v2 → v2.5 (The Hat)

Planning document only — no code lands with this file. CLAUDE.md remains the
source of truth for *what* the game is; this file is the plan for *how and in
what order* the next versions get built. When the two disagree, CLAUDE.md wins.

**Status:** v1 and v1.1 are shipped and live. The v2.5 gate in CLAUDE.md §6
("do not start until streaks are happening without prompting") has been met in
the only way that counts: the family is playing daily. Note the pleasing
irony that the *ritual* streak is real before the *streak system* exists in
code — v2's job is to catch up with reality.

---

## Constraints that shape every milestone (from CLAUDE.md)

- Single self-contained `index.html`, no build step, offline-capable.
- Two buttons only, forever. Green = screen-left, red = screen-right,
  chord = straight on. The screen-direction invariant (junctions only on
  vertical travel) applies to **generated** mazes and to World Two.
- Both worlds share every system that isn't world-specific.
- `tunnels:*` localStorage namespace, shared across worlds — one streak,
  one journal.
- Nothing is ever a failure; the tone rules apply to every new string.

## Milestones (each independently shippable, in order)

### M1 — Daily seed + procedural tunnel generation (v2)
The backbone. Everything later hangs off this.
- Date → seed: local calendar date string (YYYY-MM-DD) hashed to a 32-bit
  seed; `seedOverride` bypasses for testing. Same seed → same three stages
  for everyone, replayable all day for personal best.
- Generator (per stage, from `stageJunctions` / `stageDeadEnds` /
  `mazeCols` / `mazeRows`): carve a main path top-to-bottom built from
  vertical runs joined by horizontal connectors; junctions placed only on
  vertical runs (invariant by construction, not by luck); mix of T's and
  straight-on chord junctions rising with stage index; dead-end branches
  with a depth budget, at least one agonisingly near the exit.
- Validator (runs on every generated stage): exactly one route start→exit,
  no loops, junction count correct, every junction's answer set is a subset
  of {green, red, both}, no junction on horizontal travel. On failure,
  regenerate with a derived seed (bounded retries), then fall back to the
  v1 hand-authored stage for that slot — the pair must never see a broken
  maze.
- The v1 hand-authored stages stay in the file permanently as the fallback
  and as generator regression fixtures.
- Tests: headless sweep over ~200 seeds — auto-solve each generated run,
  assert validator invariants, assert difficulty monotonicity across the
  three stages.

### M2 — Streak system + capped difficulty (v2)
- `tunnels:streak` = { count, lastPlayedDay }. A day counts when a full
  3-stage run completes. Rules pending guidance (see Open Questions #2);
  default: strict consecutive local-calendar days.
- Difficulty nudges: every `streakDifficultyStep` days, one lever moves
  (junction count first, then dead-end depth, then dig speed), capped at
  `streakDifficultyCap`; a broken streak drops two notches, never to zero.
  All nudges applied as deltas to the generator inputs — never to the
  hand-authored fallbacks.
- Streak shown quietly on the run-complete card ("Day 6 of digging
  together"). Never shown mid-stage; never shames a break (a lapsed streak
  simply starts a new count — no "you lost your streak" copy, per tone
  rules).

### M3 — Collectible gems + the dog's gem-sense (v2)
- `gemCount` gems per stage, `gemDeadEndBias` of them inside dead ends —
  the deliberate detour conversation. Collection is automatic on touch;
  "Found a gem!"; gems tally into stats and (M4) the journal. Never gate
  progress. Decorative treasure stays untouched and untouchable.
- Dog ear-prick when the digger is within ~2 tiles of an uncollected gem —
  drawn only, never sounded (a whisper for the Navigator).

### M4 — Journal (v2)
- On run complete, stamp `tunnels:journal[]`: date, seed, total time,
  bumps, gems, streak day, and a small maze thumbnail (offscreen canvas of
  the reveal spread → JPEG data URL, ~10–15 KB each; cap the journal at
  ~200 entries, oldest dropped).
- v2 surface is deliberately minimal: the run-complete screen shows the
  freshly stamped page. A browsable journal (flipping past days) waits
  unless guidance says otherwise (Open Questions #4).

### M5 — Map panel + plan trace (v2)
- Plan trace at briefing: the one sanctioned Navigator touch. Finger
  drags from the start; the game snaps the ink to corridor centres
  (small-finger-friendly; pending #3), draws it soft grey. Skippable —
  some days you just dig. At reveal it sits under the red actual-path ink.
  Never scored; divergence is a story.
- Canvas accepts touches only during briefing (enforced in code, not
  convention).
- Map panel groundwork: `positionUpdateAtJunctionsOnly` made real (marker
  updates at junctions when set). Full map/memory *modes* remain v3.

### M6 — World Two: The Hat, hand-authored (v2.5 H1)
- **Refactor first, features second:** extract the world-agnostic engine
  (phases, chord input, junction logic, bounce, timers, reveal flow,
  stats, persistence, audio scaffolding) from World One's presentation
  (renderer, palette, sprites, sound voices, movement drawing rules).
  The full Playwright suite must pass on World One before any Hat feature
  starts — the refactor ships as its own commit.
- Then: three hand-authored desert stages (same ramp shape as v1's),
  top-down renderer with flat one-side shadows, the Seeker (90°-step
  rotation — plan view is exempt from the upright rule), the Watcher's
  slow head-tracking, shell-bonks reusing the bump system with drier
  sounds, 3–5 nameable landmarks per stage, the sunning lizard, the
  half-buried second hat, footprint-trail reveal, sunset scene into the
  shared snack.
- World picker card at boot: green = Tunnels, red = The Hat, chord =
  last played. Appears only once `tunnels:runs` ≥ `hatUnlockRuns` (5) —
  already true for this family, so the picker shows on first launch after
  shipping. Until then Tunnels boots directly.

### M7 — Procedural desert + landmark-aware generation (v2.5 H2)
- Same daily seed, third derived stream. The M1 generator gains a
  "landmark pass": every junction within `landmarkJunctionRadius` of a
  nameable landmark, landmarks distinct within a stage.
- Journal entries for Hat runs use footprint thumbnails; streaks/bests
  shared — one ritual, two worlds.

## Cross-cutting

- **Storage migration:** v1 keys (`best`, `runs`, `config`) keep working
  untouched; v2 adds `streak`, `journal`, and per-seed bests under
  `tunnels:seedbest:<seed>`. All-time bests carry over — nothing the pair
  earned is ever reset.
- **Testing:** the existing Playwright e2e stays green at every milestone;
  M1 adds the generator property sweep; M6 adds a Hat e2e (same skeleton,
  bonk instead of bump). Screenshot review at each milestone against §5/§7
  art rules.
- **Size guardrail:** single file; keep total under ~250 KB so it loads
  fast on the backyard wifi. Journal thumbnails live in localStorage, not
  the file.

## Risks and mitigations

1. **Generator breaks the invariant or produces unsolvable mazes** → the
   validator is written *before* the generator, the hand-authored stages
   are a permanent fallback, and the 200-seed sweep runs in CI-style
   testing before any push.
2. **Two worlds tangle the codebase** → M6's engine/world split is its own
   gated commit with the full suite passing before any desert code lands.
3. **Plan trace frustrates small fingers** → snap-to-corridor, generous
   touch radius, and skippable-by-chord; if playtest says it's fiddly, it
   waits — it's a teaching feature, not a toll.
4. **Difficulty creep outpaces a 4.5-year-old** → every nudge is one lever,
   capped, and reversible by two notches on a break; the floor is always
   winnable-on-an-off-day.

## Open questions (guidance wanted before M-numbers start)

1. **Sequencing.** CLAUDE.md's roadmap runs all of v2 before v2.5, but the
   Hat's gate is met and M6/M7 only truly depend on M1 (+M2 for shared
   streaks). Option A: strict order M1→M5 then the Hat. Option B: M1→M2,
   then jump to M6 (Hat, hand-authored) while gems/journal/plan-trace
   (M3–M5) follow after. Recommendation: **B** — the ritual is hot now,
   and the Hat rewards it; gems and the journal deepen it later.
2. **Streak rules.** What keeps a streak alive? (a) Strict: a completed run
   every consecutive local calendar day. (b) One-day grace: a single
   missed day bends but doesn't break (life happens on a Tuesday).
   Recommendation: **(b) one-day grace** — the difficulty system already
   assumes streaks break sometimes, but the ritual is a family habit, not
   a compliance program.
3. **Plan trace ink.** Snap-to-corridor (clean, forgiving, recommended) or
   freeform finger ink (more "hers", messier under the reveal)?
4. **Journal surface.** Minimal (auto-stamp + show the new page on run
   complete, recommended for v2) or a browsable book UI now?

Default if no guidance arrives: recommendations above (B, one-day grace,
snap-to-corridor, minimal journal).
