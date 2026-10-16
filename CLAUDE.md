# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project Overview

**Sky Fighter** is a Raiden-style vertical shoot-'em-up game implemented as a single self-contained HTML file. The game features:
- A player aircraft with weapon upgrades and smart bombs
- Multiple enemy types and boss encounters
- Canvas-based 2D rendering with procedural audio synthesis
- Responsive design supporting desktop, tablet, and mobile (touch controls)
- Difficulty levels: Easy, Normal, Hard
- Score tracking with localStorage persistence
- Accessibility features: reduced motion support, screen reader announcements

There are no external dependencies, build tools, package manager, or tests. The entire game is in `index.html` (roughly 1400 lines).

## Architecture & Game Structure

The JavaScript code is organized as a single IIFE (immediately invoked function expression) with these logical sections:

### 1. **Constants & Configuration** (lines ~485–515)
- `DIFFICULTIES`: Settings per difficulty level (spawn rates, enemy speed, boss HP multiplier, lives/bombs)
- `KILLS_PER_STAGE`: How many enemy kills trigger progression to the next stage
- `THEMES`: 6 sky/cloud color schemes, each with a name
- `ENEMY_TYPES`: Properties for grunt, interceptor, tank (width, height, HP, color, score value, stage unlock)
- `BOSSES`: 6 boss configurations with color schemes and HP multipliers

### 2. **Audio System** (lines ~542–676)
- Web Audio API synthesis engine (`ensureAudio()`, `beep()`, `noiseBurst()`)
- Sound effect library (`sfx` object): shoot, laser, explosions per enemy type, hit, bomb, powerup, boss telegraph, etc.
- All sounds are generated procedurally—no audio files
- Spatial audio: pan based on screen position (left/right)
- Muted state and volume control via master gain

### 3. **Input Handling** (lines ~768–861)
- Keyboard: Arrow keys, WASD for movement; Space for fire; X or Shift for bomb; P/Esc to pause
- Mouse: Movement steers the plane; click fires
- Touch: Left side drag for movement (joystick); FIRE/BOMB buttons on right
- Input priority: keyboard/joystick > mouse (mouse won't interfere when keys are active)

### 4. **Game State & Main Loop** (lines ~863–onwards)
- Core entities: `player`, `bullets`, `enemies`, `enemyBullets`, `particles`, `powerUps`, `boss`
- Game loop runs at ~60 FPS with adaptive quality scaling
- Each frame: update player, enemies, bullets, collisions, rendering
- **Stage progression**: Stages advance every `KILLS_PER_STAGE` kills; boss appears at stage boundaries
- **Combo/multiplier**: Kills within a short window (chainTimer) increase multiplier

### 5. **Rendering** (canvas-based, 2D context)
- Base resolution: 480×720 (portrait), scales to fit screen
- Starfield background with parallax clouds
- Entity drawing: sprites drawn as simple colored rectangles with gradient shading
- HUD overlays: score, best, multiplier, weapon, hearts (lives), bombs
- Boss health bar appears when boss is active
- Visual effects: screen shake, flash on hit, vignette for critical health

## Key Game Entities

### Player
- Position (x, y), width/height 40×40
- Speed: 260 pixels/second
- Weapons: `vulcan` (rapid fire) or `laser` (slower, more damage)
- Weapon level upgrades (1–3)
- Cooldown timer between shots
- Invulnerability timer after hit
- Lives system (3, 4, or 6 depending on difficulty)

### Enemies
Three types, scaled in difficulty:
- **Grunt**: Fast spawning, 1 HP, score 10
- **Interceptor**: Faster, weaving pattern, 1 HP, score 15
- **Tank**: Slow, tanky, 4 HP, score 35
- Spawn rate decreases (easier to spawn) as stage increases
- Enemies shoot back at player
- Death triggers explosion particle effect and sound

### Bosses
Six unique boss patterns; one appears at stage 2, 5, 8, 11, 14, 17+. Each has:
- Color scheme (hull, wing, core)
- Custom shape and attack pattern
- HP multiplier (0.8× to 1.7× base)
- Tells (warning beeps) before major attacks
- Multi-phase behavior (spiral shots, sweeps, sprays)

### Weapons & Power-ups
- **Vulcan**: Rapid small bullets, fast cooldown
- **Laser**: Slower, stronger projectiles
- Weapon drops appear from defeated enemies
- Weapon levels 1–3 increase fire rate or damage

### Bombs
- Limited count (2–3 per life depending on difficulty)
- Smart bomb clears all on-screen enemies and bullets in a radius
- Visual flash and unique explosion sound
- Cannot be used during certain boss patterns (design choice to force engagement)

## How to Add New Features

### Adding a New Enemy Type
1. Add entry to `ENEMY_TYPES` object with properties: `w`, `h`, `hp`, `speedMul`, `color`, `dark`, `score`, `minStage`
2. Add explosion sound to `EXPLODE_SFX` mapping if custom needed
3. Enemy spawn logic checks `minStage` to avoid spawning before intended
4. Enemy behavior (movement pattern, shooting) is in the main update loop—search for `enemies.forEach`

### Adding a New Boss
1. Add entry to `BOSSES` array with: `name`, color scheme (`hullTop`, `hullBottom`, `wing`, `core`), `hpMul`
2. Boss AI/attack patterns are hard-coded per boss in the main loop—look for `boss.type` checks
3. Update cycle counter; currently 6 bosses repeat (boss at stage N is `BOSSES[N % BOSSES.length]`)

### Adding a New Theme
1. Add object to `THEMES` array: `sky` (3-color gradient), `cloud`, `sun` (RGB strings), `name`
2. Theme is randomly selected each game; affects background color only (cosmetic)

### Adding a New Weapon
1. Add weapon name to weapon pool logic
2. Create bullet type with unique fire rate and damage
3. Update `weaponLabel()` to return display name
4. Add fire sound to `sfx` object if desired
5. Modify player fire logic to handle new weapon case

### Game Balance Adjustments
- Difficulty: tweak `DIFFICULTIES` object for spawn rates, enemy speed, boss multipliers
- Stage progression: change `KILLS_PER_STAGE`
- Weapon stats: cooldown and damage are baked into the update loop—search for `player.cooldown`
- Health/bombs: change `d.lives` and `d.bombs` in `resetGame()`

## Local Development & Testing

There is no build step. To test changes:
1. Edit `index.html` directly
2. Open in a modern web browser (Chrome, Firefox, Safari, Edge)
3. Use browser DevTools (F12) to debug:
   - Console for errors and `console.log()` statements
   - Network tab to verify no external requests
   - Responsive Design Mode (Ctrl+Shift+M) to test mobile
4. Clear localStorage if needed to reset high score: `localStorage.clear()` in console

### Testing Checklist for Changes
- **Rendering**: Visuals appear correctly, no overlapping artifacts
- **Gameplay**: Collision detection works, enemy/boss behavior is correct
- **Audio**: Sound effects play (if not muted), no distortion
- **Responsiveness**: Layout adapts on mobile, touch controls work
- **Accessibility**: Screen reader announcements clear, reduced motion respected
- **Storage**: Best score persists after reload, difficulty selection remembered

## Important Implementation Details

### Performance Optimization
- Adaptive quality scaling: frame time is monitored; if averaging >25ms, visual quality is reduced (fewer particle effects, etc.)
- Canvas is sized at internal resolution (480×720) and scaled via CSS—this keeps rendering fast and pixel-perfect
- No external image assets; all graphics are procedural (drawn via canvas 2D context)

### Randomization & Procedural Generation
- Enemy spawns use random type selection within difficulty constraints
- Boss appearance order is randomized (one per stage)
- All sound effects are synthesized from oscillators and noise—no pre-recorded audio

### State Persistence
- `localStorage` keys: `skyfighter.bestScore`, `skyfighter.muted`, `skyfighter.difficulty`, `skyfighter.reducedMotion`
- Persisted on every change; restored on page load

### Accessibility
- Screen reader support via `#srLive` element with `aria-live="polite"`
- Reduced motion toggle respects `prefers-reduced-motion` media query
- All controls have keyboard equivalents
- Color contrast meets WCAG standards

## Common Editing Patterns

### To adjust game difficulty parameters:
```javascript
const DIFFICULTIES = {
  easy:   { spawnBase: 75, ... },  // Lower spawnBase = harder
  normal: { spawnBase: 58, ... },
  hard:   { spawnBase: 40, ... }
};
```

### To change a constant (e.g., player speed):
Search for `player.speed = 260` and modify

### To add debug output:
Use `console.log()` or add text to `announce()` for screen reader feedback

### To trigger a specific game state:
Modify `running`, `paused`, `gameOver` flags directly in console

## No External Tools or CI

This is a standalone game—there are no:
- Tests to run
- Linters to configure
- Build steps or transpilation
- External API calls (all local)
- Package dependencies

All changes are made directly to `index.html` and tested in the browser.
