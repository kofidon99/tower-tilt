# Tower Tilt

A one-tap stacking game about balance, built to YouTube Playables spec. Single
HTML file, no build step, no dependencies, no external network requests.

## Controls

One action, everywhere.

| Action | Touch | Keyboard |
|---|---|---|
| Drop a block | Tap anywhere | `Space`, `Enter` or `↓` |
| Pause | Pause button | `Esc` or `P` |
| Mute | Speaker button | `M` |

## The hook — why this isn't a stacking clone

Most stacking games slice the overhang off each block so it shrinks until you
miss. That mechanic is well-worn, and Playables rejects games that copy an
existing title without adding anything.

Tower Tilt does something different: **blocks never shrink, and the failure
state is toppling, not missing.** Every block keeps the offset you gave it, and
the game tracks the tower's actual centre of mass. Dashed markers on the
foundation show how far the load may sit from the base centre; an amber plumb
line shows where it currently sits. Push the load past the markers and the tower
is overloaded.

That turns one tap into two decisions at once:

- **Where does this block land?** Overlap the block below by at least 70% or it
  slides off.
- **What does it do to the load?** A block placed left of centre pulls the load
  left. If you're leaning right, that's how you fix it.

And the two pull against each other. A **flush** landing — dead centre on the
block below — is worth the most points and builds a multiplier, but it doesn't
correct a lean. Correcting a lean means deliberately placing off-centre and
throwing your flush run away. That trade is the game.

**Wind** makes balance a standing concern rather than a one-off mistake: past a
certain height it pushes the load sideways, so even perfect stacking eventually
has to be counterweighted. An arrow beside the load gauge shows which way.

## When it goes wrong

- **Miss** (less than 70% overlap): the block tumbles away, costs a life.
- **Overload** (load past the markers): the tower **sheds its top few blocks**,
  costs a life. That's both what a real overloaded stack does and what actually
  fixes the problem — the highest blocks carry the most leverage, so shearing
  them brings the load back over the base. It costs a lot of height, so it
  stings without ending the run.
- **Out of lives:** the whole tower comes down.

**Gold blocks** arrive every 13 placements. Land one flush and you get a life
back, up to the tier's ceiling. The lives row shows hollow markers for that
ceiling, so you can see whether chasing one is worth it.

## Difficulty

Three tiers, tuned around the **precision window** — how long the crane spends
inside the landing zone on each pass. The crane swings sinusoidally, so it's
fastest dead centre; these are the worst-case numbers, measured there:

| | Land window | Flush window | Lives (max) | Wind from | Score |
|---|---|---|---|---|---|
| **Cruise** | ~0.76s | ~0.14s | 4 (6) | height 20 | ×0.7 |
| **Drive** | ~0.50s | ~0.08s | 3 (5) | height 12 | ×1.0 |
| **Redline** | ~0.34s | ~0.04s | 2 (3) | height 6 | ×1.45 |

Tiers also vary how fast the base's tolerance narrows as you build.

**Nothing about the control changes between tiers** — one tap, same response.
Difficulty comes from the world, never from a worse-feeling input.

Each tier keeps its own best score and its own tallest tower, so picking an
easier one costs you nothing. The game also **suggests a tier**: three short
builds in a row and it offers the gentler one; one tall clean build and it
offers the faster one. Each direction is offered once so it never nags.

Block width and crane sweep are **fixed logical sizes**, not fractions of the
viewport, so the game plays identically on a 9:20 phone and a 16:9 desktop.
Sizing the sweep to the screen would both change the difficulty per device and
hang the block off the edge of a phone, where you can't aim at it.

## Altitude

The sky is a function of height: warm low haze, then clear blue, then thin high
atmosphere, then near-space with stars. An altimeter appears on the right once
the camera starts climbing. It's the progress bar, and it costs nothing.

## Playables compliance notes

- **Initial load ~68 KB**, one file. Limit is 30 MB.
- **Zero external requests.** No CDN, no web fonts, no analytics. All art is
  drawn procedurally on canvas; all audio is synthesised with Web Audio.
  Verified automatically in `test/run.js`.
- **No copyrighted assets** — there is no image or audio file in the bundle.
- **Scales to 1:1, 16:9 and 9:16.** Screenshots in `test/shots/`.
- **60 fps** at phone and desktop resolutions, measured under load.
- **`firstFrameReady()` then `gameReady()`**, in that order, and `gameReady`
  only once the menu is interactive.
- **Pause and mute obeyed immediately.** `onPause` cancels the animation frame
  outright and zeroes the audio master gain; `onAudioEnabledChange` is honoured
  and in-game mute cannot override the platform setting.
- **Progress saved through `saveData` / `loadData`**, falling back to
  `localStorage` only when the SDK is absent.
- **No ads wired up yet** — add the hooks once the channel is onboarded and the
  portal says which placements are supported.

## Repo layout

```
index.html                    the whole game
.nojekyll                     serve files as-is
tower-tilt-playables.zip      bundle for the developer portal
src/body.html                 source of truth (title + style + markup + script)
build.js                      wraps src/body.html into index.html
test/run.js                   aspect ratios, external requests, pause, perf
test/sdk.js                   integration against a mocked ytgame SDK
test/gameplay.js              lives, tiers, balance, gold repairs
test/shot.js                  screenshot capture
```

`node build.js` regenerates `index.html`.
`node test/run.js && node test/sdk.js && node test/gameplay.js` runs everything.

### How the tests work without debug hooks

The shipped build has no test affordances. The game-over card reports height,
flushes, misses and shears, and that's enough: a run only ends when lives hit
zero, and only a miss or a shear costs a life, so **misses + shears must equal
the tier's starting lives exactly** (plus any gold repairs). That one identity
proves the whole lives system and each tier's starting count.

Anything not directly observable is tested as a **differential** — build a
variant with the feature disabled and assert the outcome measurably differs.
The balance system is checked that way: two builds that land every block, one
with a finite base and one with an unbounded one. The bounded build shears and
stays short; the unbounded one never shears and builds taller. If the
centre-of-mass calculation were inert, those two would be identical.

The automated player taps blind — it can't see where the crane is, so on the
shipped build it misses most drops, exactly as a person would with their eyes
shut. Tests that need a tower actually built use a variant with the landing
window opened up, which still scatters placements across the sweep — precisely
the input the balance system exists to punish.

## Performance notes

Same lessons as Neon Drift, plus one new one:

- **Sprite-cache anything with a shadow.** Canvas `shadowBlur` recomputes per
  fill.
- **Never build a gradient per frame.** Cache per layout.
- **Bake the sky into a small texture and stretch it.** The sky is a smooth
  gradient plus a soft sun; painting it into a 192×448 offscreen canvas and
  blitting that stretched is far cheaper than two full-screen gradient fills a
  frame — one of them radial, which is the most expensive fill there is. This
  alone was the difference between 30 and 60 fps on a desktop window.
- **Render-resolution budget:** past ~2.6M device pixels, step density down
  rather than drop frames.
