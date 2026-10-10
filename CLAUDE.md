# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project Overview

**Sky Fighter** is a browser-based Raiden-style vertical shoot-em-up arcade game. It's a single-file HTML5 Canvas game with no external dependencies — all rendering, audio, and game logic are implemented from scratch.

## Development Commands

### Running the Game

The game is a single `index.html` file. Open it in a browser:

```bash
# Simple: Open the file directly in a browser
open index.html

# Or use a local server if needed
python3 -m http.server 8000
# Then navigate to http://localhost:8000/index.html
```

### Testing Specific Features

Since this is a single-file game without automated tests, manual testing is the primary verification method:

- **Test player movement**: Use arrow keys (WASD also works), mouse movement for aiming, or touch/joystick on mobile
- **Test weapons**: Press Space or click Fire button; switch weapons via power-ups
- **Test bombs**: Press X, Shift, or click Bomb button
- **Test pause**: Press P or Esc
- **Test pause**: Press P or Esc
- **Test difficulty**: Click Easy/Normal/Hard buttons before starting (persists via localStorage)
- **Test audio**: Toggle sound with 🔊 button; test various sound effects by hitting enemies, using bombs, etc.
- **Test responsiveness**: Resize browser or test on mobile/tablet devices
- **Test accessibility**: Test screen reader announcements (lives, weapon changes, etc.)

### Linting & Quality

No build step needed. For code quality, you can manually inspect the JavaScript section (lines 422-1666).

## Architecture

### Single HTML File Structure

The entire game (CSS, HTML, JavaScript) is in `index.html`:

- **Lines 1-421**: HTML markup and CSS styling
- **Lines 422-1666**: JavaScript game logic (all inside an IIFE)

### Game Loop & Rendering

**Main Loop** (`loop` function, line 1640):
- Runs via `requestAnimationFrame` at ~60 FPS
- Calls `update(dt)` to advance game state
- Calls `draw()` to render the canvas
- Tracks performance and adapts quality dynamically

**Rendering** (`draw` function):
- Clears canvas and renders sky background with theming
- Draws player ship, enemies, bullets, particles, power-ups, and bosses
- HUD overlays (score, health, weapon, bombs) are HTML/CSS elements
- Applies screen shake effects for visual feedback

### Game State Management

**Core State Variables** (line 864-869):
- `player`: Position, health, weapon, cooldown, invulnerability
- `bullets`: Player projectiles with pierce and laser properties
- `enemies`: Enemy ships with HP, firing behavior, movement patterns
- `enemyBullets`: Enemy projectiles
- `boss`: Optional boss ship with complex AI patterns
- `score`, `lives`, `stage`, `chain`, `multiplier`: Progression tracking

**Difficulty System** (line 485-489):
- Three presets: Easy, Normal, Hard
- Controls spawn rates, enemy speed, enemy accuracy, health multipliers
- Difficulty selector buttons persist via `localStorage`

### Key Game Systems

#### 1. Enemy Spawning & Progression
- Three enemy types: grunt (1 HP), interceptor (1 HP, fast), tank (4 HP)
- Spawn rate decreases over time (difficulty ramping, line 1201)
- Stage progression: Kill 18 enemies → boss battle
- Boss spawned after KILLS_PER_STAGE kills (line 491, 1204)

#### 2. Weapon System
- **Vulcan**: Spread shot pattern, increases lanes with level (1, 2, or 3 lanes)
- **Laser**: High-speed piercing projectiles, increases pierce/lanes with level
- Weapon pickup chance: 20% per enemy kill (line 1064)
- Weapon level caps at 3 (line 902)
- Fire rate: Vulcan 150ms, Laser 100ms (line 1177)

#### 3. Power-Up System
- **Life** (6% chance): Restores one health
- **Bomb** (8% chance): Clears screen, damages boss
- **Weapon** (20% chance): Upgrades current weapon or switches
- Spawn on enemy death (line 1058); drift downward until collected or despawn

#### 4. Scoring & Multiplier Chain
- Kills extend the chain timer (2.2 seconds, line 1089)
- Multiplier increases every 5 kills: 1× + 1 per 5 kills (max 10×)
- Chain breaks if no kills during the window
- Base score: grunt 10, interceptor 15, tank 35; bombs deal 5 each

#### 5. Boss Battles
- Six boss designs, one per stage (cycling, line 1045)
- Each boss has unique:
  - Hull/wing colors and glow core
  - Bullet patterns (spread vs. aimed vs. multi-angle)
  - Movement patterns (horizontal weaving, vertical approaches)
  - Special telegraph/flash before big attacks
  - HP multiplier relative to stage/difficulty (line 1046)
- Boss health bar UI shows current HP (line 1054)

#### 6. Audio System
- Procedurally generated SFX using Web Audio API (lines 542-676)
- No external audio assets
- Functions: `beep()` (tone + sweep), `noiseBurst()` (percussion)
- SFX catalog: shoot, laser, explode (3 variants), hit, bomb, power-up, boss hit, stage clear, etc.
- Mute toggle persists via localStorage (line 737)

#### 7. Input Handling
- **Keyboard**: Arrow keys / WASD for movement, Space to fire, X/Shift for bomb, P/Esc for pause
- **Mouse**: Hover over canvas to aim, click to fire
- **Touch**: Virtual joystick in left zone, fire button in right zone
- **Joystick fallback**: When keyboard/touch inactive, mouse position provides a glide target (line 1164)

#### 8. Accessibility
- Screen reader live region (`#srLive`, line 451) announces game state changes
- High-contrast HUD with text shadows
- Reduced motion preference respected (line 520); on by default, toggle with 🚫 button
- Hearts and bombs rendered with emoji for clarity

#### 9. Performance Optimization
- **Adaptive quality levels**: 0 (full), 1 (medium), 2 (low) based on frame timing (line 474)
- Glow effects and particle counts scale with quality (e.g., line 1074)
- Canvas size fixed at 480×720; CSS scaling handles responsive display (line 753)
- Frame time tracking rolling average over 45 frames (line 475)

#### 10. Theming & Visual Effects
- Six visual themes (sky colors, cloud tints, sun highlights, names) that cycle per stage (line 493-500)
- Screen shake on impact/bomb (line 727)
- Flash vignette and white flash animations for critical health and hits
- Animated combo pop text for kill chains
- Stage banner announcement system

### Collision Detection

**Rectangle Overlap** (line 1069):
```javascript
rectsOverlap(a, b): Math.abs(a.x - b.x) < (a.w + b.w) / 2 && Math.abs(a.y - b.y) < (a.h + b.h) / 2
```

Used for:
- Player collision with enemy bullets
- Player collision with power-ups
- Bullets hitting enemies and bosses

### Save/Persistence

Uses `localStorage` for:
- Best score (`skyfighter.bestScore`)
- Mute state (`skyfighter.muted`)
- Difficulty selection (`skyfighter.difficulty`)
- Reduced motion preference (`skyfighter.reducedMotion`)

## Common Development Tasks

### Adding a New Enemy Type

1. Add entry to `ENEMY_TYPES` object (line 502-506)
2. Set `w`, `h`, `hp`, `speedMul`, `color`, `dark`, `score`, `minStage`
3. Update spawn logic if needed (line 1026-1030 filters by `minStage`)
4. Consider `weave` flag for movement pattern
5. Test spawn and drawing (enemies drawn at line 1603-1613)

### Adding a New Boss

1. Add entry to `BOSSES` array (line 508-515)
2. Set `name`, `hullTop`, `hullBottom`, `wing`, `core` colors, `hpMul`
3. Bosses are selected by stage: `BOSSES[(stage - 1) % BOSSES.length]`
4. To customize attack patterns, modify `boss.attackTimer` and bullet logic (line 1273-1343)

### Adjusting Difficulty

Edit the `DIFFICULTIES` object (line 485-489):
- `spawnBase` / `spawnMin`: Control enemy spawn rate
- `enemySpeed`: Multiplier on all enemy speeds
- `enemyShootChance`: Probability enemy fires (0.45-0.95)
- `lives`, `bombs`, `bossHpMul`: Starting lives, bomb count, boss HP scaling

### Tweaking Weapon Balance

**Vulcan** (line 1125-1134):
- Spread pattern: `[-0.24, 0, 0.24]` (level 3), `[-0.12, 0.12]` (level 2), `[0]` (level 1)
- Speed: 500 px/s
- Cooldown: 150 ms

**Laser** (line 1135-1145):
- Lane count: 2 lanes at level 2+, 3 lanes at level 3
- Pierce: 1 at level < 3, 2 at level 3
- Speed: 640 px/s
- Cooldown: 100 ms

### Adding Visual Polish

- Particle explosions controlled by `spawnExplosion()` (line 1073)
- Screen shake via `screenShake(mag, dur)` (line 727)
- Flash effects with CSS animations (`#flash.hit`, `#flash.bomb`)
- Theme colors in `THEMES` array for sky gradients and particles
- Glow effects respect quality setting (line 1002)

## Notes for Future Contributors

- **No external dependencies**: Keep it that way. Everything should be vanilla JS/Canvas/WebAudio.
- **Performance matters**: Test on older devices; prioritize 60 FPS. Adaptive quality is your friend.
- **Mobile first**: Support touch controls, landscape orientation, and safe-area insets.
- **Accessibility**: Screen reader announcements, color contrast, reduced motion support.
- **Persistence**: Use localStorage for non-game-critical state (settings, best score).
- **Code style**: The existing code uses concise variable names and functional patterns; maintain that style.
