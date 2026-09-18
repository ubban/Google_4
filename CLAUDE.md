# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project Overview

**Sky Fighter** is a Raiden-style vertical shoot-'em-up game. It's a single-file, self-contained browser game with no external dependencies. The entire game (HTML, CSS, JavaScript, and game logic) is in `index.html`.

## How to Run

Simply open `index.html` in a web browser. The game loads immediately and requires no build process, bundler, or server.

## Code Architecture

The game is structured as a single IIFE (Immediately Invoked Function Expression) containing all game logic:

### High-Level Flow
1. **Initialization**: Set up DOM elements, canvas, event listeners, and game state
2. **Game Loop**: `requestAnimationFrame(loop)` → `update(dt)` → `draw()`
3. **Input Handling**: Keyboard (arrows/WASD, Space, X, P), mouse (crosshair), touch (virtual joystick)
4. **Game States**: Menu overlay → Running → Paused/Game Over overlay

### Key Systems

#### Canvas & Rendering (`draw()`)
- 480×720 canvas (dynamically scaled to fit screen)
- Background: animated gradient sky with clouds based on theme
- Rendered each frame: player plane, enemies, boss, bullets, particles, power-ups
- Quality scaling: adaptive rendering quality (0–2) based on frame times to maintain ~60 FPS

#### Game State Management
```javascript
let running, paused, gameOver;
let stage, kills, stageKills, chain, multiplier, score;
let player, bullets, enemies, boss, enemyBullets, powerUps, particles;
```
- Stages auto-advance after defeating boss (stage counter cycles theme and boss difficulty)
- Chains: kill streaks without gaps > 2.2s; multiplier = 1 + floor(chain / 5), max 10x
- Each stage requires 18 kills to trigger boss spawn

#### Player (`player` object)
- Movement: keyboard/joystick input OR mouse tracking (mouse doesn't fight keyboard)
- Weapons: **Vulcan** (spread shots) or **Laser** (piercing beams), each upgradable 1–3
- Fire rate: Vulcan 0.15s, Laser 0.1s between shots
- Invulnerability: 1.5s after taking damage (drawn with blink effect)
- Bomb: clears all enemy bullets, damages/kills enemies + boss (limited stock)

#### Enemies (`spawnEnemy()` & update)
- **Grunt**: 1 HP, 10 points, red (spawns stage 1+)
- **Interceptor**: 1 HP, 15 points, orange, 1.7× speed, weaves (spawns stage 1+)
- **Tank**: 4 HP, 35 points, purple, slow, shows health bar (spawns stage 2+)
- Behavior: move down screen, random shoot pattern (aimed or spread), despawn if off-screen
- Weaving interceptors oscillate horizontally using `sin(frame * 0.05 + seed)`

#### Boss System (`boss` object & `updateBoss()`)
- 6 unique bosses (VANGUARD → BEHEMOTH → SPECTER → DREADNOUGHT → WRAITH → TITAN), cycle per stage
- Each boss has distinct visual design, health multiplier (0.8–1.7×), and 3 attack patterns
- Entering phase: 120px down, then attack loop (telegraph 22 frames → fire pattern → 85-frame cooldown)
- Attack patterns use helper functions (`fireRadial`, `fireAimedSpread`, `fireCross`, etc.) for bullet spray
- Boss health bar (top center) updates in real-time; kill triggers stage clear and next stage

#### Audio (Web Audio API synthesis)
- No external audio files; all sounds generated procedurally with oscillators, filters, and noise
- `ensureAudio()`: lazy-init AudioContext (suspended until user interacts)
- `beep(freq, dur, type, vol, opts)`: oscillator with optional sweep, filter, pan
- `noiseBurst(dur, vol, opts)`: filtered white noise
- Mute toggle persists in `localStorage`

#### Input Handling
- **Keyboard**: Arrow keys / WASD move, Space fires, X/Shift bombs, P/Esc pauses
- **Mouse**: Move to steer (lerp toward pointer), click to fire
- **Touch**: Left zone drag = joystick, FIRE/BOMB buttons on right, auto-detected if `ontouchstart` present
- Priority: keyboard/joystick > mouse (mouse never interrupts active input)

#### Power-ups
- Spawn 6% life, 8% bomb, 20% weapon (rest: nothing)
- Types: `life` (restore 1 up to max), `bomb` (+1 bomb), `weapon` (upgrade or switch weapon)
- Weapon upgrade: levels 1→3 increase spread/pierce, level 3 allows weapon switch (VULCAN ↔ LASER, back to level 1)

#### Difficulty Scaling
- **Easy**: slower spawns, weak enemies, 4 lives, 3 bombs, 0.8× boss HP
- **Normal**: balanced (default)
- **Hard**: fast spawns, strong enemies, 3 lives, 2 bombs, 1.3× boss HP
- Progressive difficulty: spawn rate decreases over time within a stage

#### Accessibility & UX
- **Screen reader support**: `#srLive` announces game events (kills, stage, boss defeat)
- **Reduced motion**: toggle via button, affects all animations (flashes, shakes, pulsing) — respects OS prefers-reduced-motion
- **Responsive**: canvas scales up to 1.6× with clamp sizing to fit any screen
- **Local storage**: saves high score, mute state, difficulty, reduced-motion preference

## Constants & Configuration

### Game Constants
```javascript
BASE_W = 480, BASE_H = 720;  // canvas size
KILLS_PER_STAGE = 18;         // kills to spawn boss
```

### DIFFICULTIES
Maps difficulty name to spawn rate, enemy speed, shoot chance, lives, bombs, boss HP multiplier.

### THEMES (6 stages)
Each stage cycles a color theme (sky gradient, cloud color, sun color, visual name).

### ENEMY_TYPES
Define: width, height, HP, speed multiplier, color, score, minimum stage, weave flag.

### BOSSES (6 designs)
Define: name, hull colors (top/bottom), wing color, core color, HP multiplier. Each has 3 attack patterns in `BOSS_PATTERNS`.

### BOSS_PATTERNS
Each boss has a 3-element array of functions firing different bullet patterns. Patterns use helper functions for radial, aimed spreads, columns, spirals, crosses, homing, etc.

## Key Functions

- `update(dt)` – game logic per frame (input, movement, collision, spawning)
- `draw()` – render all game objects
- `resetGame()` – initialize new game
- `spawnEnemy()` – create enemy on top of screen
- `spawnBoss()` – create boss, show warning banner
- `fireWeapon()` – spawn player bullets based on weapon/level
- `useBomb()` – clear enemies/bullets, damage boss
- `addScore(base)` – increase score with multiplier, update chain
- `rectsOverlap(a, b)` – AABB collision check
- `screenShake(mag, dur)` – add camera jitter effect
- `updateBoss(dt)` – boss movement and attack logic

## Common Changes

### Adjusting Game Balance
- **Difficulty**: Modify `DIFFICULTIES` object
- **Enemy types**: Add/modify in `ENEMY_TYPES`, update `spawnEnemy()` logic
- **Bosses**: Change `BOSSES` designs or `BOSS_PATTERNS` attack sequences
- **Player stats**: Modify `player` initialization in `resetGame()` (speed, cooldown, weapon levels)
- **Spawn rate**: Tweak `spawnBase`, `spawnMin` in difficulties

### Adding New Features
- **New weapon**: Add weapon type, add firing logic to `fireWeapon()`, define level progression in `applyPowerUp()`
- **New enemy type**: Add to `ENEMY_TYPES`, add spawn condition in `spawnEnemy()`, add update/rendering logic
- **New theme/boss**: Add to `THEMES` and `BOSSES`, define attack patterns in `BOSS_PATTERNS`
- **Visual effects**: Particles system is ready (`particles` array); spawn with `spawnExplosion()`, render in `draw()`

### Audio/UI Tweaks
- **SFX**: Add new beeps/noise bursts to `sfx` object, tweak frequency/duration/filter
- **UI text**: Modify overlay strings, HUD labels
- **Colors**: Change `#rgba` hex values in CSS (`:root`, theme sky/cloud/sun, plane colors, UI)

## Performance Notes

- **Adaptive quality**: Frame times tracked; if avg > 25ms, reduce particle count and shadow blur; if < 18ms, increase
- **Collision**: O(n²) enemy-bullet checks; acceptable for small enemy counts (20–30 max on-screen)
- **Canvas scaling**: Uses CSS `max-width/max-height` + `clamp()` for mobile-friendly responsive sizing
- **Touch handling**: Joystick clamped to 50px max distance for consistent mobile control feel

## Testing in Browser

1. Open `index.html` in Chrome/Firefox/Safari
2. Menu appears; click difficulty → Start Game
3. Keyboard: arrows move, space fires, X bombs, P pauses
4. Mouse: move to steer, click to fire
5. Touch: drag left zone to move, tap FIRE/BOMB buttons
6. Press mute/effects/pause buttons in top-right corner
7. Verify each difficulty level
8. Play through multiple stages to test boss patterns and progression

## File Structure

```
Google_4/
├── index.html      (entire game: HTML, CSS, JavaScript)
├── README.md       (project header)
└── CLAUDE.md       (this file)
```
