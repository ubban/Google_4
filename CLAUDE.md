# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project Overview

**Sky Fighter** is a Raiden-style vertical shoot-'em-up game implemented as a single HTML5 file with embedded CSS and JavaScript. The entire application (~1670 lines) lives in `index.html`:
- **Canvas API** for rendering (2D context)
- **localStorage** for persistence (best score, difficulty, mute state, motion preferences)
- **Responsive design** that works on desktop (mouse + keyboard) and mobile (touch + joystick)

## How to Run & Test

**No build step required.** Simply open `index.html` in a modern browser:
```bash
# Quick local test (Python 3)
python3 -m http.server 8000
# Then visit http://localhost:8000

# Or any static file server
npx http-server
```

**Key user flows to test:**
- Desktop: click/drag mouse to steer, Space/click to fire, X/Shift for bomb, P/Esc to pause
- Mobile: drag left zone to move, tap FIRE/BOMB buttons, landscape preferred
- Difficulty: Easy/Normal/Hard buttons on start screen (affects spawn rate, enemy speed, lives, bombs)
- Score persistence: finish a game, reload page, check best score appears
- Audio: mute button persists state in localStorage

## Code Architecture

The JavaScript is wrapped in a single IIFE (`(function() { 'use strict'; ... })()`) to avoid globals. Key sections (in order):

### 1. **DOM & State Setup** (lines 426–466)
- Canvas element and 2D context
- UI element references (score, health, bombs, overlay, buttons)
- localStorage keys for persistence

### 2. **Game Configuration** (lines 473–510)
- **Performance tracking**: frameTimes array adapts quality (0=full, 1=medium, 2=low) to maintain 40–56 FPS
- **Difficulty presets**: `DIFFICULTIES` object; difficulty affects spawn timing, enemy speed, shoot chance, starting lives/bombs, boss HP multiplier
- **Stage progression**: `KILLS_PER_STAGE = 18` enemies per stage to advance
- **Visual themes**: 6 sky gradients that cycle per stage (Daylight Front → Sunset Corridor → Night Approach, etc.)

### 3. **Game Loop & Entity System**
The game maintains several entity arrays and updates them each frame:
- **Player**: single ship object with position, velocity, health, bomb count, weapon state
- **Bullets**: player projectiles (simple, persistent until off-canvas)
- **Enemies**: dynamic spawn pool with position, velocity, type, health, shoot timer; different types (red, blue, bomber)
- **Enemy bullets**: incoming fire (handled separately for rendering order)
- **Particles**: explosions, hits, combos (animated off-canvas)
- **Power-ups**: weapon upgrades and bombs (dropped by enemies, consumed on collision)

**Entity lifecycle:**
- Enemies spawn according to `spawnCounter` (decrements each frame)
- Collision detection on every update (player ↔ enemy, player bullets ↔ enemies, player ↔ pickups)
- Off-canvas entities are pruned each frame to keep memory low

### 4. **Input Handling**
- **Desktop**: mousemove, click, keyboard (arrows, WASD, Space, X, Shift, P, Esc)
- **Mobile**: custom touch joystick (left zone draggable) + FIRE/BOMB buttons
- **Gyro/motion**: optional device motion API for tilt-to-move (iOS/Android)
- Input state stored in simple flags/coordinates; polled in update loop (not event-driven game logic)

### 5. **Rendering Pipeline**
Canvas operations in strict order:
1. Clear & draw sky gradient (from theme)
2. Draw clouds (parallax scrolling background)
3. Draw all entities: enemies → player → bullets → particles → UI overlays
4. Flash & vignette overlays for hit/bomb effects
5. HUD updates (score, health, bombs, weapon name/level)
6. Boss health bar (when active)

### 6. **Key Systems**

#### Weapon Upgrade System
- Player cycles through 4 weapon types (basic → double → triple → spread)
- Weapon level increments on pickup
- Each weapon has distinct spread pattern and fire rate
- Weapon state tied to player object; resets to level 0 on new life

#### Score & Combo
- Base enemy kill = 100 pts; multiplied by current combo
- Combo increases on consecutive kills within time window (250ms default)
- Combo displayed as floating popup (comboPulse animation)
- Hits without killing reset combo to 0

#### Boss Encounters
- Bosses spawn at stage milestones (every N stages after boss intro)
- Boss HP scales with difficulty (`bossHpMul`)
- Boss pattern: circles, spirals, or walls of projectiles
- Defeat triggers stage advance and increased enemy spawn rate

#### Smart Bombs
- Limited inventory (difficulty-dependent: 2–3)
- Clears all bullets and enemies on-screen
- Visual flash + vignette effect
- Partial bomb refund on killing enemies during detonation

### 7. **Mobile Optimizations**
- Adaptive quality: frame-time averaging triggers quality reduction (particle count, effect complexity)
- Touch-optimized UI: large buttons, safe-area insets for notches
- Joystick deadzone to prevent drift
- Gyro fallback for tilt-unaware devices

### 8. **State Management**
Game modes via overlay visibility:
- **Menu**: overlay visible, start button active, difficulty selection enabled
- **Playing**: overlay hidden, game loop running, pause button functional
- **Paused**: game state frozen, overlay shows pause message
- **Game Over**: overlay shows final stats, difficulty buttons re-enable

Persistent state (localStorage):
- `skyfighter.bestScore`: best score across sessions
- `skyfighter.difficulty`: last selected difficulty
- `skyfighter.muted`: audio mute toggle
- `skyfighter.reducedMotion`: motion-reduce preference (for accessibility)

## Development Patterns & Style

- **No classes or modules**: plain object literals for game state and entity data
- **Imperative animation**: manual property updates (no tweening library); use `requestAnimationFrame` loop
- **CSS for UI, Canvas for gameplay**: UI overlays (menu, HUD, boss bar) use CSS; game world is pure Canvas
- **Inline asset creation**: no external images; SVG icon in meta tag, shapes/gradients/text drawn programmatically
- **Responsive units**: `clamp()`, `vw`/`vh` for scaling to any screen; safe-area insets for mobile notches
- **Performance-conscious**: object pooling for bullets/particles, no excessive allocations in hot loops
- **Browser APIs**: Vibration API (buzz on hit), Web Audio (if added), localStorage, requestAnimationFrame, touch events

## Common Edits

**Adjust difficulty balance:**
- Edit `DIFFICULTIES` object (spawn rate, enemy speed, health scaling)
- Spawn rate: `spawnBase` (initial timer), `spawnMin` (final min during ramp)

**Add new enemy type:**
- Add type name to spawn logic
- Define shape/color in enemy draw section
- Customize shoot pattern in enemy update

**Modify weapon behavior:**
- Weapon definitions in player shoot logic
- Adjust spread angle, fire rate, bullet speed

**Tweak visual themes:**
- Edit `THEMES` array (sky gradients, cloud color, sun color)
- Cycle progression is automatic per stage

**Change difficulty selection on start screen:**
- Edit default difficulty in initialization (`const activeDiff = 'normal'`)
- Adjust button styling in CSS (#diffRow, .diffBtn)

## Accessibility & Inclusivity

- Screen reader live region (`srLive`) announces key events (game over, stage start, combo milestones)
- Reduced motion preference respected: skips particle animations and effects if enabled
- Touch controls as first-class: joystick and buttons sized for all hand sizes
- Colorblind enemy variants possible (current: red, blue, bomber; textures differ beyond color)
- High contrast mode compatible (all text has text-shadow for readability on any sky gradient)

## Performance Notes

- **Target:** 60 FPS on desktop, 40+ FPS on mobile
- **Adaptive quality:** if frame time > 25ms (40 FPS), reduce particle count & effect complexity
- **Entity limits:** no strict cap, but game naturally caps enemies via spawn rate
- **Canvas size:** scales to viewport with `max-width`/`max-height`; resolution adjusts via canvas.width/height in resize handler
