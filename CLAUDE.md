# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project Overview

**Sky Fighter** is a Raiden-style vertical shoot-'em-up game built as a single, self-contained HTML5 Canvas application. The entire game (HTML, CSS, and JavaScript) is contained in a single `index.html` file (~1700 lines) with no external dependencies.

### Key Technologies
- **Canvas API** for 2D rendering
- **Web Audio API** for sound effects (dynamic synthesis, no audio files)
- **Touch Events API** for mobile support
- **Vanilla JavaScript** (no frameworks)
- **CSS animations** for effects (hearts, vignette, flash)

## Game Architecture

### Core Game Systems (in order of appearance in code)

1. **Adaptive Performance System** (`trackPerf` function)
   - Monitors frame times and dynamically adjusts rendering quality
   - Three quality levels adjust enemy count and bullet complexity
   - Important: Changes are frame-rate-driven, not manually toggled

2. **Entity System**
   - All game entities (player, enemies, bullets) are objects pushed to `entities` array
   - Each entity has: `x`, `y`, `vx`, `vy`, `w`, `h`, `hp`, `dead` properties
   - Update loop: spawn → update positions → check collisions → remove dead

3. **Weapon System**
   - Player has multiple weapons with levels (1-3)
   - Weapons: VULCAN (spread), LASER (concentrated), SCATTER (bouncy)
   - Weapon selection cycles via `W` key; level increases are enemy drops
   - Fire rate and damage increase per level

4. **Enemy System** (`ENEMY_TYPES` constant)
   - **Grunt**: Basic red enemy (1 HP, score 10)
   - **Interceptor**: Fast orange enemy (1 HP, score 15, weaving movement)
   - **Tank**: Heavy purple enemy (4 HP, score 35)
   - Spawning difficulty-scaled, enemies shoot bullets and drop powerups

5. **Boss System**
   - Spawned after stage progression, has unique attack patterns
   - Three boss types with different behaviors (phases)
   - Boss health bar (`#bossBarFill`) updates during combat
   - Boss defeat triggers stage increment and wave reset

6. **Bomb System**
   - Smart bombs (activated with `X` or Shift) clear nearby enemies
   - Creates expanding damage zone (visual + functional)
   - Bomb count resets between lives, drawn from `bombs` constant

7. **Collision Detection**
   - Simple AABB (axis-aligned bounding box) with padding
   - Check: player ↔ enemy bullets, player ↔ enemies, bullets ↔ enemies
   - Collisions trigger damage, knockback, visual effects (screen shake, vignette flash)

8. **Audio System**
   - Uses Web Audio API (`AudioContext`) to synthesize sounds (no files loaded)
   - `playSound(freq, duration, type)` for sound effects
   - Volume muted/unmuted via UI button; persisted to localStorage

9. **Difficulty Scaling** (`DIFFICULTIES` constant)
   - Adjusts enemy spawn rate, speed, AI shoot chance, player lives, bombs
   - Hard mode: fewer spawns but faster + smarter enemies
   - Easy mode: more lives, more bombs, easier enemies

### Rendering Pipeline

1. **Clear canvas** with gradient background
2. **Render static elements**: HUD, boss bar, UI buttons
3. **Render game entities**: player, enemies, bullets (in order)
4. **Render effects**: screen shake (translate canvas), flash white, vignette overlay
5. **Render controls** (on mobile)

### Game Loop

```
requestAnimationFrame → update() → {
  - Adaptive performance check
  - Player input handling (keyboard + mouse/touch)
  - Enemy spawning (rate-based on difficulty + stage)
  - Entity updates (position, velocity, lifetime)
  - Collision detection
  - Remove dead entities
  - Update HUD (score, hearts, weapons, boss bar)
  - Render
}
```

## Key Code Locations

| Section | Lines | Purpose |
|---------|-------|---------|
| HTML Structure | 124–220 | Canvas, HUD, controls, buttons, overlays |
| CSS Styling | 10–122 | Layout, animations, responsive design, mobile |
| IIFE Wrapper + Globals | 221–260 | Canvas setup, global state (player, entities, difficulty) |
| Utility Functions | 280–350 | Helper: clamp, distance, random, audio, screen shake |
| Player Object | 360–420 | Position, health, weapon, firing, movement |
| Entity Update Loop | 450–600 | Collision, movement, damage, despawn logic |
| Rendering | 700–850 | Draw player, enemies, bullets, effects |
| Event Listeners | 900–1100 | Keyboard, mouse, touch controls |
| Game Flow Control | 1150–1300 | Start, pause, game over, stage progression |

## Common Tasks

### Adding a New Enemy Type

1. Add entry to `ENEMY_TYPES` constant (after `tank`):
   ```javascript
   newType: { w: 40, h: 40, hp: 2, speedMul: 1.1, color: '#color', dark: '#darkcolor', score: 25, minStage: 3 }
   ```
2. Update `spawnEnemy()` to include new type in the spawn condition
3. Test on each difficulty level (Easy, Normal, Hard)
4. Check performance at high spawn rates (use hard mode)

### Adding a New Weapon

1. Add to weapon array in player initialization
2. Update `fireWeapon()` to handle the new weapon's bullet pattern
3. Test fire rate, damage, and visual feedback
4. Add to weapon name rotation if needed

### Adjusting Game Balance

1. **Enemy difficulty**: Modify `DIFFICULTIES` constant
2. **Enemy stats**: Edit `ENEMY_TYPES` (hp, speedMul, shootChance)
3. **Spawning rate**: Change `spawnBase` and `spawnMin` in DIFFICULTIES
4. **Player survivability**: Adjust `lives`, `bombs`, or `bossHpMul`
5. **Weapon damage**: Find the damage multiplier in `fireWeapon()` or collision handling

### Performance Optimization

- The adaptive quality system reduces enemy count and bullet complexity at lower frame rates
- If adding features, ensure they don't break the quality tiers
- Test on mobile devices (lower CPU/GPU than desktop)
- Use DevTools Performance tab to identify hot loops

## Testing Approach

**No automated test suite exists.** Testing is manual:

1. **Play through each difficulty** (Easy → Normal → Hard)
2. **Test all weapons** (cycle with `W` key, check fire rate and spread)
3. **Test bombing** (trigger bombs in enemy-dense situations)
4. **Check mobile controls** (drag left zone to move, tap FIRE)
5. **Verify visual effects** (screen shake, vignette flash, animations)
6. **Check accessibility** (mute audio, toggle reduced motion, test screen reader)
7. **Performance check** (DevTools FPS, no jank at quality=0/1/2)

### Rapid Testing Tips

- Press `D` to jump to boss fight (debug hotkey, if implemented)
- Open DevTools → Lighthouse Performance to catch regressions
- Check localStorage persistence (mute state, best score, motion preference)

## State Management

Game state is stored in these global objects (within the IIFE):

| Variable | Purpose |
|----------|---------|
| `player` | Position, health, ammo, weapon level |
| `entities` | Array of all active game objects (bullets, enemies, effects) |
| `difficulty` | Current mode (easy/normal/hard) |
| `stage` | Wave counter (increments after boss kill) |
| `lives` | Remaining player lives |
| `bombs` | Remaining bombs in current life |
| `score`, `best` | Current and best scores |
| `mult` | Score multiplier (increases with enemy kills) |
| `gameRunning`, `paused` | Game flow state |

localStorage is used for: best score, mute state, motion preference.

## Mobile vs. Desktop Considerations

- **Touch controls**: Joystick-like input on left, action buttons on right
- **Responsive units**: HUD font sizes use `clamp()` for viewport scaling
- **Viewport fit**: Meta tag handles notches on iPhones
- **Performance**: May need to adjust quality thresholds for weaker devices
- **Safe areas**: HUD respects `env(safe-area-inset-*)` for notches/edges

## Common Gotchas

1. **Entity lifetime**: Dead entities have `dead = true` and are skipped; array is pruned each frame
2. **Collision padding**: Collisions use `w - 4` / `h - 4` (slight shrink for playability)
3. **Screen shake**: Applies canvas translate; ensure effects reset after rendering
4. **Vignette effect**: Uses CSS animation when activated; can conflict with reduced-motion preference
5. **Audio context**: May be suspended on mobile until first user gesture
6. **Z-index layering**: Ensure visual effects (flash, vignette) don't clip unexpected elements

## Extending the Game

### Ideas That Work Well With Architecture

- **New enemy AI patterns**: Modify movement logic in entity update loop
- **Weapon effects**: Add particles or trails in the rendering section
- **Powerups**: Add new drop types in the collision handling
- **Score combos**: Extend the multiplier logic on enemy kills
- **Boss phases**: Add conditional logic in boss entity update

### What Would Require Refactoring

- **Multiple simultaneous players**: Single player entity is hardcoded
- **Level/map design**: No map/zone system exists (all full-screen spawning)
- **Leaderboard/persistence**: Would need backend or cloud storage
- **Modular sprite/animation system**: Currently all drawn procedurally with Canvas primitives
