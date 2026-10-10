# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project Overview

**Sky Fighter** is a Raiden-style vertical shoot-'em-up game implemented as a single-file HTML5 application. The entire game runs in the browser with no external dependencies.

## Quick Start

1. Open `index.html` in a web browser to play the game
2. No build process, server, or dependencies needed
3. Edit `index.html` directly and refresh the browser to see changes

## Game Architecture

The game is organized into several interconnected systems within a single 1670-line HTML file:

### Core Systems

**Canvas Rendering** (lines 752-764, 1520-1638)
- Fixed 480×720 canvas with responsive CSS scaling
- Three rendering layers: sky/clouds/sun, gameplay objects, UI overlays
- Screen shake and flash effects for impact feedback
- Quality adaptation tracks frame times and reduces particle counts on low-end devices

**Player Control** (lines 767-861)
- Keyboard: Arrow keys or WASD to move, Space to fire, X/Shift/P to bomb/pause
- Mouse: Move to steer, click to fire (takes priority over keyboard when active)
- Touch: Left zone drag for movement, right zone tap buttons for fire/bomb, virtual joystick
- Input priority: Keyboard/Joystick > Mouse steering (prevents fighting)

**Entity Systems** (lines 863-868)
- `player`: Position, speed, weapon state, cooldown, invulnerability timer
- `bullets`: Player projectiles with pierce/spread mechanics per weapon
- `enemies`: Type-based (grunt/interceptor/tank) with AI patterns (spread/aimed shooting)
- `enemyBullets`: Marked as `radial` if boss-pattern fired, moving by `dx`/`dy` instead of `speed`
- `particles`: Explosion debris, color-based, fade over time
- `powerUps`: Life/bomb/weapon pickups that fall and rotate
- `boss`: Single entity that enters from top, moves horizontally, fires complex patterns

**Combat & Collision** (lines 1069-1302)
- Tight AABB collision via `rectsOverlap()` called on every bullet-enemy pair each frame
- Enemy destroyed when `hp <= 0`; boss destroyed after final bullet hit
- Player loses life when hit by enemy/enemy bullet/boss; invulnerability window prevents double-hit
- Damage model: player bullets deal 1 base (plus pierce for laser), bosses take 3 per hit, tanks take 4 hits to destroy
- **Bomb** (useBomb): Clears all enemy bullets, removes all on-screen enemies, damages boss for 24 HP, awards 5 points per enemy

**Progression** (lines 1043-1056, 1441-1454, 1204-1207)
- Player kills 18 enemies to trigger boss spawn warning (1200ms delay)
- 6 bosses cycle: VANGUARD, BEHEMOTH, SPECTER, DREADNOUGHT, WRAITH, TITAN
- Boss spawns every stage; defeating boss increments `stage`, resets `stageKills`, queues next boss
- 6 sky themes cycle per stage
- Difficulty adjusts spawn rate, enemy speed, enemy shoot chance, boss HP multiplier

**Weapon System** (lines 1124-1146, 1304-1322)
- **Vulcan**: Spread increases with level (1→2→3 projectiles), base cooldown 150ms
- **Laser**: Lanes increase with level, pierce allows hitting multiple enemies, cooldown 100ms
- Weapon drops cycle: level up within weapon or switch weapon+reset to level 1
- Max weapon level 3

**Scoring & Chains** (lines 1087-1101)
- Base points per kill: grunt 10, interceptor 15, tank 35, bomb 5 per enemy, boss 500
- Kill chain multiplier: every 5 kills increases multiplier (1 → 2 → 3 ... up to 10x)
- Chain resets if > 2.2 seconds pass without a kill
- "X KILLS!" popup shown every 10th kill

**Audio** (lines 542-676)
- Web Audio API synthesized sounds, no external files
- Master compressor prevents clipping; each sound is oscillator + optional filter + gain envelope
- Pan based on entity X position (left/right stereo)
- Mute button stores preference in localStorage
- Sound effects: shoot, laser, grunt/interceptor/tank explosions, hit, bomb, powerup, boss hit/telegraph/down, stage clear, UI click, gameover

**Effects & Feedback** (lines 698-730)
- **Flash**: White screen flash on player hit or bomb (100-500ms)
- **Vignette**: Red pulse animation when health critical (1 life)
- **Shake**: Screen jitter, magnitude & duration varies by event
- **Reduced Motion**: Affects animations, can be toggled in-game; respects OS prefers-reduced-motion
- **Combo Pop**: Large glowing text ("X KILLS!") floats up center screen
- **Stage Banner**: "STAGE N" or "WARNING — BOSS NAME" text intro

### Boss Fight Architecture

**Entrance** (lines 1413-1417)
- Boss `entering: true`, glides down from `y = -80` to `y = 120` at 60 px/s
- While entering, no collision damage and doesn't fire

**Movement** (lines 1418-1419)
- Bounces left-right between canvas edges at 55 px/s
- Direction flips at `x < w/2+10` or `x > width-w/2-10`

**Attack Patterns** (lines 1372-1408)
- Telegraph phase: 22 frames of core glow + sound, then fires attack
- Attack timer: 85 frames between telegraphs
- Each boss has 3 patterns in `BOSS_PATTERNS[index]`, cycled via `patternIndex`
- Pattern functions mutate `enemyBullets` array; boss bullets are `radial: true` and move by `dx`/`dy` per frame
- Pattern types: `fireRadial`, `fireAimedSpread`, `fireColumns`, `fireSpiralBurst`, `fireCross`, `fireHomingSeeded`, `fireDoubleRing`

**Boss HP Scaling** (lines 1046-1049)
- Base HP: `70 + stage * 35`, multiplied by difficulty `bossHpMul` (0.8/1.0/1.3) and boss-specific `hpMul` (0.8→1.7x)
- Boss health bar updates every frame as `width: hp/maxHp * 100%`

### UI & State Management

**Overlay System** (lines 401-419, 1479-1504)
- Single overlay div shows title screen, pause screen, or game-over screen
- Buttons and text update dynamically
- Class toggle `hidden` to show/hide elements within overlay

**HUD Elements** (lines 367-375)
- Score, best score, multiplier, weapon name/level, health (hearts), bombs (bombs emoji)
- Boss health bar (hidden until boss spawns, shown via `bossBarWrap` class `show`)
- Top-right buttons: mute, motion toggle, pause

**Local Storage Keys**
- `skyfighter.bestScore`: Best score across sessions
- `skyfighter.muted`: Audio mute state (0 or 1)
- `skyfighter.difficulty`: Selected difficulty (easy/normal/hard)
- `skyfighter.reducedMotion`: Motion preference (0 or 1)

### Main Game Loop (lines 1640-1665)

```
requestAnimationFrame(loop):
  dt = delta time (capped at 50ms)
  trackPerf(dt)
  if running && !paused:
    update(dt) → move entities, handle collision, spawn enemies/boss, award points
    draw() → render all objects
  requestAnimationFrame(loop)
```

## Common Development Tasks

### Testing the Game
- Open `index.html` directly in a browser (no server needed)
- Use difficulty buttons on title screen to test different gameplay
- Press P or Esc to pause mid-game for frame-stepping
- Browser DevTools Console shows no errors on clean runs

### Adding a New Enemy Type
1. Add entry to `ENEMY_TYPES` (line 502-506) with `w`, `h`, `hp`, `speedMul`, `color`, `dark`, `score`, `minStage`
2. Update `spawnEnemy()` choices to include new type (line 1026-1030)
3. Add explosion sound to `EXPLODE_SFX` map (line 678) if needed
4. Optionally add unique sound in `sfx` object

### Adding a New Boss Pattern
1. Add function to `BOSS_PATTERNS[bossIndex]` array (e.g., new attack at line 1375)
2. Function receives boss object, mutates `enemyBullets` array
3. Use helper functions: `fireRadial`, `fireAimedSpread`, `fireColumns`, `fireSpiralBurst`, `fireCross`, `fireHomingSeeded`, `fireDoubleRing`
4. Each bullet in pattern starts at `(b.x, b.y)`, has `w`, `h`, `radial: true`, and `dx`/`dy` velocity

### Tweaking Game Feel
- **Difficulty**: Adjust `DIFFICULTIES` object (line 485-489)
- **Spawn rate**: Modify `spawnBase` and `spawnMin` per difficulty
- **Enemy speed**: Adjust `enemySpeed` multiplier
- **Weapon balance**: Change fire rates in `fireDelay` constant (line 1177) or spread patterns (line 1127)
- **Screen shake**: Pass `mag` and `dur` to `screenShake()` calls (line 1108, etc.)
- **Audio levels**: Adjust `vol` parameter in `beep()` and `noiseBurst()` calls

### Accessibility
- Respects `prefers-reduced-motion` media query on first load
- Toggle in-game with motion button (🎆 icon)
- Live region (`#srLive`) announces major events for screen readers
- All buttons have title attributes for tooltips

## Key Constants & Magic Numbers

| Constant | Line | Value | Purpose |
|----------|------|-------|---------|
| `BASE_W`, `BASE_H` | 753 | 480×720 | Canvas resolution |
| `KILLS_PER_STAGE` | 491 | 18 | Enemies to kill before boss |
| `STORAGE_*` | 463-466 | Various | localStorage keys |
| Boss spawn delay | 1206 | 1200ms | Time before boss enters after warning |
| Player speed | 877 | 260 px/s | Base movement speed |
| Invulnerability duration | 1459 | 1.5s | After taking damage |

## Performance Notes

- Particle count scales by quality: full quality uses `count`, medium uses 60%, low uses 35%
- Quality tier auto-adjusts based on 45-frame rolling average of frame time
- Boss glow (shadow blur) disabled on medium/low quality
- Canvas resolution fixed at 480×720 to avoid reflow during gameplay

## File Structure

```
index.html               (1670 lines)
├─ <head>
│  ├─ <style>          (CSS: ~350 lines)
│  └─ <meta>           (viewport, theme-color, favicon)
├─ <body>
│  ├─ Canvas & HUD elements
│  ├─ Overlay (title/pause/gameover)
│  ├─ Touch controls (joystick + buttons)
│  └─ <script>         (Game logic: ~1200 lines, IIFE)
```

No separate files, no build step, no external assets.
