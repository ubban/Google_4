# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project Overview

**Sky Fighter** is a Raiden-style vertical shoot-'em-up game built as a single-file HTML application. The entire game (CSS, JavaScript, HTML) is contained in `index.html` (~1670 lines). No build process, bundler, or external dependencies are required.

### Quick Start

1. Open `index.html` in any modern web browser to play the game
2. The game runs at 480×720px, scales responsively to screen size
3. Game state persists to localStorage (best score, mute status, difficulty, reduced-motion preference)

### Controls

- **Desktop**: Mouse (move to steer), click to fire, Space to fire, X/Shift to bomb, P/Esc to pause
- **Mobile**: Drag left side to move, tap FIRE/BOMB buttons, touch controls enabled automatically

## Architecture

The entire game logic is in a single IIFE (Immediately Invoked Function Expression) starting at line 423. All variables are scoped locally to this function.

### Key Game Systems

**State Management**
- Game state (running, paused, gameOver) and player object live as module-scoped variables
- Arrays track enemies, bullets, enemy bullets, particles, and power-ups
- LocalStorage saves: best score, mute status, difficulty, reduced-motion preference

**Game Loop** (line 1640)
- `requestAnimationFrame(loop)` drives updates and rendering
- Updates call `update(dt)` for logic, `draw()` for rendering
- Performance is tracked and quality scales down on low frame rates (line 473–483)

**Player & Weapon System**
- Player at `player` object with position, speed, weapon type, level, cooldown, invulnerability
- Two weapons: `vulcan` (spread shot) and `laser` (piercing beam)
- Weapon fire handled in `fireWeapon()` (line 1124)
- Weapon levels unlock spread patterns and additional lanes

**Enemy System**
- Three enemy types: `grunt`, `interceptor`, `tank` (defined in `ENEMY_TYPES`, line 502–506)
- Enemy AI: movement (weaving for interceptors), shooting patterns (spread or aimed), difficulty-based spawning
- Enemies spawn continuously until KILLS_PER_STAGE (18) reached, then boss spawns after delay
- Each enemy type has HP, speed multiplier, color, and score value

**Boss System**
- Six bosses cycle through stages (line 508–515: VANGUARD, BEHEMOTH, SPECTER, DREADNOUGHT, WRAITH, TITAN)
- Boss patterns defined in `BOSS_PATTERNS` array (line 1372–1409) — 3 patterns per boss
- Boss enters from top, moves side-to-side with telegraph phase before firing patterns
- Telegraph visually white + glowing, shows 22 frames before pattern fires
- Boss HP scales with stage and difficulty, can be damaged by bullets or bombs

**Difficulty System** (line 485–489)
- Three presets: easy, normal, hard
- Affects spawn rate, enemy speed, enemy shoot chance, lives, bombs, boss HP multiplier
- Changing difficulty saved to localStorage, only allowed on menu or pause screen

**Collision Detection**
- Simple AABB (axis-aligned bounding box) with `rectsOverlap()` (line 1069–1071)
- Bullets vs enemies, bullets vs boss, enemies vs player, bombs vs everything (bomb clears screen)
- Enemy bullets vs player checked every frame

**Visual Effects**
- Canvas rendering: gradient backgrounds, procedurally drawn planes (player/enemies) and bosses
- Particle system for explosions (line 1073–1085)
- Screen shake effect (line 726–730, applies to canvas context)
- Flash overlay for hits/bombs (line 98–104 CSS animations)
- Combo pop and stage banner displays (line 711–724)
- Reduced-motion mode disables animations when `prefers-reduced-motion` is set or toggled

**Audio (WebAudio Synthesis)**
- No external audio files — all sounds are synthesized
- `ensureAudio()` initializes AudioContext on first user interaction (line 545–561)
- Beep, noise burst, and swept-frequency sounds for different events
- Sounds muted via `muted` flag, stored in localStorage
- Spatial audio with panning based on X position (line 562–564)

**Input Handling**
- Keyboard: tracks pressed keys in `keys` object, checked in update
- Mouse: hover-to-steer on canvas, click fires
- Touch: joystick in bottom-left, fire/bomb buttons in bottom-right
- Joystick logic (line 785–826): tracks touch origin, constrains stick to 50px radius

**Score & Chain System**
- Score multiplier increases with consecutive kills: `1 + floor(chain / 5)` (line 1090)
- Chain resets on miss (no kill for 2.2 seconds)
- Bonus pop-up every 10 kills

**Themes**
- Six sky themes (line 493–500) cycle per stage: gradient, clouds, sun color
- Each theme affects visual mood but not gameplay

### Performance Optimization

Quality scaling (line 474–483) adjusts based on frame time:
- **Quality 0** (full): full particle counts, glow effects, shadows
- **Quality 1** (medium): 60% particle counts, reduced glow
- **Quality 2** (low): 35% particle counts, no glow

Tracks rolling average of last 45 frame times; scales down if avg > 1/40s, up if < 1/56s.

### Canvas & Responsiveness

- Base canvas is 480×720px (line 752–765)
- Scales up to 1.6× on large screens, maintains aspect ratio
- `resize()` called on window resize and orientation change

## Development Guidelines

### Modifying Gameplay

**Add a New Enemy Type**
1. Define in `ENEMY_TYPES` (line 502–506): set w, h, hp, speedMul, color, dark, score, minStage, optionally weave
2. Update `spawnEnemy()` to include in random selection (line 1024–1040)
3. Add explosion sound mapping in `EXPLODE_SFX` (line 678) if needed

**Add a New Boss**
1. Add to `BOSSES` array (line 508–515): name, hull colors, wing color, core color, hpMul
2. Add 3 patterns to `BOSS_PATTERNS` array (line 1372–1409) — each is a function taking boss object, fires bullets via helper functions
3. Helper functions available: `fireRadial()`, `fireAimedSpread()`, `fireColumns()`, `fireSpiralBurst()`, `fireCross()`, `fireHomingSeeded()`, `fireDoubleRing()` (line 1324–1370)

**Adjust Difficulty**
1. Edit `DIFFICULTIES` object (line 485–489): spawnBase, spawnMin, enemySpeed, enemyShootChance, lives, bombs, bossHpMul
2. Ramp difficulty in `update()` at line 1201: `Math.floor(frame / 300) * 4` is the spawn acceleration

**Change Weapon Behavior**
1. Vulcan: edit spread array at line 1127
2. Laser: edit lanes array at line 1136
3. Bullet speed/size at line 1130, 1140
4. Fire rate: `fireDelay` at line 1177

### Modifying Audio

All sounds are in `sfx` object (line 624–676). Each uses `beep()` or `noiseBurst()`:
- `beep(freq, dur, type, vol, opts)` (line 566): freq (Hz), dur (seconds), type ('sine'/'square'/'triangle'/'sawtooth'), vol (0–1)
- Options: sweepTo, filterType, filterFreq, pan, delay
- `noiseBurst(dur, vol, opts)` (line 597): white noise with optional filter and pan

Example: to make explosions lower-pitched, reduce the first frequency in `explodeGrunt()` (line 630).

### Modifying Visuals

**Ship Appearance** — `drawPlane()` (line 913) draws player and enemy aircraft. Edit fill colors or path geometry to redesign.

**Boss Appearance** — `drawBossShip()` (line 970) draws boss. Modifies ellipse/arc sizes, colors from boss.design, and rotation for thruster animation.

**Particle Effects** — `spawnExplosion()` (line 1073) controls count. Particle draw is in `draw()` around line 1628.

**UI Colors & Text** — Edit CSS in `<style>` block (line 10–359). Colors are hardcoded hex values; animate via `@keyframes` or `transition`.

**Themes** — Add to `THEMES` array (line 493): sky gradient colors (RGB), cloud color, sun color. Stage colors cycle through themes.

### Modifying UI

**Start Screen** — Edit `#overlayCard` content in HTML (line 402–419). Difficulty buttons are auto-wired to `setDifficulty()`.

**HUD Display** — Elements in `#hud` (line 367–374), `#bossBarWrap` (line 376–379), `#topBar` (line 384–388). JS updates textContent and classList.

**Game Over Screen** — Toggle visibility in `endGame()` (line 1469). Title, subtitle, stats all set via JS.

### Common Tasks

**Increase enemy spawn rate**
- Lower `spawnBase` and `spawnMin` in DIFFICULTIES

**Make bosses harder**
- Increase `hpMul` in DIFFICULTIES or individual boss `hpMul`
- Add more bullets to boss patterns

**Add a new power-up type**
1. Add spawn case to `maybeSpawnPowerUp()` (line 1058–1067)
2. Add type name to powerUp object (type: 'mytype')
3. Add case to `applyPowerUp()` (line 1304–1322) to apply effect

**Change game dimensions**
- Edit `BASE_W`, `BASE_H` (line 753)
- Affects all draw calls and collision — no scaling needed if changed proportionally

## No Build Process

This project has no build step, bundler, or transpiler. The single HTML file serves directly. Testing is done by opening in a browser.

**To test changes:**
1. Edit `index.html` in any text editor
2. Refresh the browser (Ctrl+R or Cmd+R)
3. LocalStorage persists state; clear it in DevTools if needed (Application tab → Local Storage → Clear)

**Performance testing:**
- Open DevTools (F12) → Performance tab
- Record a few seconds of gameplay
- Check FPS graph; quality auto-adjusts if dropping below 40 FPS

## Files

- `index.html` — Complete game (HTML + CSS + JavaScript)
- `README.md` — Project title and timestamp
- `CLAUDE.md` — This file
