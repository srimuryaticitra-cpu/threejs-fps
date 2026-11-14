# Transcript — KURA OPS (threejs-fps-01)

Build log for the make.fun bounty entry "Make the best three.js tactical FPS"
($1,000, deadline Mon 23 Nov 2026 17:00 WIB). All game source was written in
this session (2026-11-14/15 WIB) by the agent, in Indonesian-first working
notes; the game text itself is Indonesian.

## What the game is

**KURA OPS — Operasi Tempurung.** A round-based tactical FPS in the spirit of
Counter-Strike, but its own map, art, sounds and names: the player is a
tortoise special agent; the enemy team is hares — fast, but they spray wide,
while tortoises are slow and steady and aim better. Win a round by
eliminating the enemy team or by planting the "Benih" (seed pod) at Site A
"Kebun" (garden) or Site B "Kolam" (pond) and letting it bloom; the defending
bot can defuse. First to 5 round wins (max 9 rounds). Money between rounds
buys weapons: Kura-9 pistol (free), Kilap SMG $900, Tempurung AR $1,200,
Teropong DMR $2,200, Granat Daun $300. Kills pay $300, a round win $1,000, a
loss $600, planting $300. Buy phase 12 s, round 100 s, pod timer 35 s, plant
3 s (hold E in a site), defuse 5 s. Single-player against 7 bots (3 tortoise
allies, 4 hares) on "Taman Senja", a 44 x 32 m dusk garden courtyard with a
fountain at mid, lanes, crates and hedges, amber site beacons, stars and a
moon. HUD: health, ammo, round timer, score, minimap, killfeed, buy menu.
Mouse look + keyboard movement, first-person view, crouch, jump. All sounds
are synthesized with WebAudio (no audio files). One mulberry32 stream per
round, seeded, so rounds are deterministic from the seed.

The joke the user asked for ("serius tapi fun/lucu", calm but sure, different
from the rest): the slowest animal in the garden is the special forces
operator, and the fast ones can't aim.

## Files written in this session

- `game/src/rng.js` — mulberry32, range/pick/clamp/lerp.
- `game/src/palette.js` — the 6-colour dusk palette.
- `game/src/audio.js` — procedural SFX, lazy AudioContext.
- `game/src/map.js` — geometry, COLLIDERS AABBs, `resolveCircle`,
  `segmentBlocked` slab test, WAYPOINTS/ROUTES/SITES/SPAWNS, stars + moon.
- `game/src/weapons.js` — weapon table, headshot x3, spread/recoil/bloom.
- `game/src/player.js` — accel/friction, crouch, jump, AABB collision.
- `game/src/combat.js` — hitscan `shootRay`, pooled tracers/sparks, grenade
  physics, explosion flash light.
- `game/src/bots.js` — bot meshes (tortoise shell / hare ears) and AI:
  hold/patrol/engage/investigate/plant/defuse, vision cone + 2D wall test,
  hearing radius 18 m, reaction 0.25–0.55 s, burst fire; hares faster (5.0 vs
  4.2) but x1.7 aim error.
- `game/src/hud.js` — DOM HUD setters that skip unchanged values + 2D canvas
  minimap (teammates / spotted enemies).
- `game/src/input.js` — pointer lock, look deltas, fire.
- `game/src/game.js` — the single state object; phases
  menu → buy → live → roundEnd → matchEnd; economy; plant/defuse;
  player/bot shooting; grenades; killfeed; carrier passing; `shotSetup()` for
  the four capture poses.
- `game/src/main.js` — boot, fixed 60 Hz loop, resize, overlay wiring,
  `window.__dbg()` + `window.__scene` debug hooks.
- `game/index.html` — HUD DOM, buy menu, one import map
  (`"three": "./vendor/three/three.module.min.js"`).
- `main.ts` — copies game/ + screenshots/ to /output and writes
  dossier.json via the kit's `buildDossier`.
- `serve.ts` — static file server (Node stdlib only).
- `README.md`, `LICENSE` (MIT, first line `SPDX-License-Identifier: MIT`),
  `shots.json` (4 captures, `&seed=1`), `.makefun/owner.json` (one line,
  wallet `CX5dVHNdf6jaHiurmGincNgN1dDL3r4ocMwPVjeXXo5B`).

Copied unchanged, not authored: `vendor/makefun-threejs.ts` (from the kit),
`game/vendor/three/three.module.min.js` and `three.core.min.js` (three.js
0.187.0 from npm, byte-for-byte; the two `./three.core.js` specifiers in the
module rewritten to `./three.core.min.js`), `LICENSE`.

## How it was verified

- All 12 game modules pass `node --check`. `main.ts` passes `node --check`.
- three.js verified byte-for-byte against npm `three@0.187.0`
  (sha256 `three.core.min.js` = npm `build/three.core.js`;
  `three.module.min.js` identical after the specifier rewrite).
- Played in headless Chrome 157 (SwiftShader WebGL2) via CDP, served as
  `file://` with `--allow-file-access-from-files` (this Chrome blocks
  `http://127.0.0.1` navigation). Zero console errors, zero failed requests
  in every capture.
- Four real screenshots rendered from the game's own code
  (`?shot=lobby|fight|plant|win&seed=1`, 1280x800 JPEG q85):
  lobby — buy phase, round 1, Tempurung AR 25/100, $800, 7 bots alive;
  fight — live, round 2, score 1–1, kill confirmed (kills=1, money
  $1,100 = +$300), ammo 22/100, HP 100, killfeed "Kamu ▸ Lari · Tempurung AR";
  plant — live, round 3, score 2–1, "MENANAM BENIH — SITE A" bar at 60%,
  hare + ally in frame; win — round end, round 3, score 3–1, planted pod
  glowing at Site A, fallen hares, banner "RONDE DIMENANGKAN".
- `main.ts` run twice with `MAKEFUN_OUTPUT_DIR=out`; dossier.json built,
  outputs compared for determinism.

## Bugs found and fixed in this session

- `player.js` imported `resolveCircle` from the wrong module and started
  facing the wrong way (yaw π → 0).
- `bots.js`: the foe list wrongly included the player agent for kura bots;
  planting/defusing progress reset to 0 every tick; `ctx.podSite` split into
  `targetSite` (plant target) vs `plantedSite`; a missing brace in the
  attacker/defender branch.
- `game.js`: killfeed team classes now use the victim's team; a duplicated
  plant-hint block removed; `scene.updateMatrixWorld(true)` added before
  raycasts — scripted shots before the first render were raycasting stale
  spawn-time transforms, which was why the first fight capture had 0 kills.
  Viewmodel and muzzle flash shrunk. Capture poses: round/score set before
  `beginLive()` (banner said RONDE 1 while the HUD said RONDE 2); the plant
  progress re-applied after the final settle step; the buy menu hidden during
  roundEnd; lobby money set to $2,000 then the rifle bought ($800 left).
- **The tan band in the sky (fight shot only).** A large tan wedge filled the
  upper third of the frame in every fight retake. Bisected by hiding scene
  objects in the live page and re-screenshotting: stars — stays; moon —
  stays; tracers — stays; all Plane/Box terrain — stays; all amber meshes —
  stays; all cylinders — stays; all spheres — GONE; hiding only the 16 spark
  spheres — GONE. Root cause: a bot's shot had hit the player's own head
  proxy, and `fireSpark` placed an amber spark at the hit point — about 20 cm
  from the camera, so the 0.18 m sphere filled the view. Fix in `combat.js`:
  no impact spark when the hit agent is the player (the hurt sound, shake
  and health bar already say the player was hit). Retake confirmed clean.
- Site beacons moved from the site centre to the site's far corner and
  shortened (3.4 m → 2.8 m, letter 4.1 m → 3.4 m) so the pillar never stands
  between the player and the planted pod or crosses the score HUD.

## Live play-test (no `?shot`) and two fixes it found

A 120 s headless live session (click "Mulai Misi", buy a grenade with key 4,
hold W and fire in bursts) ran menu → buy → live with zero console errors
and zero failed requests — but it exposed two real gameplay bugs:

1. **Bots fought during the buy phase.** By the time live started, only 3 of
   7 bots were still alive: the feed showed four bot-vs-bot kills while the
   phase was still 'buy'. A round must not be decided before it starts.
   Fix: `botStep` now only perceives, engages, shoots and plants while
   `ctx.phase === 'live'` (the phase is passed in the bot context from
   `game.js`); during buy bots only walk to their holds.
2. **A team wiped out before live never ended the round.** `checkRoundEnd()`
   was only called from `damageAgent` after a kill, and it returns early
   unless the phase is 'live' — so a wipe during buy left a live round with
   nobody left to fight, dragging on to the timer. Fix: `checkRoundEnd()` is
   now also called every step while live, so the round ends as soon as live
   starts if a team is already gone.

The re-test after the fix (150 s, real CDP mouse/key input, pointer lock
engaged) confirmed the whole loop live: all 7 bots alive and no feed kills
through the entire buy phase; bots engaging only once live; the player
walking north (camera z 12.50 → 2.62) and firing the Kura-9 (ammo 12 → 4,
one round per trigger pull, semi-auto); bots shooting back (player HP
100 → 78 → 12 → 0, killed in the open); the ally carrier Tama planting the
Benih at Site B after the player died (`planted=true`, feed "Tama menanam
Benih di Site B"); the round ending by elimination ("Ronde 1: KURA menang —
Tim Kelinci habis"), score 1–0, money $800 → $1,800 (+$1,000 win); then
round 2 starting clean (buy phase, HP 100, 7 bots alive). Zero console
errors, zero failed requests across the whole session. (The first live test
had shown player firing not working only because synthetic JS clicks do not
grant pointer lock in headless Chrome; with a real CDP click the lock
engaged and firing worked.)

## Not done

- No GitHub repo created, no submission made (per the task: report when the
  files are ready). The bounty opens Mon 16 Nov 2026 17:00 UTC and ends Mon
  23 Nov 2026 17:00 UTC.
