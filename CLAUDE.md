# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project Overview

**Sky Fighter** is a single-file HTML5 vertical shoot-'em-up game inspired by arcade classics like Raiden. The entire game is contained in `index.html` with embedded CSS and vanilla JavaScript — no build process or external dependencies required.

## How to Run the Game

**Local testing:**
- Open `index.html` directly in any modern web browser (Chrome, Firefox, Safari, Edge)
- The game is fully playable on desktop and touch devices (mobile/tablet)

**Development server:**
- No build tool is required, but you may use any simple HTTP server:
  ```bash
  python -m http.server 8000  # Python 3
  # OR
  npx http-server            # Node.js
  ```
- Then navigate to `http://localhost:8000`

The game persists high scores and user preferences (mute state, difficulty, reduced-motion setting) to localStorage.

## Game Architecture

### Core Systems

The game is structured around several interconnected systems, all in a single IIFE closure to avoid global scope pollution:

1. **Game State** (lines ~863–910)
   - `player`: position, weapon, level, invulnerability frames
   - `score`, `lives`, `stage`, `chain`, `multiplier`: progression tracking
   - `enemies`, `enemyBullets`, `bullets`, `particles`: entity arrays
   - `boss`: current boss (if active)

2. **Input Handling** (lines ~768–862)
   - **Keyboard**: Arrow keys / WASD for movement, Space for fire, X/Shift for bombs, P/Esc to pause
   - **Mouse**: Free steering toward pointer; click to fire
   - **Touch**: Joystick in lower-left (movement), buttons in lower-right (FIRE, BOMB)
   - Input state merges gracefully: keyboard/joystick take priority over mouse when active

3. **Rendering Pipeline** (lines ~1520–1638)
   - Canvas base is 480×720px, scaled responsively to fit window (max 1.6× scale)
   - **Background**: animated gradient sky with parallax clouds
   - **Entities**: player, enemies, boss, bullets, powerups, particles
   - **UI overlay**: HUD (score, best, multiplier, weapon, lives, bombs), stage banners, boss health bar
   - Screen shake and flash effects for feedback (respect reduced-motion preference)

4. **Gameplay Loop** (lines ~1640–1666)
   - `requestAnimationFrame` based, adaptive performance tracking
   - `update()` each frame: movement, collision, entity lifecycle
   - `draw()` each frame: render all visual elements
   - Frame-independent delta-time for smooth pacing across refresh rates

### Difficulty & Progression

**Difficulties** (lines ~485–489):
- Adjust spawn rate, enemy speed/behavior, player lives, bomb count, boss HP multiplier
- Stored in localStorage; difficulty buttons on the start screen

**Stage System**:
- Player must kill 18 enemies (`KILLS_PER_STAGE`) to advance
- Boss spawns after stage threshold met (line ~1204)
- Each stage cycles through 6 environment themes and 6 boss designs
- Stage boss HP scales: `70 + stage * 35` × difficulty multiplier × boss design multiplier

### Entity Behavior

**Enemies** (lines ~502–506, spawning ~1022–1041, logic ~1210–1227):
- Three types: grunt (fast, weak), interceptor (very fast, weaving), tank (slow, durable)
- Enemies spawn at random x-positions and descend; weaving interceptors oscillate horizontally
- Each enemy can shoot on a cooldown; patterns are either "spread" (5-shot fan) or "aimed" (single homing shot at player)
- Collision with player or bullets damages them; destroyed enemies drop weapon/life/bomb powerups

**Bosses** (lines ~508–515, design logic ~970–1020, attack patterns ~1372–1409):
- Six boss designs with unique visual themes and attack patterns
- Each boss has 3 distinct attack patterns that cycle
- Boss enters from top, bounces side-to-side, telegraphs attacks with visual cue
- Attack patterns combine:
  - Radial bursts (all directions at once)
  - Aimed spreads (fan toward player)
  - Homing projectiles (track player position with error)
  - Teleportation (swap position mid-fight)
  - Dense walls and crosses

**Bullets**:
- Player bullets: vulcan (spread shot, upgrades add width) or laser (piercing, upgrades add count)
- Enemy bullets: radial movement or aimed/homing behavior
- Collision detection uses AABB (axis-aligned bounding boxes) via `rectsOverlap()`

### Audio System

**WebAudio Synthesis** (lines ~542–676):
- Pure synthesized sound effects; no external audio files
- `beep()`: tone generation with frequency sweeps, filters, panning
- `noiseBurst()`: filtered white noise (explosions, bombs)
- Dynamic gain envelope (attack/release) for natural sound shape
- Stereo panning based on entity position (left/right channel intensity)
- Mute state persists to localStorage

### UI & Accessibility

- **HUD**: Displays score, best, kill multiplier, current weapon, health hearts, bomb count
- **Overlays**: Title screen, pause menu, game-over screen with stats
- **Accessibility**:
  - Screen-reader live region (`#srLive`) announces state changes
  - Keyboard controls fully functional
  - Touch-friendly button sizing and hit targets
  - Reduced-motion preference respected (CSS animations disabled, screen shake dampened)
  - High contrast text shadows for readability

### Storage Keys

Game state persisted to localStorage:
- `skyfighter.bestScore`: best score achieved
- `skyfighter.muted`: audio mute state
- `skyfighter.difficulty`: last selected difficulty
- `skyfighter.reducedMotion`: accessibility preference

## Key Modification Points

**Adding a new enemy type:**
1. Define in `ENEMY_TYPES` (line ~502–506)
2. Add spawn logic to `spawnEnemy()` (line ~1026–1030)
3. Add explosion SFX in `EXPLODE_SFX` map (line ~678)
4. Draw routine in `enemies.forEach()` in `draw()` (line ~1603–1613)

**Adding a new boss:**
1. Define design in `BOSSES` (line ~508–515)
2. Add attack pattern array to `BOSS_PATTERNS` (line ~1372–1409)
3. Update `drawBossShip()` if needed for custom visuals (line ~970–1020)

**Tweaking balance:**
- `DIFFICULTIES` object for spawn rates, speeds, HP scaling
- `KILLS_PER_STAGE` for pacing
- Weapon fire delays in `update()` (line ~1177)
- Collision damage values (bullets vs boss ~1259, etc.)
- Score multipliers in `addScore()` (line ~1087–1094)

**New visual effects:**
- Add particle types in `spawnExplosion()` (line ~1073–1085) or create new function
- Screen shake: `screenShake(magnitude, duration)` (line ~727–730)
- Flash overlays: `flashHit()` / `flashBomb()` (line ~698–709)

## Performance & Quality

**Adaptive Quality System** (lines ~474–483):
- Game monitors frame times and throttles particle count and visual effects if FPS drops
- Quality levels: 0 (full), 1 (medium ~60% effects), 2 (low ~35% effects)
- Glow/shadow effects disabled at quality level 2

**Browser Compatibility:**
- Requires ES6+ (arrow functions, const/let, modern Canvas API)
- AudioContext API for sound (graceful fallback if unavailable)
- Modern touch and pointer event support

## Common Tasks

**Test a balance change:**
- Modify a value in `DIFFICULTIES` or entity stats
- Reload the browser; no build step needed
- Check localStorage is working: `localStorage.clear()` to reset scores

**Debug collisions:**
- Add visual debugging to `rectsOverlap()` or draw bounding boxes in `draw()`
- Use browser DevTools console: `boss` / `player` / `enemies` to inspect state

**Tweak visual timing:**
- Most animations use frame counts: `frame * speedFactor` increments each update
- Animation keyframes in CSS: search `@keyframes`
- Canvas context transforms for effects: search `ctx.save()` / `ctx.restore()`

**Add a keyboard shortcut:**
- Handle in `keydown` listener (line ~769–775)
- Add corresponding `update()` logic
- Test on desktop and mobile (no keyboard on touch devices)
