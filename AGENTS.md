# Space Explorer — the arcade game

An endless 3D wireframe space shooter in a single vanilla JavaScript file
(`game.js`), built for the physical arcade cabinet at
[Platanus Hack 26 Caracas](https://hack.platan.us/26/arcade/space-explorer) and
embedded as the playable arcade cabinet and background 3D ship scenes on
[juanevillam.com](https://juanevillam.com/work/space-explorer).

**SHIPPED AT `v6` (2026-10-17).** Release tags (`v1` through `v6`) are pinned to
the exact commits accepted by the Platanus Hack arcade server and must never be
moved, deleted, or retagged.

## Working here

- Commit on `main`, conventional commits, signed (SSH key; `git commit` hangs if
  `ssh-add -l` is empty — never `--no-gpg-sign`). Don't push unless asked.
- **Only three files belong to the game build:** `game.js`, `metadata.json`, and
  `cover.png` (800 × 600 PNG, ≤ 500 KB). Never add external assets, modules, or
  stylesheets.
- **Keep `game.js` readable and commented.** `v1` shipped pre-minified; `v2`
  onward ships the readable, commented source, sized so every minifier lands
  under the 50 KB (51,200-byte) limit.
- **Measure all three minifiers before touching `game.js`:** `npm run check-restrictions`
  measures with SWC (`50,770` B at `v6`), while the arcade server and the
  website sync script minify with Terser (`50,706` B) and esbuild (`51,188` B).
  All three must stay ≤ 51,200 bytes.
- **Sync into `website/` after any game or cover change:** run
  `npm --prefix ../website run arcade` so `website/public/arcade/game.js` and
  `website/public/media/arcade/cover.webp` stay byte-in-sync with this repo.

## Hard cabinet constraints

1. **≤ 50 KB (`51,200` bytes) after minification (before gzip).**
   - Wrap the entire file in a top-level IIFE `(() => { ... })()` so minifiers
     mangle and inline top-level bindings.
   - Use `let` instead of `const` (Terser and esbuild preserve `const`, costing
     2 extra bytes per declaration).
   - Destructure palette and numeric constants from arrays so minifiers reference
     1-letter local variables instead of repeating hex literals.
2. **Pure vanilla JavaScript only:** no `import`, no `require`, no external
   URLs (`http://`, `https://`, `//`), and no network calls (`fetch`,
   `XMLHttpRequest`). The game runs inside a sandboxed offline iframe where
   Phaser 3.87.0 is injected globally by the host `index.html`.
3. **ASCII-only in-game text:** every string drawn on the canvas must use
   printable ASCII only (no `—`, `·`, `▸`, or `∞`) so glyphs never go missing on
   the cabinet's bitmap/monospace font.
4. **Never alter `CABINET_KEYS`:** the key map at the top of `game.js` matches
   the physical arcade cabinet wiring. Game logic must read arcade codes
   (`P1_U`, `P1_D`, `P1_L`, `P1_R`, `P1_1`..`P1_6`, `START1`), never raw keyboard
   keys.
   - Joystick (`WASD` / `P1_U..R`): pitch and horizon-locked yaw.
   - `B1` (`U` / `P1_1`): auto-aim plasma cannons (quad rapid-fire in Overdrive).
   - `B2` (`I` / `P1_2`): tap for invincible Dash (`+260` speed + wireframe
     ghosts), double-tap (`< 320 ms`) for aerobatics (Barrel Roll, full Loop, or
     Immelmann U-turn), hold for Nitro boost.
   - `B3` (`O` / `P1_3`): tap for single homing missile, hold (`> 0.4 s`) to
     paint up to 4 targets in Multi-Lock and fire a salvo on release.
   - `B4` / `B6` (`J` / `L` / `P1_4` / `P1_6`): 2.6 s spherical shield (reflects
     sniper beams back at the shooter; tap again to cancel early).
   - `B5` (`K` / `P1_5`): Overdrive (6 s invulnerability, clears enemy fire,
     quad cannons).
   - `START1` (`Enter`): start, pause, and confirm initials.
5. **Storage through `window.platanusArcadeStorage` with `localStorage` fallback:**
   top-3 high scores (`space-explorer:scores`, `{ v: 1, scores: [{ n, s }] }`)
   wrapped in `try / catch` so a rejecting bridge never blocks returning from the
   initials screen to the title.

## How `juanevillam.com` embeds Space Explorer

- **Playable cabinet (`website/src/components/ArcadeCabinet.astro`):** leads the
  **More Work** deck (`website/src/lib/work.ts`) and opens from the **Play Game**
  button, the Konami code (`↑ ↑ ↓ ↓ ← → ← → B A`), or the clickable 3D ship
  easter eggs. The `<iframe src="/arcade/index.html">` mounts lazily on play so
  Phaser's 1.2 MB bundle never loads on initial page paint.
- **Phone touch layer (`website/src/lib/arcade-touch.ts`):** lives entirely in
  `website/` so `game.js` stays keyboard-only. In landscape on phones, a virtual
  joystick and action arc dispatch synthetic `keydown` / `keyup` events into the
  iframe's `contentWindow` matching `CABINET_KEYS`.
- **Background 3D scenes (`website/src/lib/ship.ts`):** ports `game.js`'s compact
  mesh format (`SHIP`, `MISSILE`, `SENTRY`, `DESTROYER`), two-pass wireframe
  renderer, and flight/combat constants onto full-bleed background canvases in
  the Hero, More Work, and Footer.
