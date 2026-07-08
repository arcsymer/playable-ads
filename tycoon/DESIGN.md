# NOVA NOODLES — tycoon playable ad (design)

A genuine **manage-a-venue** tycoon playable: run a little space noodle stand — cook,
serve a growing queue of travellers, earn star-coins, and **expand the operation** (build
more woks, cook faster) until business is booming. The satisfaction is *building up a
business*, not tapping one object. Original IP: theme, name (**NOVA NOODLES**), and all
art (procedural Canvas). Mechanic is generic management/tycoon — no real game's art,
brand, or content.

## Core loop (~20–30 s, compressed)
1. **Travellers arrive** into a queue on the counter (queue visibly fills → pressure).
2. **Woks auto-cook** bowls of glowing noodles (a progress ring; when done the wok holds a
   ready bowl). Buffer of one per wok, so idle woks fill up and wait — clear visual state.
3. **Serve:** tap a ready wok (or the front traveller) → a bowl flies to the front
   traveller → **+coins**, they leave happy, the queue advances, that wok resumes cooking.
4. **Spend coins to grow** — the real trade-off:
   - **New Wok** — build another cooking station (more throughput, visibly appears).
   - **Faster Cook** — reduce every wok's cook time (improve what you have).
5. Throughput visibly rises as the stand grows → **milestone** (serve the target count) →
   "BUSINESS BOOMING!" end card → install CTA.

Why this shape (honest design note): a full 3-station production *chain*
(ingredient→cook→serve with routing) adds fiddly item-routing beyond a 20–30 s ad's
budget. Instead the venue is modelled as **parallel cook stations + a serve action + an
expand-vs-improve economy** — the exact "buy a second station vs upgrade the first"
trade-off, which reads unmistakably as *a growing operation* while staying tappable on a
phone. Two station-affecting upgrades is enough for a playable ad.

## Guided + never-stuck
- **Guided first action:** an animated hand points at the first ready wok ("SERVE!"), then
  at the first affordable upgrade.
- **Auto-hint on inactivity:** pulse whatever is actionable — a ready wok to serve, else an
  affordable upgrade.
- **No soft-lock / no fail:** travellers never leave angry and there is no timer to lose;
  the queue just grows. Woks always keep cooking, so a bowl is always coming and idle
  travellers are always servable. You can only ever move forward.

## Juice
Floating `+coin` numbers, coin counter roll + pop, wok cooking rings + steam, a bowl
arc-flying to the traveller, travellers sliding in/out with a happy pop, a "NEW WOK!" /
"FASTER!" flash + particles + small screen shake on purchase, a served/target progress
bar, and a "BUSINESS BOOMING" flourish before the CTA.

## Economy (tuned via automated playthrough)
- Start: 1 wok, cook time 2.4 s, price 10 coins/serve.
- **New Wok:** 45 → 110 → 240 (up to 4 woks).
- **Faster Cook:** 35 → 75 → 150 (−18% cook time each, floor ~0.9 s).
- Travellers arrive ~every 1.7 s (faster than one wok → creates the pressure to expand).
- **Win:** serve 20 travellers. Progress bar = served / 20.

## Tech / guardrails (same as the other ads)
Single self-contained `index.html`: Canvas 2D, procedural art, Web-Audio SFX, **zero
external requests**. Responsive DPR canvas, delta-time loop, unified pointer input, state
machine (play → win), MRAID `redirect()` / CTA hook, portrait + landscape, large tap
targets. Under the < 5 MB (< 2 MB ideal) budget — size recorded in the README.
