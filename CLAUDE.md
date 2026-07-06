# Blind Navigation — Digging Prototype

A cooperative maze game for a child and a blindfolded adult, played on one
phone. The whole game is `index.html` — a single self-contained file with no
build step or dependencies. Open it in a browser to play.

## The two roles

- **Driver (adult, eyes closed):** holds the phone and steers the drill using
  only two buttons. Never sees the screen; relies on the navigator's voice and
  on haptic feedback.
- **Navigator (child):** watches the map and calls out turns. The cooperative
  challenge is translating what they see into the driver's frame of reference
  (when the drill faces down-screen, the driver's "left" is the screen's right).

## Hard constraint: two buttons only

The driver has exactly two inputs — left and right. Never add more controls
(no D-pad, no forward button). The drill digs forward automatically; the two
buttons turn it 90° relative to its current heading. Any new mechanic must fit
within these two inputs.

## Key design mechanic: colour-coded left and right

Left is **green**, right is **red** — always. This teaches the child left and
right while signalling control positions to the blindfolded adult. The
on-screen controls are coloured to match this directional coding (currently
`#4CAF50` green for left, `#F44336` red for right). Keep any future UI, map
markers, or instructions consistent with green=left, red=right.

## Design choices

- **Wrong turns bounce back or slow down — never restart.** Hitting dirt
  stalls the drill in place; failure costs time, not progress.
- **Haptic feedback and celebratory audio signal progress.** Vibration tells
  the eyes-closed driver they're stalled; success should be audible and
  joyful, not just visual.
- **Only the adult is blind.** The child navigator sees everything in real
  time — map, drill position, facing, and timer. The challenge is
  communicating what they see into the driver's frame of reference, not
  remembering it.
- **Personal best timer, not a leaderboard.** The pair beats *their own* time
  together; there is no competition between players.
- **Start simple: a two-turn maze with one deliberate dead end** to force
  active guidance from the navigator. The current 5×5 maze follows this
  (two turns to the treasure, dead end at grid (1,4)).

## Current implementation notes

- Maze is a 5×5 grid in `index.html`: `0` = solid dirt (wall), `1` = tunnel,
  `2` = treasure. Start at (3,4), treasure at (1,0).
- The drill advances one tile per tick (`TICK_MS = 700`) in its heading
  direction; `turn('left'|'right')` rotates the heading 90°. Tune pace via
  `TICK_MS` if playtesting shows the navigator can't keep up.
- A yellow drill-tip dot shows the navigator the current facing.
- Rounds start (and restart) via a tap on the full-screen overlay; that tap
  also unlocks the Web Audio context, which browsers require a user gesture
  for.
- Timer HUD sits above the map; the personal best persists in
  `localStorage` under the key `bestTime` (milliseconds).
- Every game event has both haptic and audio feedback, since the driver
  can't see and not every phone vibrates: a soft blip per tile advanced, a
  low thud when stalled, and a rising jingle plus celebration vibration
  pattern on a win.
- Arrow keys (left/right) mirror the buttons for desktop testing.
