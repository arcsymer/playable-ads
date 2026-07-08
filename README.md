# Playable Ads — self-contained HTML5 prototypes

Original **playable-ad** prototypes built to demonstrate the interactive ad-network
format: each ad is a single, self-contained `index.html` — pure HTML5 + Canvas +
vanilla JS, **no engine, no external runtime, no network requests**. Tap/click-driven,
mobile-first, ~15–30 s of gameplay ending in an **install CTA**.

**Live:** https://arcsymer.github.io/playable-ads/

| Ad | Genre | Status | Live link | Size |
|----|-------|--------|-----------|------|
| **GLYPHFALL** | Match-3 (aurora runes) | ✅ Live | [play](https://arcsymer.github.io/playable-ads/match3/) | **32 KB** |
| **STARDUST FOUNDRY** | Clicker / Idle (tap-to-earn) | ✅ Live | [play](https://arcsymer.github.io/playable-ads/clicker/) | **30 KB** |
| **PRISM POUR** | Puzzle (colour-sort) | ✅ Live | [play](https://arcsymer.github.io/playable-ads/puzzle/) | **32 KB** |
| **GRILL RUSH** | Tycoon / Management (isometric run-around, warm diner) | ✅ Live | [play](https://arcsymer.github.io/playable-ads/tycoon/) | **40 KB** |

*All four use Web-Audio-generated SFX **plus a subtle muteable ambient pad** — no audio files.*

> These are **original prototypes to show the format**, not shipped campaigns. The
> operator has 5 Unity games + a TS/JS core stack but no prior shipped playable ads —
> this repo demonstrates the single-file, weight-budgeted, tap-to-CTA discipline the
> format needs. No fabricated install metrics or client claims.

---

## What is a playable ad?

An ad you can *play*. It runs inside an ad container (MRAID) in an app, gives a few
seconds of real gameplay, then shows an **Install / Play the full game** button that
deep-links to the store. The hard constraints are what make it a specialized craft:

- **One file, self-contained.** No CDN, no external assets — everything inlined or
  generated so it works the instant the container loads.
- **Tiny.** Networks enforce a hard weight budget (commonly ~2–5 MB). Every KB counts.
- **Can't get stuck.** Guided first move, auto-hint on inactivity, no soft-lock, always
  reaches the CTA.
- **Any screen, touch-first.** Portrait *and* landscape, phone-sized, large tap targets.

## GLYPHFALL (match-3) — live

![GLYPHFALL gameplay](media/match3.gif)

Swap adjacent glowing runes to make lines of 3+; matches pop, tiles cascade, and chained
clears build a **combo multiplier** that fills the "charge" bar. Hit the goal → end card
→ install CTA.

- **Six original runes** — distinct *shape and* colour (colour-blind friendly), all drawn
  procedurally on Canvas. No image or audio files anywhere.
- **Juice:** particle bursts, screen shake, floating `+score`, combo call-outs, an
  animated progress bar, Web-Audio SFX generated at runtime.
- **Guided & unstuck:** an animated hand shows the first suggested swap; after a few idle
  seconds a hint pulses; if the board runs out of moves it reshuffles.
- **Weight:** the entire ad is **~31 KB** in one HTML file — ~150× under the 5 MB ceiling.

<p align="center"><img src="media/match3-portrait.png" width="240" alt="guided first move"> <img src="media/match3-endcard.png" width="240" alt="install CTA end card"></p>

### Tech

Vanilla HTML5 / Canvas 2D / JS in a single file. Responsive full-viewport canvas
(devicePixelRatio-aware), a fixed-timestep-ish game loop with delta time, a pointer layer
that unifies mouse + touch (swipe *or* tap-tap), and a small state machine
(play → resolve cascades → win → end card). Art is procedural, audio is Web-Audio
generated — so there are **no assets to inline** and the file stays tiny.

### Ad-network integration

The store redirect and MRAID hook live at the bottom of
[`match3/index.html`](match3/index.html):

```js
// Networks (AppLovin, ironSource, Unity Ads, Mintegral…) inject the store URL at serve
// time. Prefer MRAID when present; fall back to window.open so it also runs standalone.
function redirect(url) {
  url = url || STORE_URL;                        // STORE_URL = network macro placeholder
  if (typeof mraid !== "undefined" && mraid.open) { mraid.open(url); return; }
  window.open(url, "_blank");                     // standalone / double-click fallback
}
```

That's the whole integration surface: swap `STORE_URL` for the network's macro and the
CTA is wired.

## STARDUST FOUNDRY (clicker / idle) — live

![STARDUST FOUNDRY gameplay](media/clicker.gif)

Tap the reactor core to harvest **Stardust**; spend it on two upgrades that both
*visibly change the scene* — **Tap Yield** (each tap earns more) and **Collector
Drones** (passive income, each drone appears orbiting the core). The per-second rate
accelerates, the core grows and gains rings, the counter rolls up in abbreviated
big-number form (`1.2K`, `1.8M`)… until you cross the **ignite** threshold → the star
goes supernova → install CTA. Engaged play ignites in **~18 s**.

- **The macro-over-micro loop, compressed:** tap → earn → upgrade → numbers accelerate →
  milestone, in one screen.
- **Juice:** floating `+N`, stardust motes that fly toward the counter, screen shake, a
  rolling counter, an ignite flash + particle supernova, Web-Audio SFX with a rising
  tap-pitch.
- **Guided & unstuck:** an animated "TAP TO HARVEST" hand on the core; affordable
  upgrades pulse on their own to pull you into the loop; a stronger hint after a few idle
  seconds. Tapping always earns, so it can't soft-lock.
- **Weight:** **~29 KB**, one HTML file, no assets.

<p align="center"><img src="media/clicker-portrait.png" width="240" alt="tap the core to harvest"> <img src="media/clicker-endcard.png" width="240" alt="ignite the star — install CTA"></p>

> **Genre note:** this one is a **clicker / idle** ad (tap one object → upgrades → number
> grows), not a management tycoon. It lives at `/clicker/`. The real tycoon is the
> isometric run-around **GRILL RUSH** below.

Same single-file Canvas skeleton as GLYPHFALL (responsive DPR canvas, delta-time loop,
unified pointer input, state machine, procedural art, Web-Audio SFX) and the same MRAID /
`redirect()` hook at the bottom of [`clicker/index.html`](clicker/index.html).

## PRISM POUR (puzzle) — live

![PRISM POUR gameplay](media/puzzle.gif)

A **colour-sort** puzzle: tap a crystal vial, then another, to pour the top run of
glowing light-essence across — legal only onto a matching colour (or an empty vial).
Sort every vial to a single pure colour to solve. Two short levels (3 then 4 colours),
then the install CTA.

- **Provably never stuck.** Each level is *generated then verified solvable* by a built-in
  BFS solver before it's shown. That same solver drives the animated **guided first move**
  and the **inactivity hint** (recomputed from the *current* board, so it's always a real
  solving move) — plus an **Undo** to back out of any dead-end. It cannot soft-lock.
- **Accessible:** each colour carries a distinct **emblem** (circle / triangle / square /
  diamond), so it reads by shape as well as hue.
- **Juice:** selecting a run lifts it as a rounded blob (eased, with a subtle bob); pouring
  sends that blob **arcing up out of the vial neck, over, and settling into** the target vial
  (clipped to the glass so it reads as *poured*, never snapped in). Plus a "seal" flare +
  chime on a completed vial, a level-clear flourish, particle finish, Web-Audio SFX.
- **Weight:** **~32 KB**, one HTML file, no assets.

<p align="center"><img src="media/puzzle-portrait.png" width="240" alt="tap a vial then another to pour"> <img src="media/puzzle-endcard.png" width="240" alt="perfectly sorted — install CTA"></p>

Same single-file Canvas skeleton and the same MRAID / `redirect()` hook at the bottom of
[`puzzle/index.html`](puzzle/index.html).

## GRILL RUSH (tycoon) — live · *the flagship — isometric run-around*

![GRILL RUSH gameplay](media/tycoon.gif)

A proper **isometric run-around tycoon** (the idle-runner genre): you **steer a cook** around
a warm, sunny **burger diner** — light checkered floor, wood counter, tables & potted plants,
a steaming griddle, a hanging menu sign. Walk to the **grill** → a stack of burgers piles up
on your back; run to the **counter** → the burgers sell to hungry diners and become a **cash
stack**; carry the cash to a **build-pad** → drop it to construct a new station (a 2nd grill,
a 2nd counter, a 3rd grill). Build the whole venue → **BUSINESS BOOMING!** → install CTA. See
[`tycoon/DESIGN.md`](tycoon/DESIGN.md).

- **Isometric 2.5D on Canvas — no 3D engine.** Everything is an iso projection of a tile
  grid, painter-sorted by depth so the chef passes correctly in front of / behind stations.
  All art is **procedurally drawn** iso boxes / cylinders / stacked discs (an original
  minimalist style — no external assets, no Three.js, which is what keeps it tiny).
- **Run-around controls:** a **floating joystick** (drag anywhere to steer) with a
  **follow-camera** so the venue fills the screen; stations act by proximity — no buttons to
  hunt for.
- **The whole courier loop:** cook → **carry a stack on your back** → sell → **drop money on
  build-pads** → the venue visibly grows → milestone. Engaged play wins in **~30–40 s**.
- **Guided & unstuck:** an animated arrow re-points to your next target (grill → counter →
  nearest pad) as your hands fill and empty; inactivity auto-hint. **No fail / no timer**,
  grills always cook, and there's always a pad to bank cash into — it can't soft-lock.
- **Juice + sound:** stepping walk-cycle, stacks that grow/shrink on your back, coins that
  pop off sold burgers, build-pads that fill + **poof** into a station with particles &
  screen-shake, floating `+N`, a build progress bar, Web-Audio SFX **+ a muteable ambient
  hum**.
- **Weight:** **~40 KB** — an iso renderer + a courier economy + the warm-diner props in one HTML file with no
  assets, still ~140× under the 5 MB ceiling.

<p align="center"><img src="media/tycoon-portrait.png" width="200" alt="steer the chef to the grill"> <img src="media/tycoon-carry.png" width="200" alt="carry a stack of burgers on your back"> <img src="media/tycoon-endcard.png" width="200" alt="business booming — install CTA"></p>

Same MRAID / `redirect()` hook at the bottom of [`tycoon/index.html`](tycoon/index.html).

> **Clicker vs tycoon:** [STARDUST FOUNDRY](#stardust-foundry-clicker--idle--live) (at
> `/clicker/`) is the tap-one-object *clicker/idle* ad; **GRILL RUSH** here (at `/tycoon/`)
> is the *management* tycoon — run a venue, carry stacks, and build stations to expand.

## Run it locally

Each ad is one file — **just double-click `match3/index.html`** (no server needed).
To serve the whole hub:

```bash
npx serve .        # or: python -m http.server
# then open http://localhost:3000/match3/
```

## Honest scope note

"Playable-ready" here means **builds + runs correctly in a browser, portrait and
landscape, reaching the CTA under budget** — verified on desktop Chrome via automated
playthrough. The final "hooks in the first 2 s / feels right in the hand / passes a
specific network's spec" judgment is a human check on a real device. Say so, don't fake it.

## License

MIT — see [LICENSE](LICENSE). Original IP; genre mechanics are free to use, no real game's
characters, art, or brand are copied.
