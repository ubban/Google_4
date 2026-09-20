# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project Overview

**Sky Fighter** is a Raiden-style vertical shoot-'em-up game implemented as a single-file HTML5 application. The entire game engine, rendering, audio synthesis, and UI logic is contained in `index.html` (~1670 lines).

### Key Characteristics
- **Architecture**: Single-file monolithic game (no build process, no dependencies)
- **Rendering**: HTML5 Canvas 2D context
- **Audio**: WebAudio API with procedural synthesis (no audio files)
- **Storage**: Browser localStorage for scores, settings, preferences
- **Platforms**: Desktop (keyboard/mouse) and mobile (touch controls)

## Development Quick Start

There is **no build process** for this project. The game runs directly by opening `index.html` in a browser.

### Running the Game
```bash
# Open in any modern web browser
open index.html  # macOS
xdg-open index.html  # Linux
start index.html  # Windows
```

Or use any local server:
```bash
python3 -m http.server 8000
# Then visit http://localhost:8000
```

### Common Tasks
- **Testing changes**: Open `index.html` in a browser, test gameplay/UI
- **Performance tuning**: Browser DevTools (F12) > Performance tab to profile frame times
- **Accessibility testing**: Test with keyboard-only and screen reader (the game announces key events via `srLive`)
- **Mobile testing**: Test touch controls via browser DevTools device emulation or physical device

## Architecture & Code Organization

The game is organized as a single IIFE (Immediately Invoked Function Expression) starting at line 423, with these major sections:

### 1. **UI & DOM (lines 426-461)**
- Canvas and overlay elements
- HUD elements (score, lives, weapons, multiplier)
- Boss health bar
- Touch controls UI (joystick, fire/bomb buttons)
- Screen reader element for accessibility

### 2. **Configuration & Constants (lines 463-515)**
- `STORAGE_*`: localStorage keys for persistence (scores, audio, difficulty, motion preferences)
- `DIFFICULTIES`: Three difficulty tiers (easy/normal/hard) with spawn rates, enemy AI behavior, lives/bombs, boss HP multipliers
- `KILLS_PER_STAGE`: Number of kills needed to trigger boss
- `THEMES`: Six sky/environment color themes that cycle with stages
- `ENEMY_TYPES`: grunt, interceptor, tank with stats (size, HP, speed multiplier, score, spawn stage)
- `BOSSES`: Six boss designs with unique colors and HP multipliers

### 3. **Audio System (lines 542-676)**
- `ensureAudio()`: Lazy-loads WebAudio context on first interaction (browser requirement)
- `beep(freq, dur, type, vol, opts)`: Generates tone with optional sweep, filter, pan
- `noiseBurst(dur, vol, opts)`: Procedural noise for explosions
- `sfx` object: Named sound effects (shoot, laser, explode variants, hit, bomb, powerup, boss sounds, etc.)
- **Key design**: All audio is real-time synthesis; no audio files

### 4. **Performance Adaptation (lines 473-483)**
- Adaptive quality scaling (0=full, 1=medium, 2=low) based on frame time average
- `trackPerf(dt)`: Adjusts shadow blur, glow effects, particle count based on performance
- Helps maintain 60fps on lower-end devices

### 5. **Input Handling (lines 767-861)**
- Keyboard: Arrow keys + WASD for movement, Space to fire, X/Shift for bomb, P/Esc to pause
- Mouse: Move to steer, click to fire (on canvas)
- Touch: Joystick in bottom-left quadrant, Fire/Bomb buttons in bottom-right
- Joystick state: `joyActive`, `joyId`, `joyOrigin`, `joyVec`
- Mouse steering: Blends keyboard/joystick input (priority) with mouse target when no keyboard input

### 6. **Game State (lines 863-911)**
- `player`: Position, velocity, weapon, cooldown, invulnerability timer
- `bullets`, `enemies`, `enemyBullets`, `particles`, `powerUps`: Entity arrays
- `boss`: Boss state when active (null otherwise)
- `score`, `lives`, `kills`, `stage`: Game progression metrics
- `chain`, `chainTimer`, `multiplier`: Combo system (multiplier increases per 5 kills)
- `resetGame()`: Initializes all state for a new game run

### 7. **Rendering (lines 913-1020, 1520-1638)**
- `drawPlane(x, y, w, h, color, darkColor, flame)`: Draws player or enemy aircraft
- `drawBossShip(b)`: Boss-specific rendering with gradient hull, core glow, telegraph effects
- `draw()`: Main render loop—sky gradient, clouds, entities, particles, camera shake
- **Performance-aware**: Glow effects (shadows/blur) only drawn at full quality

### 8. **Physics & Collision (lines 1148-1302)**
- `update(dt)`: Main game loop
  - **Input**: Reads keyboard, joystick, mouse and moves player with speed damping
  - **Firing**: Maintains cooldown timer, spawns bullets
  - **Enemies**: Spawn timer, movement (with optional weave pattern), shooting logic
  - **Collision**: Overlap detection between bullets/enemies, enemies/player, enemy bullets/player, powerups/player
  - **Screen boundaries**: Clamps player position, removes off-screen bullets/enemies
- `rectsOverlap(a, b)`: Simple AABB collision check

### 9. **Enemy & Boss AI (lines 1022-1439)**
- `spawnEnemy()`: Random type selection (grunt → interceptor → tank), random patterns (spread/aimed shooting)
- **Enemy patterns**:
  - Aimed: Enemy calculates angle to player and shoots
  - Spread: Enemy shoots 3 bullets in fixed fan pattern
  - Weave: Movement oscillates side-to-side (interceptor only)
- `spawnBoss()`: Creates boss for current stage; boss HP scales by stage and difficulty
- **Boss attack patterns** (lines 1372-1409): Six boss designs each have 3 unique attack patterns
  - Patterns: radial bursts, aimed spreads, columns, spirals, crosses, homing, double rings
  - Boss telegraphs 22 frames before attacking; core glows white during telegraph
  - Patterns cycle and are position-aware for telegraphic/dodgeable design

### 10. **Score & Progression (lines 1087-1101, 1304-1322)**
- `addScore(base)`: Increments chain on every kill; multiplier = 1 + floor(chain/5), max 10x
- `breakChain()`: Reset on time or damage (chainTimer = 2.2 seconds between kills)
- `applyPowerUp(type)`: Weapon upgrades (level 1→2→3, then swap vulcan/laser), health restore, bomb refill
- **Weapons**:
  - **Vulcan** (default): Rapid spread shot, 0.15s cooldown
    - Level 1: Single center bullet
    - Level 2: Two side bullets
    - Level 3: Three bullets (left, center, right)
  - **Laser**: Piercing beams, 0.1s cooldown
    - Level 1: Single center beam (pierces 1 enemy)
    - Level 2: Two side beams (each pierces 1)
    - Level 3: Three beams (center pierces 2)

### 11. **Difficulty & Game Flow**
- Three difficulties modify spawn rate, enemy speed, shoot chance, lives (3-4), bombs (2-3), boss HP (0.8-1.3x)
- **Stage progression**: Kill 18 enemies → trigger boss warning → boss spawns 1.2s later
- **Stage cycling**: Themes and boss designs cycle through arrays (6 themes, 6 bosses)
- Boss defeat adds 500 points, advances stage, resets stage kill counter, resets boss warning flag

### 12. **Settings & Persistence (lines 517-750)**
- `localStorage` keys: best score, mute state, difficulty, reduced motion preference
- `reducedMotion`: Disables animations, screen shake, flash effects (respects OS prefers-reduced-motion)
- Difficulty selection buttons, mute button, pause button, motion toggle button

## Important Design Decisions

### Single-File Approach
- **Pro**: No build tools, no module imports, runs anywhere (including offline)
- **Con**: Large file, all-or-nothing reload, harder to isolate systems
- **When modifying**: Understand you're editing one cohesive piece; coordinate changes across sections

### WebAudio Synthesis
- All sounds are procedurally generated at runtime using oscillators, filters, noise buffers
- This means audio is **not** embedded as assets; game always generates fresh sounds
- Each sound effect (`sfx.*`) is a function that creates and plays audio; modify frequency, duration, waveform to change sounds

### Difficulty Tuning
- All parameters (spawn rate, enemy speed, boss HP) scale by difficulty enum in `DIFFICULTIES`
- To add a new difficulty level, add entry to `DIFFICULTIES` object and update the UI buttons

### Mobile-First Input
- Keyboard and mouse are primary on desktop; touch controls replace them on mobile
- Joystick and button position is responsive (CSS `clamp()` for adaptive sizing)
- Game detects `ontouchstart` to decide whether to show touch UI

### Performance Adaptation
- Frame-time tracking (`trackPerf()`) adjusts visual quality in real-time
- Quality levels 0-2 control shadow blur, glow effects, particle counts
- Allows game to maintain playability on low-end devices

## Common Modifications

### Adding a New Enemy Type
1. Add entry to `ENEMY_TYPES` object (line ~502) with stats
2. Adjust `minStage` to control when it spawns
3. Update spawn probability in `spawnEnemy()` if needed
4. Add explosion SFX in `EXPLODE_SFX` map if desired
5. Add sprite/rendering logic in the enemies draw loop (around line 1603)

### Adding a New Boss Pattern
1. Create firing function (e.g., `fireSpread()` style function)
2. Add pattern array to `BOSS_PATTERNS` entry for that boss, or create new boss entry
3. Adjust attack timer and telegraph duration if pattern needs more/less warning time

### Tweaking Weapon Balance
- Modify `DIFFICULTIES` entries to change spawn rate / enemy speed for difficulty tuning
- Modify weapon cooldown in `fireWeapon()` (line ~1177)
- Modify bullet properties (speed, spread angle, pierce count) in `fireWeapon()` body

### Adjusting Screen Shake / Effects
- `screenShake(mag, dur)`: Modify `mag` for intensity, `dur` for duration
- Calls to `screenShake()` are throughout (damage, explosions, bomb)
- `reducedMotion` mode scales shake down to 15% intensity and caps duration at 0.1s

## Testing Checklist

When making changes, test:
- **Gameplay**: Movement, firing, collision, damage
- **Enemy spawning**: Correct types at correct stages
- **Boss encounters**: All patterns execute, telegraph visual works, defeat triggers stage clear
- **Weapons**: Switching works, pierce works, spread is correct
- **Powerups**: Collection, effects, UI updates
- **Score/multiplier**: Increments correctly, chain timeout works
- **Difficulty levels**: Spawn rate and difficulty changes work
- **Mobile controls**: Joystick responds, buttons fire/bomb correctly
- **Accessibility**: Screen reader announcements work (mute, motion, pause, stage/boss arrival)
- **Performance**: FPS stable on target device (check DevTools Performance tab)
- **localStorage**: Settings persist across page reload

## Files & Structure

- `index.html`: Single source file containing all game logic, rendering, audio, and UI (1670 lines)
- `README.md`: Minimal project info
- No external dependencies, no build config, no node_modules

## Browser Requirements

- **Canvas 2D**: All modern browsers
- **WebAudio API**: All modern browsers (required for sound; gracefully degrades to silent if unavailable)
- **localStorage**: All modern browsers (gracefully degrades if blocked)
- **Touch Events**: Mobile browsers (fallback to keyboard/mouse on desktop)
- **matchMedia**: Used for reduced-motion preference detection (optional, has fallback)
