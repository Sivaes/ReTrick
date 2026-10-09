# ReSkate Dark Pop

A patch for [ReSkate](https://github.com/Dingo-Shenanigans/ReSkate) (GPL-3.0) that brings back the dark catch
pop removed in skate. 0.32.0. One switch in Insert > SKATER > BOOSTS, off until you turn it on; a "Dark Pop
experiments" fold-out under it holds the probe and the things still being tested. Since 2.0 the build also carries
[Revert Boost](https://github.com/Sivaes/reverboost) 2.0 (its own switch and slider, above Dark Pop).

This repository holds only the patches (`patches/revert-boost.patch`, then `patches/dark-pop.patch`, which is a
diff against the Revert Boost tree), the Thunderstore package files (`thunderstore/`) and the workflows. ReSkate's
source is fetched from the ReSkate project when you build, so nothing of theirs is copied here. The workflow
`build-dark-pop.yml` carries copies of the patches and package files inside itself, so that one file is all a
build needs.

## Build it
1. Actions > **Build Dark Pop (tester zip)** > Run workflow.
2. `upstream_ref`: a ReSkate release tag such as `v1.1.8` (or `main`). `mod_version`: this mod's version (x.y.z).
3. Download the zip from the run's Artifacts; drop its folder into the game's `Mods` folder and run `Install.bat`.

If the **Apply the patch** step fails, ReSkate changed a file one of the patches edits and that patch needs updating.

Not affiliated with EA, Full Circle or the ReSkate developers.
