# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project Overview

**Sky Fighter** is a Raiden-style vertical shoot-'em-up game built as a single HTML5 file with Canvas 2D rendering. It features difficulty levels, weapon progression, boss encounters, and adaptive performance scaling.

The entire game (~1,669 lines) is contained in `index.html` with CSS and JavaScript inline.

## Running and Testing

### View the Game
```bash
# Open directly in browser from the file system or use a simple HTTP server:
python3 -m http.server 8000
# Then visit: http://localhost:8000/index.html
```

### Testing Changes
1. Open `index.html` in a web browser
2. Make edits to the file
3. Refresh the browser to see changes immediately
4. Test on both desktop (keyboard/mouse) and mobile (touch controls)

### Key Testing Scenarios
- **Desktop controls**: Arrow keys/WASD to move, Space to fire, X/Shift for bomb, P/Esc to pause
- **Mobile controls**: Drag left zone to move, tap FIRE/BOMB buttons
- **All difficulties**: Easy, Normal, Hard (affect spawning, enemy behavior, lives)
- **Boss encounters**: Should appear after 18 kills per stage
- **Weapon progression**: Collect weapon pickups to upgrade VULCAN/PULSE/LASER
- **Screen sizes**: Test at various viewport sizes (mobile, tablet, desktop)

## Code Architecture

### Game Loop (line ~1600)
```javascript
function loop(now) {
  trackPerf(dt);      // Adaptive performance scaling
  if (running && !paused) {
    update(dt);       // Physics, collision, AI
    draw();           // Canvas rendering
  }
  requestAnimationFrame(loop);
}
```

### Main Game State (lines 420-790)
Global variables track:
- **Player**: `player` object with position, health, weapon state
- **Entities**: `enemies[]`, `enemyBullets[]`, `bullets[]`, `powerUps[]`, `particles[]`
- **Game flow**: `running`, `paused`, `lives`, `score`, `stage`, `kills`
- **Difficulty**: `difficulty` object controls spawn rates, enemy speed, enemy fire rate
- **Boss**: `boss` object for stage-ending encounter

### Key Constants (lines 485-550)
- `DIFFICULTIES`: Three difficulty profiles affecting spawn behavior and HP multipliers
- `KILLS_PER_STAGE`: 18 kills needed to trigger boss encounter
- `THEMES`: Six visual themes with sky gradients (controls stage aesthetics)
- `WEAPONS`: Three weapon types (VULCAN, PULSE, LASER) with distinct fire patterns
- `ENEMY_TYPES`: Light fighter, tank (armored), heavy (rapid fire)

### Update Function (lines 850-1200)
Handles each frame's game logic:
- Player movement and input handling
- Weapon firing and cooldown
- Enemy spawning based on difficulty
- Enemy AI and shooting patterns
- Collision detection (bullets vs enemies, enemy bullets vs player)
- Particle and effect updates
- Scoring and stage progression
- Boss health tracking

**Performance tip**: Collision detection uses simple bounding boxes (AABB). Avoid adding pixel-perfect collision without testing frame rate impact on lower-end devices.

### Rendering Pipeline (lines 1250-1550)
Organized by layer:
1. Canvas background (sky gradient based on theme)
2. Cloud parallax effect
3. Player ship and targeting crosshair
4. Bullets and enemy fire
5. Enemy ships with health bars (tanks)
6. Boss ship (if active)
7. Power-up pickups (rotating)
8. Particle effects (explosion debris)

**Glow effects**: Lines ~1300-1320 apply `ctx.shadowBlur` when `quality === 0` (high-quality rendering mode).

### Input Handling (lines 1380-1450)
- **Keyboard**: Arrow keys, WASD, Space, X, Shift, P, Esc (parsed in `update()`)
- **Mouse**: Movement tracked via `mouseTarget`; click to fire
- **Touch**: `moveZone` for flying position; `fireBtn`/`bombBtn` buttons
- **Joystick**: Virtual joystick UI built/destroyed dynamically on touch devices

### Audio System (lines 550-610)
- Uses `AudioContext` for sound synthesis (no external audio files)
- All sounds are procedurally generated during gameplay
- `ensureAudio()` creates the audio context on first user interaction (browser requirement)
- `playSound(freq, duration, type)` covers firing, explosions, etc.
- `muteBtn` toggles sound via `muted` flag

### Accessibility & Preferences (lines 620-680)
- **Screen reader**: `srLive` (aria-live region) announces scoring events
- **Reduced motion**: `body.reduced-motion` class disables animations per user preference
- **LocalStorage keys**:
  - `skyfighter.bestScore`: Best score persistence
  - `skyfighter.muted`: Audio preference
  - `skyfighter.difficulty`: Last selected difficulty
  - `skyfighter.reducedMotion`: Motion preference

## Common Modification Patterns

### Add a New Weapon Type
1. Add entry to `WEAPONS` constant (line ~520):
   ```javascript
   WEAPONS.newWeapon = { fire: 40, label: 'NEW', damage: 18, ... };
   ```
2. Add fire pattern logic in `update()` weapon firing section (~line 1050)
3. Update `weaponNameEl.textContent` assignment to recognize new type

### Adjust Difficulty Curves
1. Modify `DIFFICULTIES` constant (line 485) to change:
   - `spawnBase`/`spawnMin`: Lower = more enemies spawning
   - `enemySpeed`: Enemy flight speed multiplier
   - `enemyShootChance`: Fire probability per frame (0.0–1.0)
   - `bossHpMul`: Boss health multiplier

### Change Visual Theme
1. Add new theme object to `THEMES` array (line 493):
   ```javascript
   { sky: [/* gradient stops */], cloud: 'r,g,b', sun: 'r,g,b', name: 'Theme Name' }
   ```
2. Themes are cycled by stage; they auto-map to stages sequentially

### Add Screen Effects (Vignette, Flash)
- **Vignette pulse** (red damage indicator): Set `vignetteEl.classList.toggle('on')`
- **White flash** (hit/bomb): Trigger class `#flash.hit` or `#flash.bomb` with `flashEl.classList.add('hit')`
- Both have CSS animations (lines 71-101); reduced motion disables them

### Modify Spawn/Wave Behavior
1. `spawnChance` calculation at line ~1100 controls spawn rate per frame
2. `spawnList` array at line ~550 defines enemy formation sequences
3. Enemy type selection at line ~1110 uses `Math.random() * 100`

### Physics Tweaks
- **Player speed**: `player.speed` constant (line ~745)
- **Bullet speed**: Set per-weapon in `WEAPONS.*.speed` (line ~520)
- **Enemy speed**: `difficulty.enemySpeed` multiplier (line 485)
- **Gravity/acceleration**: Not used; all movement is direct velocity

## Performance & Optimization

### Adaptive Quality System (lines 473-483)
Game monitors frame times and scales rendering quality automatically:
- **Quality 0** (full): Shadow/glow effects enabled (`ctx.shadowBlur`)
- **Quality 1** (medium): Reduced effects
- **Quality 2** (low): Minimal rendering

Triggers on average frame time over 45-frame window:
- If avg > 1/40 (25ms) → decrease quality
- If avg < 1/56 (~18ms) → increase quality

**Do not add expensive effects** (particle count, canvas size) without stress-testing on target devices.

### Entity Pooling
Currently uses simple array push/splice for entities. For performance-critical additions, consider object pooling if entity counts exceed ~200.

### Canvas Size
Dynamically set to match viewport (lines 747-750). Very large canvases (4K) will impact performance; consider capping max size for lower-end devices.

## File Structure Notes

- **Lines 1–359**: HTML structure and CSS styling
  - Responsive layout with `clamp()` for scaling
  - Safe area insets for notched phones
  - Dark theme (`color-scheme: dark`)
- **Lines 360–420**: HTML body with overlay, HUD, touch controls
- **Lines 422–end**: Single IIFE wrapping all game logic (prevents global pollution)

## Deployment & Testing Checklist

- [ ] Game runs at 60 FPS on target devices (use DevTools Performance tab)
- [ ] Touch controls work on mobile (drag left, tap buttons)
- [ ] Keyboard controls respond correctly (arrows, space, x, p)
- [ ] Audio works when unmuted (test in browser that supports AudioContext)
- [ ] Best score persists across page reloads
- [ ] Reduced motion preference is respected
- [ ] Boss appears after 18 kills
- [ ] Weapon pickups correctly upgrade weapon level
- [ ] All three difficulty levels spawn enemies at expected rates
