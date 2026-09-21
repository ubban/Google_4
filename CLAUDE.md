# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project Overview

**Sky Fighter** is a Raiden-style vertical shoot-'em-up game built entirely in a single HTML file using HTML5 Canvas for rendering and Web Audio API for procedurally generated sound effects. No external dependencies or assets — pure vanilla JavaScript.

## Architecture Overview

The game runs in an infinite `requestAnimationFrame` loop with three main phases:
1. **Update** (game state, physics, collisions) — only if running and not paused
2. **Draw** (canvas rendering) — happens every frame
3. **Performance tracking** — adaptive quality scaling

### Core Game Loop
- `loop()` (line 1640): Main frame callback. Calls `update(dt)` then `draw()`.
- `update(dt)` (line 1148): Handles player movement, enemy AI, collision detection, score, and state transitions.
- `draw()` (line 1520): Renders all game entities (sky gradient, clouds, player, enemies, bullets, boss, particles).

### Game State Variables
- `player`: { x, y, w, h, speed, weapon, level, cooldown, invuln }
- `bullets`, `enemies`, `enemyBullets`, `particles`, `powerUps`: Arrays of entities
- `score`, `lives`, `stage`, `kills`, `chain`, `multiplier`: Game stats
- `boss`: Current boss or null
- `running`, `paused`, `gameOver`: State flags

## Key Systems

### 1. Entity Rendering (Canvas)
- **Player plane**: `drawPlane()` (line 913) — draws using triangle geometry and filled paths
- **Enemy planes**: Same `drawPlane()` function with different colors
- **Boss ship**: `drawBossShip()` (line 970) — more complex with rotating thrusters
- **Bullets & particles**: Rectangles/circles with gradients and glow effects
- **Clouds & sky**: `drawClouds()` (line 1507) uses ellipses with animated positioning
- **Performance optimization**: Variable `quality` (0=full, 1=medium, 2=low) reduces shadows and glows when needed

### 2. Audio (Web Audio Synthesis)
- No external audio files — all sounds generated procedurally
- `beep()` (line 566): Oscillator-based sine/square/sawtooth tones with optional frequency sweep, filter, and panning
- `noiseBurst()` (line 597): White noise with envelope and filter
- `sfx` object (line 624): Named sound effects (shoot, laser, explode, bomb, powerup, etc.)
- Master gain + dynamics compressor for audio limiting
- Audio context initialized on first user input

### 3. Input Handling
- **Keyboard**: Arrow keys / WASD for movement, Space to fire, X/Shift/P for bomb/pause
- **Mouse**: Hover over canvas to steer, click to fire (mouse target glides smoothly)
- **Touch**: Joystick in bottom-left, Fire/Bomb buttons in bottom-right
- Input priority: keyboard/joystick > mouse (never conflict)

### 4. Enemy Types and Spawning
Defined in `ENEMY_TYPES` (line 502):
- **Grunt**: Red, basic enemy, 1 HP
- **Interceptor**: Orange, faster, weaves left-right
- **Tank**: Purple, slow, 4 HP, appears stage 2+

Enemies spawn with random shooting patterns:
- **Spread**: Fires 3 bullets at angles
- **Aimed**: Tracks player position
- **Weaving**: Interceptors snake side-to-side

### 5. Boss System
- 6 boss designs: VANGUARD, BEHEMOTH, SPECTER, DREADNOUGHT, WRAITH, TITAN
- Each boss has unique color scheme and HP multiplier
- Boss patterns defined in `BOSS_PATTERNS` (line 1372): Each boss has 3 attack patterns
- Pattern functions: `fireRadial()`, `fireAimedSpread()`, `fireColumns()`, `fireSpiralBurst()`, `fireCross()`, `fireHomingSeeded()`, etc.
- Bosses telegraph attacks (white flash) before firing
- Boss appears after defeating 18 enemies (KILLS_PER_STAGE)

### 6. Difficulty & Themes
- **Difficulties** (line 485): Easy, Normal, Hard with different spawn rates, enemy speed, shoot chance, lives, bombs, boss HP
- **Themes** (line 493): 6 visual themes with different sky gradients, cloud colors, sun colors. Themes cycle per stage.
- Settings persist to localStorage (best score, difficulty, mute, reduced motion)

### 7. Power-Up System
- Life (+1 health, 6% spawn chance)
- Bomb (+1 bomb, 8% spawn chance)
- Weapon (upgrade or switch, 20% spawn chance)
- Applied via `applyPowerUp()` (line 1304)

### 8. Scoring & Multiplier
- Kill chain increments `chain` counter, restarts timer
- Multiplier = 1 + floor(chain / 5), max 10x
- Chain breaks if 2.2 seconds pass without a kill
- Shows combo pop-ups at 10, 20, 30... kills

## Important Constants & Arrays

| Constant | Range | Purpose |
|----------|-------|---------|
| `BASE_W`, `BASE_H` | 480×720 | Canvas resolution (scales to fit window) |
| `KILLS_PER_STAGE` | 18 | Enemies to defeat before boss spawns |
| `frameTimes` | array | 45-frame rolling window for performance tracking |

## Common Modifications

### Add a New Enemy Type
1. Add entry to `ENEMY_TYPES` (line 502): `{ w, h, hp, speedMul, color, dark, score, minStage, weave? }`
2. Update `EXPLODE_SFX` if unique death sound (line 678)
3. Add sound effect if needed

### Add a New Boss
1. Add to `BOSSES` array (line 508) with hull colors and core color
2. Add 3 attack patterns to `BOSS_PATTERNS` (line 1372) — reuse existing fire functions
3. Boss number automatically cycles; stage controls which appears

### Adjust Difficulty Balance
Edit `DIFFICULTIES` object (line 485):
- `spawnBase`, `spawnMin`: Frame intervals for enemy spawning
- `enemySpeed`, `enemyShootChance`: Combat tuning
- `lives`, `bombs`: Starting resources
- `bossHpMul`: HP multiplier (1.0 = base, 1.3 = 30% harder)

### Change Visual Theme
Edit `THEMES` array (line 493): Set `sky` gradient stops, `cloud` and `sun` RGB values, `name`

## Rendering & Performance Notes

- **Screen shake** uses small random translation; reduced motion disables it
- **Vignette effect** pulses red when health is critical (1 HP)
- **Flash overlay** on hit or bomb detonation
- **Canvas scaling**: Maintains 480×720 aspect, scales CSS to fit window (max 1.6x)
- **Quality scaling** (line 474–483): Monitors frame times and auto-reduces shadow/glow rendering if FPS drops below ~40

## UI & State Management

- **Overlay** shows title screen, pause screen, game-over with stats
- **HUD** displays score, best, multiplier, weapon level, hearts, bombs (top-center)
- **Boss bar** shows boss health (hidden when no boss active)
- **Stage banner** animates in at stage start and boss warning
- **Combo popup** shows at 10+ kill milestones
- **Reduced motion toggle** disables animations/flash/shake for accessibility
- **Mute button** controls audio (persisted)
- **Pause button** toggles overlay

## Input & Control Flow

- **Keyboard/joystick/mouse** independently update movement vector
- **Mouse target** smoothly interpolates player position (keyboard/joystick take priority)
- **Touch joystick** drawn with base circle + stick nub; 50px max deflection
- **Audio context** resumed on first user interaction (browser autoplay policy)

## Game Flow

1. Show overlay with title, difficulty choice, start button
2. User clicks Start → `resetGame()` initializes state
3. `loop()` runs continuously:
   - Spawn enemies on timer (rate scales down as waves progress)
   - On 18 kills → boss warning animation, then boss spawns
   - Boss defeated → stage++, theme changes, enemies clear
   - Player death → lives--, restart if 0 lives left
4. Game over → show final score, best score, option to retry

## Accessibility Features

- Reduced motion support (disables animations if system prefers)
- Screen reader live region (`srLive`) for announcements
- Semantic HTML (buttons, roles)
- Icon buttons (mute, effects, pause) with title text
- High contrast UI with text-shadow for readability

## Development Workflow

**No build step required.** Open `index.html` in a browser to play.

To iterate:
1. Edit the game logic or visuals in the `<script>` section (line 422–1666)
2. Reload browser
3. Use browser DevTools console to inspect game state: `player`, `score`, `enemies.length`, etc.

**Testing changes:**
- Adjust `DIFFICULTIES[difficulty]` values to test balance
- Modify `spawnEnemy()` to spawn specific enemy types
- Edit `THEMES` to see visual changes
- Change `BOSS_PATTERNS` to test new attack patterns

## Common Bugs & Fixes

| Issue | Cause | Fix |
|-------|-------|-----|
| Enemies not spawning | `spawnTimer` not decrementing | Check `update()` dt calculation |
| Boss won't spawn | Stage kills < 18 or boss != null | Verify `KILLS_PER_STAGE` and `killBoss()` cleanup |
| Audio cuts out | AudioContext suspended | Click canvas to resume (autoplay policy) |
| Canvas blurry on high-DPI | CSS scaling mismatch | Check `resize()` function for DPI handling |
| Performance dips | Shadow/glow rendering | Quality scaling should kick in automatically |
