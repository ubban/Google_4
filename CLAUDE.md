# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project Overview

**Sky Fighter** is a browser-based Raiden-style vertical shoot-'em-up game. The entire game is contained in a single `index.html` file with embedded CSS and JavaScript. It features:

- Canvas-based 2D rendering
- Keyboard and mouse input (with touchscreen support for mobile)
- WebAudio API-based sound synthesis (no external audio files)
- Difficulty modes (Easy, Normal, Hard)
- Multiple enemy types and six boss variants
- Weapon upgrade system (Vulcan → Laser, with 3 levels each)
- Smart bomb mechanic
- Score multiplier chain system
- Adaptive graphics quality based on frame rate
- Local storage for best score and settings
- A11y features (reduced motion support, screen reader announcements)

## How to Run

The game is fully playable by simply opening `index.html` in any modern web browser. No build step or local server is required (though the game works with a local server too).

**To test locally:**
```bash
# Option 1: Open directly in browser
open index.html

# Option 2: Use a simple HTTP server
python3 -m http.server 8000
# Then visit http://localhost:8000
```

The game is playable immediately on load. The overlay screen shows difficulty selection and controls.

## Code Architecture

The game is organized as a single IIFE (Immediately Invoked Function Expression) that manages all game state and logic. Key structural sections:

### 1. **Constants & Configuration** (lines ~463-525)
- `STORAGE_*`: LocalStorage keys for best score, mute state, difficulty, reduced motion preference
- `DIFFICULTIES`: Defines spawn rates, enemy speeds, shoot chances, lives, bombs per difficulty
- `KILLS_PER_STAGE`: Enemies to defeat before boss spawns (18)
- `THEMES`: Six sky themes with gradient colors, cloud color, and sun color
- `ENEMY_TYPES`: Grunt, Interceptor, Tank (with HP, speed, scoring)
- `BOSSES`: Six boss designs with color schemes and HP multipliers
- `BOSS_PATTERNS`: Attack pattern functions for each boss

### 2. **Audio System** (lines ~542-676)
- `ensureAudio()`: Lazy-initializes Web Audio Context on first user interaction
- `beep()`: Creates tones with optional frequency sweep, filter, and panning
- `noiseBurst()`: Generates noise for explosion sounds
- `sfx` object: Pre-defined sound effects (shoot, laser, explode, bomb, powerup, etc.)
- All audio is synthesized in-memory; no external sound files

**Key insight:** Sound effects don't play until `ensureAudio()` is called after user interaction, following modern browser autoplay policies.

### 3. **Input Handling** (lines ~767-861)
- **Keyboard:** Arrow keys, WASD, Space (fire), X/Shift (bomb), P/Esc (pause)
- **Mouse:** Move to steer, click to fire
- **Touch:** Left half of screen for movement (virtual joystick), right half for fire/bomb buttons
- Input state is read in the update loop; the game never modifies the keys object directly

**Priority:** Keyboard/joystick input takes precedence over mouse steering to avoid fighting between control methods.

### 4. **Game State** (lines ~863-911)
- `player`: Position, velocity, weapon, level, invulnerability timer, cooldown
- `bullets`, `enemies`, `enemyBullets`, `particles`, `powerUps`, `boss`
- `score`, `lives`, `chain`, `multiplier`, `stage`, `stageKills`
- `running`, `paused`, `gameOver`: Game loop control flags

### 5. **Game Loop** (lines ~1640-1665)
- `loop(now)`: RequestAnimationFrame callback
- Calculates delta-time, clamps to 0.05s max per frame
- Calls `update(dt)` if running and not paused
- Calls `draw()` to render
- Tracks performance for adaptive quality

### 6. **Update Logic** (lines ~1148-1302)
- **Player movement:** Reads input (keyboard, joystick, mouse) and updates player position
- **Weapon firing:** Cooldown logic, weapon-specific firing patterns (vulcan spread, laser lanes)
- **Enemy spawning:** Difficulty-based spawn rate that increases over time
- **Bullet/Enemy collision:** Checks all player bullets vs. all enemies
- **Boss collision:** Separate handling for boss hits (no pierce, different damage)
- **Enemy bullet collision:** Checks enemy bullets vs. player
- **Powerup collection:** Life, bomb, weapon upgrades
- **Chain system:** Increases multiplier for consecutive kills within 2.2 seconds
- **Boss logic:** Handled in `updateBoss()` for movement, telegraph/attack patterns

### 7. **Boss System** (lines ~1372-1439)
- `BOSS_PATTERNS`: Six arrays of attack pattern functions
- Each boss cycles through 3 patterns from their pattern list
- Patterns include: radial bursts, aimed spreads, columns, spirals, homing, etc.
- `telegraph`: Brief warning (22 frames) before boss fires, during which core glows white
- Boss HP scales with stage and difficulty

### 8. **Rendering** (lines ~1520-1637)
- **Sky/theme:** Gradient background with sun glow and animated clouds
- **Player plane:** Drawn as polygons with flame trail when moving
- **Enemies/Boss:** Also drawn as polygons with appropriate colors
- **Bullets/particles:** Rendered with optional glow (high quality mode)
- **HUD:** Score, best, multiplier, weapon, hearts, bombs (managed via DOM)
- **Screen shake:** Applies random translation when active
- **Flash effects:** White screen flash on hit or bomb (reduced in reduced-motion mode)

**Quality tiers** (adaptive based on frame time):
- Quality 0: Full detail with glow effects
- Quality 1: Reduced glow
- Quality 2: Minimal effects

### 9. **Performance Optimization** (lines ~474-483)
- `trackPerf(dt)`: Monitors last 45 frames of delta-times
- Automatically downgrades quality if avg > 1/40 (25ms per frame)
- Upscales if avg < 1/56 (~17.8ms)
- Reduces particle count and shadow effects at lower quality levels

## Key Game Mechanics

### Weapon System
- **Vulcan:** Spread bullets (1 lane → 2 lanes → 3 lanes at levels 1–3)
- **Laser:** Piercing bullets (pierces 1 → 2 → 3 enemies; 2 lanes at level 3)
- Weapons switch and level up via power-ups
- Max of 3 levels per weapon; switching weapons resets to level 1

### Combo System
- Each enemy kill increments the chain counter
- Multiplier = 1 + floor(chain / 5), capped at 10x
- Chain resets if >2.2 seconds pass without a kill or player takes damage
- Visual feedback: "+N KILLS!" popup at 10, 20, 30, etc. kills

### Boss Mechanics
- Bosses spawn after 18 enemy kills per stage
- Telegraph phase (22 frames) with glowing core before each attack
- Three attack patterns per boss, cycled and indexed
- Boss HP: `70 + stage * 35` adjusted by difficulty multiplier and boss design multiplier

### Difficulty Scaling
- **Spawn rate:** Decreases over time in-game (difficultyRamp)
- **Enemy behavior:** Varies by difficulty (shoot chance, speed)
- **Boss HP:** Multiplied by 0.8/1.0/1.3 for Easy/Normal/Hard

## File Structure

```
Google_4/
├── index.html     (entire game in one file)
├── README.md      (basic project info)
└── CLAUDE.md      (this file)
```

## Development Notes

### Adding New Content

**New Enemy Type:**
1. Add entry to `ENEMY_TYPES` object with `w`, `h`, `hp`, `speedMul`, `color`, `dark`, `score`, `minStage`
2. Optionally add to `EXPLODE_SFX` with custom explosion sound
3. Add to `spawnEnemy()` probability check if needed

**New Boss:**
1. Add to `BOSSES` array with color scheme and HP multiplier
2. Create new 3-function pattern array in `BOSS_PATTERNS`
3. Boss appearance is generated procedurally from `design` colors

**New Weapon:**
1. Add to weapon check in `fireWeapon()` function
2. Define spread/lanes logic for each level
3. Add to `weaponLabel()` if adding a new weapon type

### LocalStorage Keys
All game state persists via LocalStorage:
- `skyfighter.bestScore`: Best score achieved
- `skyfighter.muted`: Audio mute state (1 or 0)
- `skyfighter.difficulty`: Selected difficulty
- `skyfighter.reducedMotion`: Reduced motion preference (1 or 0)

### Canvas Sizing
- Base resolution: 480×720 (vertical orientation)
- Scales up to max 1.6x on desktop, fits to window on mobile
- Uses `canvas.width/height` for game logic, `style.width/height` for display scale

### Reduced Motion Support
- Respects `prefers-reduced-motion` media query
- Disables screen shake (scales to 15%), flash effects, and animations
- Can be toggled via the 🎆 button in top-right

### Touch Controls
- Automatically enabled if touchscreen detected
- Left 55% of screen: movement zone with virtual joystick
- Right 45% of screen: fire and bomb buttons
- Buttons have visual feedback and prevent default touch behaviors

## Common Tasks

### Testing a Specific Difficulty
In the browser console:
```javascript
localStorage.setItem('skyfighter.difficulty', 'hard');
location.reload();
```

### Checking Game Stats During Play
The HUD displays:
- Score and best score (top center)
- Multiplier (x indicator)
- Current weapon and level
- Hearts (lives) and bomb count

### Adjusting Game Balance
- Modify `DIFFICULTIES` object for spawn/speed tuning
- Adjust `baseHp` calculation in `spawnBoss()` for difficulty
- Change `BOSS_PATTERNS` attack functions for boss behavior
- Tweak `fireWeapon()` spread angles for weapon feel

### Adding Sound Effects
- Use `beep(freq, duration, type, vol, opts)` for tones
- Use `noiseBurst(duration, vol, opts)` for noise/explosions
- Chain multiple calls in sfx functions for complex effects
- All audio plays through `masterGain` → compressor → destination

## Architecture Decisions

1. **Single file:** Simplifies deployment and ensures the game works immediately when opened
2. **Canvas rendering:** Provides fine control over visual effects and performance optimization
3. **Procedural sound:** No external audio dependencies; smaller total asset size
4. **Adaptive quality:** Ensures playability on slower devices without manual graphics settings
5. **LocalStorage:** Persists preferences/scores without requiring a backend
6. **IIFE scope:** All game state is private; prevents accidental global namespace pollution

## Browser Compatibility

Requires modern browser with:
- Canvas 2D context
- RequestAnimationFrame
- Web Audio API (for sound)
- Touch Events (for mobile)
- LocalStorage
- ES6 features (arrow functions, const/let)

Tested to work on:
- Chrome/Edge 90+
- Firefox 88+
- Safari 14+
- Mobile browsers (iOS Safari, Chrome Android)
