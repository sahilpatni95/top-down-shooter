# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Repository

Repository tracked on GitHub at `https://github.com/sahilpatni95/top-down-shooter`. Tracked files: `shooter.html`, `tictactoe.html`, `README.md`, `CLAUDE.md`, `.gitignore`.

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

**After every meaningful unit of work, commit and push immediately.** Do not batch multiple unrelated changes into one commit. This ensures no work is ever lost and the GitHub remote always reflects the current state.

**After every change, also update `README.md` and `CLAUDE.md`** to reflect what was added or modified, then include both files in the same commit as the code change.

```bash
git add <changed-files>
git commit -m "<type>: <description>"
git push
```

Commit type prefixes: `feat:` · `fix:` · `style:` · `refactor:` · `chore:`

### When to commit

- After adding or completing a feature
- After fixing a bug
- After any edit to a file that leaves the code in a working or clearly improved state
- After updating documentation or config files (README, CLAUDE.md, .gitignore)

### Commit message rules

- Use the imperative mood in the description (`add`, `fix`, `update`, not `added`, `fixes`)
- Be specific — describe *what changed and why*, not just *that something changed*
- Bad: `fix: changes` · Good: `fix: prevent player from moving outside canvas bounds`
- Always append the co-author trailer:

```
Co-Authored-By: Claude Sonnet 4.6 <noreply@anthropic.com>
```

## Architecture

The entire game is one self-contained file (`shooter.html`) with no external dependencies. All logic, styles, and markup are inline. The script section is divided into clearly labelled comment blocks in this order:

1. **SETUP** — Canvas context (`ctx`), constants `W=800 H=600`; `resizeCanvas()` (CSS `scale` transform to fill viewport, scales settings panel too), `toggleFullscreen()` (Fullscreen API wrapper); `resize` + `fullscreenchange` listeners; `resizeCanvas()` called once at startup
2. **SETTINGS** — `settings` object (including `moveMode: 'keyboard'|'mouse'`, `playerIcon: 'circle'|'triangle'|'diamond'|'square'|'star'`) + UI event wiring for the bottom panel; 7-column CSS grid (columns: Name, Background, Difficulty, Volume, Player Color, Movement+Fullscreen, Icon)
3. **AUDIO** — Web Audio API procedural SFX; all sounds synthesised at runtime, routed through a single `masterGain` node
4. **LEADERBOARD** — `LB_KEY` constant; `getLeaderboard()` (reads localStorage); `saveToLeaderboard(name,score,wave,kills,difficulty)` (push → sort desc → splice(10) → return rank); `rankColor(rank)` / `rankLabel(rank)`; `roundRect(x,y,w,h,r)` path helper; `drawLeaderboardPanel(px,py,pw,title,maxRows,highlightRank)` — rounded bordered panel with RANK/NAME/SCORE/WV/D columns, medal colours, gold-tint on highlighted row
5. **SCORE POPUPS** — `scorePopups[]`; `spawnScorePopup(x,y,text,color)` pushes `{x,y,text,color,alpha,vy,life}`; `updateScorePopups(dt)` floats up + fades; `drawScorePopups()` bold 15px monospace with shadow glow
6. **GAME STATE** — `STATE` enum (`MENU=0 PLAYING=1 GAME_OVER=2`), `state` variable; `paused` boolean (toggled by `P`, reset in `startGame()`); `killCount`, `playerRank`, `waveEnemiesTotal` let declarations; `joystick` object `{active,startX,startY,dx,dy,startSet}` for virtual touch joystick
7. **INPUT** — `keys{}` map (keydown/keyup) + `mouse{x,y,held}` (mousemove/down/up); mouse coords mapped via `(clientX-r.left)*(W/r.width)` to account for CSS scale; `P` toggles `paused`; `F` calls `toggleFullscreen()`; `Escape` handles both game-over and paused-while-playing states; `handleTouches(touches)` splits touches by canvas half (left=joystick, right=aim+fire); `touchstart/touchmove/touchend` listeners on canvas with `passive:false`
8. **BACKGROUND THEMES** — Four renderers (`drawBgGrid`, `drawBgStars`, `drawBgDungeon`, `drawBgNeon`) dispatched by `drawBackground()`
9. **WAVE DEFS** — `WAVE_DEFS[0..4]` for waves 1–5; `getWaveDef(n)` returns infinite-scaling config beyond wave 5
10. **ENTITY DEFS** — `ENEMY_DEF` lookup table with base stats for `grunt/fast/tank`
11. **GAME INIT** — `startGame()` resets all state (including `paused=false killCount=0 playerRank=-1 scorePopups=[]`) and reads `DIFF_PARAMS[difficulty]`; `startNextWave()` sets `waveEnemiesTotal`; `endGame()` calls `saveToLeaderboard()` and stores rank in `playerRank`
12. **PLAYER** — `updatePlayer(dt)` branches on `settings.moveMode`: keyboard mode uses WASD/Arrows with diagonal normalisation; mouse mode moves player toward cursor (stops within 6 px); both modes aim via `Math.atan2`; `firePlayerBullet()`, `drawPlayer()` switches on `settings.playerIcon` (circle/triangle/diamond/square/star) to draw the body shape; colour helpers `darken()` / `blendWhite()`
13. **ENEMIES** — `spawnEnemy(type)` picks a random screen edge; `updateEnemies(dt)` homes toward player; `drawEnemy(e)` switches on `e.type`
14. **BULLETS** — Array of `{x,y,vx,vy,owner,radius,lifetime,damage}`; bullets are filtered out when `lifetime ≤ 0` or off-screen
15. **PARTICLES** — Simple alpha-decay system; spawned on shoot, hit, and death events
16. **COLLISION** — `circleCollide(a,b)` checks `dist² < (ra+rb)²`; `handleCollisions()` iterates bullets→enemies then enemies→player; increments `killCount` on enemy death; calls `spawnScorePopup()` with enemy point value
17. **WAVE MANAGER** — `updateWaves(dt)` drives spawn timer and transitions to `waveClearing` once all queued enemies are spawned and dead
18. **HUD** — Drawn after `ctx.restore()` so it is never offset by screen shake; includes semi-transparent backing panels, kill counter (top-right below score), thin wave progress bar (under wave counter, fills as enemies die), `shadowBlur` glow on wave banner; mobile pause button (❚❚/▶ toggle) drawn in top-right corner (W-44,8,36,36) as tap target
19. **SCREENS** — `drawMenu()` has two-panel layout (left: controls + mini enemy shapes; right: HALL OF FAME leaderboard); `drawGameOver()` has 2×2 stats grid (Score/Waves/Kills/Difficulty), pulsing rank badge, embedded LEADERBOARD with player row highlighted; `drawPauseOverlay()` draws semi-transparent black veil + "PAUSED" glow text + resume hint; `drawJoystick()` draws virtual joystick ring + knob at `joystick.startX/Y + dx/dy` with 35% alpha
20. **GAME LOOP** — `requestAnimationFrame` loop; `dt` capped at 100 ms to prevent tunnelling after tab-switch; all update calls wrapped in `if (!paused)`; `drawJoystick()` called after `drawHUD()` when `joystick.active`; `drawPauseOverlay()` called after HUD when paused

## Key Data Flows

- **Difficulty** is read once at `startGame()` from `DIFF_PARAMS[settings.difficulty]` and baked into `player.hp/maxHp` and live collision damage. Enemy speed multiplier is applied at spawn time in `spawnEnemy()`.
- **Player colour** (`settings.playerColor`) is read every frame in `drawPlayer()` and `drawBullet()` — change it and it updates instantly.
- **Screen shake** is a single scalar `screenShake` decremented by `×0.82` each frame; it offsets the `ctx.save/translate` wrapping background+entities+bullets (HUD is drawn outside this block).
- **Kill tracking** — `killCount` incremented in `handleCollisions()` on enemy death; `playerRank` set in `endGame()` via `saveToLeaderboard()`; both reset to `0` / `-1` in `startGame()`.
- **Leaderboard persistence** — `saveToLeaderboard()` writes to `localStorage['tds_leaderboard_v1']` (JSON array ≤10, sorted desc by score); `getLeaderboard()` reads it with error fallback to `[]`; `drawLeaderboardPanel()` reads fresh on every call so menu always reflects current state.
