# Space Explorer

An endless 3D wireframe space shooter that fits in 50 KB. Fly through three
sectors — Orion, Nebula, and Event Horizon — at the edge of a gravitational
black hole, dogfighting predictive enemy aces, mine-laying corvettes, laser
snipers, and surface walkers and turrets. Chain flight gates to charge
Overdrive, reflect sniper beams with a timed spherical shield, lock up to four
homing missiles at once, and bring down a capital Destroyer — its deck turrets,
its two shield domes, and its command bridge — that keeps jumping back out of
hyperspace.

The entire game lives in a single vanilla JavaScript file (`game.js`, 49.5 KB
minified) on top of Phaser 3 (v3.87.0), with zero external assets, zero imports,
and zero network calls. Every ship mesh, planetary station, nebula, accretion
disk, sound effect, and two-minute choral-battle soundtrack is generated
procedurally at runtime. Built for the physical arcade cabinet at Platanus Hack
26 Caracas and embedded in [`juanevillam.com`](https://juanevillam.com/work/space-explorer)
with a site-side touch HUD for phones.

- **Web:** [juanevillam.com/work/space-explorer](https://juanevillam.com/work/space-explorer)
- **Platanus Arcade:** [hack.platan.us/26/arcade/space-explorer](https://hack.platan.us/26/arcade/space-explorer)

## Layout

| path            | what it is                                                                                                              |
| --------------- | ----------------------------------------------------------------------------------------------------------------------- |
| `game.js`       | the entire game — 3D projection engine, compact wireframe models, flight physics, enemy AI, boss stages, Web Audio synth |
| `metadata.json` | title, description, and single-player cabinet configuration                                                             |
| `cover.png`     | 800 × 600 PNG cabinet cover artwork                                                                                     |
| `index.html`    | sandboxed 800 × 600 host shell that loads Phaser 3.87.0 and `game.js`                                                   |
| `docs/`         | Phaser 3 reference notes included with the original arcade starter                                                      |

Space Explorer began as a fork of the Platanus Hack 26 Caracas arcade starter
and lives alongside [`website`](https://github.com/juanevillam/website), which
embeds `game.js` in its arcade cabinet and runs a small copy of the same 3D
simulation in the background easter egg scenes on `juanevillam.com`.

## Develop

```sh
npm install
npm run dev                  # local cabinet preview on :3001
npm run check-restrictions   # verify ≤ 50 KB minified and zero external calls
```

After any change to `game.js` or `cover.png`, run `npm run arcade` inside
`website/` to minify `game.js` with esbuild and sync the cabinet bundle into
`juanevillam.com`.

## A 3D engine and procedural synthesizer in 50 KB

Because the arcade runner enforces a strict 51,200-byte minified ceiling and an
offline sandboxed iframe, `game.js` uses Phaser only as a 2D `Graphics` canvas
and game loop, running its own perspective projection (`FOCAL = 420`, near-plane
clipping at `NEAR = 14`, frustum culling, and distance LOD) and two-pass
wireframe renderer (a wide low-alpha halo stroke under a crisp core stroke).
3D meshes are stored as flat vertex arrays paired with ASCII edge strings where
each character encodes a vertex index (`charCodeAt(i) - 48`), and the entire
file is wrapped in a single IIFE with destructured palette tokens and short
property keys so SWC, Terser, and esbuild all minify it below 50 KB.

Audio runs through the browser's Web Audio API with no samples: `Sfx` synthesizes
laser bolts, explosions, thrusters, shield parries, and arpeggiated fanfares from
oscillators and filtered white noise, while `Music` runs a lookahead step
sequencer (`0.4 s` window) that plays an eight-phrase Phrygian choral-and-brass
score in normal flight and switches to a faster tritone boss theme while the
Destroyer holds the sector.

## Standards

[`AGENTS.md`](AGENTS.md) is the engineering record: the 50 KB multi-minifier
budget, the cabinet key mapping (`CABINET_KEYS`), the storage bridge contract,
and how `juanevillam.com` embeds the game without modifying the cabinet source.

## License

All rights reserved — see [`LICENSE`](LICENSE). The source is public to read;
it is not licensed for reuse or redistribution.
