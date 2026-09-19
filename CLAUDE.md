# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project Overview

**Sky Fighter** is a Raiden-style vertical shoot-'em-up game built as a single-file HTML5 application. The game features:
- Adaptive difficulty (Easy, Normal, Hard)
- Multiple enemy types with distinct behaviors
- 6 boss encounters with pattern-based attacks
- Weapon progression system (Vulcan/Laser with 3 levels each)
- Score multiplier chain system
- Procedural cloud parallax and theme rotation
- WebAudio synthesis for all sound effects
- Full touch and keyboard/mouse controls

## Repository Structure

The entire game is contained in `index.html` (~1669 lines):
- **Lines 1-400**: CSS styling (responsive layout, animations, UI elements)
- **Lines 400-421**: HTML structure (canvas, overlays, HUD, controls)
- **Lines 422-1667**: Game logic (single IIFE wrapping all JavaScript)

No external assets are used — all graphics are drawn via Canvas 2D, and all audio is synthesized via Web Audio API.

## Core Game Architecture

### Game State Variables (Line ~864)
All mutable state is centralized:
- Entities: `player`, `bullets`, `enemies`, `enemyBullets`, `particles`, `powerUps`, `boss`
- Progression: `score`, `lives`, `stage`, `stageKills`, `kills`
- Scoring: `chain`, `chainTimer`, `multiplier`
- Flow: `running`, `paused`, `gameOver`

**Key insight**: The game avoids object-oriented class patterns; entities are plain objects with properties updated in-place.

### Main Update Loop (Lines 1640–1653, 1097–1300+)
The `loop()` function runs via `requestAnimationFrame` and:
1. Calculates delta time (clamped to 50ms)
2. Calls `update(dt)` if running and not paused
3. Calls `draw()` to render the frame
4. Schedules the next frame

The `update(dt)` function (not fully shown in excerpt) handles:
- Player movement and firing
- Weapon cooldown
- Enemy spawning based on difficulty
- Enemy AI and attack patterns
- Bullet-entity collision detection
- Power-up collection
- Stage progression and boss fights
- Life loss and game-over conditions

### Rendering Pipeline (Lines 1520–1638)
The `draw()` function:
1. Clears canvas
2. Applies screen shake if active
3. Renders sky gradient, sun glow, and parallax clouds (adaptive per `quality` level)
4. Draws player, mouse indicator, bullets, enemies, boss, power-ups, particles
5. Handles invulnerability blinking and laser weapon glow

**Performance adaptation** (Lines 473–483): Tracks frame times over 45 frames; downgrades `quality` (0=full, 1=medium, 2=low) if average frame time exceeds 1/40s, downgrades shadow effects and particle counts accordingly.

## Key Systems

### Difficulty Configuration (Lines 485–489)
`DIFFICULTIES` object defines three tiers with independent spawn rates, enemy speed, shoot chance, starting lives, bomb count, and boss HP multiplier. This is the primary lever for balancing.

### Enemy Spawning & AI (Lines 1022–1041)
- Enemies spawn at the top, descend, and shoot based on type and difficulty
- Three types: `grunt`, `interceptor`, `tank` — each with distinct HP, speed, sprite rendering
- Enemies use two attack patterns: `'spread'` (radial bullets) or `'aimed'` (player-targeted spread)
- Interceptors weave horizontally using a `weaveSeed`

### Boss System (Lines 1043–1056, 1372–1439)
- Six boss designs rotate by stage (see `BOSSES` array, lines 508–515)
- Each boss has a unique pattern set in `BOSS_PATTERNS` (lines 1372–1409)
- Boss states: `entering` → normal patrol → `telegraph` (warning) → attack → cycle
- HP scales with difficulty multiplier and boss design multiplier

Boss attack types include:
- Radial burst, aimed spreads, columns, spiral bursts, crosses, teleport-barrage, homing volleys, double rings

### Weapons & Damage (not fully shown; inferred from variables)
- Two weapon types: `'vulcan'` (spray), `'laser'` (narrow beam)
- Three upgrade levels per weapon; cycling back after level 3 switches weapon
- `player.rapid` tracks cooldown; player fires continuously while input is held

### Power-Ups (Lines 1058–1067, 1304–1322)
Three types spawn at random enemy death locations:
- `'life'`: Restore one heart (up to `maxLives`)
- `'bomb'`: Restore one bomb (up to `maxBombs`)
- `'weapon'`: Upgrade weapon level or cycle weapon type

### Scoring & Chain Multiplier (Lines 1087–1099)
- `addScore(base)` adds `base * multiplier` to score and increments `chain`
- Multiplier = 1 + floor(chain / 5), capped at 10x
- `chainTimer` resets on each kill; `breakChain()` zeros it if player takes damage

### Audio Synthesis (Lines 542–676)
No audio files — all sounds are synthesized:
- `beep(freq, dur, type, vol, opts)`: Oscillator with optional frequency sweep, filter, pan, delay
- `noiseBurst(dur, vol, opts)`: Noise sample with filter and pan
- `sfx` object maps game events to sound functions (shoot, hit, bomb, powerup, boss patterns, etc.)
- Master gain (0.9) feeds through a dynamics compressor

**Key note**: Audio context is lazily initialized on first user input (`ensureAudio()`), ensuring compliance with autoplay policies.

### Control Systems (Lines 755–862)
Three input methods coexist:
1. **Keyboard/Gamepad** (inferred): `firing` flag, arrow/WASD/Space/Shift keys
2. **Mouse**: Plane glides toward pointer; click to fire
3. **Touch**: Virtual joystick on left, FIRE/BOMB buttons on right
- All update `joyVec` or `mouseTarget` or `firing` without conflict

### Local Storage (Lines 463–527)
Persists across sessions:
- `skyfighter.bestScore`: High score
- `skyfighter.muted`: Audio mute state
- `skyfighter.difficulty`: Last selected difficulty
- `skyfighter.reducedMotion`: Accessibility preference

## Theme System (Lines 493–500)
Six themes rotate by stage, each with:
- `sky`: Three gradient stops (top, middle, bottom)
- `cloud`: RGB for parallax clouds
- `sun`: RGB for sun glow
- `name`: Display name

Themes wrap around (modulo 6) so every stage has a theme.

## Development Workflow

### Making Changes
1. **Small tweaks** (balance, UI text, colors): Edit the specific section directly
2. **New enemy type**: Add to `ENEMY_TYPES` object, update spawn logic in `spawnEnemy()`
3. **New boss pattern**: Add pattern function to relevant array in `BOSS_PATTERNS`
4. **New difficulty tier**: Extend `DIFFICULTIES` object
5. **Visual effects**: Modify `draw()` or particle spawning logic

### Testing
- **Local**: Open `index.html` in a browser; use DevTools console to inspect state
- **Difficulty balance**: Adjust `DIFFICULTIES` constants and replay stages
- **Collision tuning**: Edit `rectsOverlap()` logic (currently simple AABB)
- **Audio**: Test mute button and `ensureAudio()` autoplay compliance
- **Responsive**: Test on mobile devices or DevTools device emulation; controls adapt based on `touchControls.enabled`

### Debugging Tips
- Game state is globally accessible in `loop()` closure; use DevTools to set breakpoints
- `announce(text)` updates screen reader live region for accessibility
- `screenShake(magnitude, duration)` provides feedback (respects reduced-motion)
- Frame counter (`frame` variable) increments each update; useful for animation timing
- Quality level (`quality` variable) adapts rendering; test by simulating slow devices

## Performance Considerations

- Canvas resolution matches device pixel ratio (inferred from responsive CSS)
- Particle counts scale with quality level (full=100%, medium=60%, low=35%)
- Shadow effects and glow are disabled below full quality
- Entity arrays are not pre-allocated; new entities are pushed, dead ones filtered
- Collision detection is O(n²) (player vs bullets + enemies vs bullets); manageable for ~50 entities

## Accessibility
- Screen reader live region (`#srLive`) receives game announcements
- Reduced-motion toggle respects `prefers-reduced-motion` and disables animations/shake/flash
- All UI buttons are clickable; keyboard shortcuts (P/Esc for pause, X/Shift for bomb)
- Mouse cursor changes to crosshair during gameplay

## Common Edits Reference

| Task | Location | Notes |
|------|----------|-------|
| Adjust difficulty | Lines 485–489 | Edit `DIFFICULTIES` object |
| Change boss patterns | Lines 1372–1409 | Edit `BOSS_PATTERNS` array |
| Add new sound | Lines 624–676 | Add entry to `sfx` object |
| Modify weapon balance | Search `player.rapid`, `player.cooldown` | Update firing rate or damage calculation |
| Change theme colors | Lines 493–500 | Edit `THEMES` array |
| Adjust enemy spawning | Lines 1022–1041 | Edit `spawnEnemy()` and difficulty constants |
| UI tweaks | Lines 1–400 (CSS) + 401–421 (HTML) | Responsive CSS uses clamp() for scaling |

## Edge Cases & Quirks
- Boss enters from top at y=-80; enters complete at y=120 (harcoded safe zone)
- Player invulnerability period is 1.5s after taking damage
- Chain timer resets to 0 on damage; 2.2s between kills to maintain chain
- Enemy bullets are simple circles; player/boss bullets use square/rect with rotation (inferred)
- Collision is AABB only; no rotation handling for collision
- Boss health bar uses `transition: width 0.15s ease` for smooth animation

## Future Enhancements (Not Implemented)
Potential areas for expansion without major refactoring:
- Leaderboard server integration
- Multiple player models / skins
- Custom difficulty tuner UI
- Replay system (record inputs, replay)
- Mobile joystick haptics feedback
