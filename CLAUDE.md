# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project Overview

**Sky Fighter** is a Raiden-style vertical shoot-'em-up game built as a single HTML5 file. It features:
- Canvas-based 2D rendering
- Wave-based enemy spawning with difficulty scaling
- Boss encounters with distinct attack patterns
- Weapon upgrade system (Vulcan/Laser, levels 1-3)
- Smart bomb mechanic for area damage
- WebAudio synthesizer for procedural sound (no external assets)
- Responsive design with keyboard/mouse and touch controls
- Accessibility features and reduced motion support
- Persistent high score storage via localStorage

## Code Architecture

All game logic lives in `index.html` (1669 lines, ~307 functions/classes). The structure is:

### 1. **Game State Variables** (lines 463-524)
- `bestScore`, `muted`, `difficulty`, `reducedMotion` — persisted settings
- `player` — player ship object with position, speed, weapon, level, health
- `enemies`, `boss` — arrays of active enemy and boss objects
- `bullets`, `enemyBullets` — player and enemy projectiles
- `powerUps`, `particles` — pickups and explosion effects
- Difficulty settings define enemy spawn rates, speeds, health pools, and lives

### 2. **Render Systems**
- **Canvas rendering loop** (lines 1400+) — clears, draws sky gradient, all entities, HUD
- **drawPlane(x, y, w, h, color, darkColor, flame)** (line 913) — draws player/enemy ships with shading and flame
- **drawBossShip(b)** (line 970) — boss-specific rendering with hull, wings, core glow
- **Particle system** — small colored dots spawn on impact/explosion, fade over time
- **Screen shake & flash effects** — CSS animations triggered by collisions/bombs (lines 698, 704, 727)

### 3. **Game Loop** (line 1400+)
- **update(dt)** — called per frame, handles:
  - Player movement (keyboard, mouse, joystick input)
  - Weapon firing and cooldown
  - Bullet physics (movement, off-screen culling)
  - Enemy spawning with difficulty ramp-up
  - Collision detection (enemy-bullet, player-enemy, player-bullet, player-boss)
  - Power-up pickup logic
  - Boss state updates
- **render()** — draws all entities to canvas

### 4. **Physics & Collision**
- **rectsOverlap(a, b)** (line 1069) — axis-aligned bounding box collision; all entities have x, y, w, h
- Enemies move vertically (y += speed * dt * 60); some weave horizontally
- Bullets move in straight lines; lasers pierce enemies if level ≥ 3
- Boss collision uses same AABB, triggers player damage/knockback

### 5. **Weapon System**
- **Vulcan** — spreads based on level; level 3 has 3 bullets at angles [-0.24, 0, 0.24]
- **Laser** — pierces and may hit multiple enemies; level 3 fires 3 lanes
- Switching weapons resets to level 1
- Weapon level increases on pickup; at level 3, next pickup switches weapon type
- Firing cooldown: Vulcan 0.15s, Laser 0.1s (line 1177)

### 6. **Enemy Types** (ENEMY_TYPES, line 502)
- **Grunt** — 34×34, 1 HP, red, basic spray-shoot pattern, 10 points
- **Interceptor** — 26×26, 1 HP, orange, 1.7× speed, weaves, 15 points
- **Tank** — 46×46, 4 HP, purple, 0.55× speed, 35 points; min stage 2
- Enemies spawn at random x, top of screen; each has cooldown for shooting

### 7. **Boss System**
- Six distinct boss types (BOSSES, line 508): VANGUARD, BEHEMOTH, SPECTER, DREADNOUGHT, WRAITH, TITAN
- Each boss has unique color scheme, HP multiplier, and attack pattern
- Spawns after 18 kills per stage; enters with animation
- Fires in patterns: radial bursts, aimed spreads, columns, spirals
- Boss damage: player bullets deal 3 HP each; bombs deal 24 HP
- **updateBoss(dt)** — handles entry animation, attack cycling, exit on death

### 8. **Score & Progression**
- **addScore(base)** (line 1087) — multiplies by combo multiplier (max 5×)
- Combo multiplier increments on kill chain (reset on time-out or damage taken)
- Multiplier decreases on player hit (line 1096, **breakChain**)
- Milestone feedback: "10 KILLS!", "20 KILLS!", etc. display on screen

### 9. **Bomb Mechanic** (**useBomb**, line 1103)
- Clears all enemy bullets from screen
- Damages all enemies (instant kill) and boss (24 HP)
- Spawns orange explosion particles at each enemy
- Triggers full screen flash, camera shake, bomb SFX
- Capped at 2–3 depending on difficulty

### 10. **Audio (WebAudio Synth)**
- **ensureAudio()** (line 545) — lazy-init AudioContext on first interaction
- **beep(freq, dur, type, vol, opts)** — creates oscillator-based sound with optional sweep, filter, pan
- **noiseBurst(dur, vol, opts)** — noise generator for punch effects
- **sfx object** (line 624) — pre-defined sounds: shoot, laser, powerup, bomb, explode, bossHit, etc.
- No external audio files; all synthesized on-the-fly

### 11. **Difficulty Modes** (DIFFICULTIES, line 485)
- **Easy** — slower spawns, slower enemies, 4 lives, lower boss HP
- **Normal** — balanced (default)
- **Hard** — faster spawns, faster enemies, higher boss HP, 3 lives
- Difficulty can be changed before each game; setting persisted

### 12. **Themes** (THEMES, line 493)
- Six visual themes (Daylight, Sunset, Night, Storm, Arctic, Volcanic)
- Randomly chosen per stage; define sky gradient and cloud/sun colors

### 13. **Input Handling**
- **Keyboard** — Arrow keys / WASD for movement, Space to fire, X or Shift to bomb, P to pause
- **Mouse** — move to steer, click to fire
- **Touch** — left half of screen for movement (with joystick visual), right half has FIRE and BOMB buttons
- Mobile: joystick base appears on first touch; buttons are circular overlays

### 14. **Accessibility & Settings**
- **Reduced motion** — disables animations on body.reduced-motion (CSS @media)
- **Screen reader** — announce() updates #srLive for live region (line 468)
- **Mute button** — toggles audio off
- **Motion toggle** — toggles screen shake, flash, animations
- Settings stored in localStorage (STORAGE_* keys)

### 15. **Render Pipeline**
1. Clear canvas with sky gradient (theme-dependent)
2. Draw clouds (scrolling parallax, line 1420+)
3. Draw all particles
4. Draw enemy bullets
5. Draw enemies
6. Draw player bullets
7. Draw power-ups
8. Draw boss (if active)
9. Draw player ship
10. Update HUD (score, lives, bombs, weapon level, boss bar)

## Common Development Tasks

### Running the Game
- Open `index.html` in a modern browser (Chrome, Firefox, Safari, Edge)
- Game initializes with overlay menu; select difficulty and click "Start Game"
- No build step or server required; it's a standalone single-file game

### Adding a New Enemy Type
1. Add entry to `ENEMY_TYPES` object (line 502) with w, h, hp, speedMul, color, dark, score, minStage
2. Update **spawnEnemy()** (line 1022) to select from new type based on stage
3. Add explosion SFX entry in `EXPLODE_SFX` object (line 678) if desired
4. Collision and rendering use generic enemy properties; no additional code needed

### Adding a New Boss Attack Pattern
1. Define a new pattern function (e.g., `firePattern_newAttack(b)`, lines 1324+)
2. Add to **updateBoss()** (line 1356) switch statement: add case for new pattern index
3. Pattern functions call helpers like fireRadial, fireAimedSpread, fireColumns, fireSpiralBurst
4. Each fires enemyBullets with x, y, w, h, radial, dx, dy (velocity)

### Tweaking Difficulty or Spawn Rates
- Edit **DIFFICULTIES** object (line 485): spawnBase, spawnMin, enemySpeed, enemyShootChance, lives, bombs, bossHpMul
- Edit **KILLS_PER_STAGE** (line 491) to change when boss spawns
- Spawn timer ramps down over time (line 1201); increase/decrease difficultyRamp factor

### Adjusting Weapon Balance
- **Vulcan** bullets: line 1129-1132 (spread, speed, pierce)
- **Laser** bullets: line 1139-1142 (lanes, spread, pierce)
- Fire delay: line 1177 (0.1s laser, 0.15s vulcan)
- Adjust damage multipliers in collision code (line 1259 for boss, 1237 for enemies)

### Adding New Sound Effects
- Define a function in **sfx** object (line 624) using **beep()** or **noiseBurst()**
- Call **ensureAudio()** on first use to initialize AudioContext
- Use opts: { sweepTo, filterFreq, filterType, pan, delay } for modulation
- Example: `sfx.newEffect = () => beep(440, 0.2, 'sine', 0.1, { sweepTo: 220 });`

### Responsive Design & Touch
- Canvas resizes on window resize via **resize()** (line 754)
- Touch controls scale with viewport; buttons are 78×78 on mobile
- Joystick appears at first touch in move zone; visual feedback via opacity
- Test on various screen sizes; CSS uses `clamp()` for scalable text/spacing

## Key Invariants & Patterns

- **Delta time (dt)**: All physics use dt for frame-rate independence; update() receives seconds/frame
- **Coordinate system**: (0,0) at top-left; y increases downward (canvas default)
- **Entity structure**: All dynamic objects (player, enemies, bullets, boss) have x, y, w, h for collision
- **Game states**: running (boolean), gameOver (boolean), paused (boolean), boss (object or null)
- **Stages**: incremented on boss defeat; determines which enemy types spawn, theme rotation, difficulty ramp
- **Kill counter (frame)**: incremented every update; used for difficulty ramp and visual cues
- **Particle pool**: particles array reused; old particles simply fade (life -= dt) and are filtered out
- **Screen shake**: applies translation to canvas; disabled if reducedMotion is true

## Debugging Tips

- Check console for audio errors (AudioContext not supported on some browsers)
- Use localStorage to inspect saved state: `localStorage.getItem('skyfighter.bestScore')`
- Frame rate drops often correlate with particle count; check **trackPerf()** (line 476) for adaptive quality
- Test touch controls on actual mobile device or browser DevTools device mode
- Collision debug: temporarily draw red rects around all AABB entities to verify hit detection
- Boss pattern timing: adjust phase durations in **updateBoss()** cycle logic (line 1356+)

## File Size & Performance

- Single 59 KB HTML file (all CSS, JS, HTML bundled)
- No external dependencies or assets; all graphics are canvas-drawn, sounds synthesized
- Targets 60 FPS on mid-range devices; adaptive quality degrades rendering if frame time exceeds 25ms
- Touch controls hidden on non-touch devices; keyboard/mouse is primary input on desktop
