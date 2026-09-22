# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project Overview

**Sky Fighter** is a Raiden-style vertical shoot-'em-up game built as a single HTML file with embedded CSS and JavaScript. No build process or external dependencies are required—just open `index.html` in a browser to play.

### Key Features
- **Player ship** with mouse/touch/keyboard controls
- **Multiple weapons** that upgrade through pickups (base, spread, piercing)
- **Smart bombs** (limited per stage) with area-of-effect damage
- **Enemy types**: grunts, fast interceptors, heavy tanks
- **Boss battles** across multiple stages with unique boss designs
- **Difficulty levels**: Easy, Normal, Hard with scaling parameters
- **Procedural visuals**: 6 themed sky backdrops with dynamic lighting
- **Web Audio synth**: No external audio assets; all sounds generated procedurally
- **Adaptive performance**: Scales quality down on low-end hardware
- **Accessibility**: Reduced motion support, screen reader announcements, keyboard-safe input
- **Persistence**: High scores and settings stored in localStorage

## Working with the Codebase

### Running/Testing
Simply open `index.html` in a browser. No local server required.

To test on mobile: open on a phone or use your browser's device emulation (F12 → Device Toolbar).

### Code Architecture

The entire game lives in a single `<script>` block (line 422+) inside `index.html`. The structure follows this pattern:

1. **DOM Setup & Globals** (~line 423–500)
   - Canvas and UI element references
   - Constants: `DIFFICULTIES`, `THEMES`, `ENEMY_TYPES`, `BOSSES`
   - Configuration like `KILLS_PER_STAGE`, storage keys

2. **Audio Engine** (~line 542–650)
   - `ensureAudio()`: Lazy-initializes WebAudio context on first sound
   - `beep(freq, dur, type, vol, opts)`: Core synth; plays tones with optional sweep, filter, pan, delay
   - Sound effects are procedurally generated (e.g., weapon fire, explosions, boss hits)

3. **Game State & Update Loop** (~line 650–1000)
   - `gameState`: Object tracking game mode (`start`, `play`, `pause`, `gameOver`), score, lives, stage, combo
   - `player`: Ship position, weapon, bomb count, fire cooldown
   - `enemies`, `bosses`, `projectiles`, `pickups`: Gameplay entities
   - `update()`: Main tick function; advances all entities, checks collisions, respawns/despawns
   - `render()`: Draws canvas background, all entities, UI overlay

4. **Entity Managers** (~line 1000–1200)
   - `spawnEnemy(type, x, y)`: Creates enemy with AI (seeks player, shoots with probability)
   - `spawnBoss(index)`: Places boss at top-center, scales HP by difficulty
   - `spawnProjectile(x, y, vx, vy, owner, ...)`: Bullets/bombs for player and enemies
   - `spawnPickup(x, y, type)`: Weapon upgrades and bomb refills

5. **Collision & Damage** (~line 1200–1300)
   - Rectangle collision checks between player/enemies, projectiles/enemies, player/pickups, boss arena
   - Damage application, health tracking, entity removal
   - Particle effects (visual feedback, not particles—just canvas drawing)

6. **Input Handling** (~line 1300–1450)
   - Mouse movement (player tracking)
   - Keyboard: arrow keys or WASD for movement, Space for fire, B for bomb
   - Touch: virtual joystick (left zone) and buttons (fire, bomb)
   - Pause button, difficulty selection, mute/motion toggles

7. **Boss AI & Patterns** (~line 1450–1550)
   - Each boss type in `BOSSES[]` has hardcoded movement and attack patterns
   - Boss position updates per frame based on stage and time
   - Projectile spawning on intervals

8. **UI & Overlays** (~line 1550+)
   - Start screen (difficulty selection), HUD (score, weapon, lives)
   - Combo pop animation, stage banners, boss health bar
   - Game over screen with final stats and "New Best" badge

### Making Common Edits

**Adjust difficulty parameters:**
```javascript
// Around line 485–489
const DIFFICULTIES = {
  easy:   { spawnBase: 75, ... },  // spawnBase: enemy spawn interval (ms)
  normal: { spawnBase: 58, ... },  // enemySpeed: AI speed multiplier
  hard:   { spawnBase: 40, ... }   // enemyShootChance: 0–1 probability
};
```

**Add a new enemy type:**
- Add entry to `ENEMY_TYPES` object (line ~502) with `{ w, h, hp, speedMul, color, dark, score, minStage, weave? }`
- Update spawn logic in `spawnEnemy()` to handle the new type
- Adjust `KILLS_PER_STAGE` if needed to balance progression

**Add a new boss:**
- Add entry to `BOSSES` array (line ~508) with `{ name, hullTop, hullBottom, wing, core, hpMul }`
- Extend boss position/attack logic in `update()` where boss movement is computed
- Projectile spawning for the boss is in the same section

**Add a new weapon:**
- Add to weapon system logic (search for weapon type checks in `update()`)
- Update `spawnProjectile()` to handle spread/pierce patterns
- Modify visual rendering in `render()` for projectiles

**Tweak balance numbers:**
- Enemy spawn rate: `spawnBase`, `spawnMin` in `DIFFICULTIES`
- Damage: `player.baseDmg` (line ~900), `bossDamage` (line ~1000)
- Player speed: `player.speed` constant
- Weapon upgrade probability: pickup spawn rate in `update()`

**Add/remove visual themes:**
- Edit `THEMES` array (line ~493)
- Each theme defines gradient colors, cloud/sun color components, and a name
- Themes rotate per stage automatically

### Performance Considerations

- **Adaptive quality**: `quality` variable (line ~474) downgrades rendering if frame times exceed 1/40 FPS. Check `if (quality ...)` blocks for level-specific rendering choices.
- **Entity pooling not used**: Enemies, projectiles, and particles are created/destroyed directly. For thousands of entities, consider object pools.
- **Canvas 2D is synchronous**: No partial rendering or async draws. Complex scenes may stall; optimize entity counts if needed.

### Accessibility & Browser Compatibility

- **Reduced motion**: Toggleable via button; disables animations and camera shake. Stored in localStorage.
- **Screen reader**: Live region (#srLive) for announcements. Use `announce(text)` function.
- **Safe area (notch) support**: CSS env(safe-area-inset-*) for mobile; buttons/HUD respect device notches.
- **Touch controls**: Auto-enabled on touch devices; separate from keyboard. Virtual joystick in bottom-left, buttons in bottom-right.
- **Audio**: WebAudio synth is not universally supported; gracefully falls back to muted (no errors).

## Project Structure

```
.
├── index.html         # Complete game (CSS + JS inline, 1669 lines)
└── README.md          # Minimal project description
```

## Storage & Defaults

LocalStorage keys used:
- `skyfighter.bestScore`: High score (number)
- `skyfighter.muted`: Sound toggle (0 or 1)
- `skyfighter.difficulty`: Last selected (easy/normal/hard)
- `skyfighter.reducedMotion`: Reduced motion toggle (0 or 1)

## Tips for Efficient Editing

1. **Line numbers are stable**: The game code is in a single file. Use Ctrl+G (or Cmd+G) to jump to line numbers mentioned in comments (e.g., "line ~485").
2. **Edit in any text editor**: No transpiling or minification. Refresh the browser to see changes.
3. **Console debugging**: Open DevTools (F12), type `gameState`, `player`, etc., to inspect live state during gameplay.
4. **Disable animations for quick testing**: Set `reducedMotion = true` in console to speed up visual debugging.
5. **Quick balance tweaks**: Modify `DIFFICULTIES` or entity `speed`/`hp` values, then refresh to test immediately.
6. **Audio debugging**: Mute and unmute via the button, or `muted = false` in console to toggle without UI.
