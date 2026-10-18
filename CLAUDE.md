# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project Overview

**Sky Fighter** is a Raiden-style vertical shoot-'em-up browser game. It's built as a single HTML file with embedded JavaScript and CSS — no build step, no external dependencies, no assets (synthesized audio only). The entire game runs in the browser with responsive canvas rendering and full touch/keyboard/mouse support.

## Architecture

### Single-File Structure
The game is a monolithic HTML file (`index.html`) with three main sections:
1. **HTML/DOM** (~420 lines): Canvas container, HUD elements, overlay UI, touch controls, accessibility elements
2. **CSS** (~360 lines): Responsive design with flexbox, animations (keyframes), glassmorphism effects, dark theme
3. **JavaScript** (~1250 lines): Game logic wrapped in an IIFE (Immediately Invoked Function Expression)

### Game Structure (JavaScript)

The JavaScript is organized as a large IIFE containing these functional areas:

#### State Management
- `player`: Position, health, weapon, cooldown
- `enemies[]`: Spawned enemies with pathfinding and behaviors
- `bullets[]`: Player projectiles
- `enemyBullets[]`: Enemy projectiles
- `boss`: Current boss if active
- `particles[]`: Visual effects
- `powerUps[]`: Collectible drops
- Global state: `score`, `lives`, `stage`, `chain`/`multiplier`, `running`/`paused`/`gameOver`

#### Core Systems

**Rendering** (`draw()`):
- Canvas clearing and shader setup (screen shake, gradient sky)
- Theme-based backgrounds with scrolling clouds
- Entity drawing: planes (player/enemies), bullets, particles, UI
- Performance-based quality reduction

**Update Loop** (`update(dt)`):
- Input handling and movement
- Firing and cooldowns
- Collision detection (bullets vs enemies, enemies vs player, power-ups)
- Spawn timing and stage progression
- Boss behavior updates

**Audio** (Web Audio API):
- Synthesized SFX using oscillators and noise bursts
- `beep()`: Frequency sweep synthesis
- `noiseBurst()`: Filtered white noise
- Spatial audio with panning
- Compressor and gain control

**Difficulty System**:
- 3 presets: Easy, Normal, Hard
- Per-difficulty: spawn rates, enemy speed, shoot chance, HP multipliers, bomb count
- Stored in localStorage

#### Game Loop
`requestAnimationFrame(loop)` → `update()` → `draw()` → repeat
- Delta-time based (capped at 50ms per frame)
- Adaptive performance tracking with quality downscaling

### Game Progression

1. **Stages**: Numbered 1-6 (then repeats), each with 18 kills-per-stage target
2. **Enemy Types**: 
   - `grunt` (red, 1 HP, common)
   - `interceptor` (orange, fast, weaving)
   - `tank` (purple, 4 HP, slow, harder on stage 2+)
3. **Bosses**: 6 designs (VANGUARD, BEHEMOTH, SPECTER, DREADNOUGHT, WRAITH, TITAN) with different attack patterns
4. **Weapons**: Vulcan (spread shots) and Laser (piercing beams), each with 3 levels
5. **Progression**: Kill minions → reach kill threshold → boss warning → boss fight → next stage

### Key Constants

- `BASE_W: 480, BASE_H: 720` — Canvas resolution
- `KILLS_PER_STAGE: 18` — Enemy kills needed before boss
- `DIFFICULTIES` — Per-difficulty spawn/HP/lives config
- `THEMES` — 6 sky themes (backgrounds + cloud colors)
- `ENEMY_TYPES` — 3 enemy templates
- `BOSSES` — 6 boss designs with color schemes
- `BOSS_PATTERNS` — Attack pattern functions per boss

## Development Workflow

### Running the Game

**Live preview in browser:**
```bash
# Method 1: Use Python HTTP server (common)
python -m http.server 8000
# Then open http://localhost:8000

# Method 2: Use Node.js
npx http-server

# Method 3: Open index.html directly in browser
# Works fine since there are no CORS restrictions for this single file
```

### Making Changes

**Structure for modifications:**
1. Locate the relevant section in the JavaScript IIFE:
   - Game state: Top of IIFE after variable declarations
   - Mechanics: Look for named functions (`update()`, `fireWeapon()`, `spawnEnemy()`, etc.)
   - Drawing: `draw()` and helper draw functions (`drawPlane()`, `drawBossShip()`, etc.)
   - Audio: `sfx` object with sound effect functions
   - Input: Event listeners for keyboard, mouse, touch

2. Make changes with the game semantics in mind:
   - Timing is always `dt`-based (delta-time in seconds) — use `dt` for frame-independent motion
   - Canvas coordinates: (0,0) at top-left, x right, y down
   - Rendering is fully imperative Canvas 2D — no retained graphics
   - Audio context must be created and resumed on user interaction (see `ensureAudio()`)

**Common tasks:**

- **Add/modify enemy behavior**: Edit `spawnEnemy()` to change spawn logic, edit the update section in `update()` for enemy AI
- **Add/modify boss pattern**: Add a new function like `fireRadial()` or edit `BOSS_PATTERNS`
- **Change difficulty**: Edit `DIFFICULTIES` object
- **Add/modify weapon**: Change `fireWeapon()` logic or add new weapon type to player state
- **Add visual effect**: Add to `draw()` or create new helper like `drawPlane()`
- **Add audio**: Add new function to `sfx` object using `beep()` or `noiseBurst()`
- **Change theme/colors**: Edit `THEMES` for backgrounds, or color values in drawing functions

### Testing

**Manual testing:**
1. Start the server and open in browser
2. Test each difficulty (Easy, Normal, Hard)
3. Play through multiple stages to verify progression
4. Test both keyboard and touch controls
5. Check audio (mute/unmute button)
6. Check that best score persists (localStorage)
7. Test pause/resume
8. Test bomb usage and power-ups
9. Reach a boss and verify boss patterns

**No automated tests** — the game is small enough for manual testing. Focus on the golden path (start game → kill enemies → fight boss → survive/die) and edge cases (controls, audio context resumption, touch input on mobile).

### Performance Considerations

- **Quality scaling** (`quality` variable): 0=full detail, 1=medium, 2=low — automatically reduced if FPS drops
  - Reduces particle count, disables shadows, reduces blur
- **Particle limits**: Reduced by quality setting in `spawnExplosion()`
- **Canvas sizing**: Responsive, scales via CSS while internal canvas stays 480×720
- **Reduced motion**: Respects `prefers-reduced-motion` media query and has a toggle in the UI

### Browser Compatibility

- **Canvas 2D**: Required (widely supported)
- **Web Audio API**: Required (fallback gracefully if unavailable)
- **localStorage**: Used for best score, settings — falls back to zero if unavailable
- **Touch events**: Full support including multi-touch joystick
- **Responsive viewport**: Uses safe-area-inset for notched devices

## Key Code Locations

| Feature | Location |
|---------|----------|
| Game loop | Line 1640–1653 |
| Input handlers | Lines 768–862 |
| Update logic | Lines 1148–1302 |
| Drawing | Lines 1520–1638 |
| Boss patterns | Lines 1372–1409 |
| Audio synthesis | Lines 542–676 |
| Game state reset | Lines 875–911 |
| Difficulty system | Lines 485–489, 741–750 |
| Particle/effects | Lines 1073–1085, 711–729 |

## File Limits

- Total file size: ~59 KB uncompressed (all-in-one)
- No external files to load (no images, audio, CSS imports)
- Everything is inlined for simplicity and performance

## Guidelines for Modifications

1. **Don't split the file** — Single file is a feature, not a limitation. The IIFE scoping keeps everything isolated.
2. **Preserve responsive design** — Use `clamp()` for font sizes, `max()` for safe areas, `%` for layout
3. **Keep audio synthesis** — No external assets; synth-based SFX are part of the charm
4. **Maintain accessibility** — Keep reduced motion toggle, screen-reader announcements, semantic HTML
5. **Use `dt` always** — Frame timing must be delta-based for consistency
6. **Cache DOM elements** — All elements are cached at the top to avoid repeated queries
7. **Use localStorage carefully** — Prefix keys with `skyfighter.` to avoid collisions

## Common Patterns in Code

**Circular collision detection:**
```javascript
function rectsOverlap(a, b) {
  return Math.abs(a.x - b.x) < (a.w + b.w) / 2 && Math.abs(a.y - b.y) < (a.h + b.h) / 2;
}
```

**Sprite drawing with transform:**
```javascript
ctx.save();
ctx.translate(x, y);
// Draw relative to origin
ctx.restore();
```

**Movement with delta-time:**
```javascript
player.x += mvx * player.speed * dt;
```

**Timed effects (e.g., invulnerability):**
```javascript
if (player.invuln > 0) player.invuln -= dt;
```

**Animation via frame counter:**
```javascript
const t = frame * 0.15; // Continuous animation tied to frame count
ctx.rotate(Math.sin(t + i) * 0.15);
```

## Known Tradeoffs

- **Single file = large lines count** — But keeps deployment trivial (just `index.html`)
- **Canvas-based = no accessibility-by-default** — Mitigated with `aria-live`, semantic overlays, screen reader announcements
- **Synth audio = no realistic SFX** — But provides unique retro feel and zero licensing issues
- **No persistence beyond localStorage** — Save only best score and settings, not game state

---

Last updated: 2026-10-18
