# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Overview

**Sky Fighter** is a Raiden-style vertical shoot-'em-up game, entirely contained in a single `index.html` file (~1670 lines, 60KB). It's a self-contained, dependency-free game with no build tools required.

## Quick Start

**Opening the game:** Simply open `index.html` in any modern web browser. No server or build step needed.

**Game features:**
- Vertical scrolling space shooter with weapon upgrades and bomb power-ups
- Three difficulty levels (Easy, Normal, Hard)
- Six themed stages, each with unique boss encounters
- Score persistence via localStorage
- Desktop (keyboard/mouse) and mobile (touch) controls
- Procedurally spawned enemies with intelligent patterns
- WebAudio synthesis-based sound effects
- Accessibility features (reduced motion support, screen reader announcements)

## Code Architecture

The entire game logic is wrapped in a single IIFE (Immediately Invoked Function Expression) to maintain closure scope and avoid global pollution.

### Main Sections

1. **DOM References** (lines ~426–461)
   - Canvas and UI element references cached at startup
   - All state updates flow through these references

2. **Game Configuration** (lines ~463–515)
   - `STORAGE_*` constants for localStorage keys
   - `DIFFICULTIES` object: tunable parameters for Easy/Normal/Hard modes (spawn rates, enemy speed, lives, bombs, boss HP multiplier)
   - `THEMES`: six visual themes with sky gradients and color palettes
   - `ENEMY_TYPES`: grunt, interceptor, tank (each with HP, speed, color, score)
   - `BOSSES`: six unique boss definitions with visual properties and HP multipliers
   - `KILLS_PER_STAGE`: constant controlling stage progression

3. **Performance Tracking** (lines ~474–483)
   - Adaptive quality system that degrades rendering quality if average frame time exceeds 1/40s
   - Tracks last 45 frame times to smooth out spikes

4. **Audio System** (lines ~542–695)
   - WebAudio API synthesis (no external audio files)
   - `beep()`: tone generation with optional frequency sweep and filtering
   - `noiseBurst()`: procedural noise (used for explosions, hits)
   - Spatial audio panning based on entity X position
   - Can be globally muted

5. **Input Handling** (keyboard, mouse, touch)
   - Keyboard: Arrow keys / WASD for movement, Space for fire, X/Shift for bombs
   - Mouse: movement tied to cursor position, click to fire
   - Touch: drag-to-move, tap FIRE/BOMB buttons

6. **Game State Variables**
   - `stage`, `score`, `mult` (score multiplier), `lives`, `bombs`
   - `player` object: position, health, current weapon
   - `enemies`, `bullets`, `enemyBullets`, `effects` arrays
   - `gameState`: 'menu', 'playing', 'paused', 'gameOver'

7. **Entity System**
   - Player: position and collision detection
   - Enemies: spawn based on difficulty, move down screen, shoot, drop pickups on death
   - Bullets: from player and enemies, collision detection each frame
   - Items: weapon pickups and bombs, collected on overlap
   - Effects: visual feedback (explosions, damage hits, combo popups)

8. **Weapon System**
   - Vulcan (default rapid fire)
   - Laser (narrow beam)
   - Spread (multiple projectiles)
   - Each level increases fire rate and damage

9. **Boss Encounters**
   - Triggered at fixed kill counts per stage
   - Large health pool, unique color scheme per boss
   - Distinct attack patterns (spray of bullets)
   - Boss health bar displayed while active
   - Completion progresses to next stage

10. **Rendering Pipeline**
    - Clear canvas with gradient sky (from current theme)
    - Draw clouds (with parallax), sun
    - Draw all entities (player, enemies, bullets, effects)
    - Draw HUD overlay (score, lives, weapon status)
    - Draw vignette effect on damage
    - Screen shake for impacts (disabled if reduced motion enabled)

11. **Game Loop** (requestAnimationFrame)
    - Delta time calculation and clamping
    - Update entity positions
    - Collision detection (player/enemy bullets, player/items, bullets/enemies)
    - Spawn enemy waves
    - Update UI elements
    - Render frame

## Working with the Code

### Making Changes

**Difficulty Tuning:**
Adjust `DIFFICULTIES` object (lines ~485–489). Keys: `spawnBase` (spawn interval), `spawnMin` (minimum spawn rate), `enemySpeed`, `enemyShootChance`, `lives`, `bombs`, `bossHpMul`.

**Visual Theme/Colors:**
- Modify `THEMES` array for sky gradients and cloud colors
- Adjust `ENEMY_TYPES` color properties (`.color` and `.dark` for shading)
- Edit CSS variables in `<style>` section for UI colors

**Enemy/Boss Behavior:**
- Enemy types live in `ENEMY_TYPES` object (line ~502)
- Enemy spawning logic and movement is in the game loop update
- Boss definitions in `BOSSES` array (line ~508)

**Audio Tuning:**
- Weapon fire: adjust `beep()` frequency and duration calls in fire handler
- Enemy explosion: adjust `noiseBurst()` parameters
- Global volume: modify `masterGain.gain.value` in `ensureAudio()` function

**Weapon Balance:**
- Weapon definitions in game state include ROF (rate of fire) and damage
- Adjust bullet speed, pattern, and damage values in the firing code

### Testing

**In-browser:** 
- Open `index.html` in a browser, test gameplay through multiple stages
- Press P or Esc to pause and inspect game state in browser console
- Test on mobile by resizing browser window or using device emulation

**Difficulty validation:**
- Play each difficulty to verify spawn rates, enemy behavior, and boss difficulty
- Check localStorage persistence: open browser DevTools → Application → Local Storage

**Audio:**
- Verify sound effects play (mute button toggles global muting)
- Test audio panning: fire near screen edges and verify left/right speaker variation

**Accessibility:**
- Toggle reduced motion button (🎆 → 🚫) and verify screen shake/flash/pulse animations disable
- Test keyboard navigation and verify screen reader announcements via `announce()` function

## Performance Notes

- Adaptive quality degrades rendering at low frame rates to maintain 60 FPS
- Enemy and bullet counts scale with difficulty; watch for performance cliffs on lower-end devices
- Touch controls inject a JS/layout thrashing loop; test on older mobile devices
- Canvas drawing is unoptimized; consider culling off-screen entities if adding many effects

## Browser Compatibility

Requires:
- ES2015+ (arrow functions, const/let, template literals)
- Canvas 2D context
- WebAudio API (for sound; game is fully playable muted if unsupported)
- localStorage (for score persistence)
- Touch Events API (for mobile controls)

Tested on: Chrome 90+, Firefox 88+, Safari 14+, mobile Safari, Chrome Android.
