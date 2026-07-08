# GRILL RUSH — isometric run-around tycoon (design)

Replaces the previous static NOVA NOODLES tycoon with a proper **isometric run-around
tycoon** — the idle-runner genre where you steer a character around a venue, pick up a
stack of product, carry it to the counter for a stack of money, and drop that money on
build-spots to expand. Original IP: theme, name (**GRILL RUSH** — a cosmic burger stand),
and **all art are procedurally drawn** (isometric boxes / cylinders / discs with shading —
an original minimalist iso style, no external assets, no 3D engine). Mechanic is the
generic idle-runner tycoon; nothing copied from any real game (and deliberately **not**
pizza, to stay clearly distinct).

## Isometric approach (2.5D, no 3D engine)
- **Iso projection of a 2D tile grid.** World floor is an `N×M` tile grid; screen position
  `sx = ox + (wx − wy)·TW/2`, `sy = oy + (wx + wy)·TH/2` (2:1 iso). Everything lives at a
  world `(wx, wy)` on the floor plane.
- **Depth sorting:** every drawable (stations, chef, customers, stacks, build-spots) is
  collected each frame and painter-sorted by `wx + wy` (then height), so the chef correctly
  passes in front of / behind objects.
- **Art is procedural:** iso cuboids (top + two shaded faces), cylinders, and stacked discs.
  Grill = shaded cuboid + hot-plate; counter = cuboid + till; chef = capsule body + dome +
  little stepping feet; carried stack = a pile behind the chef whose height scales with the
  carried count. No Three.js / WebGL — pure Canvas 2D, keeps the weight budget.

## Controls (mobile-first)
- **Floating joystick.** Touch/click-drag anywhere on the floor → a joystick appears at the
  touch point; drag to steer the chef continuously (screen direction is mapped back to iso
  world direction). Release to stop. Works with mouse for desktop. Big, forgiving, no
  precise tapping needed.
- Stations auto-interact by **proximity** — walk the chef onto a station and it acts. No
  extra buttons to hunt for.

## Core loop (~25–40 s, compressed)
1. **Grill (producer):** grills auto-cook burgers into a small tray. Stand the chef by a
   grill → burgers transfer onto the chef's **back-stack** (up to carry capacity).
2. **Counter (seller):** travellers queue at the counter. Carry burgers there → each burger
   is **sold** to a traveller (they leave happy) and becomes **cash** on your back; coins
   are earned (`+N` floater).
3. **Build-spots (grow):** clearly-marked ghost pads with a cost. Carry cash onto a pad →
   the cash **drops in** and fills its build meter; when funded it **builds** (a 2nd grill,
   a 2nd counter, a 3rd grill) and starts working — throughput visibly rises.
4. **Win:** build every spot → the venue is fully grown → **BUSINESS BOOMING!** → install
   CTA. (Win = "grew the whole venue", so cash always has a pad to go to — the loop never
   stalls and can't soft-lock.)

Single stack kind at a time (burgers **or** cash): you must sell your burgers before the
grill will load more, and bank your cash at a pad before grabbing more burgers — this is
what keeps the courier loop legible.

## Guided + never-stuck
- **Guided path:** an animated arrow floats from the chef to the **next** correct target —
  grill when empty-handed, counter when carrying burgers, nearest unbuilt pad when carrying
  cash. It re-points as your state changes, teaching the whole loop.
- **Auto-hint:** the arrow re-asserts after a few idle seconds.
- **No fail / no soft-lock:** no timer, travellers never leave angry, grills always cook, and
  there is always an unbuilt pad to bank cash into until the moment you win.

## Juice + sound
Stepping/bobbing walk cycle, stack items that hop on/off the back, coins that pop off sold
burgers, build-pads that fill and **poof** into a new station with particles + a screen
shake, floating `+N`, a served/target-style **build progress bar**, and a "BUSINESS
BOOMING" flourish → end card. Web-Audio SFX (pickup, sell/coin, build, win jingle) + a very
subtle muteable ambient pad; audio starts on first interaction (autoplay policy) and a mute
toggle is always present.

## Layout / economy (tuned via automated playthrough)
- Room ~7×6 tiles. Start: 1 grill (back-left), 1 counter (front-right), chef centre, and 3
  build-pads. Pads: **2nd grill**, **2nd counter**, **3rd grill** (costs tuned so the whole
  venue builds in ~30 s of play). Burger sell price and grill cook-rate tuned so cash flows
  fast enough to feel good but the growth still reads.

## Guardrails
Single self-contained `index.html`, Canvas 2D, procedural art, Web-Audio SFX, **zero
external requests**. Responsive DPR canvas, delta-time loop, floating-joystick pointer
input, state machine (play → win), MRAID `redirect()` / CTA hook, portrait **and**
landscape, large touch targets. This is the biggest ad (an iso renderer + a courier
economy) so it weighs more than the 29–35 KB ads — the real size is recorded in the README;
it is still one small self-contained file, far under the < 5 MB budget.
