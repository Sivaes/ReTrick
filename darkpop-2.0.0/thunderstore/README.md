# ReSkate Dark Pop @MOD@

skate. patch 0.32.0 removed the **dark pop**: catch a flip dark side, work the D-pad menu around the
landing, and the board pops you back up. Patch 0.32.0 deleted the code behind it, so this mod puts it back as a build of ReSkate.

## Use
Everything is in **Insert > SKATER > BOOSTS**. Switch **Dark Pop** on. That is all: catch a flip trick dark
(board upside down), press **D-pad Left, Down or Right** any time in the air, and land with the board still dark.
The body sinks into the ground and is thrown back up: the dark pop. Any flip trick, any height, pre-wind or not.
Turn the board back over before landing and you land normally. No Bail can stay off.

This build also carries **Revert Boost 2.0** (its own switch and strength slider, right above Dark Pop): land an
auto revert or a board bend and gain forward speed. Off until you turn it on.

**Dark Pop experiments** (a fold-out under the switch) is for testing and sending logs; nothing in it needs
changing to play. The switch already carries what tested out (a one-tick sink, the landing catches, the pop guard,
keep speed).

- **Dark pop probe**: records what the game does to `dark-pop-probe-<time>.csv` and `dark-pop-fast-<time>.csv`
  in the game's `logs` folder. Turn it on, do your attempts, turn it off, and send both files with `logs\ReSkate.log`.
- **Hold the landing** (on): new in 2.0, untested. A longer trick (the nollie 360 flip) can run out its air
  animation with the board already resting upside down but the landing not yet declared; the game then drops to
  physics, and the landing is caught late, which pops half-size. On, the game's own landing is held open for up to
  10 chooser calls so it can declare the landing itself. Off, those landings are only caught.
- **Sink length (ticks)** (0): how long the sink lasts before the pop. 0 pops biggest and most evenly (two
  sessions, 29 landings, every one 0.7-2.1 s); longer sinks roll the board over for hops.
- **Pop strength (m/s)** (0 = off): a floor on the body's downward speed at the sink entry, written to a value the
  game turned out to rewrite every tick (1.10 test: no effect). Kept so the write and its read-back stay in the log.
- **Feet-only hold (board freeze)** (off) and **Pop when the board rolls** (on): the 1.7-1.8 long-sink
  experiments. The freeze held the board dark through the sink but the release flipped it in five of seven landings.
- **Settled (leave on)**: Catch timed-out landings, Catch early dismounts, Keep speed, One press one pop, Unpatch,
  Hold (foot attach), Pop guard (12). The parts of the pop that tested out; off only to reproduce an old result.

Works with Xbox and PlayStation controllers.

**Game settings:** Dark Catch Assist **off**, Legacy Dark Catches **on** (the settings the glitch was found with;
with them RB need not be held at the D-pad press), No Bail **off**, offline.

## How it works
Patch 0.32.0 removed one animation signal. It gated the board placement's hold (the foot attach that glued the board
through the landing) and it let the physics-state chooser leave the landing for state 601, whose exit to the air is
the pop. The mod hooks the board placement to restore the hold, raises the chooser's two 601 inputs on the landing
tick, keeps 601 from bailing (bail package, wipeout requests and the animation's bail request cleared before each
decision, No Bail's protection borrowed), cancels the game's stay signal after one tick so the chooser falls
through to the pop, protects the pop's first ticks the same way, and puts the landing's forward speed back. A dark
landing the game wants to turn into a dismount straight from the air is sent to 601 the same way.

## Install
1. Install ReSkate normally and run it once.
2. Put this folder in the game's `Mods` folder (`Mods\<author>-ReSkate_DarkPop`).
3. **Close the game and the launcher**, then double-click **Install.bat**. It saves your current
   `ReSkate.dll` and `ReSkateLauncher.exe` in a `backup` folder there and copies this mod's over them.
4. Start `ReSkateLauncher.exe` as usual.

**Uninstall:** close the game and the launcher and run **Uninstall.bat**.

The mod is a build of ReSkate itself, so it replaces `ReSkate.dll`. Do not combine it with other mods that
do that (Revert Boost): the last one installed wins.

## Crashes
The start-up notice says `Modified build: ReSkate_DarkPop <version>`, and crash reports are labelled with
the mod's name and version (`build.mod`, `build.mod_version`) so the ReSkate developers can tell them apart.
Please report crashes here, not to them. If you suspect the mod, switch Dark Pop off and see if it still
happens, and send the end of `logs\ReSkate.log` (lines starting `Dark Pop`).

## Good to know
- Built on **ReSkate @BASE@** (mod version @MOD@), with Revert Boost 2.0 included.
- Play offline or in a private session. It follows ReSkate's session rules: if a host turns off boosts, Dark Pop
  is off too.
- Each dark landing and each pop's landing are written to `logs\ReSkate.log` (lines starting `Dark Pop:`).

## Source and license
GPL-3.0, the same as ReSkate. Not affiliated with EA, Full Circle or the ReSkate developers.
