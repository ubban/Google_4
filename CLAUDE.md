# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project Overview

**Sky Fighter** is a Raiden-style vertical shoot-'em-up game built as a single-file HTML5 application. Players control a fighter jet, switch between weapons, use smart bombs, and battle progressively harder enemy waves leading to stage bosses.

## Architecture

The entire game is implemented in `index.html` as an inline HTML/CSS/JavaScript application (~1670 lines). This monolithic approach ensures the game runs standalone without dependencies—just open the file in a browser.

### Core Systems

**Game Loop & State**
- Frame-based timing with delta-time calculations (`dt`)
- Global variables manage game state: `player`, `enemies`, `bullets`, `boss`, `powerUps`, `explosions`, `floatingTexts`
- Frame counter tracks progression for difficulty ramping and boss spawning

**Player Mechanics**
- Position clamped to canvas bounds; WASD/arrow keys for movement
- Two weapon modes: basic bullets (fast fire rate) and laser (slower, piercing, 1-pixel width)
- Invulnerability frames after taking damage
- Lives system (hearts) and bomb supply for smart bombs (kills all enemies and damages boss)
- Weapon/bomb pickup via power-ups

**Enemy System**
- Three enemy types: `fighter`, `tank`, `specialist`; each has distinct firing patterns
- Waves spawn on a decreasing timer (difficulty ramps by reducing spawn intervals every 5 seconds)
- Weaving motion for some enemies; spread-fire vs. homing patterns
- Kill milestone triggers boss spawn after 50 enemy kills per stage

**Boss Fight**
- Single boss spawns after stage kill target; entering phase before combat
- Distinct attack pattern (predictable bullet spread) and high HP pool
- Defeated boss ends stage; score multiplier persists to next stage

**Collision Detection**
- Rectangle-overlap checks between: player-bullets-enemies, player-bullets-boss, player-enemies, player-powerups
- Laser bullets pierce multiple enemies (with piercing limit); normal bullets vanish on contact

**Scoring & Progression**
- Kill chain/combo system: +1x multiplier per kill, resets after 2-second silence
- Difficulty variables (`d`) ramp spawn rate, enemy HP, and fire speed across stages
- Score display and end-game final score screen

**Visual & Audio**
- Canvas rendering: background gradient, sprites (player/enemies as rectangles with rotation), particles (explosions, floatingText)
- Sound effects (SFX) synth-generated on-demand; spatial audio panning by entity X position
- Vignette effect on low-health warning; screen flash on hit or bomb use

**Control & Input**
- Keyboard: WASD/arrow keys move, Space fires, B drops bomb
- Mouse/touch: click/tap fires
- Pause on blur; resume on focus

## Development Workflow

### Running the Game

Open `index.html` directly in a web browser (Chrome, Firefox, Safari, Edge). No build step or server required. Game runs immediately with full audio/visuals.

**Testing a specific feature:**
- To test enemy spawning/difficulty: play through waves and observe spawn timing decrease
- To test weapons: collect weapon power-ups or edit `player.weapon` in DevTools console to switch modes
- To test boss: reach 50 kills to trigger boss spawn
- To test scoring: verify kill counter increments and combo multiplier resets on delay

### Code Structure

The file follows this logical flow (line numbers approximate; search for keywords below):

1. **Setup** (`<head>`, initial `<style>`) – HTML structure, fonts, theme
2. **Canvas & Container** – Game viewport rendering target
3. **Inline Styles** – HUD styling, animations (heartBeat, vignettePulse, flashHit)
4. **Global State** – `const` declarations (canvas, player, enemies, stage config, etc.)
5. **Game Difficulty Table** – `stages` array defining progression
6. **Audio/SFX Factory** – `createSfx()` synth-based sound generation
7. **Utility Functions** – collision detection, AABB helpers
8. **Rendering** – `draw()`, particle/text rendering
9. **Game Logic** – `update()` loop: input, movement, spawning, collision checks
10. **Input Handlers** – keydown/keyup, mouse/touch listeners
11. **Animation Frame Loop** – `requestAnimationFrame` with frame counting

### Modifying Game Balance

**Difficulty Progression** (line ~400–450):
```javascript
const stages = [
  { spawnBase: 120, spawnMin: 30, spawnDecrease: 4, ... },
  ...
];
```
Adjust `spawnBase` (initial delay) and `spawnMin` (floor) to speed/slow enemy waves.

**Enemy Behavior**:
- `enemies.push({ type, hp, speed, canShoot, pattern, ... })` in `spawnEnemy()` — modify HP or speed directly
- Shooting patterns in `update()` loop — change `spread` vs. homing logic

**Weapon Balance**:
- Bullet speed: `bullets.push({ speed: 400 })` (basic), `{ speed: 600 }` (laser)
- Fire rate: `player.cooldown = fireDelay` (0.1s laser, 0.15s basic)
- Laser piercing: `b.pierce` limit (currently unlimited for laser)

**Boss Mechanics**:
- Boss HP: `boss.hp = 100` in `spawnBoss()` — increase for harder fight
- Boss pattern: `updateBoss()` function — modify bullet spread angles or frequency

### Adding New Features

**New Enemy Type**:
1. Add entry to `spawnEnemy()` where types are checked (search "fighter" | "tank" | "specialist")
2. Define shooting pattern in the enemy collision loop in `update()`
3. Add explosion SFX mapping in `EXPLODE_SFX`

**New Weapon**:
1. Add weapon name to `player.weapon === 'newweapon'` checks
2. Create firing logic in `fireWeapon()` function
3. Define bullet properties (speed, size, behavior)

**New Power-Up**:
1. Add type to `maybeSpawnPowerUp()`
2. Handle pickup logic in collision check where `powerUps` intersect with player
3. Add visual representation in rendering section

## Common Commands & Tasks

Since this is a single-file app, common workflows are:

- **Run**: Open `index.html` in browser
- **Debug**: Open DevTools (F12), use console to inspect game state (e.g., `console.log(player)`, `player.lives = 5`)
- **Profile**: Use DevTools Performance tab to check frame rate during intense action
- **Test balance**: Adjust stage difficulty values and replay to verify feel
- **Mobile test**: Open on phone/tablet to test touch controls and viewport scaling

## Key Constants & Config

- `KILLS_PER_STAGE = 50` – enemy kills before boss spawns
- `stages[]` – difficulty progression for each level
- `EXPLODE_SFX` – sound effect mapping per enemy type
- `chainTimeout = 2` – seconds of silence before combo resets
- `player.invuln` – invulnerability duration after hit (in seconds)

## Notes for Future Development

- **Performance**: Canvas rendering is CPU-bound; consider WebGL if targeting very high entity counts
- **Mobile UX**: Touch controls work well; consider adding on-screen buttons for constrained layouts
- **Accessibility**: Add keyboard alternative for color-blind players (currently relies on visual cues for HUD)
- **Persistence**: No save system; consider localStorage for high scores if desired
- **Platform**: Works on any modern browser with HTML5 Canvas and Web Audio API support
