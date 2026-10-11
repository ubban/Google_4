# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project Overview

**Sky Fighter** is a Raiden-style vertical shoot-'em-up game built entirely in a single HTML5 file. Players fly a fighter jet, destroy enemy waves, collect power-ups, and defeat stage bosses across multiple difficulty levels. The game uses Canvas for rendering and synthesizes all sound effects procedurally via the Web Audio API.

## Running the Game

Simply open `index.html` in a modern web browser. The game launches immediately with a title screen overlay allowing difficulty selection.

### Testing

No build step or tests exist. Manual testing is the verification method:
- **Desktop**: Use keyboard (Arrow keys/WASD to move, Space to fire, X/Shift for bombs) or mouse (move to steer, click to fire)
- **Mobile**: Use touch controls (drag left area to fly, tap FIRE/BOMB buttons)
- **Accessibility**: Test with reduced motion enabled (button in top-right corner) to verify animations work correctly in that mode

## Architecture & Structure

The entire game logic lives in a single `<script>` tag in `index.html`. It's organized conceptually as:

### Core Systems

**Game State**
- `player`: position, weapon, level, invunerability
- `enemies`, `bullets`, `enemyBullets`, `particles`, `powerUps`: pooled entity arrays
- `boss`: current stage boss (null when not active)
- `score`, `lives`, `stage`, `multiplier`: player stats
- `running`, `paused`, `gameOver`: control flow flags

**Update Loop**
- `update(dt)`: processes all game logic each frame
  - Player movement (keyboard, joystick, mouse steering)
  - Weapon firing with cooldown and spread patterns
  - Enemy spawning with difficulty-based timers
  - Collision detection (bullets vs enemies, enemies vs player, etc.)
  - Boss state machine (entering, attacking, telegraph)
  - Chain multiplier decay
- `draw()`: renders all entities to canvas with theme-based sky gradient and parallax clouds
- `loop(now)`: requestAnimationFrame wrapper managing dt calculation and adaptive performance

**Input Handling**
- Keyboard: ARROW_KEYS/WASD (move), SPACE (fire), X/SHIFT (bomb), P/ESC (pause)
- Mouse: position tracking for steering, click to fire
- Touch: two zones — left for joystick movement, right for FIRE/BOMB buttons
- Mobile joystick rendered as visual feedback showing stick position

**Rendering**
- Canvas size: 480x720 (scaled to fit window, max 1.6x)
- Theme system cycles through 6 environmental themes (Daylight, Sunset, Night, Storm, Arctic, Volcanic) — sky gradients and cloud colors change per stage
- Custom plane-drawing function renders player and enemies as geometric shapes
- Boss ships have color-coded designs per BOSSES array (e.g., VANGUARD is purple, TITAN is gold)

### Key Subsystems

**Difficulty & Progression**
- Three difficulty presets: Easy, Normal, Hard — control spawn rate, enemy speed, enemy bullet fire rate, starting lives/bombs, boss HP multiplier
- 18 kills per stage triggers boss spawn (configurable via `KILLS_PER_STAGE`)
- 6 stage bosses cycle (VANGUARD, BEHEMOTH, SPECTER, DREADNOUGHT, WRAITH, TITAN), each with unique attack patterns
- Boss patterns scale by stage via `baseHp = 70 + stage * 35`

**Weapon System**
- Two weapons: Vulcan (spread bullets) and Laser (piercing beams)
- Each weapon has 3 levels controlling projectile count/spread
- Level 3 + power-up toggles to the other weapon at level 1
- Weapon cooldown differs: Laser 0.1s, Vulcan 0.15s

**Combat Mechanics**
- Chain multiplier: increments on each kill, resets if player takes damage or time expires (2.2s timeout)
- Multiplier: 1 + floor(chain / 5), max ×10
- Score formula: `base * multiplier` where base varies by enemy type
- Boss telegraph: brief 22-frame warning before attack pattern, audible beep cue

**Enemy Types**
- `grunt`: 1 HP, slow, common
- `interceptor`: 1 HP, fast, weaves side-to-side
- `tank`: 4 HP, slow, heavy (displays HP bar above unit)
- Minimum stage requirement prevents early spawning (e.g., tanks only stage 2+)
- Enemies fire in either "spread" (5 bullets cone) or "aimed" (homing bullet to player)

**Audio (Web Audio Synth)**
- No external files — all SFX procedurally generated via oscillators, noise, and filters
- Master gain connected through compressor (threshold -20dB, ratio 5)
- Mute toggle persists to localStorage
- Pannable sound effects (bullets wider apart have spatial cues)
- SFX keys: `shoot`, `laser`, `explode*`, `hit`, `bomb`, `powerup`, `boss*`, `stageClear`, `uiClick`, `gameover`

**Adaptive Performance**
- Quality setting tracks frame times; adjusts rendering detail
- Quality 0 (full): glows and shadows enabled
- Quality 1 (medium): reduced particle count (~60%)
- Quality 2 (low): minimal particles (~35%)
- Thresholds: drop quality if FPS < 40, improve if FPS > 56 (averaged over 45 frames)

**Accessibility**
- Reduced motion preference respects `prefers-reduced-motion` media query on first load
- Persists user choice to localStorage
- Removes keyframe animations when enabled (vignette pulse, flash, hearts, combos)
- Screen reader live region (`#srLive`) announces: stage changes, life loss, combo milestones, boss defeat
- Color contrast meets WCAG AA for HUD text

**Local Storage**
- `skyfighter.bestScore`: high score
- `skyfighter.muted`: audio mute state (boolean as '0' or '1')
- `skyfighter.difficulty`: last selected difficulty (easy/normal/hard)
- `skyfighter.reducedMotion`: accessibility preference

## Modifying Game Content

### Adding a New Enemy Type
1. Add entry to `ENEMY_TYPES` object with `w`, `h`, `hp`, `speedMul`, `color`, `dark`, `score`, `minStage`, optional `weave`
2. Optionally add explosion sound to `EXPLODE_SFX` mapping
3. Update `spawnEnemy()` filter logic if conditional spawn behavior needed

### Adding a New Boss
1. Add entry to `BOSSES` array (color scheme: `hullTop`, `hullBottom`, `wing`, `core`, `hpMul`)
2. Add attack patterns array to `BOSS_PATTERNS` (each index matches BOSSES index)
3. Boss spawns automatically cycling through BOSSES based on (stage - 1) % BOSSES.length

### Adding a New Theme
1. Add entry to `THEMES` array with `sky` (3-color array), `cloud`, `sun` RGB strings, and `name`
2. Themes auto-cycle based on stage; update `draw()` theme selection if different progression desired

### Tweaking Difficulty
Modify `DIFFICULTIES` object entries: `spawnBase`, `spawnMin`, `enemySpeed`, `enemyShootChance`, `lives`, `bombs`, `bossHpMul`

## Game Loop Mechanics

1. **Frame**: increment every update; used for animation timing and pattern indexing
2. **Spawn Timer**: counts down; enemy spawns when <= 0, timer resets with difficulty-based value
3. **Stage Progression**: 18 kills → boss warning (1.2s delay) → boss spawn at Y = -80, enters to Y = 120
4. **Boss Patterns**: telegraph phase (22 frames) → silence (85 frames) → repeat; pattern cycles from BOSS_PATTERNS[patternIndex % patterns.length]
5. **Collision**: rectangular AABB using `rectsOverlap(a, b)` checks a.x ± a.w/2, a.y ± a.h/2
6. **Particle Cleanup**: particles die when `life <= 0`; pools (arrays) are filtered at end of update

## Performance Considerations

- Single canvas, no off-screen buffer
- Particle count scales with quality setting
- Boss glow/shadow disabled in quality > 0
- Touch event handlers use `{ passive: false }` to prevent default scrolling
- Frame time capped at 50ms to prevent large dt jumps during lag

## Hardcoded Constants

- Base canvas: 480×720
- Max visible scale: 1.6x window size
- Kill threshold per stage: 18
- Boss base HP: 70 + (stage * 35)
- Chain timeout: 2.2s
- Boss telegraph duration: 22 frames
- Boss attack cooldown: 85 frames
- Player invulnerability: 1.5s

## Browser APIs Used

- Canvas 2D Context
- Web Audio API (AudioContext, OscillatorNode, BiquadFilterNode, GainNode, StereoPannerNode)
- localStorage
- matchMedia (prefers-reduced-motion)
- Touch Events
- requestAnimationFrame
- getBoundingClientRect

## Common Development Tasks

**Adjust Game Balance**
- Enemy spawn rate: modify `DIFFICULTIES[difficulty].spawnBase` and `spawnMin`
- Boss difficulty: tweak `BOSSES[index].hpMul` or `DIFFICULTIES[difficulty].bossHpMul`
- Player weapon output: change cooldown values in `fireWeapon()` or projectile count in spread arrays

**Debug Collision Issues**
- `rectsOverlap()` is the only collision function; verify a.w, a.h, b.w, b.h are set on all entities
- Tank HP bar uses `ENEMY_TYPES.tank.hp` hardcoded; keep in sync with tank spawn setup

**Test Audio**
- Mute toggle in top-right; verify each sound plays (SFX buttons for easy testing)
- Panning: test on wide stereo speakers or headphones to hear left/right bullet cues

**Mobile Testing**
- Joystick needs two-finger tracking tolerance; `moveZone` covers 55% of left side
- Fire/Bomb buttons are 78×78px; ensure touch targets on small screens aren't cramped
- Safe area insets for notched devices use `env(safe-area-inset-*)` in CSS

**Accessibility Audit**
- Reduced motion: verify animations are smooth/null without keyframe animations active
- Screen reader: check `#srLive` announcements (stage banner, "3 lives remaining", "Boss defeated")
- Keyboard: all actions (move/fire/bomb/pause) must work via keys alone
