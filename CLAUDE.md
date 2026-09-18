# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project Overview

**Sky Fighter** is a Raiden-style vertical shoot-'em-up game implemented as a single-file HTML5 application. The entire game (~1,700 lines) is contained in `index.html`, requiring no build tools or external dependencies.

### Key Features

- Canvas-based 2D rendering with adaptive quality scaling
- WebAudio API for synthesized sound (no audio files)
- Three difficulty levels (Easy, Normal, Hard)
- Six stages, each with unique enemy patterns and boss encounters
- Multiple weapon types with levels and power-up system
- Smart bomb mechanic for area clearing
- Combo/multiplier system for consecutive kills
- Mobile support with touch controls (joystick + action buttons)
- Keyboard controls (mouse steering, arrow keys, WASD)
- Accessibility: screen reader announcements, reduced motion support
- Persistent score tracking via localStorage

## Development

### Running the Game

No build process is required. Simply open `index.html` in a modern web browser:

```bash
# Direct file opening
open index.html
# Or serve locally
python3 -m http.server
```

The game is responsive and works on desktop (1024px minimum recommended for best experience) and mobile devices.

### Code Structure

The entire application is a single IIFE (Immediately Invoked Function Expression) in strict mode, wrapping all game logic. The structure follows this pattern:

1. **HTML & CSS** (lines 1-359): DOM structure and responsive styling
2. **JavaScript** (lines 421-1669): Game implementation in one scope

### Game Architecture

#### Core Systems

1. **Player & Input**
   - Mouse: move to steer, click to fire
   - Keyboard: Arrow keys/WASD for movement, Space to fire, X/Shift for bomb
   - Touch: drag left side to fly, tap FIRE/BOMB buttons
   - Joystick tracking with dead zone handling

2. **Rendering Pipeline**
   - Canvas context with performance-adaptive quality (3 levels: full, medium, low)
   - Sky background with parallax cloud layers
   - Entity rendering: player plane, enemies, bosses, projectiles, explosions
   - HUD overlay with score, health, weapons, boss bar
   - UI overlays (start screen, pause, game-over)

3. **Game State Management**
   - State variables: `running`, `paused`, `gameOver`
   - Difficulty selection stored in localStorage
   - Score/best score tracking
   - Chain/multiplier tracking for combo scoring

4. **Entity Systems**
   - Player: health, bomb count, weapon, position, rotation
   - Enemies: spawn at top, move down, shoot at player, award points on death
   - Bosses: multi-stage health, complex firing patterns, phase transitions
   - Projectiles: bullets from player, bullets from enemies
   - Power-ups: weapon upgrades, health, bombs (spawn on enemy death)
   - Particles: explosions, visual effects

5. **Audio**
   - WebAudio synthesizer with no external files
   - `beep()`: configurable oscillator sounds with filters, panning, frequency sweeps
   - `noiseBurst()`: noise-based percussion effects
   - Master gain + dynamics compressor for mix management
   - Triggered by: weapon fire, hits, explosions, UI interaction

#### Difficulty Settings

```javascript
DIFFICULTIES = {
  easy:   { spawnBase: 75, spawnMin: 40, enemySpeed: 1.0, ... },
  normal: { spawnBase: 58, spawnMin: 26, enemySpeed: 1.4, ... },
  hard:   { spawnBase: 40, spawnMin: 18, enemySpeed: 1.9, ... }
}
```

Each difficulty scales enemy spawn rate, speed, aggression, and boss health.

#### Theme System

Six visual themes (Daylight Front, Sunset Corridor, Night Approach, Storm Front, Arctic Wastes, Volcanic Ridge) each define:
- Gradient sky colors
- Cloud color (for parallax effect)
- Sun/light color
- Selected per stage, cycling through the theme array

#### Enemy Types

- **Grunt**: Fast, weak, common (HP 1)
- **Interceptor**: Very fast, weaves, starts at stage 1 (HP 1)
- **Tank**: Slow, durable, heavy scoring, stage 2+ (HP 4)

#### Boss Types

Six bosses (VANGUARD, BEHEMOTH, SPECTER, DREADNOUGHT, WRAITH, TITAN) with:
- Distinct color schemes (hull top/bottom, wing, core)
- Health multiplier per boss
- Unique firing patterns defined in `updateBoss()`

#### Weapon System

Weapon upgrade path: VULCAN → SPREAD → HOMING → RADIAL, with defined level caps and fire patterns. Each weapon has a fire rate and spread pattern. Power-ups randomly grant weapon upgrades.

#### Performance Adaptation

Frame time tracking (last 45 frames) automatically scales render quality:
- `quality = 0`: full rendering
- `quality = 1`: medium (skip some particle updates)
- `quality = 2`: low (minimal effects, no vignette)

### Key Functions

- **`update(dt)`**: Main game loop physics and state updates
- **`draw()`**: Render all entities and UI
- **`loop(now)`**: Animation frame handler, delta-time tracking
- **`fireWeapon()`**: Detect weapon type, call appropriate fire pattern
- **`updateBoss(dt)`**: Boss AI, phase transitions, special moves
- **`maybeSpawnPowerUp(x, y)`**: Random drop on enemy death
- **`applyPowerUp(type)`**: Apply power-up effect to player
- **`useBomb()`**: Trigger bomb effect, clear nearby projectiles
- **`addScore(base)`**: Add score with multiplier, update best score

### Styling & Responsive Design

- CSS Grid/Flexbox for layout
- `clamp()` for responsive font sizes (scales from mobile to desktop)
- Safe area insets for notched devices
- Touch-action and user-select properties for mobile-friendly interaction
- Dark theme with semi-transparent overlays and backdrop filters
- Animations respect `prefers-reduced-motion` media query

### Local Storage Keys

```javascript
'skyfighter.bestScore'    // Best score ever achieved
'skyfighter.muted'        // 1 = muted, 0 = unmuted
'skyfighter.difficulty'   // 'easy', 'normal', 'hard'
'skyfighter.reducedMotion' // 1 = reduced, 0 = full, null = system default
```

## Modification Patterns

### Adding a New Enemy Type

Define in `ENEMY_TYPES` object with properties: `w`, `h`, `hp`, `speedMul`, `color`, `dark`, `score`, `minStage`, `weave` (optional). Then reference in `spawnEnemy()` and `update()` when spawning or updating enemies.

### Adding a New Weapon

Add fire pattern function (e.g., `fireCustomPattern(b)`), add weapon to the upgrade chain in `fireWeapon()`, and set fire rate/damage parameters in weapon type comparisons.

### Adding a New Boss

Add entry to `BOSSES` array with name, hull colors, core color, hpMul. Then add unique firing logic in `updateBoss()` within the boss-index switch.

### Adjusting Game Balance

- Enemy spawn rates: modify `spawnBase` and `spawnMin` in `DIFFICULTIES`
- Enemy/boss health: adjust `hp` values or `hpMul` multipliers
- Scoring: change base scores in `ENEMY_TYPES` or `BOSSES`, or multiplier scaling in `addScore()`
- Weapon balance: modify fire rate or spread pattern count in respective functions

### Audio Tweaks

- Modify frequency ranges in `beep()` calls (search for `beep(`)
- Adjust master gain in `ensureAudio()` (default 0.9)
- Tune compressor settings for different mix character

## Testing & Verification

- **Manual play**: Test all difficulty levels, all weapons, all bosses
- **Responsive**: Verify on mobile, tablet, and desktop at various sizes
- **Accessibility**: Check keyboard controls, screen reader announcements, reduced motion toggle
- **Audio**: Verify mute button works, sounds play on first interaction (browser policy)
- **Storage**: Verify best score, difficulty selection, and motion preference persist

## Known Constraints

- Single HTML file (no module system, all scope shared in IIFE)
- No external libraries (Canvas 2D, WebAudio, vanilla DOM APIs only)
- Canvas rendering targeted at 2D API (no WebGL)
- Audio is synthesized on-the-fly (no asset loading)
- localStorage used for persistence (cleared if user clears browser data)
