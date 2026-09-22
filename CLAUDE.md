# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project Overview

**Sky Fighter** is a Raiden-style vertical shoot-em-up game implemented as a single-file HTML game with embedded CSS and JavaScript. It runs in any modern browser with no build step or external dependencies. The game features:

- Procedurally spawned enemies with multiple types (grunt, interceptor, tank)
- Weapon upgrade system (Vulcan → Laser with leveling)
- Boss battles with telegraph mechanics
- Smart bombs with screen-clearing effects
- Score multiplier chain mechanics
- Touch and keyboard controls
- Adaptive difficulty selection (Easy/Normal/Hard)
- Synthesized sound effects using WebAudio API
- Persistent high-score storage via localStorage
- Accessibility features (screen reader support, reduced-motion mode)

## File Structure

```
index.html (1669 lines)
  ├─ HTML structure (1-420 lines) — canvas, HUD elements, overlay UI
  ├─ CSS (10-359 lines) — layout, animations, responsive design
  └─ JavaScript (422-1669 lines) — game engine, input, rendering
      ├─ DOM element caching & constants
      ├─ Adaptive performance (frame time tracking)
      ├─ Audio synthesis (WebAudio API)
      ├─ Canvas sizing & input handling
      ├─ Game state & initialization
      ├─ Rendering functions (plane, boss, particles, enemies)
      ├─ Spawn logic & collision detection
      ├─ Update loop (player, enemies, bullets, particles)
      └─ Main animation loop (requestAnimationFrame)
```

## Architecture & Key Concepts

### Game Loop

The game runs on `requestAnimationFrame` with a frame-rate independent update using delta time:

```javascript
function gameLoop(now) {
  const dt = (now - lastTime) / 1000;
  lastTime = now;
  
  // Calls update(dt), applies screen shake, renders everything
  if (running && !paused) update(dt);
  
  requestAnimationFrame(gameLoop);
}
```

### Game State

Core state is tracked in module-scoped variables:
- **player** — position, health, weapon, cooldown, invulnerability frames
- **enemies**, **boss** — array of enemy objects with position, type, HP
- **bullets**, **enemyBullets** — projectile arrays
- **particles**, **powerUps** — visual and item arrays
- **score, multiplier, chain** — scoring mechanics
- **stage, stageKills** — progression

### Game Objects

All objects are plain JavaScript objects (not classes). Key structures:

**Player:**
- `x, y` — screen position (center)
- `w, h` — collision size
- `speed` — movement speed (260 px/s)
- `weapon` — 'vulcan' or 'laser'
- `level` — weapon upgrade level (1–5+)
- `cooldown, rapid` — fire rate modulation
- `invuln` — invulnerability timer (after hit)

**Enemy:**
- `type` — 'grunt', 'interceptor', or 'tank' (defines HP, speed, score)
- `x, y` — position
- `hp` — health
- `shootTimer` — reload between shots
- `weave` — (interceptors only) horizontal dodge behavior

**Boss:**
- `hp, maxHp` — health bar
- `design` — colors and appearance
- `telegraph` — wind-up timer for telegraphed attacks
- `pattern` — current attack sequence
- `spawnTimer` — controls burst spawn timing

### Difficulty & Progression

**Difficulty** (stored in localStorage) affects enemy spawn rate, movement speed, and boss HP:
- Easy: slower spawns, weaker enemies
- Normal: balanced
- Hard: faster spawns, more aggressive enemies, 30% boss HP boost

**Stage Progression**: Kill 18 enemies to advance. Boss fight at stage transition. Bosses increase in HP by stage number.

### Rendering System

Canvas rendering is split by layer:

1. **Background** — gradient sky (changes per stage/theme)
2. **Game objects** — rendered by type (drawPlane, drawBossShip, particles, enemies)
3. **HUD** — DOM elements (not canvas) — score, health hearts, bomb count, weapon name
4. **Overlays** — flash effect, vignette (critical health), combo popups

Enemy/player drawing uses procedural geometry (no sprites), making the game easily moddable.

### Audio

All sound effects are synthesized in real-time using WebAudio API:
- **beep()** — tone generator with frequency sweep, filter, panning
- **noiseBurst()** — white noise with filter envelope

Sound pool is created on-demand on first interaction (`ensureAudio()`). Muting is toggled via button and persisted to localStorage.

### Input Handling

Three input modes coexist:
1. **Keyboard** — Arrow/WASD for movement, Space for fire, X/Shift for bomb
2. **Mouse** — Player follows pointer when hovering canvas
3. **Touch** — Virtual joystick + fire/bomb buttons (shown on touch devices)

Priority: Keyboard/touch always take precedence over mouse.

## Common Development Tasks

### Running the Game

Simply open `index.html` in any browser. No build step required.

```bash
# Start a local HTTP server to serve the game
python3 -m http.server 8000
# Then navigate to http://localhost:8000/index.html
```

### Adding a New Enemy Type

1. Add entry to `ENEMY_TYPES` constant (around line 502):
   ```javascript
   ENEMY_TYPES.newType = { 
     w: 30, h: 30, hp: 2, speedMul: 1.2, 
     color: '#new-color', dark: '#dark-color', 
     score: 20, minStage: 3 
   };
   ```
2. Add explosion sound effect to `sfx` object if needed
3. Add case in `spawnEnemy()` to include it (around line 1022)
4. Add rendering logic in the enemy rendering section of gameLoop

### Adjusting Game Balance

All game constants are at the top of the file:
- **DIFFICULTIES** (line 485) — spawn rates, enemy speeds, boss HP multiplier
- **KILLS_PER_STAGE** (line 491) — enemies per stage
- **Player speed** (line 877) — movement velocity
- **Weapon properties** — in the update() function (search for `player.weapon`)

### Modifying Visuals

- **Sky colors** — THEMES array (line 493)
- **Enemy colors** — ENEMY_TYPES.*.color fields
- **Boss appearance** — BOSSES array (line 508), drawBossShip() function
- **Animations** — CSS keyframes in `<style>` section

### Testing Specific Scenarios

The game writes to browser console for basic debugging:
- Set `running = true` and `stage = X` in browser DevTools to jump to stage X
- Set `player.hp = 10` to increase health
- Call `spawnBoss()` to immediately spawn a boss

### Collision Detection

Collision is rect-based and checked per update:
- **Bullet-to-enemy** — in update() around line 1200
- **Enemy bullet-to-player** — checks `player.invuln` timer to allow grace period
- **Bomb** — clears all on-screen enemies/bullets in one frame

## Performance Considerations

The game uses **adaptive quality** to maintain 60 FPS on weaker devices:
- Quality level 0 (full) — shadows, glow effects
- Quality level 1 (medium) — reduced shadow blur
- Quality level 2 (low) — no shadows, fewer particles

Tracks frame times and adjusts quality automatically (around line 476).

## Accessibility

- **Reduced-motion mode** — Toggle via 🎆 button, disables animations and screen shake for prefers-reduced-motion users
- **Screen reader support** — Uses `aria-live` region for game state announcements
- **Color contrast** — All UI text uses white on dark/semi-transparent backgrounds

## Browser Compatibility

Requires:
- Canvas 2D Context API
- WebAudio API (with fallback graceful degradation if unavailable)
- localStorage
- Modern CSS (flexbox, CSS variables)

Tested on desktop and mobile (iOS Safari, Android Chrome).

## Key Implementation Details

### Stage Management

`stage` variable increments when `stageKills >= KILLS_PER_STAGE`. Boss spawns via `spawnBoss()` using BOSSES array (round-robin + difficulty scaling).

### Weapon System

Two weapons toggle via pickup:
- **Vulcan** — fires 2 bullets per trigger, faster fire rate
- **Laser** — fires 1 wider beam, longer cooldown, higher damage per hit

Weapon `level` (1–5+) increases damage. Level increases persist across weapon switches and are found in pickups.

### Scoring & Multiplier

- Base enemy score × difficulty × level multiplier
- **Chain system** — consecutive kills within 0.8s without breaking chain increase multiplier (1x → max 3x)
- Combo popup displays current multiplier when triggered

### Boss Attack Patterns

Bosses use a simple state machine:
- `telegraph` > 0 — wind-up phase, boss flashes white
- Spawns enemy bullets in burst at end of wind-up
- Repeats with random interval (1–2 seconds)

Boss defeated when `hp <= 0`; triggers stage clear sound, next stage begins on next spawn cycle.

## Future Extension Points

- **New enemy types** — add to ENEMY_TYPES, implement spawn logic and rendering
- **Weapon variants** — add new weapon to weapon selection logic
- **Boss attack patterns** — modify `updateBoss()` to add phase mechanics or special moves
- **Visual themes** — expand THEMES array with new sky gradients
- **Progressive difficulty** — modify DIFFICULTIES or create dynamic scaling based on playtime
