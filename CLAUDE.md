# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project Overview

**Sky Fighter** is a Raiden-style vertical shoot-'em-up game built as a self-contained HTML5 application. The entire game logic, rendering, audio synthesis, and UI are implemented in a single 1670-line HTML file (`index.html`).

- **Type**: Vanilla JavaScript game with no external dependencies
- **Platform**: Browser-based (Chrome, Firefox, Safari, Edge)
- **Key Technologies**: Canvas 2D API, WebAudio synthesis, touch/keyboard input
- **Mobile**: Full touch support with on-screen joystick and action buttons

## File Structure

```
Google_4/
├── index.html          # Complete game implementation (HTML + CSS + JS)
├── README.md           # Basic project description
└── CLAUDE.md          # This file
```

There is no build step, bundler, or package manager. The game is playable by opening `index.html` in any modern browser.

## Architecture Overview

### 1. Game Loop & Rendering

The core engine uses `requestAnimationFrame` for a 60fps game loop:

- **Loop function** (line 1640): Main animation frame callback
- **Update cycle** (line 1148): Physics, collision detection, entity state updates
- **Draw cycle** (line 1520): Canvas rendering, backdrop animation, sprite drawing
- **Performance scaling** (line 474): Adaptive quality (0=full, 1=medium, 2=low) based on frame time

**To modify game speed/difficulty**: Adjust `DIFFICULTIES` object (line 485) or difficulty multipliers in `updateBoss()` (line 1411).

### 2. Entity System

Game entities (player, enemies, bullets, particles) are plain JavaScript objects with position, velocity, and type-specific properties:

- **Player** (line 877): `{ x, y, w, h, speed, weapon, level, cooldown, invuln }`
- **Bullets** (line 1129-1145): `{ x, y, vx, speed, w, h, pierce, laser }`
- **Enemies** (line 1032-1040): `{ x, y, type, hp, pattern, canShoot, weave, ... }`
- **Boss** (line 1047-1052): Similar to enemies but with phases and attack patterns
- **Particles** (line 1076-1084): Simple explosion/visual effects with velocity and lifetime

All entities are stored in arrays (`enemies[]`, `bullets[]`, `enemyBullets[]`, `powerUps[]`, `particles[]`). Collision detection iterates these arrays (lines 1232-1295).

### 3. Audio System

Procedural sound effects using WebAudio API (no external audio files):

- **Initialization** (line 545): Lazy-init audio context on first user interaction
- **beep()** (line 566): Square/sine/sawtooth tones with frequency sweeps
- **noiseBurst()** (line 597): White noise with frequency filtering
- **SFX dictionary** (line 624): Named effects (shoot, laser, explode, bomb, powerup, bossHit, etc.)
- **Panning** (line 562): Spatial audio positioned relative to canvas X coordinate

**To add sound effects**: Add functions to the `sfx` object and call them from game events. Use `beep()` and `noiseBurst()` primitives.

### 4. Rendering & Visual Effects

Canvas-based 2D rendering with stage-based theming:

- **Backgrounds** (line 1529-1544): Gradient sky with scrolling clouds per stage theme
- **Sprites** (line 913-1020): Procedurally drawn planes (player, enemies, boss) using Canvas paths
- **Particles** (line 1628-1636): Simple circles with fading alpha and gravity
- **Screen shake** (line 727): Adds randomness to canvas transform during impacts
- **Flash overlay** (line 95-96, 698-709): Full-screen white flash for damage feedback
- **Stage themes** (line 493): 6 different color palettes applied based on stage number

**To modify visuals**: Edit `drawPlane()`, `drawBossShip()`, colors in `THEMES[]`, or particle spawn code.

### 5. Input Handling

Supports mouse, keyboard, and touch input:

- **Keyboard** (line 769): Arrow keys / WASD for movement, Space/X/Shift for firing/bombs, P/Esc for pause
- **Mouse** (line 847): Move to steer, click to fire (glides toward cursor)
- **Touch** (line 814-834): Left side joystick for movement, right side fire/bomb buttons
- **Firing state** (line 783, 1176): Continuous fire when key/button held down
- **Input priority**: Keyboard/joystick > mouse (they never conflict)

**To change controls**: Modify key bindings in keyboard event handlers or touch zone dimensions.

### 6. Difficulty & Persistence

Three difficulty modes with localStorage persistence:

- **DIFFICULTIES object** (line 485): `easy`, `normal`, `hard` with spawn rates, enemy speed, HP multipliers
- **Storage keys** (line 463-466): Best score, muted state, selected difficulty, reduced-motion preference
- **Difficulty affects**: Enemy spawn frequency, enemy speed, shoot chance, boss HP

**To add new difficulties**: Add entry to `DIFFICULTIES` and corresponding buttons in HTML.

### 7. Boss System

Boss encounters have unique designs, attack patterns, and difficulty scaling:

- **BOSSES array** (line 508): 6 boss designs (VANGUARD, BEHEMOTH, SPECTER, DREADNOUGHT, WRAITH, TITAN)
- **BOSS_PATTERNS array** (line 1372): Indexed 1:1 with BOSSES, each has 3 attack phases
- **Pattern functions** (line 1324-1370): `fireRadial()`, `fireAimedSpread()`, `fireColumns()`, etc. spawn bullets in specific formations
- **Boss lifecycle** (line 1043-1056, 1411-1439): Entering animation → patternIndex cycles → telegraph phase → attack phase

**To create new boss**: Add entry to `BOSSES` array and corresponding pattern array in `BOSS_PATTERNS`. Use existing fire functions or create new ones.

### 8. Game States

- **`running`** (line 865): Game is active
- **`paused`** (line 865): Game loop still runs but entities don't update
- **`gameOver`** (line 865): Final state after losing all lives
- **`overlay`** (line 200): Menu UI shown before game start or after end

State transitions: `resetGame()` → update/draw → `endGame()` or `togglePause()`.

## Common Development Tasks

### Running the Game

Simply open `index.html` in a browser (no build required). The game is fully playable immediately.

**To test on mobile**: Use browser DevTools device emulation or access the file on a mobile device.

**To test specific difficulty/features**:
- Use browser DevTools console to call functions: `resetGame(); running = true;`
- Or modify `DIFFICULTIES` values directly in the source

### Adding New Enemy Types

1. Add entry to `ENEMY_TYPES` object (line 502-506) with stats: `w`, `h`, `hp`, `speedMul`, `color`, `dark`, `score`, `minStage`
2. Add optional weave behavior flag if needed
3. Update `spawnEnemy()` (line 1022-1041) conditional logic to include the new type
4. If needed, add explosion sound to `EXPLODE_SFX` mapping (line 678)

### Adding New Weapon Types

1. Modify `fireWeapon()` function (line 1124-1146) to handle new weapon type
2. Add weapon name to `weaponLabel()` (line 871)
3. Adjust bullet spread, speed, or pierce values based on level

### Tweaking Game Balance

- **Enemy difficulty**: Adjust `DIFFICULTIES[].spawnBase`, `spawnMin`, `enemySpeed`, `enemyShootChance`
- **Player speed/power**: Change `player.speed` (line 877) or bullet speed/spread in `fireWeapon()`
- **Boss difficulty**: Multiply `hpMul` in `BOSSES` or adjust damage/speed in patterns
- **Rewards**: Change base scores in `ENEMY_TYPES` or multiplier logic in `addScore()`

### Adding Accessibility Features

The game already supports:
- **Reduced motion** (line 349-358): Disables animations for users with vestibular sensitivity
- **Screen reader announcements** (line 468-471, `announce()`)
- **Keyboard-only control** (no mouse required)

**To add more**: Use `announce()` for important state changes, respect `reducedMotion` flag in visual effects.

## Code Patterns & Style

### Entity Management

All game entities follow a simple object pattern with optional lifecycle:

```javascript
// Spawning
enemies.push({ x, y, type, hp, ... });

// Updating
enemies.forEach(e => { e.y += e.speed * dt; });

// Cleanup
enemies = enemies.filter(e => e.y < canvas.height);

// Collision
for (let i = enemies.length - 1; i >= 0; i--) {
  if (rectsOverlap(enemies[i], bullet)) {
    // remove and handle
  }
}
```

### Particle System

Simple particle spawning for explosions and visual feedback:

```javascript
spawnExplosion(x, y, color, count);
```

Particles are updated each frame and automatically removed when `life <= 0`.

### Canvas Coordinate System

- Origin (0,0) is top-left
- X increases rightward, Y increases downward
- Canvas size: 480×720 (BASE_W, BASE_H) scaled to fit window
- Player starts at `(canvas.width/2, canvas.height - 100)`

### Performance Optimization

- **Quality scaling** (line 474-483): Frame times tracked; quality reduced if avg > 25ms
- **Particle count scaling** (line 1074): Fewer particles at lower quality settings
- **Glow effects conditional** (line 1579): Shadow effects only at full quality
- **Reduced motion support** (line 728-729): Less intense shake at 15% when enabled

## Testing Considerations

**Golden path**: Start game → spawn enemies → defeat enemies → trigger boss → defeat boss → advance stage → new boss → game over.

**Edge cases**:
- Pause during boss telegraph
- Fire bomb while enemy bullets are incoming
- Hit invulnerability frame timing
- Weapon switch/level up mechanics
- Stage/theme cycling through multiple cycles
- Mouse vs keyboard vs touch input switching
- Low-performance device quality scaling

**Accessibility check**:
- Play with reduced motion enabled
- Play with audio muted
- Play keyboard-only
- Play touch-only on mobile
- Listen to screen reader announcements

## Performance Characteristics

- **Frame time**: Target 60fps (16.67ms per frame)
- **Entity limits**: Typically 50-200 enemies/bullets on-screen without noticeable slowdown
- **Memory**: No memory leaks; entities cleaned up each frame, audio buffers released
- **Quality auto-scaling**: Adapts to device performance over 45 frames

## Common Gotchas

1. **Canvas sizing**: Always use `canvas.width` and `canvas.height` for game math (not CSS size). DOM size is separate for scaling.
2. **Audio context**: Must be initialized after user gesture (first click/key).
3. **Delta time capping** (line 1644): Prevents huge jumps if tab loses focus; important for physics stability.
4. **Entity array mutations**: Iterate backwards when removing during collision checks to avoid skipping items.
5. **Invulnerability frames**: Player invuln state (line 1174, 1459) prevents repeated damage in same frame.
6. **Stage cycling**: Themes and bosses use modulo (`%`) to cycle back after stage 6.

## Future Extensibility

The codebase is designed for easy extension:

- **New bosses**: Add to `BOSSES` and `BOSS_PATTERNS` arrays
- **New enemies**: Add to `ENEMY_TYPES`, update spawn logic
- **New weapons**: Extend `fireWeapon()`, update `weaponLabel()`
- **New stages**: Extend `THEMES` array, adjust spawn difficulty per stage
- **New effects**: Use particle system or WebAudio synthesis
- **Leaderboard**: Use localStorage or external API (requires cross-origin handling)

The single-file structure keeps everything visible and makes refactoring straightforward.
