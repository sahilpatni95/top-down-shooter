# Top-Down Shooter

A browser-based top-down shooter built entirely in a single HTML file — no frameworks, no bundlers, no dependencies. Just open `shooter.html` and play.

**Live repo:** https://github.com/sahilpatni95/top-down-shooter

---

## Table of Contents

- [Features](#features)
- [How to Play](#how-to-play)
- [Controls](#controls)
- [Enemy Types](#enemy-types)
- [Wave System](#wave-system)
- [Leaderboard](#leaderboard)
- [Difficulty Modes](#difficulty-modes)
- [Settings Panel](#settings-panel)
- [Background Themes](#background-themes)
- [Audio](#audio)
- [Tech Stack](#tech-stack)
- [Project Structure](#project-structure)
- [Development Workflow](#development-workflow)

---

## Features

- **Zero dependencies** — single `shooter.html` file, works offline in any modern browser
- **requestAnimationFrame game loop** with `deltaTime` capping for frame-rate-independent movement
- **3 enemy types** with distinct shapes, speeds, and HP pools
- **5 handcrafted waves** + infinite procedurally-scaled waves beyond wave 5
- **Web Audio API** — all sound effects generated procedurally at runtime (no audio files)
- **Particle system** — bullet muzzle flash, enemy death explosions, player hit sparks
- **Screen shake** on player damage
- **Invincibility frames** after being hit (duration varies by difficulty)
- **Settings panel** — name, background theme, difficulty, volume, sound toggle, player colour
- **HUD** — colour-coded health bar, score, kill counter, wave counter with progress bar, enemy count, animated wave banner with glow
- **localStorage leaderboard** — top-10 scores persisted across sessions; visible as "HALL OF FAME" on the menu and "LEADERBOARD" on the game-over screen
- **Score popups** — floating `+N` text rises from each enemy kill (colour-coded by enemy type)
- **Kill counter** — tracks total kills per run; shown in HUD top-right and on the game-over stats panel
- **Wave progress bar** — thin bar under the wave counter fills as enemies in the current wave are eliminated
- **Redesigned menu** — two-panel layout: controls reference (left) + HALL OF FAME leaderboard (right)
- **Redesigned game-over** — 2×2 stats grid (Score/Waves/Kills/Difficulty) + pulsing rank badge + embedded leaderboard with player's row highlighted

---

## How to Play

1. Clone or download the repository
2. Open `shooter.html` in any modern browser (Chrome, Firefox, Edge, Safari)
3. Customise your settings in the panel at the bottom of the screen
4. Click the canvas or press **Enter** to start
5. Survive as many waves as possible and maximise your score

No server, build step, or installation required.

---

## Controls

| Input | Action |
|-------|--------|
| `W` / `↑` | Move up |
| `S` / `↓` | Move down |
| `A` / `←` | Move left |
| `D` / `→` | Move right |
| **Mouse** | Aim |
| **Hold left click** | Fire (continuous, ~6-7 shots/sec) |
| **Enter** | Start game / Restart after game over |
| **Esc** | Return to menu (from game over screen) |

> Movement is diagonal-corrected: holding two directions simultaneously normalises speed so you don't move faster diagonally.

---

## Enemy Types

| Enemy | Shape | HP | Speed | Damage | Points |
|-------|-------|----|-------|--------|--------|
| **Grunt** | Red triangle | 40 | 90 px/s | varies | 10 |
| **Fast** | Orange diamond | 20 | 160 px/s | varies | 20 |
| **Tank** | Purple circle + ring | 120 | 50 px/s | varies | 50 |

- All enemies spawn off-screen on a random edge and home directly toward the player
- Enemies show a green HP bar only once damaged
- Each enemy type plays a distinct death sound

---

## Wave System

| Wave | Enemy Count | Types Available | Spawn Interval |
|------|-------------|-----------------|----------------|
| 1 | 5 | Grunt | 2000 ms |
| 2 | 8 | Grunt, Fast | 1800 ms |
| 3 | 10 | Grunt, Fast | 1500 ms |
| 4 | 8 | Grunt, Fast, Tank | 1500 ms |
| 5 | 12 | Grunt, Fast, Tank | 1200 ms |
| 6+ | 12 + 3×(wave−5) | All | max(600, 1200−50×(wave−5)) ms |

- A new wave starts only after all enemies from the previous wave are dead
- An animated **WAVE N** banner fades in and out at each wave transition
- Beyond wave 5 the game scales infinitely — more enemies spawn faster with each wave

---

## Leaderboard

Scores are persisted in `localStorage` under the key `tds_leaderboard_v1` as a JSON array of up to 10 entries, sorted by score descending.

| Column | Description |
|--------|-------------|
| Rank   | Medal icon: gold (#1), silver (#2), bronze (#3), grey (#4–#10) |
| Name   | Player name from the settings panel (truncated to 7 chars) |
| Score  | Final score at time of death |
| WV     | Wave number reached |
| D      | Difficulty initial: E / N / H |

- On game over, your row is **highlighted in gold** and a rank badge pulses ("NEW RECORD!" for rank #1)
- The leaderboard is visible on both the **menu** (HALL OF FAME, top 7) and **game-over** (LEADERBOARD, top 6) screens
- Clear via DevTools → Application → Local Storage → delete `tds_leaderboard_v1`

---

## Difficulty Modes

| Setting | Enemy Speed | Enemy Damage | Player HP | Invincibility |
|---------|-------------|--------------|-----------|---------------|
| **Easy** | ×0.72 | 10 per hit | 150 | 1100 ms |
| **Normal** | ×1.0 | 15 per hit | 100 | 800 ms |
| **Hard** | ×1.45 | 22 per hit | 75 | 500 ms |

The health bar colour changes based on current HP:
- **Green** — above 60 %
- **Orange** — between 30 % and 60 %
- **Red** — below 30 %

---

## Settings Panel

The panel is visible on the menu screen and hidden during gameplay. All settings take effect immediately.

| Setting | Options | Default |
|---------|---------|---------|
| **Player Name** | Free text, max 16 characters | `Player` |
| **Background** | Navy Grid, Starfield, Dungeon, Neon City | Navy Grid |
| **Difficulty** | Easy, Normal, Hard | Normal |
| **Volume** | 0 – 100 % slider | 70 % |
| **Sound** | Toggle on/off | On |
| **Player Colour** | Blue, Green, Yellow, Pink, Purple | Blue (`#44aaff`) |

---

## Background Themes

| Theme | Description |
|-------|-------------|
| **Navy Grid** | Dark navy background with subtle white grid lines |
| **Starfield** | Deep space with 180 randomised stars (varying size, brightness, colour) |
| **Dungeon** | Stone brick tiles with randomised shading and mortar lines |
| **Neon City** | Near-black background with glowing cyan horizontal and magenta vertical grid lines |

---

## Audio

All audio is generated at runtime using the **Web Audio API** — no external sound files are loaded.

| Event | Sound Description |
|-------|------------------|
| **Shoot** | Sawtooth oscillator with fast frequency drop (700 → 90 Hz) |
| **Enemy hit** | Short square-wave pulse (320 → 160 Hz) |
| **Grunt / Fast death** | Filtered noise burst + sine pitch drop |
| **Tank death** | Longer, deeper noise burst + lower sine pitch drop |
| **Player hit** | Sine tone (140 → 35 Hz) layered with noise burst |
| **Wave start** | Rising four-note arpeggio (440, 554, 659, 880 Hz) |
| **Game over** | Descending four-note sawtooth motif (392, 330, 262, 196 Hz) |

Volume is controlled by a single master `GainNode`; the sound toggle simply disables all playback calls.

---

## Tech Stack

| Technology | Purpose |
|------------|---------|
| HTML5 Canvas 2D | Rendering (800×600) |
| Web Audio API | Procedural sound effects |
| `requestAnimationFrame` | Game loop |
| Vanilla JS (ES6+) | All game logic |
| CSS Grid | Settings panel layout |

No build tools, no npm, no external libraries.

---

## Project Structure

```
top-down-shooter/
├── shooter.html    # The entire game — HTML, CSS, and JavaScript in one file
└── .gitignore
```

All game systems live inside `shooter.html`:

```
shooter.html
├── <style>          — Canvas layout + settings panel CSS
├── <canvas>         — Game viewport (800×600)
├── <div#settings>   — Settings panel (name, bg, difficulty, volume, colour)
└── <script>
    ├── SETUP        — Canvas context, constants
    ├── SETTINGS     — Settings object + UI event wiring
    ├── AUDIO        — Web Audio API procedural SFX
    ├── LEADERBOARD  — localStorage persistence, rankColor/rankLabel, drawLeaderboardPanel
    ├── SCORE POPUPS — floating kill-reward text system
    ├── GAME STATE   — State machine (MENU / PLAYING / GAME_OVER), killCount, playerRank
    ├── INPUT        — Keyboard and mouse handlers
    ├── BACKGROUNDS  — Four theme renderers
    ├── WAVE DEFS    — Wave progression table + infinite scaler
    ├── ENTITY DEFS  — Enemy base stats
    ├── GAME INIT    — startGame (resets killCount/playerRank/scorePopups) / startNextWave (sets waveEnemiesTotal) / endGame (saves to leaderboard)
    ├── PLAYER       — Update, fire, draw, colour helpers
    ├── ENEMIES      — Spawn, update, draw (3 shapes + HP bar)
    ├── BULLETS      — Update, draw (glow via shadowBlur)
    ├── PARTICLES    — Spawn, update, draw
    ├── COLLISION    — Circle-circle detection, damage, death, kill counting, score popups
    ├── WAVE MANAGER — Spawn timer, clearing logic
    ├── HUD          — HP bar, score, kill counter, wave counter + progress bar, banner with glow
    ├── SCREENS      — Menu (two-panel: controls + HALL OF FAME) and Game Over (stats grid + rank badge + leaderboard)
    └── GAME LOOP    — requestAnimationFrame loop with deltaTime cap
```

---

## Development Workflow

Every change to the game follows this commit convention:

```bash
# After making changes to shooter.html:
git add shooter.html
git commit -m "<type>: <short description>"
git push
```

| Prefix | Use for |
|--------|---------|
| `feat:` | New feature or gameplay addition |
| `fix:` | Bug fix |
| `style:` | Visual / UI change with no logic change |
| `refactor:` | Code restructure, no behaviour change |
| `chore:` | Housekeeping (gitignore, config, docs) |

---

## License

MIT — do whatever you like with it.
