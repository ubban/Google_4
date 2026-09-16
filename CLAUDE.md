# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project Overview

**Sky Fighter** is a Raiden-style vertical shoot-'em-up game built entirely as a single-file HTML5 Canvas application. No build process, bundler, or external dependencies are required—it's designed to work immediately in any modern browser.

## Architecture

The entire game is contained in `index.html` with three main sections:

### 1. **CSS Styling** (lines 10–359)
- Responsive canvas sizing with viewport scaling
- HUD elements: score, lives (hearts), bombs, weapon stats
- Overlay screens: title, pause, game over
- Full-screen visual effects: vignette for critical health, flash on hit, screen shake
- Touch-friendly buttons and joystick visualization
- Accessibility: reduced-motion media query support, screen-reader-only text

### 2. **HTML Structure** (lines 361–420)
- Single `<canvas id="gameCanvas">` for all game rendering
- Overlay system using `#overlay` for start screen, pause, and game-over screens
- Touch controls: joystick zone and action buttons (fire, bomb)
- HUD elements: hearts (❤️), bombs (💣), score, multiplier, weapon level, boss health bar
- Accessibility: `#srLive` for screen-reader announcements

### 3. **JavaScript Game Loop** (lines 422–1666)
The script is wrapped in an IIFE and uses a single-threaded game loop via `requestAnimationFrame`.

## Game Flow & Key Systems

### **Initialization & Reset**
- `resetGame()` (line 875): Initializes/resets all game state (player, enemies, score, lives, stage)
- Loads difficulty settings, best score, and user preferences from `localStorage`

### **Main Game Loop** (line 1640)
- `loop(now)`: Called every frame via `requestAnimationFrame`
- Calculates delta time, updates game state if running/not paused, renders
- Adaptive performance: tracks frame times and reduces quality (particle count, shadows) if framerate drops below 40fps

### **Player Movement & Weapon System**
- **Input**: Keyboard (arrows + WASD), mouse steering, joystick (touch)
- `fireWeapon()` (line 1124): Fires vulcan (spread) or laser (piercing) based on weapon type and level
- Weapon levels increase spread; after level 3, switches to opposite weapon
- Mouse input has priority over keyboard/joystick when pointer is active

### **Enemy System**
- **Types**: Grunt (fast, weak), Interceptor (very fast), Tank (slow, armored, worth more points)
- Each type has min stage requirement; tanks appear at stage 2+
- `spawnEnemy()` (line 1022): Enemies have random firing patterns (spread or aimed), weave behavior for some types
- Enemies spawn at increasing rates as difficulty ramps up (lines 1201–1203)
- Collision detection with bullets triggers damage/death, particle explosions, score with combo multiplier

### **Boss System**
- Spawns after player kills `KILLS_PER_STAGE` (18) enemies
- Six boss types cycle through: VANGUARD, BEHEMOTH, SPECTER, DREADNOUGHT, WRAITH, TITAN
- Each boss has unique attack patterns stored in `BOSS_PATTERNS` array (lines 1372–1409)
- Boss enters with telegraph animation (frame count), then fires complex bullet patterns (radial, aimed, homing, crossing)
- `updateBoss()` (line 1411): Handles boss movement, attack telegraphing, and pattern execution

### **Scoring & Combo System**
- `addScore(base)` (line 1087): Multiplies by combo multiplier (1–10x based on consecutive kills within 2.2 seconds)
- Chain breaks when no enemies killed for 2.2 seconds
- Boss kill grants fixed 500 points
- Kill milestones (every 10th kill) trigger combo pop animation

### **Input Handling**
- **Keyboard**: Arrow keys / WASD move, Space fires, X/Shift bombs, P/Esc pause
- **Mouse**: Pointer steers player to cursor (gliding motion), click fires
- **Touch**: Left 55% screen drag for movement (joystick), right action zone with FIRE/BOMB buttons
- Audio context initialized on first user interaction

### **Audio System** (lines 542–676)
- WebAudio synth-based SFX (no audio files required)
- `beep()`: Generates oscillator-based sounds with frequency sweeps and filters
- `noiseBurst()`: Generates noise-based explosion/boom effects
- Sound objects (`sfx`): shoot, laser, explodeGrunt, explodeInterceptor, explodeTank, hit, bomb, powerup, bossHit, bossTelegraph, bossDown, stageClear, uiClick, gameover
- Panned stereo audio for horizontal bullet positions
- Mute state persisted in localStorage

### **Visual Systems**
- **Themes**: Six color schemes (sky gradient, cloud, sun colors) cycle per stage
- **Clouds**: Parallax scrolling at different speeds
- **Particles**: Explosion effects, fade over 0.5s
- **Screen Shake**: Temporary canvas translation with random offset (reduced if reduced-motion enabled)
- **Flash Effects**: White screen flash on damage/bomb, fades over 0.25–0.5s
- **Glow**: Shadow blur on bullets/powerups enabled only at highest quality setting

### **Difficulty Levels** (line 485–489)
- **Easy**: Slower spawns, weak enemies, longer shoot intervals, 4 lives, 3 bombs, boss 0.8x HP
- **Normal**: Balanced spawn, standard enemies, normal shooting, 3 lives, 2 bombs, 1.0x HP
- **Hard**: Dense spawns, aggressive enemies, constant shooting, 3 lives, 2 bombs, 1.3x boss HP

### **Power-Ups**
- **Life**: Recovers one health (capped at max)
- **Bomb**: Recovers one bomb (capped at max)
- **Weapon**: Increases weapon level or switches weapon type (resets to level 1)
- `applyPowerUp()` (line 1304): Applies effect and plays powerup SFX

## Common Tasks

### **Adjust Difficulty Balance**
- Edit `DIFFICULTIES` object (lines 485–489)
- Parameters: spawnBase/Min (frame delay), enemySpeed, enemyShootChance, lives, bombs, bossHpMul

### **Add/Modify Enemy Types**
- Add to `ENEMY_TYPES` object (lines 502–506)
- Define: w, h, hp, speedMul, color, dark, score, minStage, weave (optional)
- Add spawn logic in `spawnEnemy()` if type needs special handling

### **Add/Modify Boss Patterns**
- Add boss to `BOSSES` array (lines 508–515) with name and color scheme
- Add corresponding pattern array to `BOSS_PATTERNS` (lines 1372–1409)
- Each pattern function receives boss `b` and fires bullet patterns via helper functions:
  - `fireRadial()`: Bullets spread evenly in all directions
  - `fireAimedSpread()`: Bullets aimed at player with spread
  - `fireColumns()`: Vertical columns below boss
  - `fireSpiralBurst()`: Rotating spiral pattern
  - `fireDoubleRing()`: Two concentric rings

### **Adjust Visual Themes**
- Edit `THEMES` array (lines 493–500)
- sky: [start, mid, end] gradient colors
- cloud, sun: RGB string for parallax elements and glow
- name: Display name for stage transition banner

### **Tweak Performance**
- Quality levels 0 (full), 1 (medium), 2 (low) auto-detected from frame timing
- Adjust frame time thresholds in `trackPerf()` (lines 476–483)
- Modify particle count reduction in `spawnExplosion()` (line 1074)
- Disable shadow blur for quality > 0 in `drawBossShip()` (line 1002)

### **Change Canvas Resolution**
- Base resolution: 480×720 (lines 753–754, stored in `BASE_W`, `BASE_H`)
- Scales up to 1.6x on large screens via `resize()` function
- Modify `BASE_W` and `BASE_H` to change aspect ratio

## Testing the Game

### **Start the Game**
Open `index.html` directly in any modern browser. No server required.

### **Test Different Scenarios**
- **Difficulty**: Click Easy/Normal/Hard on the start screen (persists via localStorage)
- **Sound**: Click 🔊 button to toggle mute
- **Reduced Motion**: Click 🎆 button to disable screen shake/flash animations
- **Pause**: Press P or Esc during gameplay, or click ⏸ button
- **Mobile**: Use browser dev tools device emulation to test touch controls

### **Debug Tips**
- Open browser console (F12) to check for JavaScript errors
- Canvas rendering is all procedural (no images), so look at drawing functions (drawPlane, drawBossShip, etc.)
- Game state: `score`, `lives`, `stage`, `chain`, `multiplier`, `boss` are globals within IIFE scope
- Performance: Check console for frame time warnings if game feels slow

## Key Decisions & Trade-offs

- **Single File**: No build process or bundler needed, but harder to modularize. All state is local to the IIFE.
- **WebAudio Synth**: No audio file downloads, but limited to procedurally-generated sounds. Mute check happens per-sound for performance.
- **Canvas 2D**: No 3D rendering, but mature browser support and sufficient for 2D shmup effects.
- **Responsive Scaling**: Canvas fixed 480×720, but scales proportionally on all screen sizes. Touch controls adapt to viewport.
- **Accessibility**: Reduced-motion support disables animations entirely. Screen reader announcements for key events (stage, boss defeat, lives).

## Browser Compatibility

Requires:
- ES6 (const, arrow functions, template literals)
- Canvas 2D context
- WebAudio API (audio.suspend/resume)
- localStorage
- requestAnimationFrame
- Touch Events API (for mobile)

Works on all modern browsers (Chrome, Firefox, Safari, Edge). IE11 not supported.
