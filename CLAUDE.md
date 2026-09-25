# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project Overview

**Sky Fighter** is a Raiden-style vertical shoot-'em-up game implemented as a single-file web application (`index.html`). It features adaptive performance, responsive touch controls, procedurally generated audio, and roguelike difficulty progression.

**Key characteristics:**
- Zero external dependencies (pure vanilla JavaScript, Canvas, Web Audio API)
- ~1670 lines of self-contained code (HTML + CSS + JS)
- Cross-platform: desktop (mouse/keyboard) and mobile (touch controls)
- Persistent state via localStorage (best score, audio mute, difficulty, accessibility preferences)

## Architecture Overview

The game uses a **monolithic architecture** within a single file:

### Game Loop & Rendering (lines 1640–1665)
- `requestAnimationFrame` loop driving 60 FPS target
- Delta-time based updates for framerate independence
- Performance tracking (`trackPerf()`) monitors frame times and adjusts quality settings (particle count, shadow blur)

### Input Handling (lines 768–861)
- **Keyboard**: Arrow keys / WASD for movement, Space/Shift+X to shoot/bomb, P/Esc to pause
- **Mouse**: Hover to target, click to fire, move cursor glides the plane
- **Touch**: Virtual joystick (left zone) + Fire/Bomb buttons (right zone)
- Input has priority: keyboard/joystick > mouse (prevents fighting between input methods)

### Game State & Objects (lines 863–912)
Central state lives in module closure:
- `player`: { x, y, w, h, speed, weapon, level, invuln, cooldown }
- `enemies`, `bullets`, `enemyBullets`, `particles`, `powerUps`, `boss`
- Scoring: `score`, `chain`, `multiplier` (chain-based combo system)
- Lifecycle: `running`, `paused`, `gameOver`

### Core Systems

**Collision Detection (line 1069)**
- Rectangle-based AABB (axis-aligned bounding box): `rectsOverlap(a, b)`
- Applied to: bullets ↔ enemies, bullets ↔ boss, enemy bullets ↔ player, enemies ↔ player

**Difficulty Levels (lines 485–489)**
- Easy / Normal / Hard preset by `spawnBase`, `enemySpeed`, `enemyShootChance`, lives, bombs, boss HP multiplier
- Stored and restored from localStorage; difficulty affects spawning and stats

**Weapon System (lines 1124–1146)**
- **Vulcan** (default): spreads fire based on level (1: narrow, 2: dual, 3: triple)
- **Laser**: pierces enemies, higher level adds lanes (1: single, 2: dual, 3: triple)
- Upgrades cycle: weapon level 1→2→3, then switch weapon and reset to level 1

**Scoring & Combo (lines 1087–1101)**
- `addScore()` increments chain, which raises multiplier (1 + floor(chain/5), capped at 10x)
- Chain resets after 2.2 seconds without kills
- Boss defeats award flat 500 points

**Enemy Spawning (lines 1022–1041)**
- Spawn timer decreases each frame; at 0, spawn a new enemy
- Enemy type weighted by difficulty & stage:
  - Grunt (always): simple red enemy
  - Interceptor (stage 1+): fast, weaving side-to-side
  - Tank (stage 2+): slow, high HP, high score
- Enemies shoot in either "aimed" (at player) or "spread" pattern

**Boss System (lines 1043–1056, 1411–1439)**
- 6 themed boss designs cycle per stage (VANGUARD, BEHEMOTH, SPECTER, DREADNOUGHT, WRAITH, TITAN)
- Each boss has unique attack patterns (6 attacks per boss × 6 bosses)
- Boss telegraph phase (22 frames) warns player before attack barrage
- Boss enters from top and bounces horizontally; killing advances stage

**Power-ups (lines 1058–1067, 1304–1322)**
- Life, Bomb, Weapon (random drops from killed enemies)
- Weapon power-ups upgrade weapon level or cycle weapon type

**Audio (lines 543–676)**
- Web Audio API synth (no audio files)
- Dynamic SFX generated with oscillators, filters, noise bursts
- Pan support (left/right based on X position)
- Mutable via localStorage; respects reduced-motion preference

### Rendering (lines 1520–1638)
- **Background**: Parallax clouds + sky gradient (changes per stage theme)
- **Player**: 2D composite shape (cockpit, wings, flame when firing)
- **Enemies**: Same plane drawing function with different colors
- **Boss**: Complex multi-part design with glowing core and rotating engines
- **HUD**: Score, best, multiplier, weapon level, hearts, bombs (HTML overlaid)
- **Effects**: Screen shake (camera jitter), flash vignette (on low HP), combo popups

### Responsive Design
- Base canvas: 480×720 pixels
- Auto-scales to fit window (max 1.6x) while maintaining aspect ratio
- Touch controls only show on touch devices; hidden on desktop
- Viewport configured for mobile (touch-action: none, viewport-fit: cover)

## High-Level Workflows

### Adding a New Enemy Type
1. Define in `ENEMY_TYPES` (line 502): `{ w, h, hp, speedMul, color, dark, score, minStage, weave }`
2. Update spawn weights in `spawnEnemy()` (line 1026)
3. Add explosion SFX mapping in `EXPLODE_SFX` (line 678) if custom sound
4. Add rendering logic in `draw()` if custom shape (line 1603)

### Adding a New Boss Pattern
1. Define attack function in module scope (e.g., `b => fireRadial(b, 18, 150)`)
2. Add to corresponding boss's pattern array in `BOSS_PATTERNS` (line 1372)
3. Boss patterns are indexed by `(stage - 1) % BOSSES.length`

### Tweaking Balance
- Enemy spawn rate: adjust `spawnBase` / `spawnMin` in `DIFFICULTIES`
- Boss HP: adjust `baseHp` in `spawnBoss()` (line 1046)
- Weapon balance: change fire rates (`fireDelay` line 1177) or bullet speed
- Difficulty ramp: modify `difficultyRamp` calculation (line 1201)

### Testing
- Open `index.html` directly in a browser (no build step needed)
- Use browser DevTools console to patch state: `player.level = 3`, `lives = 10`, etc.
- Inspect localStorage: `localStorage.getItem('skyfighter.bestScore')`

## Key Constants & Tweakables

| Constant | Line | Purpose |
|----------|------|---------|
| `BASE_W`, `BASE_H` | 753 | Canvas logical size |
| `KILLS_PER_STAGE` | 491 | Enemies killed before boss |
| `THEMES` | 493 | Stage background gradients (6 per stage cycle) |
| `ENEMY_TYPES` | 502 | Enemy stats (hp, speed, score) |
| `BOSSES` | 508 | Boss designs & color schemes (6 total) |
| `BOSS_PATTERNS` | 1372 | Boss attack sequences |
| `DIFFICULTIES` | 485 | Game parameters per difficulty |

## Accessibility & Polish

- **Reduced Motion**: Disables animations, screen shake, flashing when OS prefers-reduced-motion
- **Audio**: Mute button persists to localStorage; Web Audio API synthesized (no external files)
- **Screen Reader**: ARIA live region (`#srLive`) announces kills, stage changes, game over
- **Mobile**: Full touch support with on-screen joystick and buttons; safe-area-inset respects notches

## Debugging Tips

- **Performance issues**: Check `quality` variable (0=full, 1=medium, 2=low); adjust shadow/particle thresholds
- **Collision bugs**: Add debug rects in `draw()` with `ctx.strokeRect()`
- **Audio missing**: Check browser console for AudioContext errors; some browsers require user gesture first
- **Touch controls not appearing**: Verify `isTouchDevice` detection (line 778); may need explicit touchmove handler
- **Score/state not persisting**: Check localStorage key spelling in `STORAGE_*` constants (lines 463–466)

## Common Tasks

**Run the game**: Open `index.html` in a browser. No build, server, or dependencies needed.

**Edit and test**: Make changes to the single file, refresh the browser. Changes are immediate.

**Profile performance**: Open DevTools → Performance tab, record while playing, check main thread time and frame drops.

**Test on mobile**: Use Chrome DevTools device emulation or deploy to a URL and visit from a phone.

**Test different difficulties**: Click difficulty buttons on the start screen; choice persists via localStorage.
