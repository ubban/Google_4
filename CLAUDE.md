# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project Overview

**Sky Fighter** is a Raiden-style vertical shoot-'em-up game implemented as a single self-contained HTML5 file. The game features canvas-based rendering, procedural audio synthesis (no external assets), responsive controls for desktop and mobile, and a complete boss battle system.

## Running the Game

Since this is a single HTML file with no build process, simply open `index.html` in a web browser:

```bash
# Open directly (most browsers support file:// protocol)
open index.html

# Or serve locally to test network features
python3 -m http.server 8000
# Then visit http://localhost:8000/index.html
```

The game works on:
- Desktop browsers (Chrome, Firefox, Safari, Edge)
- Mobile browsers (iOS Safari, Chrome Mobile)
- Requires WebAudio API support (all modern browsers)

## Architecture Overview

The entire game is contained in a single `index.html` file (~1670 lines) with three main sections:

### 1. HTML Structure & UI
- Game canvas (`#gameCanvas` - 480×720px, responsive scaling)
- HUD elements: score, best score, multiplier, weapon, lives (hearts), bombs
- Boss health bar (shown during boss encounters)
- Overlay system for title screen, pause menu, and game over
- Accessibility features: screen reader support (`#srLive`)
- Touch control zones (left side for movement, right side for actions)

### 2. CSS Styling (Lines 10-359)
- Responsive design with clamp() for mobile/desktop scaling
- Canvas scaling: maintains 480×720 aspect ratio while fitting window
- Visual feedback: animations for combos, stage banners, effects
- Reduced motion support: respects `prefers-reduced-motion` preference
- Color scheme: dark blue theme with accent colors (yellow, green, red, cyan)

### 3. Game Logic (Lines 422-1666, wrapped in IIFE)

#### Core Game State
```javascript
player          // {x, y, w, h, speed, cooldown, weapon, level, invuln}
bullets         // Player projectiles
enemies         // Active enemies
enemyBullets    // Enemy projectiles  
powerUps        // Life/bomb/weapon pickups
boss            // Active boss (null when not fighting)
score, lives, stage, chain, multiplier
```

#### Difficulty System (Lines 485-489)
Three difficulty tiers affecting spawn rates, enemy AI, and boss HP:
- **Easy**: slower spawn, lower enemy speed, more lives/bombs, weaker bosses
- **Normal**: balanced (default)
- **Hard**: faster spawn, aggressive enemies, fewer resources, stronger bosses

#### Key Game Systems

**Weapon System (Lines 871-1146)**
- Vulcan: rapid-fire spread shot (upgradeable to 3 lanes)
- Laser: slower but piercing shots (2-3 lanes at max level)
- Power-ups cycle through levels 1-3, then switch weapons
- Firing rates: vulcan 0.15s, laser 0.1s

**Enemy Types (Lines 502-506)**
- Grunt: basic red enemy
- Interceptor: fast orange weaver
- Tank: slow purple tank with HP (appears stage 2+)
- Each type has different stats: speed multiplier, HP, score value

**Boss System (Lines 508-1439)**
- 6 unique boss designs with distinct visual styles and attack patterns
- Boss health calculated: `(70 + stage*35) × difficulty_multiplier × boss_hpMul`
- Pattern-based attacks: radial bursts, aimed spreads, telegraphed attacks
- Boss appears after 18 kills per stage
- Visual telegraph (25 frame warning) before each attack

**Input Handling (Lines 768-861)**
- Keyboard: Arrow keys/WASD for movement, Space for fire, X/Shift for bomb, P/Esc for pause
- Mouse: Movement follows cursor while hovering canvas
- Touch: Virtual joystick on left, FIRE/BOMB buttons on right
- Input priority: keyboard/joystick > mouse steering (prevents fighting)

**Collision & Scoring (Lines 1069-1094)**
- AABB collision detection: `rectsOverlap()`
- Chain multiplier: up to 10x for consistent kills (2.2s timeout)
- Boss defeat grants 500 points + combo multiplier
- Enemy destruction triggers particle effects and sound

**Audio System (Lines 542-676)**
- WebAudio API synthesis (no audio files)
- 12 distinct sound effects: shoot, laser, explosions (by type), hit, bomb, powerup, boss attacks, stage clear, UI, game over
- Master gain with compressor for consistent volume
- Spatial audio: bullet sounds pan based on screen position
- Mute toggle persists in localStorage

**Visual Effects**
- Screen shake: intensity and duration adjusted by difficulty/event
- Flash effects: white flash on hit, bright flash on bomb
- Vignette: red pulsing vignette when critical health (1 life)
- Particle explosions: color and count by source type
- Combo/stage banners with animated entrance/exit

**Performance Adaptive Quality (Lines 473-483)**
- Tracks frame timing over 45 frames
- Reduces particle count and shadow blur at lower quality levels
- 3 quality tiers: 0 (full), 1 (medium), 2 (low)
- Auto-downgrade if avg frame time > 1/40s, auto-upgrade if < 1/56s

**Visual Theming (Lines 493-500)**
- 6 stage themes with unique sky gradients and color palettes
- Theme cycles per stage (Daylight → Sunset → Night → Storm → Arctic → Volcanic)
- Cloud parallax layers at different speeds
- Sun glow with theme-appropriate colors

#### Lifecycle Functions

- `resetGame()`: Initialize all state for new game
- `update(dt)`: Game logic per frame (input, movement, collision, spawning)
- `draw()`: Canvas rendering with camera/shake effects
- `loop(now)`: RequestAnimationFrame callback managing dt and calling update/draw
- `endGame()`: Transition to game over screen, save best score
- `togglePause()`: Pause/resume with overlay

#### Data Persistence (Lines 517-527)
Uses localStorage for:
- `skyfighter.bestScore` - high score
- `skyfighter.muted` - audio mute state
- `skyfighter.difficulty` - selected difficulty
- `skyfighter.reducedMotion` - motion preference override

## Development Notes

### Adding New Features

**New Enemy Type:**
1. Add entry to `ENEMY_TYPES` object with stats
2. Update `spawnEnemy()` filter condition if stage-gated
3. Add explosion sound effect to `EXPLODE_SFX` mapping
4. Define sound in `sfx` object if needed

**New Boss Pattern:**
1. Create attack function using existing `fire*()` helpers (radial, aimed, columns, spiral, homing, etc.)
2. Add pattern array to `BOSS_PATTERNS` in correct index
3. Pattern function receives boss object `b` and can modify its position

**Weapon Modification:**
Update `fireWeapon()` function and adjust `fireDelay` constant if changing rate.

**UI Changes:**
Elements are selected at top of IIFE. Add new elements to HTML, query in JavaScript section, and update in game state functions.

### Common Tasks

**Adjust Game Balance:**
- Enemy difficulty: modify `DIFFICULTIES` object values
- Boss HP: change `baseHp` formula in `spawnBoss()` or `hpMul` in `BOSSES`
- Weapon power: adjust bullet speed, spread angle, or pierce value
- Spawn rate: adjust `spawnBase`/`spawnMin` in difficulties

**Debug Game State:**
Open browser DevTools console and access global game variables:
```javascript
console.log(score, lives, stage, chain, multiplier)
console.log(player.weapon, player.level)
```

**Performance Tuning:**
- Lower `quality` setting affects particle counts and shadow rendering
- Adjust particle spawn rate in `spawnExplosion()`
- Reduce complexity of boss patterns if needed

**Audio Adjustments:**
- Master volume: modify `masterGain.gain.value` (currently 0.9)
- Effect volume: adjust `vol` parameter in `beep()` and `noiseBurst()` calls
- Compressor settings: adjust `threshold`, `knee`, `ratio` in `ensureAudio()`

## Testing Checklist

When making changes, verify:
- Game starts and plays on desktop and mobile
- Keyboard and touch controls work
- Audio plays correctly (if not muted)
- Boss battles spawn correctly and attacks pattern properly
- Collision detection works for all entity types
- Screen scaling/responsiveness at various window sizes
- Reduced motion preference respected (no animations when enabled)
- Best score persists across page reloads
- Difficulty selection persists
