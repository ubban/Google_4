# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project Overview

**Sky Fighter** is a Raiden-style vertical shoot-'em-up arcade game built as a single HTML file. It requires no build process, external dependencies, or server. The entire game (~59KB) is self-contained in `index.html` with embedded CSS and JavaScript.

## Running & Development

### Opening the Game
Simply open `index.html` directly in a web browser (Chrome, Firefox, Safari, Edge). The game is fully responsive and works on desktop, tablet, and mobile devices.

```bash
# Quick preview (macOS)
open index.html

# Or serve locally if needed
python3 -m http.server 8000
# Then visit http://localhost:8000
```

### Testing Changes
- **During development:** Open DevTools (F12) to monitor console errors and Canvas performance.
- **Mobile testing:** Use Chrome DevTools device emulation or test on a physical device via localhost.
- **Audio testing:** The game generates all sounds via Web Audio API (synth, no external assets). Test mute button functionality.

### Performance Testing
- Use DevTools Performance tab (⌘⇧P → "Performance") to record gameplay and check frame times.
- The game includes adaptive quality system: monitors frame times and reduces rendering quality (`quality` variable, 0–2) if FPS drops below 40.
- Watch the console for quality adjustments during drops.

## Code Architecture

The entire game runs in a single IIFE (Immediately Invoked Function Expression) at `<script>` line 422+. All state and logic are encapsulated within this closure.

### Key Systems

#### 1. **Game Loop & Rendering**
- `requestAnimationFrame` drives the main loop (no fixed timestep; adapts to screen refresh rate).
- Canvas 2D context draws all game objects each frame.
- Rendering order: background sky gradient → enemies → player → bullets → UI overlays.

#### 2. **Game State Management**
- **Player state:** position, health (lives), weapon level, bomb count, score, multiplier.
- **Enemies:** array of enemy objects with position, type, health, animation state.
- **Bullets:** arrays for player bullets, enemy bullets, and bomb effects.
- **Game phases:** `INIT` (menu) → `PLAYING` → `PAUSED` → `STAGE_CLEAR` → `GAME_OVER`.

#### 3. **Audio System (Web Audio API)**
- `ensureAudio()` initializes the AudioContext (deferred until first user interaction due to browser autoplay policy).
- Two categories:
  - **Beep synthesis:** `beep(freq, dur, type, vol, opts)` creates oscillator sounds with optional sweeps and filters.
  - **Noise synthesis:** `noiseBurst(dur, vol, opts)` generates white noise bursts (explosions, etc.).
- All SFX defined in the `sfx` object (shoot, laser, explode, bomb, powerup, bossHit, bossTelegraph, etc.).
- Master gain/compressor prevents clipping. Mute state stored in localStorage.

#### 4. **Collision Detection**
- Axis-aligned bounding box (AABB) checks: `checkCollision(a, b)` used for:
  - Player vs. enemy bullets → damage player.
  - Player bullets vs. enemies → damage enemy.
  - Player vs. enemies → damage player.
  - Bombs vs. everything → clear projectiles, damage enemies in radius.

#### 5. **Enemy & Boss System**
- **Enemy types:** grunt, interceptor, tank (3 types with varying speeds, HP, AI).
- **Spawning:** Waves spawn at intervals, difficulty affects spawn rate and enemy stats.
- **Boss fights:** After `KILLS_PER_STAGE` (18) enemy kills, a boss appears. Six unique bosses with custom colors and HP multipliers.
- **Behavior:** Enemies move down screen, shoot randomly, weave side-to-side (interceptors).

#### 6. **Difficulty Settings**
- Three presets: `easy`, `normal`, `hard` (DIFFICULTIES object).
- Affects spawn rates, enemy speed, enemy shoot chance, lives, bombs, boss HP.
- Player selection persists in localStorage.

#### 7. **Adaptive Quality System**
- Tracks frame times in rolling 45-frame window.
- If average frame time > 1/40s (FPS < 40), increment quality level (reduces particles/effects).
- If average frame time < 1/56s (FPS > 56), decrement quality (enables more effects).
- Quality levels: 0 (full effects), 1 (medium), 2 (low effects only).

#### 8. **UI & Overlays**
- **HUD (in-game):** Score, best score, multiplier, weapon level, hearts, bombs (updated each frame).
- **Overlays:** Menu (start), stage banners, game-over screen, boss health bar.
- **Animations:** CSS keyframes (`cardIn`, `stagePulse`, `comboPulse`, `vignettePulse`, etc.) for menu cards and on-screen popups.
- **Accessibility:** Screen reader live region (`srLive`) announces game state changes.

#### 9. **Input Handling**
- **Keyboard:** Arrow keys (movement), Space/Z (fire), X (bomb), P (pause).
- **Touch:** Joystick zone (left) + fire/bomb buttons (right). Position tracked via `touch` events.
- **Mouse:** Can move player, doesn't fire (intentional, prevents accidental gameplay).

#### 10. **Visual Themes**
- Six visual themes (THEMES object): day, sunset, night, storm, arctic, volcanic.
- Each defines sky gradient colors, cloud/sun colors. Theme cycles per stage.

#### 11. **Persistence & Settings**
- **localStorage keys:**
  - `skyfighter.bestScore` — high score.
  - `skyfighter.muted` — audio mute state.
  - `skyfighter.difficulty` — selected difficulty.
  - `skyfighter.reducedMotion` — motion reduction toggle.

### Important Global Objects

| Variable | Type | Purpose |
|----------|------|---------|
| `canvas`, `ctx` | DOM, CanvasRenderingContext2D | Game render target |
| `player` | Object | Player state (x, y, hp, score, weapon, etc.) |
| `enemies` | Array | Active enemy objects |
| `bullets` | Object | { player: [], enemy: [] } — projectile tracking |
| `bombs` | Array | Active bomb effect objects |
| `quality` | 0–2 | Adaptive rendering quality level |
| `difficulty` | String | Selected difficulty (easy/normal/hard) |
| `gamePhase` | String | Current game state (INIT, PLAYING, etc.) |

## Development Tips

### Adding a New Enemy Type
1. Add entry to `ENEMY_TYPES` object (define width, height, HP, speed, color, score).
2. Add rendering case in the render loop (look for `switch (enemy.type)`).
3. Add SFX in `EXPLODE_SFX` for custom explosion sound.

### Adding a New Boss
1. Add to `BOSSES` array with name and color scheme.
2. Boss rendering is procedural (drawn from hull/wing/core color properties); no sprite needed.
3. Adjust `hpMul` to balance difficulty.

### Modifying Game Balance
- Adjust `KILLS_PER_STAGE` to change when bosses appear.
- Edit `DIFFICULTIES` object to tweak spawn rates, enemy stats.
- Change `THEMES` for visual variety.

### Debugging
- The game is vanilla JS with no hidden dependencies; use browser DevTools console.
- Set breakpoints on game loop or input handlers.
- Inspect `player`, `enemies`, `bullets` objects in console during gameplay.
- Check Performance tab for frame-time bottlenecks.

### Mobile & Accessibility
- Test reduced motion with DevTools → Rendering → Emulate CSS media feature: `prefers-reduced-motion`.
- Verify touch controls respond correctly across device sizes (game uses `clamp()` for responsive scaling).
- Confirm screen reader announcements fire via `announce(text)` function (posts to `srLive` element).

## No External Build Step
This project has **no build system, bundler, or compilation step**. All changes are live:
- Edit CSS in `<style>` block → refresh browser.
- Edit JavaScript → refresh browser.
- No transpilation, no minification, no test runner.

If you need to optimize for production (minify, split), use an external tool, but development is HTML-as-is.
