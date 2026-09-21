# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project Overview

**Sky Fighter** is a Raiden-style vertical shoot-em-up game built as a single-file HTML5 canvas application. The entire game logic, rendering, and UI are contained in `index.html` (~1670 lines of inline JavaScript and CSS).

### Key Characteristics
- **Single-file architecture**: All code is in index.html; no build step or external dependencies
- **Canvas-based rendering**: 2D graphics drawn via canvas API with performance adaptation
- **Game loop**: Frame-based update/render cycle running at target 60 FPS
- **Local storage**: Persists best score, mute state, difficulty setting, and reduced-motion preference
- **Progressive enhancement**: Works with keyboard, mouse, and touch controls

## Architecture & Code Organization

The game follows a classical game loop structure with these main components:

### Game State
Global variables in the `<script>` section manage:
- **Player**: Position, health, weapon, bombs, score, multiplier/combo chain
- **Enemies**: Array of active enemies with type, position, health, AI state
- **Boss**: Single boss object per stage with health, telegraph warning, attack patterns
- **Projectiles**: Player bullets and enemy bullets (separate arrays)
- **Particles**: Explosion debris and effects for visual feedback
- **Power-ups**: Floating pickups (weapon, bomb, health)

Game progression tracked via `stage`, `kills`, `spawnCounter`, `enemyKillCount`.

### Core Functions & Patterns

**Drawing/Rendering**:
- `draw()` — Main render loop; clears canvas, draws background gradient, then all entities
- `drawPlane(x, y, w, h, color, darkColor, flame)` — Player ship with optional thrust flame
- `drawBossShip(b)` — Boss graphics with design theme colors, rotating parts, telegraph glow
- Entity drawing is inlined in `draw()` loop; no sprite sheets

**Game Logic**:
- `update(dt)` — Called once per frame with delta time; moves all entities, checks collisions
- `spawnEnemy()` — Randomly selects enemy type based on stage, difficulty, and spawn chance
- `spawnBoss()` — Creates boss with stage-dependent design and health scaling
- `maybeSpawnPowerUp(x, y)` — 6% life, 8% bomb, 20% weapon; 66% no spawn
- Collision detection via `rectsOverlap(a, b)` — Axis-aligned bounding box

**Weapon System**:
- Player has a `weapon` object with `type`, `level`, and `ammo`
- Firing functions: `fireRadial()`, `fireColumns()`, `fireCross()`, `fireDoubleRing()`, `fireHomingSeeded()`, `fireAimedSpread()`, `fireSpiralBurst()`
- Weapon types unlock per stage; each has unique bullet pattern, speed, and count
- Weapon level increases damage; multiplier (from kill combo) amplifies score

**Difficulty & Performance**:
- Three presets in `DIFFICULTIES`: easy, normal, hard with spawn/enemy/boss modifiers
- `quality` (0=full, 1=medium, 2=low) adapts particle/shadow/glow based on frame time avg
- Tracks last 45 frames; upgrades quality if avg < 1/56 FPS, downgrades if > 1/40

**Input Handling**:
- Keyboard: Arrow keys or WASD move, Space fires, X or Shift drops bomb
- Mouse: Move to steer, click to fire
- Touch: Two-zone system (left 55% for move, right 45% for buttons) with joystick option
- All input updates `inputs` object; `update()` reads it

**Audio**:
- `ensureAudio()` creates AudioContext on first user interaction (mobile requirement)
- `beep(freq, duration)` generates simple sine-wave tones; respects mute toggle
- No pre-recorded assets; all synthesis

**Persistence**:
- LocalStorage keys: `skyfighter.bestScore`, `skyfighter.muted`, `skyfighter.difficulty`, `skyfighter.reducedMotion`
- Best score persists across sessions; read on init, written on game end
- Respects prefers-reduced-motion media query

### Styling Notes
- Inline `<style>` uses CSS Grid/Flexbox; responsive with `clamp()` and viewport units
- Dark theme (#05122b background); accent colors: #1976d2 (blue), #ffd54f (gold), #7CFC00 (lime), #ff5252 (red)
- Animations use keyframes; reduced-motion media query disables them
- HUD elements (top bar, boss bar, combo pop) positioned absolutely; z-index layering

## Development Workflow

### Running the Game
No build step. Simply open `index.html` in a browser or serve locally:
```bash
# Python 3
python -m http.server 8000

# Node http-server (if available)
npx http-server

# Then visit http://localhost:8000
```

### Testing Changes
- **Visual testing**: Changes to sprite drawing, colors, or animations should be verified in-browser
- **Game balance**: Adjust `DIFFICULTIES`, weapon stats in `WEAPON_TYPES`, enemy spawn rates to tune difficulty
- **Performance**: Use browser DevTools (F12) Performance tab to measure frame time; target 60 FPS
- **Accessibility**: Test keyboard controls (arrows/WASD, Space, X); verify screen reader announcements via `announce()`

### Common Modifications

**Adding a new weapon type**:
1. Add entry to `WEAPON_TYPES` object with name, level scaling, firing pattern
2. Create a `fire*()` function or reuse existing pattern
3. Call it from the `fireWeapon()` conditional

**Changing enemy difficulty**:
1. Modify `DIFFICULTIES[diff]` values (spawnBase, enemySpeed, bossHpMul, etc.)
2. Adjust `ENEMY_TYPES` table (hp, speed multiplier, score)
3. Test in-game across all three difficulties

**Tuning visual effects**:
- Boss telegraph glow: `ctx.shadowBlur = quality === 0 ? 22 : 10` (lines ~1002)
- Explosion particle count: `count || 14` scaled by quality factor (line ~1074)
- Vignette intensity: `#vignette.on opacity` animation values

**Boss patterns**:
- Each boss has a `design` object: `{ name, wing, hullTop, hullBottom, core, hpMul }`
- Attack patterns hard-coded in `update()` boss logic; relies on `telegraph`, `attackTimer`, `patternIndex`

## File Structure
```
index.html
  └─ <head> metadata + inline <style>
  └─ <body>
      ├─ #container (game wrapper)
      │  ├─ #gameCanvas (2D context)
      │  ├─ #hud (score, lives, weapon, combo)
      │  ├─ #bossBarWrap (health bar)
      │  ├─ #overlay (menu, game-over screen)
      │  ├─ #touchControls (mobile-only)
      │  └─ #topBar (mute, motion, pause buttons)
      └─ <script> (entire game logic)
README.md (project metadata timestamp)
```

## Notable Implementation Details

- **No external dependencies**: Game runs offline after page load
- **Mobile-first touch**: Layout adapts to viewport; touch handlers create joystick feedback
- **Combo multiplier**: Kill chain decays over 2.2 seconds; resets on shot miss or pause
- **Boss telegraphing**: Boss emits bright glow + white fill before attack, telegraphing danger
- **Reduced-motion support**: Respects OS accessibility setting; disables all animations
- **Quality tiers**: Particle/shadow rendering scales with frame budget to maintain 60 FPS on lower-end devices

## When Making Changes

1. **Keep it self-contained**: Any new feature should fit within the single-file model
2. **Test all difficulties**: Changes to balance must work across Easy, Normal, Hard
3. **Check mobile**: Use DevTools device emulation to test touch controls
4. **Accessibility**: Maintain keyboard controls and screen reader announcements
5. **Performance**: Profile with DevTools; ensure adaptive quality prevents frame drops

---

For questions about the game mechanics, weapon patterns, or enemy AI, search the `index.html` for the relevant function or `CONSTANT_NAME`.
