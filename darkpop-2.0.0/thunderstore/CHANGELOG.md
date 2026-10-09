# Changelog

## 2.0.0 (ReSkate 1.1.8, Revert Boost included; tidied; on ReSkate 1.1.8)
- Rebuilt on **ReSkate 1.1.8** with **Revert Boost 2.0** in the same build (its switch and slider sit above Dark
  Pop in Insert > SKATER > BOOSTS). One ReSkate.dll carries both.
- The menu: one **Dark Pop** switch with everything that tested out behind it, and a **Dark Pop experiments**
  fold-out holding the probe, the new experiment and the long-sink knobs, with the settled parts in a nested
  "Settled (leave on)" fold-out.
- 1.10.0 play test (the user, 35 landings): the Pop strength write to context +0x4f0 was gone by the sink's first
  tick every time (the read-back was ~0.7x the original value whether written or not; the game rewrites that value
  each tick) and the flights did not change at 6, 10 or 0. **Pop strength defaults to 0** and stays only for its log line.
- **Hold the landing** (new experiment, on): the chooser's 201 case was read (0x47460b9): with context +0x13cc
  bit 20 set it keeps 201; with it clear and +0x18c0 below -0.2 it drops to 200, which is what cut the nollie 360
  flip's landing sequence short. On an armed flight in 201 with the board dark and resting, the landing sequence
  started (+0x13c4 0x880000) and no landing stage yet, the bit is raised before the chooser and taken down after,
  for at most 10 calls, so the game can stage the landing itself (a staged landing carries the landing crouch the
  pop bounces: staged landings enter the sink at -5..-7 m/s and fly 1.0-2.1 s, caught ones at ~-2 and fly 0.4-0.9 s).
  Untested. If the stage never comes the timed-out catch still fires after the hold.
- Probe: three new fast-CSV columns, `gate_18c0`, `gate_1838` (the 201 case's gate floats) and `w13f8`.

## 1.10.0 (pop strength; on ReSkate 1.1.1)
- 1.9.0 play test (the user, 56 landings): the timed-out catch fired on 30 landings (the nollie 360 flips among
  them) and every one of them popped at sink 0, but half-size: median 0.65 s against 1.30 s for the landings the
  game staged itself. At sink 11 the caught ones were 0.15-0.27 s hops with the board flipped at the freeze's
  release, the 1.8 failure again; the three biggest flights of the day (2.30, 1.88, 1.98 s) were staged sink-11
  landings whose board happened to stay dark through the release.
- Why half-size: the probe's `pup_*` columns are the skater context's velocity (+0x4f0; x/z match the board's at
  0.99). Its vertical part on the sink-entry tick predicts the pop within each group (corr -0.67 caught, -0.55
  staged): staged landings enter at -5..-7 m/s, timed-out ones mostly at -2 (the board has rested 6+ ticks and the
  velocity has decayed); the pop's `bv_y` two ticks after the exit is ~1.1x it. The pop is a bounce of that
  number. Fits 1.3 (no plant, no pop) and the sink-11 hops (velocity zeroed for 11 ticks, nothing to bounce).
- **Pop strength** (new number, default 6.0 m/s): on the tick a landing goes to 601, staged or caught, the
  context's vertical velocity is floored to -6 if it is less negative than that; nothing else is touched. 0 = off.
  The sink line logs the velocity found, whether it was floored and the value read back on the first sink tick.
  Untested: whether the write reaches the plant (the read-back says) and whether the bounce follows it.

## 1.9.0 (timed-out landings caught; sink 0 and no freeze by default; on ReSkate 1.1.1)
- 1.8.0 play tests (the user, 8 Oct): with the freeze (sink 11, 7 landings) the board's axes and velocity were
  identical on every sink tick and every exit was dark, but only two of seven popped big (1.64, 2.12 s), four
  were hops. The pop's vertical velocity (`bv_y` 5-9.6 m/s, flight time follows it) appears on the SECOND air tick
  after the sink, and only when the board is still dark on the first; the freeze's release flipped the board
  (-0.94 -> +0.73 in one tick) in five of seven. The freeze is not the way. Sink 0 (two sessions, 29 real
  landings): every one 0.68-2.10 s, 25 over 1 s.
- **Defaults: Sink length 0, Feet-only hold off** (the 1.0 configuration). Feet-only hold stays as an experiment.
- **Catch timed-out landings** (new toggle, on): the landing window opens on the game's landing stage
  (+0x13c8 bit 26), raised in 201 at the end of a short sequence in +0x13c4 (0x300000, then 0x880000, then the
  stage). 201 is capped at ~50 ticks; when the cap comes first the game drops 201 -> 200 with the sequence
  unfinished, the board (already resting upside down on the ground) rolls over in physics and the landing rides
  away with no pop. 8 Oct: 10 of 19 armed landings, among them every nollie 360 flip (a longer animation: 0x880000
  came on 2 ticks before the cap, the stage never). Now an armed flight the game wants to send 201 -> 200 with the
  board dark and resting (|vertical velocity| < 2 m/s on the last 3 chooser calls) is routed to 601 like the
  landing tick would have been. Logged as `timed-out catch on (N caught so far)` and `a timed-out landing, caught`.
  Untested: whether a sink entered without the game's own landing stage pops the same (the 1.0 dismount catch did).
- Board freeze counter fixed: it compared each axis row's 4th float too and called drift there "overwritten"
  while the probe showed the axes identical. Axes x/y/z only now.

## 1.8.0 (board freeze in the sink; on ReSkate 1.1.1)
- 1.7.0 play test (the user, 40 landings): the stage probe settled where the roll happens. On every sink tick
  `up_pre_tick`, `up_post_tick` and `up_post_move` were identical and equal to the previous row's `board_up`: the
  board does not change inside the footplant tick, the solver or the mover. It changes between chooser calls, in the
  physics step; and its vertical velocity goes positive (+1 to +4.6 m/s) in the sinks that roll, around zero in the
  ones that stay dark. Reading: the plant poses the rig into the upside-down board and the physics step kicks it
  over from below. The `bw_*` columns read 0 throughout: +0x80 is not the angular velocity.
- Adaptive sink: fired on ticks 2-5 of the first seven sinks (sink 11) and turned the rolled half into mids
  (0.6-1.1 s), no bigs. Sink 0 afterwards (30 landings): mostly 1.0-1.9 s, the most consistent setting so far.
- **Board freeze** (part of Feet-only hold): on a routed sink's first tick the board body's transform (+0x20..+0x5c:
  three axis rows and the position row) is taken; on every later sink tick it is written back and the velocity
  (+0x70) zeroed with the +0x60 bit-3 flag, the write keep speed uses. At each later chooser call the axes are
  compared with the snapshot to see whether the previous write survived the physics step (within 0.05).
- Log: the sink line gains `board freeze wrote the board's transform on N tick(s), found held on N, overwritten on
  N`. Held = the engine honours the write and the board should stay dark; overwritten = the engine restores its own
  actor state each step and a transform write is not the way (the next thing is the physics scene's own API).
- Untested in game. For the test: Feet-only hold on, **Pop when the board rolls off**, Sink length 11, so the freeze
  is seen on its own. Syntax-checked against ReSkate 1.1.1 with diagnostics identical to the 1.2.0 tree.

## 1.7.0 (stage probe + adaptive sink; on ReSkate 1.1.1)
- 1.6.0 play test (the user, 16 landings): the mover hook ran on every sink tick (`captured on 1, held on 11`) and
  the board still rolled in 12 of 16. The mover's orientation input is not the roll either. Four builds have each
  removed one thing from the footplant tick (its attaches; all attaches; its orientation input) and the roll
  survived every one; the rule held every time (4 of 4 big pops from dark exits, 0 from rolled). The roll is being
  written somewhere the tick does not control, most likely the animation/IK pass that runs after the chooser.
- **Stage probe** (new fast CSV columns): `up_pre_tick`, `up_post_tick`, `up_post_move` are the board's up axis as
  last sampled at the footplant tick's entry, after it returned, and after the skater mover returned (9 = no sample
  since the last chooser call); `bv_x/y/z` and `bw_x/y/z` are the board body's +0x70 linear and +0x80 angular
  (unverified) velocities at the chooser. Reading them: if `up_pre_tick` on a sink tick already differs from
  `board_up` on the previous row, the roll happens between the chooser and the tick (physics/IK); if `up_post_tick`
  differs from `up_pre_tick`, it is in the tick; if `board_up` differs from `up_post_tick`, after it. An orientation
  change with zero angular velocity is a pose write, not physics.
- **Pop when the board rolls** (new toggle, on): in a routed sink, the tick the board's up axis rises past -0.75 the
  stay bit is cancelled, whatever the sink length, so the chooser pops while the board is still dark. On the
  traces so far it would fire on sink tick 2-4 of the rolled half. It does not explain the roll; it catches the pop
  before it. The sink line says `adaptive sink ended the sink on tick N` when it fired.
- The 1.6 orientation hold is gone (the mover hook stays, sampling only). Feet-only hold = 1.4 (the plant's own
  attaches cleared, a two-tick attach requested).
- Untested in game. Syntax-checked against ReSkate 1.1.1 with diagnostics identical to the 1.2.0 tree.

## 1.6.0 (the plant's orientation held through the sink; on ReSkate 1.1.1)
- 1.5.0 play test (the user, 27 landings, no attach at all in the sink): the board still rolled in 18 of 27 sinks,
  same tick-1-2 climb, same rule (every big pop from a dark exit, every rolled sink a hop). The attaches were never
  the roll. The session ended off-board after a post-pop dismount (the known class, 200 -> 504).
- Read since: the solver's tail (0x4820630..) builds a full pose, position and an orientation from the plant normal
  and the foot direction, blended toward its target at two tuning rates (the steady climb), and hands it to the
  mover 0x47752a0 with the flag set; the mover writes it into the rig root (+0x49e0..+0x4a10), publishes the world
  transform, raises context+0x13c4 bit 14 and poses the rig and its bodies, the board among them. The roll is the
  orientation half of the plant; the position half is the spring (1.3).
- **Orientation hold** (part of Feet-only hold, new fingerprint `skater_mover_contract`): a hook on the mover. On a
  routed sink tick for the local skater, the first call's two direction vectors (input +0x10, +0x20) are kept and
  written over every later call's, so the rig keeps the orientation the landing left it with. The position is not
  touched. Every other caller of the mover is untouched (the hook only acts while the sink is live for the armed
  skater's context).
- The two-tick foot attach of 1.4 is back (it is not the roll, and the feet sat right with it).
- Log: `Dark Pop orientation hold ready` at start-up; after each sink `Dark Pop orientation hold: captured on 1
  tick(s), held on N tick(s)` (N = sink ticks - 1). If `captured on 0`, the mover was not called in the sink.
- Prediction: the board stays dark through every sink. Open: whether the orientation half is part of the spring; if
  the pops shrink, it is, and the adaptive sink (end the sink when the board starts to rise) is the fallback.
- Untested in game. Syntax-checked against ReSkate 1.1.1 with diagnostics identical to the 1.2.0 tree.

## 1.5.0 (no attach at all in the sink; on ReSkate 1.1.1)
- 1.4.0 play test (the user, 34 landings): the hook ran on every sink tick (`cleared on 12/15/9`), the pops were back
  (9 big, up to 2.7 s, every one from a dark exit) and so was the roll (22 of 34 sinks, the same tick-1-2 climb, every
  one a hop). So the setters' slots are not the roll. Read since: the solver (0x481f630) is a world query (raycast,
  sphere overlap, closest point) writing only the placement's +0x270 plant point, +0x280 found flag and +0x284; the
  epilogue (0x47e8860) is a real early-out with the jump flag down (0x47e8cf8 is a `ret`); `-Find` for the +0x59c0
  slot shows only writers. What fits 1.0, 1.3 and 1.4 together: the plant pulls the feet to the ground point under
  the board and any foot attach pins the feet to the board, so the board follows the feet over. 1.3 (attach, no
  plant): no roll, no pop. 1.4 and 1.0 (plant + an attach): pop and roll.
- **Feet-only hold** now requests nothing in the sink: the footplant tick runs, its attach requests are cleared, and
  no two-tick attach is requested in their place. Prediction: the board stays dark through every sink and the pop
  stays. Cost: the feet may not sit on the board during the sink (cosmetic). If the board still rolls with no attach
  at all, the roll comes from somewhere unread (the solver's tail past 0x4820630, or the foot IK itself).
- Log: `through the pop` says `attach requests cleared on N tick(s), attaches requested 0`.
- Untested in game. Syntax-checked against ReSkate 1.1.1 with diagnostics identical to the 1.2.0 tree.

## 1.4.0 (the footplant tick's attaches cleared, the plant kept; on ReSkate 1.1.1)
- 1.3.0 play test (the user, 42 landings with the tick replaced, every sink length 0-20): the board never moved
  (exit board-up -0.79 on every landing, flat through every sink) and there was no pop at all (flights 0.03-0.65 s,
  the board taking the exit alone while the skater stayed down). With the tick running again (23 landings at sink
  11 and 3) the 1.4-2.3 s pops and the roll-over hops both came back. So the tick's two halves are two things: the
  solver's plant loads the pop; the per-tick foot attaches roll the board.
- **Feet-only hold** now leaves the footplant tick to run unchanged and, right after it returns in a routed sink,
  clears the request slots its attach setters filled on the skater (regular bone +0x59c0..+0x59fc, switch bone
  +0x5a00..+0x5a3c, read from the setters' code), then requests the two-tick foot attach as before. Defaults
  unchanged from 1.3 (on, Sink length 11).
- Prediction, not measurement: the pop stays (the plant is untouched) and the board stays dark through the sink (no
  active attach), so the roll-over hops go. Open: whether the foot IK reads those slots before or after the tick in
  a frame, and whether the plant itself tips the board through the feet without the attach. The planted frames stay.
- Log: per landing `Dark Pop feet-only hold: the footplant's attach requests cleared on N tick(s), two-tick foot
  attach requested on N tick(s)`; the landing line prints 0 (it is written on the landing tick), the `through the
  pop` line after the flight carries the sink's counts.
- Untested in game. Syntax-checked against ReSkate 1.1.1 with diagnostics identical to the 1.2.0 tree.

## 1.3.0 (the pre-0.32.0 sink: 601's footplant tick replaced; on ReSkate 1.1.1)
- 1.2.0 play tests (the user, two sessions, 46 + 46 landings): **Feet-only hold** had no effect, on or off (sink 11:
  7 of 21 big with it off, 5 of 10 with it on; sink 5 off: 3 of 9). Its hook, on the board-placement apply, saw 0
  calls on every landing: that function is not on 601's path.
- What decides the pop (79 routed landings, both sessions): the board is dark at every sink entry (-0.8 to -1.0);
  in half the sinks it starts rolling upright on tick 1-2 and climbs steadily; every landing where it had rolled by
  the exit hopped (0 of 40 big), every landing where it was still dark popped (36 of 46 big, 1-2 s). Nothing at the
  landing tick predicts which half (board up, tilt, speed, air time, pad bits, roll rate: all flat). The user's
  60 fps frames: the mod plants the skater for the sink; the original kept sliding with the flip's speed.
- Read from both exes: state 601's per-tick controller (old 0x4435e40, now 0x4803ea0) is the same function but for
  one fork the old one had after `timer += dt`: with the removed signal X up it moved the body on a Bezier and jumped
  to the epilogue, past the tuning reads, the footplant solver (old 0x44516e0 / now 0x481f630, which plants the body)
  and the per-tick foot attaches (a type-1 attach with an offset, old 0x43a28d0 / now 0x4770c50, plus a blended one),
  which pull the feet to the deck and turn the board over. 601's step (0x4772ac0) is small: the tick, then the mover
  if the output is not valid, then the launch if the jump flag is up. The setters only fill request slots on the
  skater (+0x59c0..+0x5a3c). The epilogue (old 0x441a270 / now 0x47e8860) returns at once when the jump flag is 0.
  So 0.32.0's removal of X made every routed 601 run a real footplant on the dark landing: the plant (the 6 planted
  frames) and the roll (the hops) are one change.
- **Feet-only hold** is now a hook on the footplant tick. While the game is in 601 after a routed landing the
  original is not called: the tick's bookkeeping is done (ticked byte +0x29c, timer +0x228), the step's output is
  written empty with `valid` set (the step neither moves the body nor launches; the body stays on its physics), and
  the two-tick foot attach is requested so the feet ride the board where it lies. The pop is unchanged: the 1.0 exit
  when the stay bit is cancelled at Sink length. The apply hook is gone. Defaults: Feet-only hold **on**, Sink length
  **11**; with it off and Sink length 0 the build is 1.0.
- Not the Bezier of the old X path: 1.1 ran it with the solver still fighting it, so the body's motion is left to
  physics first. Open: whether the body still sinks without the solver (1.1's frames did not, with the solver in);
  whether the 1.0 exit pops as big when the body has not planted.
- Log: `Dark Pop feet-only hold ready: footplant tick hooked ...` at start-up; per landing `Dark Pop feet-only hold:
  the game's footplant tick replaced on N tick(s), two-tick foot attach requested on N tick(s)`. If `replaced` reads
  0 on a routed landing, the hook is not reaching 601's ticks and the result is 1.0's.
- Untested in game. Syntax-checked against ReSkate 1.1.1 with diagnostics identical to the 1.2.0 tree (clang 18
  targeting mingw-w64, -Wall -Wextra).

## 1.2.0 (feet-only hold, as an experiment; the launch removed; on ReSkate 1.1.1)
- 1.1.0 play test (the user, 25 landings with Original launch on): the port ran as written (hooks in, plant point
  live from the context on every landing, the jump fired on sink tick 5 or 16) and produced nothing: median flight
  0.07 s, no rise, 7 dismounts, the speed gone. The footplant tuning caps its jump at about 2.2 m/s (forward speed
  2.0, target pitch factor 0.08): a step-off for a footplant trick, not the dark pop. The jump theory is wrong;
  **Original launch** and **Stick steers the launch** are removed. (The old steering's two context fields read 0
  with the stick held, so +0x19c0 is not the stick in this build.)
- Read from both exes, the board-placement apply (old 0x441e740, new 0x47ecd30): on the ticks the removed signal X
  was held, the old apply computed a board position and handed it to two calls that are the game's empty stub, then
  requested a two-tick foot attach while the blend was under 0.2. The old hold did nothing to the board; it pinned
  the feet to the board as it lay. The new apply is an active placement (an offset ramped in through a type-1
  attach every tick plus a blended attach from the stored placement) that pulls the feet to the deck's grip side and
  turns the board over. The mod's hold starts the hold the old way and the game's apply then runs every hold tick:
  the frame-by-frame the user took (feet drawn through the deck in 5 frames, board flipping at the pop, feet
  pointing up while attached). The original's frames show feet on the trucks of the dark board, the body sinking
  to the ground, rising with the board still dark, the board rolling over only near the top. The placement helper's
  change (old 0x442c1c0 / new 0x47fa7c0) is minor: the old read the skater's up axis from a one-tick-old copy
  (old +0x140, written at the end of every update from context +0x4b0); the new reads it live with a velocity term.
- **Feet-only hold** (new toggle, off by default): a hook on the board-placement apply skips it while the dark
  landing window or a routed sink is open for the local skater, and requests the two-tick foot attach itself on
  every such tick (the apply's ticks and the 601 ticks, which never call the apply). The hold window's mode-1
  release at the pop is unchanged. Switching it on sets Sink length to 11 if it was 0.
- Log: `Dark Pop feet-only hold ready` at start-up; per landing `Dark Pop feet-only hold: the game's placement
  apply skipped on N tick(s), two-tick foot attach requested on N tick(s)`.
- Probe: the 14 extra columns from 1.1 stay (`plant_*`, `pup_*`, `stick_x/z`, `start_*`).
- Untested in game. With the toggle off nothing changes from 1.0.0. Syntax-checked against ReSkate 1.1.1 with
  diagnostics identical to the baseline tree (clang/mingw, -Wall -Wextra).

## 1.1.0 (the original launch, as an experiment; on ReSkate 1.1.1)
- Read from both exes (old = before patch 0.32.0): state 601 is `PHYSICS_STATE_FOOTPLANT`. Before 0.32.0 its
  controller planned the dark pop as a jump at the landing: direction from the start position to the plant point
  (context +0x4e0) with the up axis there (+0x4f0), swing = cross(up, delta), a tuning curve of the vertical fraction,
  angular speed = |swing| * curve, a target pitch clamped by two tuning values, **sink duration = pitch change /
  angular speed**, the launch velocity from the bisector of the old and new directions, a cubic Bezier for the sink,
  the feet attached for duration/dt + 4 ticks (the foot-attach setters take a tick count, not a mode), and once the
  timer passed the duration the game's own launch code fired with that velocity and a 0.2 m lift. 0.32.0 deleted
  that whole half of the controller; 601's own methods and the launch code are byte-identical in both exes. The 1.0
  pop is the body bouncing out of a jump-less one-tick footplant, which is why its size followed the board's tilt.
- **Original launch** (new toggle, off by default): the deleted half, ported into the mod. A hook on the footplant
  tick (new 0x4803ea0) computes the launch on the sink's first tick (after the game's own enter) with the game's own
  tuning-curve lookup (0x485c8a0), signed-angle function (0x47e86a0) and tuning block (0x47635e0 + offsets 0x214,
  0x218, 0x224, 0x240, 0x24c, 0x254, 0x25c, curve 0xb0), attaches the feet through the game's setters, then overrides
  the tick's output each sink tick (Bezier position with valid = 0, so the state step hands it to the old mover) and
  sets the jump flags with the velocity when the timer runs out. The stay bit is cancelled once the jump has fired
  (the Sink length slider is not used). Fallbacks: start position from the board body if the placement's +0x140 is
  empty; plant point ahead of and below the board if +0x4e0 is unusable (the log says `FALLBACK`); if the calculation
  fails, that sink runs as in 1.0. A 120-tick cap fires the jump regardless.
- **Stick steers the launch** (new toggle, off): the old steering with a blend of 1 (its two constants, old
  0x5d2757c and 0x5d4bde8, were not read).
- Probe: 14 new fast-CSV columns: `plant_*` (context +0x4e0), `pup_*` (+0x4f0), `stick_x/z` (+0x19c0/4), `start_*`
  (placement +0x140). 9 = not readable.
- Log: `Dark Pop original launch ready` at start-up; per sink `Dark Pop original launch: plant point ..., pitch a ->
  b deg, sink N s, feet attached N tick(s), launch V m/s along (...), jump fired on sink tick N`; the tuning once.
- Untested in game. With the toggle off nothing changes from 1.0.0. Syntax-checked against ReSkate 1.1.1 with
  diagnostics identical to the baseline tree (clang/mingw, -Wall -Wextra).

## 1.0.0 (one switch, sink of one tick, early dismounts caught; on ReSkate 1.1.1)
- 0.15.0 play tests (the user, four probe sessions, 98 dark landings, No Bail off, sink length stepped 4 -> 0):
  the pops were good at every sink length but their size varied, and what decided it was the board's tilt at the
  601 exit, not the speed coming in (4-15 m/s, no relationship), the air time or the landing height (all alike).
  The sink animation tips the board onto its end a little more each tick (tilt 0.3 -> 0.5 -> 0.75 over three
  ticks); a tilted board at the exit gave a hop. At sink length 0 (one tick): 21 landings, 15 pops over 1 s,
  3 of 0.7-1 s, no hops, 3 dismounts (the pop fired, flew 0.2 s with the board never flipping, and landed dark).
  At 2 ticks: 5 / 8 / 1 of 14; at 3: 5 / 1 / 1 of 7; at 4 and 5: all hops. The user: "feels good at 0, maybe
  even more consistent".
- The glue (0.15.0) changed nothing measurable (the same 35% big pops as 0.14.0, the same tilt at the exit: it
  works on the feet, the tilt is the board's) and is removed, with its code and log fields.
- **Sink length defaults to 0.** The slider stays in the advanced fold-out.
- **Catch early dismounts** (new, on by default): 6 of the 98 dark landings were never routed because the game
  went from the air (200) straight to the dismount (504) with no landing tick (201) for the route to fire on; all
  six came off a shorter flight (0.55-0.63 s against 0.72-0.80 s for every routed flip; the user's 180s). With the
  catch on, an armed flight with the board still dark whose decision is 504 straight from the air is sent to 601
  instead, as the landing tick would have been. Untested: the risk is a landing that then goes 601 -> wipeout.
  The landing line says `went to 601 (the sink; an early dismount, caught)` and counts them.
- Menu: one **Dark Pop** switch; everything else (Sink length, Pop guard, Catch early dismounts, Keep speed, One
  press one pop, Unpatch, Hold, probe) under a **Dark Pop advanced** fold-out. The "Sink and pop (601)" switch is
  gone (the route is the mod). Log lines say `sink length` instead of `stay`, and `Dark Pop enabled`.
- Untested in game. Syntax-checked against ReSkate 1.1.1 with diagnostics identical to the baseline tree
  (clang/mingw, -Wall -Wextra).

## 0.15.0 (glue in the sink, on ReSkate 1.1.1)
- 0.14.0 play test (the user, 145 dark landings, No Bail off): 130 routed, 128 sinks exited to the pop, 123 pops
  rode away, 15 dismounted, 1 bail. The pops were bimodal: 19 of 55 in the probe were 2-5.8 m, 36 were hops under
  0.7 m, and the board's roll during the sink decided which. Big pops had the board still fully upside down at the
  601 exit (up axis -0.80 median), hops had it half rolled (-0.53); landings were otherwise alike. The dismounts
  (12 of 15 at the guard's end) had the board never rolling over in the flight. The 13 landings that were not
  routed were the game popping on its own before the landing-stage bit returned (no earlier signal exists), or a
  pop's flight ending on a still-dark board; not a window problem.
- **Glue in the sink** (new toggle, on by default): foot-attach mode 2 is requested on every 601 tick, on the
  local skater's board-placement object (remembered by the entry hook; 601 never calls it) and the bone it last
  chose, and mode 1 once at the exit, so the board stays upside down through the sink and is free to flip in the
  flight. The sink line says `glue on N tick(s), released` (or `on but no placement object yet` before the first
  placement call of the session).
- Built on ReSkate 1.1.1 (same game build 20260929; the only difference in a file this mod touches is overlay.h,
  no conflict) so a tester on 1.1.1 can run it. Untested in game. Syntax-checked against ReSkate 1.1.1 with no new
  diagnostics (clang/mingw, -Wall -Wextra).

## 0.14.1 (same mod on ReSkate 1.1.6)
- Rebased on ReSkate 1.1.6 (same game build 20260929; the only upstream change in a file this mod touches is two
  overlay settings, no conflict). No Dark Pop change since 0.14.0, which is itself the untested cleanup of the tested
  0.13.5. Syntax-checked against ReSkate 1.1.6 with no new diagnostics (clang/mingw, -Wall -Wextra).

## 0.14.0 (cleanup, on ReSkate 1.1.4)
- 0.13.5 play test (the user, 25 routed landings, No Bail off): 23 rode away, flights 0.2-1.9 s, rises up to
  5.8 m, forward speed kept (8-10 m/s in, 7-10 out on most). A block of five pre-wind 360s and five plain flips all
  popped: the pre-wind requirement belonged to the game's own landing decision, which the route skips. Guard 12
  had no dismounts; 4 and 20 each let one through. The user: "so much better".
- Removed: the launch (0.9.x), the contact assist (0.6.0), "hold in the air" and the contact clear, with their
  code, log fields and settings. What is left: Dark Pop, Sink and pop (601), Sink length, Pop guard, Keep speed,
  One press one pop, Unpatch, Hold, probe.
- Defaults: everything on, sink length 2, pop guard 12.
- Menu and log lines renamed to plain words; the landing line says `went to 601 (the sink)`, the sink line
  `the sink (601) lasted N tick(s) ... -> state 200, the pop`.
- Untested in game. Syntax-checked against ReSkate 1.1.4 with no new diagnostics (clang/mingw, -Wall -Wextra).

## 0.13.5 (No Bail's protection borrowed, keep speed sustained, on ReSkate 1.1.4)
- 0.13.4 play test (the user): with No Bail off every guarded pop still ended in 504 the tick the guard stopped
  (bit 27 cancelled or not); with No Bail on the pops were 0.6-2.0 s, rises up to 6.3 m. The keep-speed write
  landed (7.6-8.3 m/s added back to 32 bodies) but the board was at about 0.9 m/s a tick later: the 200 entry
  re-seeds the body velocities after the exit.
- **601 route** now borrows No Bail's protection (ReSkate's hooks on the cause recorder, the impact producers and
  the bail-animation consumers, through protected_owner, the way the contact assist always did) while the game is
  in 601 and through the pop's flight. No Bail itself can stay off.
- **Keep speed** re-asserts the landing's horizontal velocity on every guard tick on which the board has fallen
  under 70% of it (the forward boost's write each time). The pop's-flight line adds `re-asserted on N guard
  tick(s)`. The guard length now also bounds how long the speed is kept up; 12 ticks is 0.2 s of the flight.
- Untested in game. Syntax-checked against ReSkate 1.1.4 with no new diagnostics (clang/mingw, -Wall -Wextra).

## 0.13.4 (bail animation held off, keep speed, on ReSkate 1.1.4)
- 0.13.3 play test (the user): the guard kept 300 away everywhere it was on, but with No Bail off every guarded
  pop ended in 504 the tick the guard stopped (0.22 s with 12 ticks, 0.41 s with 24). With No Bail on the pops were
  1-2 s (guard 12: 2.08, 1.22, 1.75, 0.98, 1.23 s). No Bail hooks the game's two bail-animation consumers; the
  animation goes into its bail on its own path and asks for the dismount afterwards. Forward speed collapsed on
  every pop, No Bail or not (about 9 m/s before the landing, 0.6-6.8 after): 601's own flatten plants the feet.
- **601 route**: in 601 and in the pop guard the animation's independent bail request (+0x13d4 bit 27) is
  cancelled too, before the chooser, so the animation does not go into the bail and ask for the dismount.
- **Keep speed** (new toggle, on by default): the board body's horizontal velocity on the landing tick is saved; at
  the 601 exit the missing part is added back to every physics body at once, the forward boost's write (velocity
  +0x70, flag +0x60 bit 3). Nothing is written unless every body reads back sane, the shortfall is between 0.5 and
  40 m/s, and the skater is on the board. The pop's-flight line says `keep speed landing X m/s, Y m/s added back to
  N bodies`. Off to see the pop without it.
- Untested in game. Syntax-checked against ReSkate 1.1.4 with no new diagnostics (clang/mingw, -Wall -Wextra).

## 0.13.3 (pop guard, on ReSkate 1.1.4)
- 0.13.2 play test (the user, four runs): the stay-bit cancel exits 601 to 200 on schedule every time. Without No
  Bail the pop's first tick went to 300 (the contact package was still up). With No Bail and a stay of 2 ticks the
  pops were real (flights 0.9-2.1 s, rises 1.2-6.3 m, "ended on a dark pop"); a stay of 63 left only hops. Forward
  speed collapsed on every pop (about 9 m/s before the landing, 0.6-6.8 after), most likely the wipeout the game
  still decided on the first pop tick doing its damage before No Bail swapped the state back.
- **Pop guard (ticks)** (new slider, default 12 = 0.2 s): for the pop's first ticks after a 601 exit the bail
  package, wipeout requests and causes are cleared before the chooser, as in 601. No Bail is not needed. The
  pop's-flight line says `pop guard N tick(s), package cleared on M`.
- 601 stay default is now 2.
- The sliders send one value when let go (0.13.2 logged every notch).
- Still unknown: whether 601's own animation also kills the forward speed. If it survives with the guard, that was
  the wipeout; if not, the next step is saving the landing velocity on entry to 601 and restoring it at the exit.
- Untested in game. Syntax-checked against ReSkate 1.1.4 with no new diagnostics (clang/mingw, -Wall -Wextra).

## 0.13.2 (601 stay length, on ReSkate 1.1.4)
- 0.13.1 play test (the user): five routed landings, 601 stays of 210-766 ticks (4-13 s), none ended through the
  chooser. The impact request never came back after the first clear; the game raised the stay bit (+0x13d4 bit 1)
  on the first 601 tick and never dropped it. The body flattened into the ground (601's own animation; the
  placement entry is not called from 601, so the hold port did not run), the board rolled back upright within 0.3 s.
  The user: the original was a very smooth bounce, like jumping from a surface onto a trampoline; and with unpatch +
  hold on (route off) dark pops land nearly 100% when landing into a clean 360 pre-wind.
- **601 stay (ticks)** (new slider, default 20, 0-120): after that many ticks in 601 the stay bit is cancelled on
  every tick before the chooser (the game re-raises it), so the chooser falls through to its next line, 200, the
  pop. Raise it for a deeper sink, lower it for a quicker bounce; 0 cancels from the first tick.
- The safety window is 3 s (180 ticks, was 10 s). When it runs out, or the game leaves 601 without a chooser call,
  the 601 line still appears: `... -> state S (the window was let go: no chooser exit)`. The line now also shows
  the stay length and how many ticks the stay bit was cancelled.
- Untested in game. Syntax-checked against ReSkate 1.1.4 with no new diagnostics (clang/mingw, -Wall -Wextra).

## 0.13.1 (601 kept alive, menu tidied, on ReSkate 1.1.4)
- 0.13.0 play test (the user, 20 dark landings): with the 601 route on, the chooser took 601 on all 12 routed
  landings and the game itself raised the "stay" signal (+0x13d4 bit 1) on the first 601 tick, but 601 lasted 1-2
  ticks and went to 300 every time, with contact assist on or off, unpatch on or off. Route off, unpatch + hold on:
  3 pops / 2 bails of 5, the 0.11/0.12 split. The chooser's 601 case (identical in both exes): +0x13c4 bit 15 (the
  impact-bail request) -> 300; else +0x13d4 bit 1 -> stay; else +0x13c4 bit 27 -> 201; else 200, the pop. Every
  exit to 300 had bit 15 up, raised by the landing impact one tick after entry.
- **601 route**: while the game is in 601 after a routed landing, on every tick before the chooser the bail package
  is cleared, the wipeout requests cancelled and the pending bail causes reset (the contact assist's retry calls),
  so only the stay bit and the pop exit remain. A 601 exit to 200 is tracked as a pop (the pop's-flight line
  follows). The 601 line now says on how many of its ticks the package was found raised and names the exit.
- Menu tidied (Insert > SKATER > BOOSTS): Dark Pop, 601 route, One press one pop, Unpatch (entry), Hold (foot
  attach), Contact clear, then the old experiments (Hold in the air, Contact assist, Launch), then the probe.
  Shorter descriptions; no toggle removed, defaults unchanged.
- Untested in game. Syntax-checked against ReSkate 1.1.4 with no new diagnostics (clang/mingw, -Wall -Wextra).

## 0.13.0 (one press one pop, 601 route, on ReSkate 1.1.4)
- 0.12.0 play test (the user, own pad): contact clear did not fix the scrape bails (the bail is decided inside the
  game's contact test, which re-raises the flags in the same call); "hold in the air" never bailed but never sank
  either, and with the D-pad held it bounced (200 -> 201 -> 200 every ~0.55 s, the mod re-arming on every hop).
- What was read before writing this: the physics-state chooser of both Skate.exe builds (old 0x43777b0, now
  0x4745a70), found in the old exe by body fingerprints after its prologue fingerprint had no match. The two are the
  same function apart from the X removal, which shifted every later bit of the packed animation->physics words down
  by one (old +0x13d4 bit 0 is now +0x13d0 bit 31, old +0x13d4 bit n is now bit n-1, through +0x13e0). The bail
  producers (reasons 9 and 14, the function before the placement helper) are identical too. From the air (200) and on
  the landing tick (201) the chooser's first test, before any contact or bail test, is "+0x13d0 bit 31 and +0x13d4
  bit 2 -> state 601" (air-to-board placement; 600-605 are the board interaction states, 604 the walk-on mount).
  Neither signal was set on any tick of two recorded sessions (riding, flips, dark catches, an underflip, walking),
  so 601 is dormant today. Underflips do not use it (an underflip landed as 100 with no 60x state).
- Also read: old apply ran only while X was held and switched to mode 1 the moment X was released; the new apply
  never requests mode 2 (mode 1 with a weight x offset only). Old update started the hold on X && latched && blend
  < 0.3 or on B && arg+0x56, reset it when timer + blend < 0, and withheld `applied` on the first X tick while
  +0x13d4 bit 3 (now bit 2) was clear. The helper's target direction changed (built from the arg's position + blend
  x velocity now). Old 0x442e350/0x4435e40 are the launch-onto-board state, not a landing path.
- **Dark Pop one press one pop** (new toggle, on by default): after a dark landing the D-pad has to be seen let go
  before a press arms a flight again. Kills the bounce in every mode.
- **Dark Pop 601 route** (new toggle, off by default): on the dark landing ticks (the same ticks as the contact
  clear) +0x13d0 bit 31 and +0x13d4 bit 2 are raised before the chooser and taken down after it. If the chooser
  picks 601 the placement window (B, latched, hold) is kept open while the game stays in 601 (at most 10 s), and
  the log says how long 601 lasted and where it went. Risk: 601's handlers have not been read; it may be gutted.
- Landing line gains `601 route {off | bits set on N tick(s)}`; a landing that went to 601 says so. New line:
  `Dark Pop: state 601 lasted N tick(s) -> state S`.
- Side observation from the control recordings (mod off): two nollies rose 2.7-3.0 m on a single arc (normal 0.4-1.1
  m) with the catch registered within 0.03-0.15 s of takeoff and +0x13c8 bit 28 up from the first catch tick; the
  game added about +2 m/s in state 201 during the catch, no landing, no chooser involvement. Not chased yet.
- Untested in game. Syntax-checked against ReSkate 1.1.4 with no new diagnostics (clang/mingw).

## 0.12.0 (contact clear, on ReSkate 1.1.4)
- 0.11.0 play test (L, PS pad, hold on, contact assist off, 22 dark landings): 13 pops, 9 bails. The tester saw the
  body sink into the ground with the original timing: the hold brought the mechanism back. Pop airtime median 1.31 s
  (0.10.0: 0.95 s), board rise median 3.1 (0.10.0: 1.5). Every landing whose bail package was already raised on the
  landing tick (the board touched during the catch; the tester's 360 hardflips) bailed on the next tick; every clean
  one (nollie tre, nollie 360 inward) popped. With contact assist on the pops were a mess (micro-pops, stuck
  landings), as in 0.10.0: the stuck bug has only ever appeared with the assist on.
- **Dark Pop contact clear** (new toggle, on by default): clears the bail package (context+0x13c4 bit 0x8000,
  +0x13c8 bits 0x2c000, +0x13cc bit 0x400, +0x13d4 bit 0x100) on the landing ticks before the game's decision, as the
  contact assist does, but with none of its retries, touchdown hold-off or clearing in the pop's flight. The hold
  takes the landing from there. The landing line says `contact clear: flags cleared on N tick(s)`.
- Fixed: the landing line's `board up` is now the value on the decision tick (it was read up to 100 ms later, during
  the pop, so it showed the board already turning over).
- The fast probe CSV gains nine columns `a_x..c_z`: the board body's three unit axes every tick (the middle one is
  up), so a flight's shove direction and amount, flip and nollie/regular can be computed afterwards instead of asked.
- Untested in game. Syntax-checked against ReSkate 1.1.4 with no new diagnostics.

## 0.11.0 (hold port, on ReSkate 1.1.4)
- What was read before writing this: the pre-0.32.0 and current board-placement entry, update and apply functions,
  the old mode setters and the constants, from byte dumps of both Skate.exe builds. Findings: the hold state machine
  (active +0x29a, applied +0x29b, blend +0x220, timer +0x234) survived 0.32.0 at new offsets; what the patch removed
  was (1) the old start of the hold (entry latched this tick and blend under 0.30; today it starts only from B plus
  the caller's +0x56 flag) and (2) the apply's X path, which snapped the board to the foot bone and requested
  **foot-attach mode 2** on bone 0x13 (regular) or 0xf (switch). Today's apply computes the same target but only ever
  requests mode 1 through a blended offset (the "feet sinking" fix). The mode-2 setters are still in the exe
  (0x4769020 / 0x4768fb0, same fields +0x1c80/+0x1c70 of the skater's +0x2f10 object), unreferenced since the patch.
- **Dark Pop hold** (new toggle, on by default): hooks the board-placement update (0x47fac60). After the game's own
  prologue, if the hold is neither active nor started by the game, the old X block runs: start when latched and blend
  < 0.30 (and run the tail the game skipped: applied = 1, timer = dt, apply), else the old idle reset. After the
  game's (or that) apply, with the hold active, applied, latched and blend < 0.20, mode 2 is requested on the chosen
  bone. Constants are the old exe's (0.30, 0.20), not guesses. When the window closes after mode 2 was requested, mode
  1 is requested once on the same bone (a release, in case the request outlives the tick). Off = 0.10.0 behaviour.
- One deliberate difference from the old code: it skipped the apply on the hold's first tick unless context+0x13d4's
  "ground" output was set. In the 0.10.0 recordings that output is never set at a dark landing, and the game decides
  the bail on the tick after the landing, so waiting would put the first mode-2 request after the decision. 0.11.0
  requests it from the first active tick; the output's state at the start is logged.
- **Dark Pop hold in the air** (new toggle, off): the hold's window and B start at arming (D-pad pressed, board
  dark) instead of at the landing. An experiment for when the hold alone changes nothing.
- Log: the dark landing line adds `hold ... (started N, blend at start b, ground at start g, active on T tick(s),
  mode-2 requests M on bone 0x13|0xf, idle resets R)`; the pop line adds the board's up axis at the pop's landing and
  the hold's final counts. Start-up logs `Dark Pop hold ready` or which fingerprint failed (update, apply, or a setter);
  with a failed fingerprint the hold is unavailable and everything else works.
- Not changed: the 0.10.0 landings with `placement calls 0` are pops the game made on the first contact tick, before
  the landing stage bit; they are not a detection miss, so the window logic is as in 0.10.0.
- Untested in game. Syntax-checked against ReSkate 1.1.4 with no new diagnostics.

## 0.10.0 (unpatch experiment, on ReSkate 1.1.4)
- Built on ReSkate 1.1.4 (the trainer is inside ReSkate now); the 0.9.1 code applies unchanged.
- What 0.32.0 removed, by field identity: a third animation signal ("X", old context+0x13d0 bit 29). The other two
  were renumbered: old bit 30 = today's bit 29 (the 0.9.x launch, "A"), old bit 31 = today's bit 30 ("B"). The old
  board-placement entry test was `(B || (A && !X)) && (X || latched || !ground) && (arg flag || flag)`; today's is the
  same with X false. X also drove a timed hold that 0.32.0 deleted outright, so no data mod can bring it back.
- **Dark Pop unpatch** (new toggle, on by default): emulates the old `B && X` entry. The board-placement entry
  function (0x4804750) is hooked; in the dark landing window (landing tick through the pop's first 4 ticks) B is set
  and the placement object's latched byte (+0x294) is forced before each call, which is what X used to skip. The log
  line now reports what those calls saw (latched before/after, the 0x295 flag, the arg's +0x54 flag and +0x55 side).
- **Dark Pop launch** now defaults **off** (the old game never took the A and X paths at once).
- Fixed: the "landing stage" bit is also on during the catch (first ~0.5 s), so 0.9.1 set its bit mid-air. A landing
  is now the stage bit coming back after it has been clear for 5 ticks since arming.
- If the hook's fingerprint does not match, the log says "Dark Pop unpatch is unavailable" at start-up and the
  landing line says "unpatch on but not hooked"; everything else still works.

## 0.9.1 (launch)
- Lean build: Dark Pop is now the dark landing, the **launch**, and the **contact assist**, each with its own toggle.
- **Dark Pop launch** (on by default): patch 0.32.0 removed one animation signal (context+0x13d0 bit 31) that switched
  on the game's board pull at a dark landing. Bit 29 still opens that path; it is set from the dark landing through
  the pop's first 4 ticks. In play tests it brought back the launch (median 2.9 m above the landing, 1.5 s in the air,
  against 0.4 m and 0.55 s without).
- **Dark Pop contact assist** (on by default): 0.6.0's bail-flag clearing, retries and bail hold-off around the pop.
  Switch it off to test the launch on its own.
- Removed: the bit 30 option, the "Dark Pop extra pop" slider and the animation-input recorder. The probe stays.
- Fixed: the launch toggle no longer logs the wrong state (each switch now has its own setting).
- The log's dark landing line says whether the assist was on and on how many ticks bit 29 was set.

## 0.9.0 (legacy launch experiment)
- New toggle **Dark Pop legacy launch** (off by default) and **Legacy launch: use bit 29**. Comparing Skate.exe from
  before patch 0.32.0 with today's: the patch removed one animation signal (context+0x13d0 bit 31) whose only job was
  to switch on the game's board-pull path at a dark landing. This sets one of the two bits that still open that path
  (bit 30, or 29) through the dark landing and the pop's first 4 ticks.
- The log's dark landing line says on how many ticks it was set. Still records the animation inputs (0.8.1).

## 0.8.1 (recorder)
- Same behaviour as 0.6.2 (0.6.0's code); 0.8.0's flip assist is gone (the board is held by the animation during a
  dark catch, so spinning its physics body did nothing).
- The fast probe file records the animation's inputs every physics tick (new columns anim_ok and anim: 80 bytes the
  game's physics publishes to the animation), to find which one makes the board flip back over at a dark pop.

## 0.6.2
- Back to 0.6.0's code, unchanged (0.7.x's RB hold and flag clearing are gone: play tests went worse).
- Controllers: the D-pad press is read through ReSkate's own controller reader, so Xbox pads and DualShock 4 /
  DualSense / DualSense Edge pads (USB or Bluetooth, or through DS4Windows / Steam Input) all arm it.
- Tip: keep RB (R1) held until you touch down. Letting go before the landing is what makes the pop come out dark.

## 0.6.0 (contact assist)
- The game makes every pop now. At a dark landing the mod clears the game's bail flags on each tick until the game
  decides, so the game chooses the pop itself, with its own animation and height. The forced pop state is gone (it
  could leave the rider detached from the board until Y was pressed).
- A dark catch is read from the board itself (upside down), not from RB or timing: any flip trick, any height.
  Turning the board back over before landing disarms it.
- If the game still asks for a bail at the landing, the decision is put off for up to 3 ticks with the flags cleared
  again (the game decides again; nothing is forced).
- The release slider and the controller tricks behind it are gone; the game sees your real D-pad.
- The fast probe file has two new columns: board_up (+1 upright, -1 upside down) and dark_pop (1 pressed, 2 armed,
  4 pop's flight, 8 after its touchdown).

## 0.5.2
- RB held up to 0.5 s before the takeoff now counts (the dark catch set during the wind-up). 0.5.1 only looked for
  RB in the air, which missed players who let go of RB before leaving the ground.

## 0.5.1
- Works with PlayStation controllers. 0.5.0 only noticed the RB + D-pad press through Xbox-style (XInput) input,
  which skate. does not use for a PlayStation pad, so it never fired for a DualSense. The mod now reads the press
  through ReSkate's own controller reader, which handles both.
- With a PlayStation pad the release slider cannot hide the D-pad from the game (the menu still opens); the pop
  itself does not depend on it.

## 0.5.0
- D-pad Left, Down or Right fires it (any of the old dark pop menus). RB only has to be seen once in the air.
- After the pop lands, a dismount in the first 0.3 s is held off (the old "glitch").
- The log gives the pop's flight length and how it landed.

## 0.4.2
- Dark Pop release can go down to 0: the game never sees the D-pad Right press, so no menu shows, and the
  landing still pops. 1.20 (default) keeps the first Dark Pop's feel (held through the landing, menu shows).
- A D-pad Right press after the release time still fires the flight (it is hidden from the game at once).

## 0.4.1
- Matches the first Dark Pop (the one that worked with No Bail on): every landing of a fired flight becomes the pop,
  and by default the game sees D-pad Right held through the landing (Dark Pop release 1.20 s). No Bail stays off;
  bails are held off only from the D-pad Right press until the pop lands.

## 0.4.0
- Back to what made the first Dark Pop work: when a fired flight (RB, then D-pad Right) would land in a bail, the pop
  is chosen instead. 0.3.3 held off wipeouts but 9 of 11 landings were dismounts, which No Bail never stops.
- The dismount request is dropped at that moment, and during the pop's flight further dismounts are held off.
  Bails are held off until 0.3 s after the pop lands (at most 2.5 s).
- The log says what the game wanted at the landing, whether the mod turned it into the pop, and how the pop's own
  flight landed.

## 0.3.3
- A flight is primed when you hold RB (the catch) in it, and fires when you press D-pad Right after that. D-pad
  Right in the air without RB is passed to the game untouched (no timed release, no bail protection).
- Includes 0.3.2 (bails held off from the D-pad Right press until 0.3 s after the landing).

## 0.3.2
- Bails are held off again (No Bail's protection, lent by the mod) from the moment the game sees D-pad Right
  pressed in a timed flight until 0.3 s after its landing. The recordings that showed natural pops were made with
  0.2.0, which did this without anyone noticing; 0.3.0 and 0.3.1 dropped it and 20 of 20 bailed, mostly wipeouts.
- Release timing unchanged (0.715 s after takeoff).

## 0.3.1
- The release is now timed from the takeoff (default 0.715 s after it) instead of from the expected landing.
  0.3.0 released too early (about 0.67 s after takeoff) and nothing popped: the game marks a bail's landing at
  0.82 s but a pop's at 0.85 s, so timing from landings learned the wrong clock.

## 0.3.0
- Built on ReSkate 1.1.1 (made against its real source).
- Dark Pop now does the timing for you: press D-pad Right in the air and the game sees it held and then let go
  a set time before the landing (Dark Pop release slider, default 0.16 s), timed from your last three flights.
  Only what the game reads changes; you can hold the button or let go whenever.
- The forced pop and the borrowed No Bail protection are gone: the game makes the pop itself. No Bail is not used.
- The log notes each flight with D-pad Right: its length, when the release was shown and whether it popped.
- The fast probe's pad column marks what the mod showed the game (0x100000 held, 0x200000 hidden).

## 0.2.0
- Back to the first Dark Pop (hold D-pad Right as the flight ends and the landing becomes a new pop), which
  worked, but it **no longer needs No Bail**. Turning Dark Pop on leaves No Bail as it is. Bails are held off
  only while D-pad Right is held in the air and from a forced pop until just after the next landing.
- The contact assist and its Assist extra pop slider are gone. The Dark Pop extra pop slider (0-25 m/s) is back.
- The log notes each forced pop and the state the next landing went to, so a bail after a pop shows up.
- The probe is unchanged.

## 0.1.0
- First standalone Dark Pop build, split out of the Revert Boost repository. Revert Boost is not in it.
- Contact assist (with the Assist extra pop slider) and the probe.
