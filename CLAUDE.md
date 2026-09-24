# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project Overview

**Sky Fighter** is a Raiden-style vertical shoot-'em-up game implemented as a single-file HTML5 game. The entire game engine, UI, input handling, and audio synthesis are contained in `index.html` (~1,670 lines).

### Running the Game

- Open `index.html` in any modern web browser
- No build process, server, or external dependencies required
- Game runs entirely in the browser with no network requirements
- Responsive design: works on desktop (mouse/keyboard) and mobile (touch controls)

## Architecture Overview

### Game Engine (`loop` function, lines 1640–1665)

The core game loop uses `requestAnimationFrame` with:
- **Delta-time management**: Capped at 50ms per frame to prevent large jumps
- **Adaptive performance tracking**: Monitors average frame time and scales visual quality (full → medium → low) via the `quality` variable (lines 474–483)
- **State management**: Conditional update/draw only when `running && !paused`

### Input System (lines 768–861)

- **Keyboard**: Arrow keys/WASD for movement, Space/X/Shift for weapons, P/Esc for pause
- **Mouse**: Move to steer (glides toward pointer), click to fire; `mouseActive` suppresses keyboard input when hovering canvas
- **Touch**: Virtual joystick on left 55% of screen; fire and bomb buttons on right; multi-touch handling with touch ID tracking
- Priority: keyboard/joystick > mouse (mouse only active when no keyboard input)

### Rendering System (lines 1520–1638)

- **Canvas context**: `ctx.save()/restore()` for transform management
- **Theme system**: 6 rotating sky themes (lines 493–500); theme changes with stage progression
- **Performance-aware rendering**: Glow effects (shadows) only render when `quality === 0` (high quality)
- **Layer structure**: Background gradients → clouds → player/enemies/bullets → particles → UI (HUD elements are separate DOM)

### Game State (lines 864–911)

**Core state variables:**
- `player`: x, y, w, h, speed, cooldown, rapid, weapon type, level, invuln timer
- `enemies[]`, `bullets[]`, `enemyBullets[]`, `particles[]`, `powerUps[]`
- `boss`: Active boss or null; includes `entering` flag for spawn animation
- Game flow: `running`, `paused`, `gameOver`; `stage`, `kills`, `stageKills`; `score`, `chain`, `multiplier`

**Difficulty system**: 3 presets (easy/normal/hard) control spawn rate, enemy speed, enemy shooting chance, lives, bombs, and boss HP multiplier

### Entity System

**Enemy Types** (lines 502–506):
- Grunt: Basic enemy, 1 HP, spawns from stage 1
- Interceptor: Fast, 1 HP, weaving movement, spawns from stage 1
- Tank: Slow, 4 HP, requires multiple hits, spawns from stage 2

Enemy spawn logic (lines 1022–1041):
- Random type selection weighted by difficulty
- Weaving pattern stored per enemy with `weaveSeed` (deterministic but varied)
- Shoot patterns: "spread" (3-way burst) or "aimed" (tracking shots)

**Player Weapon System** (lines 1124–1146):
- Vulcan (default): Fires 1–3 bullets in arc based on level; no pierce
- Laser: Fires 1–3 beams with pierce mechanic (pierce count decrements per enemy)
- Cooldown: 150ms (Vulcan) / 100ms (Laser) between shots

**Power-Up Types** (lines 1304–1322):
- Life: +1 health (up to max)
- Bomb: +1 bomb count (up to max)
- Weapon: Level up current weapon (1→2→3), then toggle to other weapon

### Boss System (lines 1043–1409)

**Boss Components:**
- 6 designs with unique colors, patterns, and HP multipliers (lines 508–515)
- 6 pattern sets per boss design (lines 1372–1409); each boss fires 3 different attack patterns
- Boss spawn triggers after `KILLS_PER_STAGE` (18 kills per stage) with warning banner
- Boss telegraph animation: 22 frames of warning before attack, played through `sfx.bossTelegraph()`

**Boss Attack Patterns:**
- Firearm functions (`fireRadial`, `fireAimedSpread`, `fireColumns`, etc.) generate enemy bullets with velocity (`dx`/`dy`)
- Radial bullets have property `radial: true` and use velocity-based movement (lines 1436–1438) rather than speed + direction
- Boss movement: Horizontal oscillation within canvas bounds (line 1418–1419)

### Scoring & Combo System (lines 1087–1101)

- **Chain mechanic**: Each kill increments `chain` and resets `chainTimer` to 2.2 seconds
- **Multiplier calculation**: 1x base, +1x per 5 consecutive kills (max 10x for 50 kills)
- **Chain break**: No kill within 2.2 seconds resets chain to 0
- **Visual feedback**: Combo popups at 10-kill milestones

### Audio System (lines 542–676)

- **No external files**: Procedurally generated using WebAudio API
- **Audio context**: Lazy-initialized on first user interaction via `ensureAudio()`
- **Compression chain**: Dynamic compressor at master output for consistent level
- **Sound effect categories**: Weapon fires, explosions (type-specific), bosses, UI, power-ups
- **Synthesis techniques**: 
  - Frequency sweeps for pitch effects (e.g., laser ramps up in pitch)
  - Filtered noise bursts for impact/explosion sounds
  - Stereo panning based on horizontal position (`panFor()`, line 562–564)

### Collision & Damage (lines 1232–1294)

- **Collision check**: Simple AABB overlap test via `rectsOverlap()` (lines 1069–1071)
- **Bullet-to-enemy**: Reduces enemy HP; laser bullets can pierce (decrement `pierce` count)
- **Bullet-to-boss**: 3 damage per bullet hit (line 1259)
- **Bomb damage**: 24 damage to boss, destroys all enemy bullets and enemies on screen
- **Enemy collision**: Damages player, breaks combo, applies invulnerability (1.5s)
- **Boss collision**: Like enemy, but only when `!boss.entering` (after spawn animation)

### Persistence (lines 517–523)

LocalStorage keys (with `skyfighter.` prefix):
- `bestScore`: High score
- `muted`: Audio state
- `difficulty`: Last selected difficulty
- `reducedMotion`: Animation preference (respects system `prefers-reduced-motion` on first visit)

### Accessibility

- **ARIA live region** (`#srLive`): Announces game state changes (boss defeats, stage progression, life count)
- **Reduced motion support**: All animations (screen shake, vignette pulse, flashes, combo popups) disabled when active; glow effects scaled back (line 728–729)
- **Screen reader hints**: Control instructions, mute/motion/pause button titles
- **Keyboard accessible**: All UI buttons are native `<button>` elements

## Common Development Tasks

### Tweaking Gameplay

**Difficulty/Balance** (lines 485–489):
- Spawn rates: Decrease `spawnBase` for faster spawning
- Enemy damage: Adjust `enemySpeed` multiplier or enemy bullet speed in fire functions
- Boss HP: Adjust `bossHpMul` or modify base HP calculation (line 1046)

**Weapon Balance** (lines 1124–1145):
- Bullet speed: Modify `speed` property (Vulcan: 500, Laser: 640)
- Spread pattern: Adjust angle offsets in `spread` and `lanes` arrays
- Fire rate: Adjust `fireDelay` (150ms Vulcan, 100ms Laser)

**Boss Patterns** (lines 1372–1409):
- Create new attack pattern function and add to `BOSS_PATTERNS` array
- Use existing helper functions (`fireRadial`, `fireAimedSpread`, etc.) or write custom bullet generation
- Pattern executes at telegraph end (line 1427)

### Adding Content

**New Enemy Type:**
1. Define in `ENEMY_TYPES` object (lines 502–506) with properties: `w`, `h`, `hp`, `speedMul`, `color`, `dark`, `score`, `minStage`, optionally `weave`
2. Update enemy spawn logic (line 1026) to include in choices by stage
3. Add explosion sound effect to `EXPLODE_SFX` map (line 678) and define in `sfx` object (lines 624–676)

**New Boss Design:**
1. Add to `BOSSES` array (lines 508–515) with colors and HP multiplier
2. Create attack pattern array in `BOSS_PATTERNS` (line 1372)
3. Boss index maps to theme and pattern set automatically

**New Visual Theme:**
1. Add to `THEMES` array (lines 493–500) with gradient colors, cloud colors, sun color, and name
2. Themes rotate by stage (`stage - 1 % THEMES.length`)

### Performance Optimization

- Quality scaling is automatic; low-quality disables shadows/glows (line 1002, 1579, 1583, etc.)
- Particle count scales with quality (line 1074)
- Cloud rendering uses modulo arithmetic to avoid off-screen overdraw (lines 1507–1518)
- Canvas resolution fixed at 480×720; CSS scaling handles responsive sizing (lines 753–765)

### Testing in Browser

- Open DevTools → Console to monitor performance and state
- Inspect `window` object for game variables (e.g., `player`, `score`, `difficulty`)
- Use `localStorage.clear()` to reset saved high score and settings

## File Structure

- `index.html`: Single-file game with embedded HTML, CSS, and JavaScript
- `README.md`: Minimal metadata
- No external dependencies; no build process

## Key Code Sections

- **Lines 1–359**: HTML structure and inline CSS
- **Lines 422–424**: IIFE wrapper and strict mode
- **Lines 426–461**: DOM element references
- **Lines 468–483**: Performance tracking (quality scaling)
- **Lines 485–528**: Game configuration and persistence
- **Lines 542–676**: Audio synthesis and SFX library
- **Lines 753–862**: Input handling (keyboard, mouse, touch, joystick)
- **Lines 864–1147**: Game state and entity systems
- **Lines 1148–1302**: Main update loop (player, enemies, collisions, scoring)
- **Lines 1411–1454**: Boss-specific update and defeat logic
- **Lines 1520–1638**: Rendering system
- **Lines 1640–1665**: Main game loop
