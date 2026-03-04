# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Repository

Single-game repository tracked on GitHub at `https://github.com/sahilpatni95/top-down-shooter`. Only `shooter.html` and `.gitignore` are tracked — `tictactoe.html` is excluded via `.gitignore`.

## Running the Game

No build step. Open `shooter.html` directly in any modern browser:

```bash
# Windows — open in default browser
start shooter.html

# Or serve locally to avoid any browser file:// restrictions
npx serve .
python -m http.server 8080
```

## Committing Changes

Every change to `shooter.html` follows this workflow:

```bash
git add shooter.html
git commit -m "<type>: <description>"
git push
```

Commit type prefixes: `feat:` · `fix:` · `style:` · `refactor:` · `chore:`

## Architecture

The entire game is one self-contained file (`shooter.html`) with no external dependencies. All logic, styles, and markup are inline. The script section is divided into clearly labelled comment blocks in this order:

1. **SETUP** — Canvas context (`ctx`), constants `W=800 H=600`
2. **SETTINGS** — `settings` object + UI event wiring for the bottom panel
3. **AUDIO** — Web Audio API procedural SFX; all sounds synthesised at runtime, routed through a single `masterGain` node
4. **GAME STATE** — `STATE` enum (`MENU=0 PLAYING=1 GAME_OVER=2`), `state` variable
5. **INPUT** — `keys{}` map (keydown/keyup) + `mouse{x,y,held}` (mousemove/down/up)
6. **BACKGROUND THEMES** — Four renderers (`drawBgGrid`, `drawBgStars`, `drawBgDungeon`, `drawBgNeon`) dispatched by `drawBackground()`
7. **WAVE DEFS** — `WAVE_DEFS[0..4]` for waves 1–5; `getWaveDef(n)` returns infinite-scaling config beyond wave 5
8. **ENTITY DEFS** — `ENEMY_DEF` lookup table with base stats for `grunt/fast/tank`
9. **GAME INIT** — `startGame()` resets all state and reads `DIFF_PARAMS[difficulty]`; `startNextWave()` / `endGame()`
10. **PLAYER** — `updatePlayer(dt)`, `firePlayerBullet()`, `drawPlayer()`; colour helpers `darken()` / `blendWhite()`
11. **ENEMIES** — `spawnEnemy(type)` picks a random screen edge; `updateEnemies(dt)` homes toward player; `drawEnemy(e)` switches on `e.type`
12. **BULLETS** — Array of `{x,y,vx,vy,owner,radius,lifetime,damage}`; bullets are filtered out when `lifetime ≤ 0` or off-screen
13. **PARTICLES** — Simple alpha-decay system; spawned on shoot, hit, and death events
14. **COLLISION** — `circleCollide(a,b)` checks `dist² < (ra+rb)²`; `handleCollisions()` iterates bullets→enemies then enemies→player
15. **WAVE MANAGER** — `updateWaves(dt)` drives spawn timer and transitions to `waveClearing` once all queued enemies are spawned and dead
16. **HUD** — Drawn after `ctx.restore()` so it is never offset by screen shake
17. **SCREENS** — `drawMenu()` and `drawGameOver()` each call `drawBackground()` then overlay text
18. **GAME LOOP** — `requestAnimationFrame` loop; `dt` capped at 100 ms to prevent tunnelling after tab-switch

## Key Data Flows

- **Difficulty** is read once at `startGame()` from `DIFF_PARAMS[settings.difficulty]` and baked into `player.hp/maxHp` and live collision damage. Enemy speed multiplier is applied at spawn time in `spawnEnemy()`.
- **Player colour** (`settings.playerColor`) is read every frame in `drawPlayer()` and `drawBullet()` — change it and it updates instantly.
- **Screen shake** is a single scalar `screenShake` decremented by `×0.82` each frame; it offsets the `ctx.save/translate` wrapping background+entities+bullets (HUD is drawn outside this block).
