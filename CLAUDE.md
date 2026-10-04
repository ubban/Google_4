# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project Overview

**Sky Fighter** is a Raiden-style vertical shoot-em-up game implemented as a single-file HTML canvas application. The game features multiple difficulty levels, themed visual environments, procedurally-generated audio effects, and boss encounters.

The entire application (HTML, CSS, JavaScript) is contained in `index.html` (~58KB).

## Running the Game

### Development / Local Testing
- **Open in browser**: Simply open `index.html` directly in any modern web browser (no server required)
- **Best experience**: Use a local HTTP server (e.g., `python -m http.server 8000` or Live Server in VS Code) for proper audio context behavior
- **Mobile testing**: Open on mobile device or use DevTools device emulation for touch controls

### Difficulty Selection
- The game starts with an overlay showing difficulty options (Easy, Normal, Hard)
- Difficulty preference persists in localStorage under key `skyfighter.difficulty`
- Difficulty affects: enemy spawn rates, speeds, shoot frequency, player lives, bomb count, and boss HP

## Code Architecture

The entire JavaScript application is organized as a single IIFE (Immediately Invoked Function Expression) in `<script>` tags, starting around line 422. The code structure follows this logical flow:

### Constants & Configuration (lines 485-515)
- **DIFFICULTIES**: Spawn rates, enemy behavior, and player resources by difficulty level
- **THEMES**: Six visual themes with color schemes for sky gradient, clouds, and sun
- **ENEMY_TYPES**: Three enemy types—grunt (basic), interceptor (fast), tank (slow/tough)
- **BOSSES**: Six boss variants with different appearance and HP multipliers
- **KILLS_PER_STAGE**: Number of enemy kills required to advance to next stage

### State Management (lines 517-527)
- Best score, mute state, difficulty, and reduced-motion preference all persisted in localStorage
- Game state variables (running, paused, current score, multiplier, current weapon, etc.) set up as `let` variables in the outer scope

### Audio System (lines 542-679)
- **ensureAudio()**: Lazy-initializes Web Audio context with dynamics compressor
- **beep()**: Generates oscillator-based tones with frequency sweeps and envelope control
- **noiseBurst()**: Creates white-noise bursts (explosions, impacts) with filtering
- **sfx object**: Named sound effects (shoot, laser, explosions, powerups, boss sounds, etc.)
- All audio is procedurally generated—**no external audio files needed**
- Audio respects mute state and uses spatial panning based on X coordinate

### Canvas & Rendering (lines 752-765)
- **BASE_W / BASE_H**: Fixed 480x720 game canvas
- **resize()**: Scales canvas to fit window while maintaining aspect ratio (max 1.6x)
- Canvas responds to window resize and orientation change events

### Input Handling (lines 767-830+)
- **Keyboard**: Arrow keys, WASD (move), Space (fire), X/Shift (bomb), P/Esc (pause)
- **Mouse**: Click to fire, mouse position steers the player ship
- **Touch**: Two zones—left 55% moves ship, right 45% has FIRE and BOMB buttons
- **Joystick simulation**: For touch, a visual joystick base appears and follows finger movement

### Game Loop & Render System
The main game loop uses `requestAnimationFrame` and manages:
- **Entity updates**: Player position, enemy movement, projectile movement, explosions
- **Collision detection**: Player bullets vs enemies, enemy bullets vs player, bomb area vs entities
- **HUD updates**: Score, multiplier, weapon level, hearts (lives), bombs, boss health bar
- **Performance scaling**: Adaptive quality reduction if frame times exceed thresholds

### Key Game Systems

#### Player Entity
- Position-based movement (mouse or keyboard input)
- Health/lives system (displays as hearts in HUD)
- Smart bombs (limited count, clears screen with explosion)
- Current weapon tracked (affects firing pattern and speed)
- Fire rate controlled by weapon type

#### Enemy Spawning
- Spawn rate decreases as difficulty/stage increases (pulls from DIFFICULTIES config)
- Enemy types introduced gradually by stage (`minStage` property in ENEMY_TYPES)
- Enemies spawn at top, move downward and shoot at player
- Three behaviors: basic patrol (grunt), weaving (interceptor), stationary + shooting (tank)

#### Weapon System
- Multiple weapon types with different firing patterns (Vulcan, Laser, Spread, Rapid)
- Power-ups drop from defeated enemies
- Weapon level increases with pickups (affects damage/speed)
- Weapon name and level displayed in HUD

#### Staging & Bosses
- Stages advance after defeating KILLS_PER_STAGE enemies
- Boss appears after stage completion
- 6 boss types cycle through; boss selected based on stage number
- Boss bars (health) shown in top-center HUD
- Boss defeated triggers stage clear animation and advances to next stage

#### Visual Themes
- Theme selected per-stage (cycles through THEMES array)
- Each theme defines sky gradient, cloud color, and sun color
- Applied to canvas background rendering

#### Score & Multiplier
- Base score from defeating enemies (varies by type)
- Multiplier tracks consecutive hits without taking damage
- Resets on player hit
- Displayed in HUD with different colors

#### Accessibility
- **Reduced motion support**: Detects prefers-reduced-motion media query, disables animations/shakes if enabled
- **Screen reader support**: ARIA live region (`#srLive`) announces game state changes
- **Keyboard controls**: All game actions available via keyboard
- **Color contrast**: HUD and UI designed with accessibility in mind

### Key Variables & State (Outer Scope)

Important state variables that persist across the game loop:
- `running`: Boolean; false during overlay, true during active gameplay
- `paused`: Boolean; true when game is paused (P key or pause button)
- `score`, `combo`, `mult`: Current run stats
- `lives`, `bombs`: Player resources
- `weapon`, `weaponLvl`: Current weapon and its level
- `enemies[]`, `projectiles[]`, `explosions[]`: Active entities
- `particles[]`: Visual effects (stars, flashes, etc.)
- `quality`: Adaptive quality level (0=full, 1=medium, 2=low)
- `shakeTime`, `shakeMag`: Screen shake state for hit feedback

### Common Development Tasks

#### Tweaking Difficulty Balance
Edit the `DIFFICULTIES` object (line 485):
- `spawnBase`: Initial spawn interval (in frames, higher = easier)
- `spawnMin`: Minimum spawn interval (controls difficulty cap)
- `enemySpeed`: Multiplier on enemy speed (1.0 = normal)
- `enemyShootChance`: Probability enemy fires per frame (0.0–1.0)
- `lives`: Starting player health
- `bombs`: Starting bomb count
- `bossHpMul`: Boss health multiplier

#### Adding a New Theme
1. Add entry to `THEMES` array (line 493) with `sky`, `cloud`, `sun` colors, and a `name`
2. Format: `sky` is array of 3 CSS color strings for gradient stops
3. `cloud` and `sun` are comma-separated RGB values (used in canvas rendering)
4. Theme is applied based on stage number (cycles through themes)

#### Adding a New Enemy Type or Boss
1. Add to `ENEMY_TYPES` (for enemies) or `BOSSES` (for bosses)
2. Enemy requires: `w`, `h` (dimensions), `hp`, `speedMul`, `color`, `dark`, `score`, `minStage`, optional `weave` (boolean for side-to-side motion)
3. Boss requires: `name`, `hullTop`, `hullBottom`, `wing`, `core` (color properties), `hpMul`
4. Boss geometry is drawn procedurally based on these colors; see rendering code for how colors map to visuals

#### Adding a New Sound Effect
1. Add function to `sfx` object (line 624):
   - Call `beep()` for tonal sounds (oscillator-based)
   - Call `noiseBurst()` for percussive/explosion sounds
   - Use `opts` parameter for sweep, filter, pan, delay
2. Register in code where the event occurs (e.g., on enemy defeat, add `sfx.mySound()`)
3. Check `muted` flag before playing (beep/noiseBurst already handle this)

#### Adjusting Game Feel (Screen Shake, Flash Effects)
- Screen shake: Call `screenShake(magnitude, duration)` (affects canvas transform in render)
- Flash effects: Call `flashHit()` or `flashBomb()` to trigger CSS animations on `#flash` element
- Both respect `reducedMotion` setting (lower intensity if enabled)

#### Modifying HUD Elements
- HUD elements are DOM nodes updated via `.textContent` or `.innerHTML` during game loop
- All HUD element references cached at top of script (lines 426–461)
- Update HUD by setting element text or modifying class lists (e.g., `vignetteEl.classList.toggle('on', condition)`)

## Performance & Quality Settings

The game includes adaptive quality scaling to maintain 60 FPS on lower-end devices:
- Tracks frame times over 45 frames
- If average frame time > 1/40s (< 40 FPS) and quality < 2, decreases quality
- If average frame time < 1/56s (> 56 FPS) and quality > 0, increases quality
- Quality level affects entity render detail, particle count, and collision checks

## Storage Keys

Game state persisted in localStorage:
- `skyfighter.bestScore`: All-time high score
- `skyfighter.muted`: Audio mute flag ('1' or '0')
- `skyfighter.difficulty`: Last selected difficulty (easy/normal/hard)
- `skyfighter.reducedMotion`: User's reduced-motion preference ('1' or '0')

## Browser Compatibility

- Requires: ES6 support (arrow functions, const/let, template literals)
- Canvas 2D API for rendering
- Web Audio API for sound (gracefully degrades if unavailable)
- localStorage for persistence
- Touch Events API for mobile input
- CSS animations and filters

## Design Notes

- **Single-file approach**: Keeps deployment simple (one HTTP request), all assets are procedural or inline
- **Procedural audio**: Eliminates external audio asset management, reduces bandwidth, allows dynamic sound tweaking
- **Responsive scaling**: Works on any screen size without distortion
- **Accessibility-first**: Supports keyboard, touch, reduced motion, and screen readers from the ground up
- **No external dependencies**: Runs standalone with no npm packages or CDN resources needed
