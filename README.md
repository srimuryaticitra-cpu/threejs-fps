# KURA OPS — Operasi Tempurung

A round-based tactical FPS in the browser, built on three.js 0.187.0: slow and
steady wins. You are a tortoise special agent; the enemy team is a squad of
hares — fast, reckless, and easy to headshot. Plant the Benih (seed pod) at
Site A (Kebun, the garden) or Site B (Kolam, the pond), or eliminate the whole
Kelinci team. Money earned each round buys better guns in the next buy phase.
First to 5 round wins takes the match.

## How to play

Serve `game/` with any static server and open it in desktop Chrome:

```sh
node serve.ts
# game/ is on http://localhost:8080
```

No build step, no network requests: every file the game needs is under `game/`,
and three.js 0.187.0 is vendored there.

## Controls

| Key | Action |
| --- | --- |
| W A S D | Move (acceleration + friction) |
| Mouse | Look (pointer lock) |
| Click | Fire (hold for auto weapons) |
| R | Reload |
| G | Throw Granat Daun |
| E (hold) | Plant the Benih, standing inside a site |
| Ctrl / C | Crouch (tighter spread) |
| Space | Jump |
| 1–4 | Buy during the buy phase (Tempurung AR, Kilap SMG, Teropong DMR, Granat Daun) |
| B | Hide / show the buy menu |

## The tactics

- **Stop to aim.** Spread grows while moving and with every shot in a burst;
  crouching halves it. A headshot deals 3× damage.
- **The hares are fast but spray wide.** Hold an angle, let them come to you.
- **Money matters.** Kills pay $300, a round win $1000, a loss $600, planting
  $300. Die and you lose your bought gun — back to the free Kura-9.
- **Bots play routes.** Allies push a site along the lanes; defenders hold the
  sites, rotate toward gunfire, converge on a planted pod and defuse it.
  The pod blows 35 seconds after planting.
- **The minimap** (top left) shows teammates and enemies you have spotted.

## Structure

```
main.ts                  entrypoint: copies game/ + screenshots/ to /output, writes dossier.json
serve.ts                 tiny static server for local play
vendor/makefun-threejs.ts  kit library, unchanged
game/
  index.html             the page: one import map ("three" -> vendor three), the HUD
  vendor/three/          three.js 0.187.0, byte for byte from npm (two ./three.core.js
                         specifiers rewritten to ./three.core.min.js)
  src/
    main.js              boot + fixed 60 Hz loop + deterministic ?shot= captures
    game.js              the match: phases, economy, plant/defuse, shooting, grenades
    bots.js              bot bodies + AI (hold, patrol, engage, investigate, plant, defuse)
    player.js            acceleration movement, crouch, jump, AABB collision
    combat.js            hitscan rays, pooled tracers/sparks, grenade physics
    weapons.js           the armoury: stats, spread, recoil, bloom
    map.js               Taman Senja: geometry, colliders, waypoints, sites, lighting
    hud.js               DOM HUD + 2D minimap
    input.js             keyboard, mouse look, pointer lock
    audio.js             procedural WebAudio sounds (no audio files)
    palette.js           the six-colour dusk palette
    rng.js               seeded RNG (mulberry32) — one stream per round
screenshots/              4 real renders of the game's own code (see shots.json)
```

## Screenshots

Taken in headless Chrome from the game's own deterministic capture mode
(`?shot=lobby|fight|plant|win&seed=1`):

1. `1-lobby.jpg` — buy phase at the Kura spawn, Toko Tempurung open.
2. `2-fight.jpg` — mid-lane exchange, tracer and hitmarker live.
3. `3-plant.jpg` — planting the Benih at Site A, 60% in.
4. `4-win.jpg` — round won, the pod glowing at Site A, Kelinci down.

## License

MIT — see LICENSE.
