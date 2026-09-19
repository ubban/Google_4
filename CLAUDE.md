# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project Overview

**Sky Fighter** is a Raiden-style vertical shoot-em-up game built as a single-page HTML5 canvas application with vanilla JavaScript (no frameworks, no external assets beyond the HTML file itself). It features:

- Multiple enemy types and boss encounters with unique attack patterns
- Weapon/bomb/power-up systems with progression
- Touch and mouse controls, keyboard input, and difficulty modes
- Procedural audio synthesis (no audio files)
- Adaptive performance scaling
- Accessibility features (reduced motion toggle, screen reader support)
- Local storage for score and settings persistence

The entire game is in a single `index.html` file (~1670 lines). Open it directly in a browser to play.

## Code Architecture

The game is organized as a single IIFE (Immediately Invoked Function Expression) that encapsulates all state and functions. The logical flow is:

1. **Initialization** (lines ~420–1000): DOM element caching, difficulty/storage setup, audio context initialization
2. **Game State** (lines ~863–870): Core variables like `player`, `enemies`, `bullets`, `score`, `stage`, etc.
3. **Update Loop** (lines ~1148–1302): Physics, collision detection, spawn timers, boss AI—called every frame
4. **Rendering** (lines ~1507–1638): Canvas drawing of game objects, parallax clouds, particle effects
5. **Entry Point** (lines ~1640–1665): `requestAnimationFrame` game loop and button event listeners

### Key Subsystems

**Game Loop**: `loop(now)` → `update(dt)` + `draw()`. The frame time is capped at 50ms and adaptively scales particle/glow quality based on frame timing.

**Input Handling** (lines ~767–861):
- Keyboard: arrow keys, WASD for movement; Space for fire; X/Shift/P for bomb/pause
- Touch: left zone (joystick), right zone (fire/bomb buttons), joystick rendering in `#touchControls`
- Mouse: pointer position steers the plane toward it at 1.7× speed when no keyboard input is active; click to fire

**Collision Detection**: Single rect-based `rectsOverlap()` used for all hits (enemy–bullet, enemy–player, powerup–player, boss–bullet).

**Game State Flow**: `resetGame()` → `running=true` → game loop → `endGame()` or stage advance. Boss spawns after `KILLS_PER_STAGE` (18 kills) with a 1.2s delay.

**Weapon/Power-up System**:
- Player weapon: "vulcan" (3-level spread) or "laser" (pierce, multiple lanes)
- Power-ups: life (+1 HP), bomb (+1), weapon (level up or switch)
- Weapon switch resets level to 1; damage is baked into collision handling (enemy hp decrement in `update()`)

**Score Chain/Multiplier**: Killing enemies within 2.2 seconds chains them; multiplier caps at `1 + floor(chain / 5)` → max 10x at 45+ consecutive kills.

**Boss Patterns**: Each boss design (6 total) has 3 attack patterns (in `BOSS_PATTERNS` array). Patterns fire during a "telegraph" window (22 frames) and are selected sequentially, looping. Enemy bullets use `.radial` flag to move via `dx`/`dy` instead of `speed`.

**Performance Adaptive System** (lines ~473–483): A rolling 45-frame average of frame times maps to a quality tier (0=full, 1=medium, 2=low), reducing particles and glow effects. This runs passively in `trackPerf()`, so adding frames shouldn't require explicit tuning.

## Common Development Tasks

**Running the game locally**: Simply open `index.html` in a web browser. Use DevTools to inspect canvas rendering or check console for audio context warnings on restricted browsers.

**Testing with Playwright headless**: The commits show this project was verified with headless runs testing firing, collisions, power-ups, and boss patterns. A minimal Playwright test would:
```javascript
const browser = await chromium.launch({ headless: true });
const context = await browser.createContext();
const page = await context.newPage();
await page.goto('file:///path/to/index.html');
await page.click('#startBtn'); // start game
await page.keyboard.press('Space'); // fire
await page.waitForTimeout(1000);
const hasNoErrors = !(await page.evaluate(() => {
  return window.hasUnhandledError; // custom flag on window if needed
}));
```

**Testing game mechanics manually**:
- Spawn control: Enemy spawn rate varies with difficulty and frame count (spawn timer decreases faster as waves progress).
- Boss patterns: Set `stage = 3` in DevTools console before starting to skip to the 3rd boss (Specter) immediately.
- Audio: Mute button (`#muteBtn`) toggles `muted` flag; ensure `audioCtx` is initialized via `ensureAudio()`.
- Touch: Emulate a touch device in DevTools to see `#touchControls` activate.

## Important Code Patterns and Quirks

**Canvas sizing**: Base size is 480×720 (viewport-fit safe). Scaled up to fill the window (max 1.6×). `resize()` handles orientation changes and recalculates canvas style dimensions.

**Audio synthesis**:
- `beep(freq, dur, type, vol, opts)`: single oscillator with optional pitch sweep, biquad filter, and stereo pan
- `noiseBurst(dur, vol, opts)`: white noise filtered with optional pan
- Pan values are spatial (left=-1 to right=1) based on `panFor(x) = (x / width) * 2 - 1`
- Master gain is 0.9; all sounds routed through a dynamics compressor to prevent clipping

**Reduced motion**: Stored in `localStorage[STORAGE_REDUCED]`; defaults to OS `prefers-reduced-motion` preference. Affects particle counts (`trackPerf()` tier scaling), screen shake magnitude, and animation visibility (CSS class `.reduced-motion` disables keyframes).

**Enemy weaving**: "Interceptor" type enemies use `weave: true` and oscillate around a `baseX` position using `Math.sin(frame * 0.05 + weaveSeed)`. Weave seed is randomized per spawn to desynchronize waves.

**Boss telegraph**: Bosses have a `telegraph` counter (frames). When it reaches 0, the attack pattern fires and `attackTimer` resets. This visual cue ensures attacks feel fair.

**Invulnerability frames**: Player gets `invuln = 1.5` (seconds) on damage and blinks (drawn on even frames only). This prevents rapid multi-hit death spirals.

**Particle life**: Particles use `.life` and `.maxLife` (both in seconds). Alpha fades as `life / maxLife` during render.

**Mouse steering priority**: Keyboard and joystick input take precedence; mouse only moves the plane when `mag < 0.02` (no active key/stick). This prevents awkward double-steering.

## Customization Points

**Adding a new enemy type**: Add to `ENEMY_TYPES` object with `w`, `h`, `hp`, `speedMul`, `color`, `dark`, `score`, `minStage`, optional `weave`. Update `EXPLODE_SFX` if explosion sound differs.

**Adding a new boss design**: Append to `BOSSES` array with `name`, hull/wing/core colors, and `hpMul`. Create a 3-pattern array in `BOSS_PATTERNS` using existing pattern builders (`fireRadial`, `fireAimedSpread`, `fireColumns`, etc.). Boss rendering in `drawBossShip()` uses `b.design` to pick colors.

**Adding a new stage theme**: Append to `THEMES` with `sky` (3-stop gradient), `cloud` and `sun` (RGB as strings), and `name`. Used cyclically by `(stage - 1) % THEMES.length`.

**Tweaking difficulty**: `DIFFICULTIES` object has `spawnBase/spawnMin` (frame intervals), `enemySpeed`, `enemyShootChance`, `lives`, `bombs`, `bossHpMul`. Affects wave density and player challenge.

**Weapon balance**: Vulcan spread is in `fireWeapon()` (angles); laser lanes and pierce count are also there. Boss damage is hard-coded as `3` per bullet in collision check; adjust as needed.

**New power-up type**: Add case to `applyPowerUp()` and update `maybeSpawnPowerUp()` spawn rate. Rendering is in `draw()` under the powerUps loop.

## Debugging Tips

- **Frame-rate issues**: Open DevTools Performance tab, record a short run, check the flame graph. `trackPerf()` will auto-downscale quality if avg frame time exceeds 25ms; manually set `quality = 0` in console to force full quality.
- **Collision bugs**: Add a debug flag to draw collision boxes: `ctx.strokeStyle='red'; enemies.forEach(e => ctx.strokeRect(e.x - e.w/2, e.y - e.h/2, e.w, e.h));` in `draw()` before particles.
- **Audio not playing**: Check browser console for AudioContext warnings (usually requires user interaction first). `ensureAudio()` is called on key/mouse/touch input; if still silent, verify `muted` is false.
- **Boss not spawning**: Ensure `stageKills >= KILLS_PER_STAGE` (18) before the 1.2s timer fires. Check console and set `stageKills = 17` to trigger it quickly.
- **Storage issues**: Clear `localStorage` (e.g., `localStorage.clear()` in DevTools) to reset best score and difficulty to defaults.

