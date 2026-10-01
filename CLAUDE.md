# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project

Asteroids clone in vanilla HTML5 Canvas + ES6. No dependencies, no bundler, no package.json, no tests, no linter. UI text and code comments are in Spanish; keep that convention.

## Running

Open `index.html` in a browser, or serve locally with `npx serve .` (http://localhost:3000). There is no build step. The game is deployed as static files (GitHub Pages demo linked from README).

## Architecture

All logic lives in `game.js` (loaded by `index.html` onto a fixed 800×600 `<canvas id="canvas">`). It uses top-level globals and plain classes, in this order:

- **Input**: `keys` (held state) and `justPressed` (edge-triggered). Use `pressed(code)` for one-shot actions like shooting — it consumes the flag on read. Read `keys[...]` directly for held actions (rotate/thrust).
- **Entities** (`Bullet`, `Asteroid`, `Ship`, `Particle`): each has `update(dt)`, `draw()` and a `dead` flag. Dead entities are removed by array filtering in `update()`, not in the entity itself.
- **Game state**: module-level `let` vars (`ship`, `bullets`, `asteroids`, `particles`, `score`, `lives`, `level`) plus `state` = `'playing' | 'dead' | 'gameover'`. `'dead'` is the 2s respawn delay (`deadTimer`); `'gameover'` waits for Space to call `initGame()`.
- **Loop**: `loop(ts)` → `update(dt)` → `draw()` with `dt` in seconds, clamped to 0.05 to avoid tunneling after tab switches. All motion/cooldowns are dt-based.

Key mechanics:
- World is toroidal: positions wrap via `wrap(v, max)`. Collisions use circle distance (`dist`) with no wrap-aware handling across edges.
- Asteroid size 1–3 indexes the parallel arrays `RADII`, `SPEEDS`, `POINTS`; note they are indexed by size (index 0 unused). `split()` spawns two of `size - 1`.
- Ship-asteroid collision uses `a.radius * 0.82` for forgiveness and is skipped while `ship.invincible > 0` (3s after `reset()`).
- Disparo triple: `PowerUp` se suelta con `TRIPLE_CHANCE` al destruir un asteroide, solo una vez por partida (`tripleSpawned`, reseteado en `initGame()`). Al recogerlo, `ship.tripleTimer` (10s) hace que `tryShoot()` dispare 3 balas en abanico; `ship.reset()` lo limpia.
- Clearing all asteroids calls `nextLevel()` (spawns `3 + level`, resets the ship).

README note: it still mentions power-ups and a "shooting star" asteroid; those were removed from the code (commit 13e713f) — don't assume they exist.
