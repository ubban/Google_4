# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project Overview

**Sky Fighter** is a Raiden-style vertical shoot-'em-up game built entirely in HTML5 Canvas with synthesized audio (WebAudio API). The entire game is self-contained in `index.html` with no external dependencies or build process.

## Running the Game

Open `index.html` in a web browser. The game runs immediately without any build or installation step.

- **Desktop**: Mouse to steer, click to fire, Space/X to drop bombs, P/Esc to pause
- **Mobile**: Touch left side to move, FIRE/BOMB buttons on right, smart bombs with gesture
- **Keyboard**: Arrow keys or WASD to move, Space to fire, X/Shift for bombs

## Code Architecture

The single `index.html` file contains three integrated systems:

### 1. **Game State & Loop** (frames ~863–900, loop function)
- Core game loop via `requestAnimationFrame`
- Delta-time based updates (`dt` in seconds, clamped to 50ms max)
- Adaptive performance tracking (scales particle/effect quality based on frame times)
- Game states: `running`, `paused`, `gameOver`

### 2. **Game Systems** (frames ~1148–1302)
- **Player**: position, velocity, weapon (vulcan/laser with levels 1–3), cooldown, invulnerability frames
- **Enemies**: three types (grunt, interceptor, tank) spawned based on stage and difficulty
- **Boss**: spawns after 18 enemy kills per stage; each boss has unique attack patterns
- **Bullets & Damage**: player projectiles with pierce (laser) or spray (vulcan); enemy bullets track player
- **Scoring & Multipliers**: kill chain with 2.2s window; multiplier scales 1–10x with kill count
- **Power-ups**: life restore, bomb refill, weapon upgrade/swap

### 3. **Rendering & Audio** (frames ~543–623 audio, 1507–1638 draw)
- **Rendering**: sky gradient per theme, clouds, player/enemy planes, bullets, particles, UI (HUD, boss bar, overlays)
- **Audio Synthesis**: No external audio files; all SFX generated with WebAudio API (square/sine/sawtooth oscillators, filters, noise bursts)
- **Visual Effects**: screen shake (reduced in accessibility mode), flash on hit/bomb, combo pop-ups

## Key Data Structures

- **Player** `{ x, y, w, h, speed, cooldown, rapid, weapon, level, invuln }`
- **Enemy** `{ x, y, w, h, speed, shootTimer, type, hp, color, weave, pattern }`
- **Boss** `{ x, y, w, h, hp, maxHp, design, telegraph, patternIndex, attackTimer, vx }`
- **Bullet** `{ x, y, w, h, speed, vx, pierce, laser }`

## Common Modifications

### Adding a New Enemy Type
1. Add entry to `ENEMY_TYPES` object (line ~502): `{ w, h, hp, speedMul, color, dark, score, minStage }`
2. Update spawn probabilities in `spawnEnemy()` (line ~1029)
3. Add explosion SFX mapping to `EXPLODE_SFX` (line ~678) if custom sound needed

### Adjusting Difficulty
- `DIFFICULTIES` object (line ~485): tweak `spawnBase`, `spawnMin`, `enemySpeed`, `enemyShootChance`, `lives`, `bombs`, `bossHpMul`
- Difficulty ramps spawn timer by `-4` per 300 frames (line ~1201)

### Adding a Boss Attack Pattern
1. Add pattern array to `BOSS_PATTERNS` (line ~1372) with three functions: `b => firePattern(b, ...)`
2. Use helper functions: `fireRadial()`, `fireAimedSpread()`, `fireColumns()`, `fireSpiralBurst()`, `fireDoubleRing()`, `fireHomingSeeded()`
3. Each pattern should assign new bullets to `enemyBullets` array with `{ x, y, w, h, radial, dx, dy }`

### Visual Themes
- `THEMES` array (line ~493): each theme has `{ sky: [color1, color2, color3], cloud, sun, name }`
- Sky is a 3-stop gradient; cloud and sun use RGB strings for dynamic opacity
- Themes cycle per stage: `THEMES[(stage - 1) % THEMES.length]`

### Audio SFX
- All SFX defined in `sfx` object (line ~624)
- Core functions: `beep(freq, dur, type, vol, opts)` and `noiseBurst(dur, vol, opts)`
- Options: `sweepTo` (frequency ramp), `filterType`/`filterFreq` (highpass/lowpass), `pan`, `delay`
- Mute state stored in localStorage; respects user preference and OS reduced-motion setting

## Collision & Gameplay Logic

- **Hitbox overlap**: `rectsOverlap(a, b)` uses AABB (axis-aligned bounding box) — simple distance check
- **Boss immunity**: boss only takes damage when not `entering` (line ~1255)
- **Weapon switching**: Power-up at level 3 swaps weapon; each weapon restart at level 1
- **Chain multiplier**: resets on damage taken or after 2.2s inactivity (line ~1096)

## Accessibility Features

- Reduced-motion mode: disables animations (shake, flash, pulse) when `prefers-reduced-motion: reduce` or toggled via button
- Screen reader announcements via `srLive` element with `aria-live="polite"`
- Full keyboard support; mobile touch controls

## Storage & Persistence

- Best score, mute state, difficulty, reduced-motion preference stored in `localStorage`
- Keys: `skyfighter.bestScore`, `skyfighter.muted`, `skyfighter.difficulty`, `skyfighter.reducedMotion`

## Performance Tuning

Adaptive quality setting (`quality` 0–2, line ~474):
- **Quality 0**: Full effects, glow, shadows (target 60 fps)
- **Quality 1**: Reduced particles & shadows (if avg frame time > 1/40s)
- **Quality 2**: Minimal effects (if avg frame time exceeds 1/40s again)

Tracked via 45-frame rolling average of delta times.
