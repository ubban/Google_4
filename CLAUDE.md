# Sky Fighter - Codebase Documentation

## Project Overview

**Sky Fighter** is a Raiden-style vertical shoot-'em-up game built with vanilla HTML, CSS, and JavaScript. The entire game is contained in a single `index.html` file (~1670 lines) with embedded styles and game logic.

- **Type**: Web-based browser game
- **Tech Stack**: Vanilla HTML5, Canvas 2D API, Web Audio API
- **Single File**: All code, styles, and game logic in `index.html`
- **No Dependencies**: Completely self-contained, no external libraries or assets

## Game Mechanics

### Core Systems

1. **Player Movement & Controls**
   - Keyboard: Arrow keys / WASD for movement, Space to fire, X/Shift for bomb
   - Mouse: Move to steer, click to fire
   - Touch: Left zone for joystick movement, right zone for fire/bomb buttons
   - Adaptive input: Keyboard/joystick priority over mouse when active

2. **Weapons System**
   - **Vulcan**: Rapid fire bullets with spread pattern based on level
     - Level 1: Single shot center
     - Level 2: Dual spread
     - Level 3: Triple spread
   - **Laser**: Slower but piercing shots, can pass through multiple enemies
     - Level 1: Single lane
     - Level 2: Dual lanes
     - Level 3: Triple lanes
   - Weapon switching: Collect 3 weapon powerups to switch weapons (Vulcan ↔ Laser)

3. **Enemy Types**
   - **Grunt** (Red, appears stage 1+): Basic enemy, 1 HP, score 10
   - **Interceptor** (Orange, appears stage 1+): Fast with weaving pattern, 1 HP, score 15
   - **Tank** (Purple, appears stage 2+): Slow, 4 HP, score 35, displays HP bar

4. **Boss System**
   - One boss per stage (6 boss designs, cycling)
   - Boss names: VANGUARD, BEHEMOTH, SPECTER, DREADNOUGHT, WRAITH, TITAN
   - Each boss has 3-4 unique attack patterns (radial bursts, aimed spreads, homing, etc.)
   - Boss HP scales with stage and difficulty
   - Telegraph animation before each attack

5. **Scoring & Multiplier**
   - Base kill scores: Grunt 10, Interceptor 15, Tank 35, Enemy 5 (bomb), Boss 500
   - Kill chain multiplier: Increases for consecutive kills without pause (~2.2s timeout)
   - Max multiplier: 10x (at 50+ consecutive kills)
   - Multiplier breaks on enemy bullet hit or taking damage

6. **Power-ups** (spawn on enemy death)
   - **Life** (red): Restore 1 life (max 3-4 depending on difficulty)
   - **Bomb** (yellow): Add 1 bomb (max 2-3 depending on difficulty)
   - **Weapon** (green): Upgrade weapon level or switch weapons

7. **Difficulty Levels**
   - **Easy**: 4 lives, 3 bombs, slower enemies, lower HP bosses
   - **Normal**: 3 lives, 2 bombs, medium enemies, standard bosses
   - **Hard**: 3 lives, 2 bombs, fast enemies, tanky bosses
   - Settings persist in localStorage

### Game State

Key variables (line 864-869):
- `player`: Position, size, weapon, level, cooldown, invulnerability timer
- `bullets`, `enemies`, `enemyBullets`, `powerUps`, `particles`: Entity arrays
- `score`, `lives`, `bombCount`, `stage`, `multiplier`, `chain`
- `running`, `paused`, `gameOver`: Game flow flags

## Code Structure

### HTML Structure (line 362-419)
- Game canvas container
- HUD (score, best, multiplier, weapon, hearts, bombs)
- Boss health bar (shown when boss active)
- Status overlays (combo pop, stage banner, boss warning)
- Top bar (mute, motion reduction, pause buttons)
- Touch controls (joystick base, fire/bomb buttons)
- Main overlay (title screen, game over, pause menu)

### JavaScript Organization

#### Constants & Configuration (line 463-515)
- `STORAGE_*`: localStorage keys for persistence
- `DIFFICULTIES`: Difficulty parameters (spawn rates, speeds, lives, bombs, HP multipliers)
- `KILLS_PER_STAGE`: 18 kills triggers boss spawn
- `THEMES`: 6 sky visual themes with gradients and cloud colors
- `ENEMY_TYPES`: Grunt, Interceptor, Tank definitions
- `BOSSES`: 6 boss designs with color palettes and HP multipliers
- `BOSS_PATTERNS`: Attack pattern functions for each boss

#### Audio System (line 542-676)
- Web Audio synthesis (no external audio files)
- `beep()`: Oscillator-based tones with optional frequency sweep and filter
- `noiseBurst()`: Procedural noise for impact sounds
- `sfx` object: 13 sound effect generators (shoot, laser, various explosions, bomb, powerup, UI clicks, boss sounds, game over)
- Stereo panning based on screen position
- Muted state toggleable and persistent

#### Input Handling (line 767-861)
- Keyboard event listeners for arrow keys, WASD, Space, P, Esc, X, Shift
- Touch controls with virtual joystick:
  - `joystickStart/Move/End()`: Track touch position and radius
  - `moveZone`: Left 55% of screen for movement
  - `actionZone`: Right 45% for fire/bomb buttons
- Mouse controls:
  - `mouseActive`: Detected on canvas hover
  - `mouseTarget`: Smooth player glide toward cursor
  - `firing`: Set by click

#### Game Logic (line 875-1490)
- `resetGame()`: Initialize new game state (line 875-911)
- `update(dt)`: Per-frame game state update (line 1148-1302)
  - Player movement and collision bounds
  - Weapon firing and cooldown
  - Enemy spawning and AI
  - Boss behavior
  - Collision detection (player-enemy, bullets-enemy, player-enemy bullets, boss hits)
  - Score and multiplier management
- `fireWeapon()`: Emit bullets based on weapon type/level (line 1124-1146)
- `spawnEnemy()`: Spawn random enemy type (line 1022-1041)
- `spawnBoss()`: Spawn boss for current stage (line 1043-1056)
- `updateBoss(dt)`: Boss movement, attack patterns, telegraph (line 1411-1439)
- `killBoss()`: Handle boss defeat, advance stage (line 1441-1454)
- `useBomb()`: Clear all enemies/bullets, damage boss (line 1103-1122)
- `addScore(base)`: Add score and update multiplier (line 1087-1094)
- `breakChain()`: Reset kill multiplier (line 1096-1101)
- `endGame()`: Game over state (line 1469-1489)
- `togglePause()`: Pause/resume gameplay (line 1491-1504)

#### Rendering (line 1507-1638)
- `draw()`: Per-frame canvas rendering
  - Sky gradient with theme
  - Sun glow effect
  - Animated clouds (2 layers, different speeds)
  - Player ship (with laser aura if applicable)
  - Mouse cursor crosshair
  - Bullets with glow effects
  - Enemy bullets as circles
  - Enemies with optional health bar (tanks)
  - Boss ship with animated core and wings
  - Power-ups as rotating squares
  - Particles (explosions)
  - Screen shake effect applied to entire canvas
- `drawPlane(x, y, w, h, color, darkColor, flame)`: Draw aircraft shape (line 913-968)
- `drawBossShip(b)`: Draw boss with design-specific colors (line 970-1020)
- `drawClouds(speed, color, rx, ry)`: Draw parallax clouds (line 1507-1518)

#### Visual Effects
- `flashHit()`, `flashBomb()`: White flash animation overlay
- `showComboPop(text)`: Floating combo counter animation
- `showStageBanner(text)`: Stage intro text animation
- `screenShake(mag, dur)`: Camera shake effect (reduced in reduced-motion mode)
- `renderHearts()`, `renderBombs()`: HUD heart/bomb indicators
- Particle system for explosions

#### Utility Functions
- `rectsOverlap(a, b)`: AABB collision detection (line 1069-1071)
- `spawnExplosion(x, y, color, count)`: Create explosion particles (line 1073-1085)
- `applyPowerUp(type)`: Handle powerup collection (line 1304-1322)
- `panFor(x)`: Calculate stereo pan value from screen position (line 562-564)

#### Performance & Optimization
- `quality` variable (0=full, 1=medium, 2=low): Adaptive graphics quality
- `trackPerf(dt)`: Monitor average frame time, adjust quality (line 476-483)
- Particle count scales with quality setting
- Shadow blur and glow effects disabled on lower quality
- `requestAnimationFrame()` for smooth 60 FPS timing

#### Accessibility & UX
- Screen reader support via `#srLive` (ARIA live region)
- `announce(text)` function for status updates
- Reduced motion preference detection (`prefers-reduced-motion`)
- Toggle for reduced motion in UI, disables shake/flash/animations
- Touch-friendly button sizes
- High contrast colors
- Mute button for audio toggle

## Visual Design

### Canvas Resolution
- Base: 480x720 (portrait)
- Responsive scaling: Up to 1.6x with aspect ratio preservation
- Safe area insets for notched devices

### Color Themes (6 rotating)
1. Daylight Front: Blue sky, white clouds, golden sun
2. Sunset Corridor: Purple sky, pink clouds, orange sun
3. Night Approach: Dark blue sky, light blue clouds, muted blue sun
4. Storm Front: Green storm, green clouds, green sun
5. Arctic Wastes: Light blue sky, cool clouds, cool sun
6. Volcanic Ridge: Dark red sky, orange clouds, orange sun

### Enemy Design
- Procedurally drawn with `drawPlane()` function
- Different colors per type (red grunt, orange interceptor, purple tank)
- Simple geometric shapes (triangles for wings, ellipse for hull)

### Boss Design
- 6 unique designs (VANGUARD, BEHEMOTH, SPECTER, DREADNOUGHT, WRAITH, TITAN)
- Each with distinct color schemes and glowing cores
- Animated wings that rotate
- Telegraph glow effect before attacks

## Key Implementation Details

### Collision Detection
- AABB (axis-aligned bounding box) using `rectsOverlap()`
- Player-enemy collision: Damages player
- Enemy bullets-player collision: Damages player (unless invulnerable)
- Player bullets-enemy collision: Damages enemy
- Player bullets-boss collision: 3 damage per hit
- Bomb effect: Destroys all on-screen enemies/bullets, damages boss 24 HP

### Enemy Behavior
- Linear downward movement with optional weaving
- Two firing patterns: `'spread'` (three bullets outward) or `'aimed'` (single toward player)
- Spawn timer scales with frame count (difficulty ramp)
- Enemies removed when off-screen below

### Boss Attack Patterns
- Each boss has 3 unique patterns
- Pattern functions use helper functions:
  - `fireRadial(b, count, speed)`: Circular spread
  - `fireAimedSpread(b, count, speed, spreadStep)`: Fan toward player
  - `fireColumns(b, speed)`: Vertical columns
  - `fireSpiralBurst(b, speed)`: Rotating spiral
  - `fireCross(b, speed)`: 8-way cross
  - `fireHomingSeeded(b, count, speed)`: Seeded homing projectiles
  - `fireDoubleRing(b, speedA, speedB)`: Two concentric rings
- Telegraph timer (22 frames) before executing pattern
- Attack timer (85 frames) between patterns

### Invulnerability & Visual Feedback
- Player blinks (invisible every 4th frame) when invulnerable (1.5 seconds after damage)
- Red vignette pulse effect when at 1 HP (critical health)
- Heart animation pulse in HUD when critical
- Flash overlay on damage and bombs

### Score Persistence
- Best score saved to localStorage
- Difficulty setting saved
- Audio mute state saved
- Reduced motion preference saved
- Keys: `skyfighter.bestScore`, `skyfighter.muted`, `skyfighter.difficulty`, `skyfighter.reducedMotion`

## Common Tasks

### Adding a New Enemy Type
1. Add entry to `ENEMY_TYPES` object (line 502-506)
2. Define: `w`, `h`, `hp`, `speedMul`, `color`, `dark`, `score`, `minStage`, optional `weave`
3. Update `spawnEnemy()` filter logic if needed (line 1026)
4. Add explosion SFX mapping if custom sound needed (line 678)
5. Optional: Add `EXPLODE_SFX` entry for custom explosion sound

### Adding a New Boss
1. Add entry to `BOSSES` array (line 508-515) with colors and HP multiplier
2. Add 3-4 attack pattern functions to `BOSS_PATTERNS` array (line 1372-1409)
3. Boss index maps automatically to BOSSES index (line 1045)

### Adding a Sound Effect
1. Add function to `sfx` object (line 624-676)
2. Use `beep()` for tones or `noiseBurst()` for noise
3. Call `ensureAudio()` before playing if user hasn't interacted
4. Use `panFor(x)` to calculate stereo pan

### Modifying Difficulty
1. Edit `DIFFICULTIES` constants (line 485-489)
2. Adjust: `spawnBase`, `spawnMin`, `enemySpeed`, `enemyShootChance`, `lives`, `bombs`, `bossHpMul`

### Changing Visual Theme
1. Add new theme to `THEMES` array (line 493-500)
2. Specify: `sky` (3-stop gradient), `cloud` (RGB string), `sun` (RGB string), `name`
3. Themes cycle based on stage number

## Browser Compatibility

- **Requires**: Modern browser with Canvas 2D, Web Audio API, ES6 support
- **Audio Context**: Falls back to `webkitAudioContext` on some browsers
- **Touch**: Detected via `ontouchstart` or `navigator.maxTouchPoints`
- **Storage**: localStorage for persistence (graceful fallback if unavailable)
- **Tested on**: Chrome, Firefox, Safari, Edge (desktop and mobile)

## Performance Considerations

- Single requestAnimationFrame loop for consistent timing
- Adaptive quality system prevents performance degradation
- Particle count caps based on quality level
- Enemy/bullet cleanup removes off-screen entities
- Canvas context state saved/restored to prevent state leaks
- No image decoding or external asset loading (all procedural)

## Future Enhancement Ideas

- Leaderboard system (with server storage)
- More boss patterns or procedural pattern generation
- In-game item shop for upgrades
- Score multiplier visuals/animations improvements
- Mobile vibration feedback (Vibration API)
- Save/load game progress
- Custom difficulty builder
- Procedural enemy generation
- Background music (currently SFX only)
- Particle effect customization per theme
