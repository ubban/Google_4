# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project Overview

**Sky Fighter** is a Raiden-style vertical shoot-'em-up game implemented as a single-file HTML5 application. No build process, no external assets, and no dependencies—just open `index.html` in a browser to play.

## Key Architecture

### Single-File Structure
- All code (HTML, CSS, JavaScript) is embedded in `index.html`
- CSS is in `<style>` tags; JavaScript is in a `<script>` IIFE (Immediately Invoked Function Expression)
- This design makes the game portable and instantly playable

### Core Game Loop
The game runs via `requestAnimationFrame()`, calling `loop(now)` at ~60fps:
1. **Timing**: `loop()` calculates `dt` (delta time in seconds) capped at 50ms
2. **Update**: If running and not paused, `update(dt)` advances game state
3. **Render**: `draw()` clears canvas and renders all entities
4. **Performance tracking**: `trackPerf()` monitors frame times and adjusts `quality` level (0=full, 1=medium, 2=low) to maintain performance

### State Management

**Game State** (global):
- `player`: Position, size, weapon, level, invulnerability timer
- `bullets`, `enemies`, `enemyBullets`, `particles`, `powerUps`: Arrays of entities
- `boss`: Boss ship (null when no boss active)
- `score`, `lives`, `stage`, `chain` (combo), `multiplier`: Gameplay progress
- `running`, `paused`, `gameOver`: Game state flags

**Persistence** (localStorage):
- `skyfighter.bestScore`: High score
- `skyfighter.muted`: Audio mute state
- `skyfighter.difficulty`: Selected difficulty
- `skyfighter.reducedMotion`: Motion accessibility setting

### Rendering Pipeline
- **Sky**: Theme-based gradient (6 themes rotate per stage)
- **Clouds**: Parallax scrolling, performance-scaled
- **Entities**: Player, enemies, boss, bullets, power-ups, particles
- **UI Overlays**: HUD (score, health, weapon), boss health bar, combo popups, pause/game-over screens

### Input Handling
- **Keyboard**: Arrow keys or WASD for movement, Space to fire, X/Shift for bombs, P/Esc to pause
- **Mouse**: Pointer position steers player (when canvas is focused); click to fire
- **Touch**: Left 55% of screen is movement joystick; right 45% has FIRE and BOMB buttons
- **Priority**: Keyboard/joystick input takes priority over mouse steering to prevent fighting

### Collision Detection
Uses axis-aligned bounding box overlap: `rectsOverlap(a, b)` checks if two rectangles overlap. All hit detection (player-enemy, bullets-enemy, bullets-boss, player-powerup) uses this.

### Audio System (Web Audio API)
Synthesized sound effects—no audio files:
- **Oscillators** (`beep()`): Square, sawtooth, sine waves with frequency sweeps and filters
- **Noise bursts** (`noiseBurst()`): White noise for explosions, impacts
- **Effects**: Panning (spatial audio), filtering (lowpass/highpass), dynamic compression
- `ensureAudio()` initializes AudioContext on first user interaction (browser policy)

### Difficulty System
Each difficulty (easy, normal, hard) defines:
- Enemy spawn rates and movement speed
- Enemy fire frequency and accuracy
- Starting lives and bombs
- Boss HP multiplier

### Boss System
- **6 bosses** with unique designs (VANGUARD, BEHEMOTH, SPECTER, DREADNOUGHT, WRAITH, TITAN)
- **Pattern-based attacks**: Each boss has 3 attack patterns in `BOSS_PATTERNS[]`
- **Telegraph mechanic**: 22 frames of warning before each attack (visual + audio)
- **Spawn trigger**: After 18 kills per stage

### Weapon System
Two weapons, each with 3 levels:
- **Vulcan**: Straight shots, spread pattern at level 3
- **Laser**: Piercing shots with screen glow, multi-lane fire at level 2+
- **Switching**: Collect 4 weapon pickups to upgrade; at level 3, collecting again switches weapons and resets to level 1

## Quick Start

1. **Run**: Open `index.html` in a modern browser (Chrome, Firefox, Safari, Edge)
2. **Play**: Select difficulty, press Start, use keyboard/mouse/touch to play
3. **Develop**: Edit `index.html` in any text editor, save, and refresh browser

## Development Guidelines

### Modifying Game Constants

- **Difficulty tuning**: Edit `DIFFICULTIES` object (lines 485–489)
  - `spawnBase`/`spawnMin`: Frame intervals for enemy spawning
  - `enemySpeed`, `enemyShootChance`: Enemy aggression
  - `lives`, `bombs`, `bossHpMul`: Balance knobs
- **Enemy types**: Edit `ENEMY_TYPES` (lines 502–506)
  - `w`, `h`: Sprite size
  - `hp`, `speedMul`, `score`: Balance
  - `minStage`: First stage this enemy appears
- **Bosses**: Edit `BOSSES` (lines 508–515); adjust HP with `hpMul`
- **Attack patterns**: Edit `BOSS_PATTERNS` (lines 1372–1409) for boss behavior

### Visual Polish

- **Canvas size**: `BASE_W = 480, BASE_H = 720` (lines 753–754); scales up to fit screen
- **Theme colors**: `THEMES` array (lines 493–500) defines sky, cloud, and sun colors per stage
- **Animations**: Most animations use `frame` counter (incremented each update). Keyframe animations in CSS use `@keyframes`
- **Particle effects**: `spawnExplosion()` creates temporary particles; adjust count for intensity

### Performance Tuning

- **Quality levels**: Controlled by `quality` variable (0=full effects, 2=low)
  - Reduced shadow blur, fewer particles, lower cloud density at lower quality
  - Automatically adjusted based on frame time; disable this by removing `trackPerf()` calls if needed
- **Reduced motion**: Respected throughout (check `reducedMotion` flag before animations)

### Adding Features

**New enemy type**:
1. Add entry to `ENEMY_TYPES` with size, HP, speed, score
2. Set `minStage` to control availability
3. Add to `spawnEnemy()` selection logic
4. Create explosion sound in `sfx` object and add to `EXPLODE_SFX` mapping

**New power-up type**:
1. Add spawn chance in `maybeSpawnPowerUp()` (line 1058)
2. Handle in `applyPowerUp()` (line 1304) and `draw()` rendering (line 1621)

**New boss pattern**:
1. Add pattern functions (e.g., `fireRadial()`, `fireAimedSpread()`) returning nothing; they mutate `enemyBullets` directly
2. Add array of patterns to `BOSS_PATTERNS`
3. Adjust `BOSSES` HP multiplier if needed for balance

### Accessibility

- **Reduced motion**: Disabled animations, screen shake, flash effects when flag is set
- **Screen reader support**: `srLive` element announces key events (kills, stage, game over)
- **Color contrast**: All text uses dark backgrounds or text shadows for legibility
- **Touch controls**: Automatically enabled on devices with `ontouchstart` support

## File Structure (all in one file)

- Lines 1–359: `<style>` — CSS for layout, animations, responsive design
- Lines 422–676: Audio system and SFX definitions
- Lines 753–862: Canvas sizing and input handling
- Lines 863–1001: Drawing functions (player, boss, clouds)
- Lines 1022–1070: Spawning (enemies, boss, power-ups, particles)
- Lines 1087–1302: Game logic (collision, scoring, boss updates)
- Lines 1507–1653: Render and main game loop
- Lines 1640–1666: Entry point (event listeners, animation loop start)

## Testing & Debugging

- **Performance**: Open browser DevTools, check Frames/Performance tab. Game should maintain ~60fps even on low-end devices.
- **Audio**: Mute button (🔊) in top-right toggles sound; check browser console for AudioContext errors
- **Mobile**: Test on real devices or DevTools device emulation; touch controls require actual touch input
- **Accessibility**: Toggle reduced motion (🎆 button) and test with screen reader or DevTools Accessibility tab

## Known Limitations

- **No persistence of gameplay**: Each run is fresh (only best score and settings are saved)
- **No network**: Entirely client-side; no leaderboards or multiplayer
- **Audio synthesis only**: All sound is generated; no sampling or streaming
- **Canvas-only rendering**: No WebGL or other graphics APIs
