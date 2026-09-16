# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project Overview

**Sky Fighter** is a Raiden-style vertical shoot-'em-up game built entirely in HTML5 with:
- Canvas-based rendering (no external assets or libraries)
- WebAudio API for procedurally generated sound effects
- Touch and keyboard input support
- Three difficulty levels with persistent local storage
- Six unique boss encounters with distinct attack patterns
- Adaptive performance scaling based on frame times

## Running the Game

Simply open `index.html` in a web browser. The game is self-contained with no build process or dependencies.

```bash
# Option 1: Open directly
open index.html

# Option 2: Serve locally (recommended to avoid CORS issues)
python3 -m http.server 8000
# Then visit http://localhost:8000
```

## Architecture

### Single-File Structure
The entire game (HTML, CSS, JavaScript) is in `index.html`. This design choice prioritizes simplicity and portability over modularity.

### Core Systems

**Game Loop** (lines 1640-1653)
- Runs at 60 FPS via `requestAnimationFrame`
- Caps delta time at 50ms to prevent spiral of death
- Tracks performance metrics for quality adaptation

**Input Handling** (lines 767-861)
- Keyboard: WASD/Arrows for movement, Space for fire, X/Shift/P for bomb/pause
- Mouse: Aim by moving, click to fire
- Touch: Left zone is joystick, right buttons fire/bomb, procedurally generated visual feedback
- Mouse and keyboard input never conflict (keyboard takes priority)

**Gameplay State** (lines 863-911)
- `player`: Position, speed, weapon, level, invulnerability frames
- `enemies`: Array of enemy objects with type-specific properties (grunt/interceptor/tank)
- `boss`: Single boss with entering animation, pattern cycling, telegraphing
- `score`, `chain`, `multiplier`: Kill chain mechanics reward rapid consecutive kills
- `stage`, `stageKills`: Progress tracking; 18 kills per stage triggers boss

**Collision Detection** (line 1069-1071)
- Simple rectangle overlap using axis-aligned bounding boxes
- Checked between: bullets/enemies, bullets/boss, enemies/player, bullets/player, powerups/player

**Difficulty System** (lines 485-489)
- Three presets: easy/normal/hard with knobs for spawn rate, enemy speed, shoot chance, lives, bombs
- Stored in localStorage; difficulty affects `spawnBase`, `spawnMin`, `enemySpeed`, `enemyShootChance`
- Spawn timer auto-scales downward over time (line 1201) independent of difficulty

**Boss AI** (lines 1372-1434)
- Six unique boss designs (VANGUARD, BEHEMOTH, SPECTER, DREADNOUGHT, WRAITH, TITAN)
- Each has 3 distinct attack patterns cycling on a timer
- Telegraphing system: 22-frame warning pulse before each attack
- Patterns include: radial bursts, aimed spreads, homing missiles, teleport dashes, spirals, crosses
- Boss enters from top with `entering` flag; collision only after fully arrived

**Weapon System** (lines 1124-1146)
- Two weapons: VULCAN (spread spray), LASER (piercing beams)
- Three levels per weapon; level 3+ grants alternate fire behavior
- Vulcan: spread width increases with level; Laser: beam count/pierce increases
- Weapon pickups toggle between weapons or level up current weapon

**Rendering** (lines 1520-1637)
- Canvas size fixed at 480x720 internally; CSS scaling handles DPI/window size (line 757)
- Screen shake applied via canvas context translate (line 1526)
- Theme system: 6 preset sky gradients + cloud sprites rotate per stage
- Glowing effects (shadow blur) disabled at low quality settings
- Particle system for explosions (line 1073-1085)

**Performance Tuning** (lines 474-483)
- Quality levels: 0=full, 1=medium (60% particles), 2=low (35% particles)
- Auto-adjusts based on 45-frame rolling average of delta times
- Adaptive quality reduces particle counts, glows, and animation complexity

**Audio** (lines 542-676)
- WebAudio synth-only; no audio files
- Master gain with dynamic range compressor
- Procedural effects for each action: shoot, laser, explosions (grunt/interceptor/tank), bomb, powerup, boss hit, stage clear, game over
- Pan effects position sounds across stereo field based on x-coordinate
- Muted state persisted in localStorage

**Accessibility** (lines 530-539, 468-471)
- Reduced motion support via `prefers-reduced-motion` media query
- Toggleable via UI button; persisted to localStorage
- Eliminates screen shake, flash effects, and animations when enabled
- ARIA live region for screen reader announcements (kills, stage clear, game over)

### State Machines

**Game State**: `running` → `paused` → `running` → `gameOver` → reset
**Boss Lifecycle**: `entering` (sliding in) → `attacking` → `defeated` → `killBoss` (next stage)
**Player**: Constant movement/firing loop, invulnerability frames after hit (1.5 seconds)

## Development Notes

### Adding Content

**New Enemy Type**:
1. Add to `ENEMY_TYPES` object (line 502-506) with `w`, `h`, `hp`, `speedMul`, `color`, `dark`, `score`, `minStage`, optional `weave`
2. Update `spawnEnemy()` (line 1022-1041) to include in spawn logic
3. Add explosion SFX function to `sfx` object if unique death sound (line 624-676)
4. Add to `EXPLODE_SFX` mapping (line 678)

**New Boss**:
1. Add design to `BOSSES` array (line 508-515): name, hull colors, wing color, core color, hpMul
2. Add 3 attack pattern functions to `BOSS_PATTERNS` array (line 1372+)
3. Function receives boss object; call `fireRadial`, `fireAimedSpread`, `fireColumns`, `fireSpiralBurst`, `fireCross`, `fireHomingSeeded`, or `fireDoubleRing` helper

**New Theme**:
1. Add to `THEMES` array (line 493-500) with sky gradient colors, cloud color, sun color, and name
2. Boss battle intro screen pulls name automatically

### Performance Considerations
- Quality level affects particle count and shadow blur; test with `quality = 2` to verify low-end performance
- Boss pattern density should stay under ~30 bullets per attack to remain responsive
- Canvas is 480x720 fixed; scaling is CSS-only to keep rendering fast
- Particles are pooled by reusing array; old particles filtered out each frame (line 1296-1297)

### Modifying Game Feel
- `player.speed`: Movement speed (currently 260 pixels/sec)
- `fireDelay`: Fire rate cooldown per weapon (line 1177)
- `KILLS_PER_STAGE`: How many enemy kills before boss appears (line 491, currently 18)
- Boss `vx`: Horizontal patrol speed (line 1418, currently 55 pixels/sec)
- Screen shake magnitude/duration in `screenShake()` calls throughout

### Common Tasks

**Change difficulty balance**: Edit `DIFFICULTIES` object (line 485-489)

**Adjust combo/multiplier rates**: 
- `chain` increments in `addScore()` (line 1088)
- `multiplier` formula at line 1090
- Chain timeout at line 1098 (2.2 seconds between kills)

**Modify visual styles**:
- Player color: `drawPlane()` call at line 1558 uses hardcoded `#2196f3` blue
- Enemy colors: Stored in individual enemy objects (line 1038)
- Boss telegraph effect: Line 999-1006 (core glow intensity)

**Fix input issues**:
- Keyboard bindings: Line 769-776
- Touch zone layout: Line 299-341 (moveZone is 55% width left, actionZone is 45% width right)
- Mouse sensitivity: Line 1165-1170 (interpolation toward mouseTarget)

## File Organization

```
index.html (1670 lines)
├── <head> with meta tags and full inline CSS
└── <body>
    ├── HTML structure (container, canvas, overlays, HUD, touch controls)
    └── <script> (entire game implementation)
        ├── DOM queries and constants
        ├── Adaptive performance system
        ├── Audio synthesis (sfx library)
        ├── UI/HUD rendering functions
        ├── Game state and initialization
        ├── Update loop (player, enemies, collisions, boss)
        ├── Rendering functions
        ├── Boss attack patterns
        └── Main game loop
```

## Responsive Design
- Canvas stays aspect-ratio locked at 480x720 (4:6)
- Scales up to fit window while respecting max 1.6x scale (line 757)
- Touch-friendly UI with 78x78px buttons
- HUD uses `clamp()` for responsive typography
- Safe-area insets respected for notch-aware devices (env(safe-area-inset-*))

## Browser Compatibility
- Requires Canvas 2D Context and WebAudio API
- Gracefully degrades if WebAudio unavailable (muted flag checked)
- Touch events polyfilled via pointer detection
- Modern CSS features (clamp, gap, backdrop-filter) assumed available
