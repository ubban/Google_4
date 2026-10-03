# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project Overview

**Sky Fighter** is a Raiden-style vertical shoot-'em-up game built entirely in a single HTML file (index.html). The game is self-contained with no external dependencies — all graphics are drawn via Canvas 2D API, all sound is synthesized using WebAudio API, and all data is stored in localStorage.

## Running the Game

Simply open `index.html` in a web browser. The game runs entirely client-side with no build step, bundler, or server required.

**Target browsers:** Modern evergreen browsers (Chrome, Firefox, Safari, Edge) with ES2015+ support, Canvas 2D, WebAudio API, and localStorage.

## Game Architecture

The game uses a single-file, imperative architecture with a game loop pattern. All code runs in a single IIFE (immediately invoked function expression) to avoid polluting the global scope.

### Core Systems

1. **Game Loop** (lines 1640–1666)
   - `requestAnimationFrame` drives the main loop at ~60 FPS
   - Each frame: `update(dt)` → game state, then `draw()` → render
   - Delta time clamped to 0.05s to prevent physics instability when frame time spikes

2. **Player & Input** (lines 768–861)
   - **Keyboard:** Arrow keys / WASD for movement, Space / mouse click to fire, X / Shift / mouse to bomb, P / Esc to pause
   - **Mouse:** Player glides toward pointer; click fires; takes priority over keyboard when active
   - **Touch:** Joystick in left zone (55% width), Fire/Bomb buttons in right zone (45% width)
   - All input handlers call `ensureAudio()` to resume AudioContext if suspended by browser

3. **Game State** (lines 863–903)
   - Player object: position (x, y), weapon, level, cooldown, invulnerability timer
   - Arrays: bullets, enemies, enemyBullets, particles, powerUps, boss
   - Scoring: chain counter (resets if no kills for 2.2s), multiplier (1–10x based on chain), lives/bombs
   - Stage progression: 18 kills per stage triggers boss warning; boss spawns 1.2s later

4. **Rendering** (lines 1520–1638)
   - **Background:** Vertical gradient + radial sun glow + scrolling cloud layers (two speeds for parallax)
   - **Entities:** Player plane, enemies, boss, bullets, particles, power-ups all drawn as Canvas shapes
   - **HUD:** HTML overlay with score, best, multiplier, weapon level, hearts, bombs (updated in real-time, not redrawn each frame)
   - **Screen shake:** Applies random translation to canvas context when `shakeTime > 0`
   - **Quality scaling:** Adaptive particle count and glow shadows based on frame time history

5. **Collision Detection** (lines 1069–1294)
   - `rectsOverlap()`: Simple AABB overlap for all collision pairs
   - Enemy bullets cleared by bomb; boss can be damaged by bomb too
   - Invulnerability frames (1.5s) after hit prevent rapid damage

6. **Difficulty Scaling** (lines 485–489)
   - Three presets: easy, normal, hard
   - Affects: spawn rate, enemy speed, enemy shoot chance, lives, bomb count, boss HP multiplier
   - Spawn rate accelerates: `spawnBase - (frame / 300) * 4` (tighter over time)
   - Player can change difficulty while paused or on title screen

### Enemy Types & Bosses

**Enemies** (lines 502–506):
- **Grunt:** Fast, weak, common early (red)
- **Interceptor:** Very fast, weaving pattern, 1.7× speed multiplier (orange)
- **Tank:** Slow, 4 HP, high score, armored (purple, appears stage 2+)

**Bosses** (lines 508–515):
- 6 designs (Vanguard, Behemoth, Specter, Dreadnought, Wraith, Titan) cycling by stage
- Each has unique attack pattern suite: radial bursts, aimed spreads, columns, spirals, homing volleys, etc.
- HP scales with stage + difficulty + boss-specific multiplier (0.8–1.7×)

### Weapons & Power-ups

**Weapons** (lines 1124–1146):
- **Vulcan:** Spread shot (1–3 projectiles at level 1–3); limited range; rapid fire (0.15s cooldown)
- **Laser:** Piercing beams (can hit multiple enemies); longer range; faster fire (0.1s cooldown); visual glow aura

**Power-ups** (lines 1304–1322):
- Life: +1 life (capped at initial difficulty lives)
- Bomb: +1 bomb (capped at difficulty max)
- Weapon: Level up current weapon → switch to other weapon at level 1 (cycles)

### Audio (lines 542–676)

**WebAudio Synthesis (no samples):**
- `beep(freq, dur, type, vol, opts)`: Oscillator with optional frequency sweep, filter, panning, and gain envelope
- `noiseBurst(dur, vol, opts)`: Filtered white noise for percussion/explosion effects
- All effects are spatial (pan based on x position) and mutable
- **SFX Map:** `sfx` object with ~15 effects (shoot, laser, explode variants, bomb, powerup, hit, boss sounds, etc.)

**Master gain chain:** Osc/Noise → optional Filter → optional Panner → Gain (envelope) → Master Gain → Dynamics Compressor → speaker

## Key Patterns to Modify

### Adding a New Enemy Type

1. Add entry to `ENEMY_TYPES` (line 502) with `w`, `h`, `hp`, `speedMul`, `color`, `dark`, `score`, `minStage`
2. If weaving, set `weave: true` and weave logic will activate in enemy update (line 1213)
3. Add explosion sound to `EXPLODE_SFX` map (line 678) and `sfx` object
4. Enemy choice weighted in `spawnEnemy()` (lines 1028–1030) by `Math.random()` thresholds

### Adding a New Boss

1. Add to `BOSSES` array (line 508) with name, hull colors, wing color, core color, `hpMul`
2. Create attack pattern function suite in `BOSS_PATTERNS` (line 1372)
   - Each pattern function receives boss object and fires bullets via `fireRadial()`, `fireAimedSpread()`, etc.
   - Fire functions are composable (e.g., `fireRadial()` + `fireColumns()` in one pattern)
3. Boss will auto-cycle by `(stage - 1) % BOSSES.length`

### Tweaking Gameplay

- **Enemy spawn rate:** Edit `DIFFICULTIES` object
- **Player speed/weapon balance:** Modify `player` init values (line 877) and weapon cooldown (line 1177)
- **Chain multiplier curve:** Line 1090 defines cap and slope
- **Visual feedback:** Adjust `screenShake()` magnitude/duration or particle spawn counts (line 1074)

## Performance Considerations

1. **Adaptive quality:** Frame time is sampled; if avg > 25ms, quality ramps to 1–2, reducing particle count and shadow blur. Visible in boss telegraphing glow.
2. **Particle pooling:** Particles created on-demand and filtered after life expires (no reuse pool, but low enough count that GC is manageable)
3. **Canvas size:** Fixed 480×720 logical size, scaled to fit viewport (max 1.6× zoom). Responsive on resize/orientation.
4. **Touch/pointer events:** Touch controls are hidden on desktop via CSS (no event listener bloat)
5. **LocalStorage:** Best score, mute state, difficulty, reduced-motion preference all cached on load and updated on change

## Accessibility

- **Screen reader support:** `#srLive` div with `aria-live="polite"` announces key events (game start, boss defeat, etc.)
- **Reduced motion:** `prefers-reduced-motion: reduce` query disables animations by default; toggle via UI button (stores in localStorage)
- **Focus & keyboard:** Tab order respects overlay buttons; P/Esc for pause, X/Shift for bomb
- **Color:** Enemy types differ in both color and speed/shape; boss glow telegraphs attack

## Testing & Debugging

### Manual Testing Checklist

1. **Input:** Test keyboard (arrows + space + x), mouse (hover + click), touch (drag joystick + tap buttons) on target browsers
2. **Collision:** Verify enemy bullets don't miss, boss patterns spawn correctly, power-ups are collectible
3. **Audio:** Mute button toggles, spatial panning works, sound level is comfortable
4. **Responsiveness:** Game canvas fits different aspect ratios (phone portrait, tablet, desktop widescreen)
5. **Persistence:** Play, set best score, reload page → best score persists; change difficulty → loads on restart

### Common Tweaks

- To test a specific stage/boss: Modify `stage` init (line 893) or stage threshold (line 1204)
- To test boss attack patterns: Set `boss.attackTimer = 0` in browser console to trigger next pattern
- To debug collisions: Add `ctx.strokeRect()` calls around enemy/bullet rects in `draw()`

## File Structure

```
.
├── index.html        (complete game: HTML + CSS + Canvas + WebAudio + state)
└── README.md         (metadata)
```

No separate CSS, JS, or asset files — everything is embedded.

## Browser Compatibility Notes

- **Canvas 2D & WebAudio API:** Required
- **ES2015 features:** Arrow functions, const/let, template literals, destructuring (common in modern browsers)
- **LocalStorage:** Required for score persistence
- **Touch events:** Desktop events work on mouse; tested on iOS Safari and Android Chrome
- **Audio context:** Requires user interaction (click/touch) to resume if suspended by browser policy

## Future Enhancement Ideas

- Difficulty curves (e.g., weekly challenges with fixed parameters)
- Stage/boss themes tied to visual themes
- Leaderboard integration (via backend)
- Replay recording (store input sequence, replay from seed)
- Mobile app wrapper (Cordova/React Native)
