# Neon Drift

A one-thumb neon lane racer, built to YouTube Playables spec. Single HTML file,
no build step, no dependencies, no external network requests at runtime.

## Controls

|Action|Touch|Keyboard|
|-|-|-|
|Change lane|Tap or swipe left / right|`←` `→` or `A` `D`|
|Boost|Tap the ring (when charged)|`Space` or `W`|
|Pause|Pause button|`Esc` or `P`|
|Mute|Speaker button|`M`|

## How it plays

Traffic comes at you down four lanes and never stops. Squeeze past a car and
you bank a **near miss**, which adds score and builds a combo up to ×9. Energy
gems charge the **boost** ring; fire it and you become briefly invincible at
1.55× speed with a doubled multiplier, smashing through anything in the way.

A hit costs a **life**, not the run. You get a couple of seconds of
invulnerability to recover, your speed drops so the next wave is readable, and
your combo resets. Green **repair cells** appear on the road and restore a life
up to the tier's ceiling; at full lives they pay out as score and a full boost
charge instead. Lives are shown top-left — filled chevrons for what you have,
hollow for the ceiling you could repair back up to.

Mechanics arrive one at a time rather than all at once: traffic first, then
energy, then boost, then repair cells, then lane-changing drifters, then
two-lane barriers. When each of those starts depends on the tier.

## Difficulty

Three tiers, tuned around **reaction time**. A car spawns about 861 logical
units above you and closes at `speed × (1 − frac)`, so the window you get to
read it and move is `861 / (speed × (1 − frac))` seconds. Each tier is built
backwards from the window it should give at its top speed:

||Reaction window|Lives (max)|Traffic|Drifters / barriers|Score|
|-|-|-|-|-|-|
|**Cruise**|\~1.4–2.4s|4 (6)|sparser|55s / 95s|×0.7|
|**Drive**|\~0.9–1.5s|3 (5)|baseline|35s / 60s|×1.0|
|**Redline**|\~0.7–1.1s|2 (3)|denser|18s / 34s|×1.45|

Steering feel is **identical** in all three — `LANE\_SHIFT\_TIME` never changes.
Difficulty comes from the world, never from sluggish controls, because a game
that gets harder by becoming less responsive just feels broken.

Each tier keeps its own best score, so the tiers can't be gamed against each
other and picking an easier one costs you nothing.

**The game also suggests a tier.** Three short runs in a row and it offers the
gentler one; a long clean run and it offers the faster one. Each direction is
offered once so it never nags.

## Playables compliance notes

Built against the published requirements:

* **Initial load \~84 KB**, one file. Limit is 30 MB, so there's enormous headroom.
* **Zero external requests.** No CDN, no web fonts, no analytics, no multiplayer.
All art is drawn procedurally on canvas; all audio is synthesised with Web
Audio. Verified automatically in `test/run.js`.
* **No copyrighted assets.** Nothing is sampled, traced or licensed — there is
no image or audio file in the bundle at all.
* **Scales to 1:1, 16:9 and 9:16.** Scale is driven by height so the road keeps
a constant on-screen size; extra width reveals more city rather than
stretching gameplay. Screenshots in `test/shots/`.
* **60 fps** at phone and desktop resolutions, measured under load.
* **`ytgame.game.firstFrameReady()` then `ytgame.game.gameReady()`**, in that
order, and `gameReady` only once the menu is actually interactive.
* **Pause and mute are obeyed immediately.** `ytgame.system.onPause` cancels the
animation frame outright — no rendering, no gameplay — and zeroes the audio
master gain. `onAudioEnabledChange` is honoured and in-game mute cannot
override the platform setting.
* **Progress is saved through `ytgame.game.saveData` / `loadData`**, falling back
to `localStorage` only when the SDK is absent.
* **No ads wired up yet.** Interstitial and rewarded hooks are deliberately left
out rather than stubbed — add them once the channel is onboarded and the
portal tells you which placements are supported.

Every SDK call is feature-detected and wrapped in try/catch, so the same file
runs identically on GitHub Pages, in an artifact, and inside Playables.

## Uploading to the Playables developer portal

`neon-drift-playables.zip` is the bundle to upload once your channel is
onboarded — it contains `index.html` at the archive root, which is the layout
the portal expects.

## Repo layout

```
index.html                    the whole game
.nojekyll                     tells GitHub Pages to serve files as-is
neon-drift-playables.zip      bundle for the developer portal
src/body.html                 source of truth (title + style + markup + script)
build.js                      wraps src/body.html into index.html
test/run.js                   aspect ratios, external requests, pause, perf
test/sdk.js                   integration against a mocked ytgame SDK
test/gameplay.js              lives, difficulty tiers, repairs, save migration
test/shot.js                  screenshot capture at 9:16, 1:1, 16:9
```

`node build.js` regenerates `index.html`.
`node test/run.js \&\& node test/sdk.js \&\& node test/gameplay.js` runs everything.

### How the gameplay tests work without reaching into the game

`test/gameplay.js` deliberately uses no debug hooks. Because a hit destroys the
obstacle and the run only ends at zero lives, a run that collects no repair cell
must end with exactly as many hits as the tier grants lives — so the hit count
on the game-over card proves both that lives work and that each tier starts with
the right number. Invulnerability is checked as a differential: the same build
with the grace window removed dies measurably faster under dense traffic.

## Changelog

**v2** — Lives (3 on Drive) replace instant death, with post-hit invulnerability
and a speed stumble; green repair cells restore a life; three difficulty tiers
tuned by reaction time with per-tier best scores; the game suggests a tier based
on how you're actually doing. Fixed: per-tier spawn density was defined but never
applied to the spawn timer.

**v1** — Endless four-lane racer: traffic, energy, boost, near-miss combos,
drifters, barriers. Playables SDK bridge, procedural art and audio.

