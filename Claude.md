# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project Overview

**Sky Fighter** is a Raiden-style vertical shoot-'em-up game implemented as a single-file HTML5 web application. The game features a player aircraft that navigates through stages, defeats enemies, collects weapon power-ups, and battles stage bosses. It supports three difficulty levels (Easy, Normal, Hard), multiple weapon types, bomb mechanics, scoring with combo multipliers, and theme variations.

## Architecture

The entire game is implemented in a single `index.html` file (~1669 lines) with inline CSS and JavaScript. The architecture follows a monolithic pattern with these major components:

### Game Loop and State
- Single continuous animation frame loop using `requestAnimationFrame`
- Game state tracking: `running`, `paused`, `gameOver`
- Frame time tracking for adaptive performance quality adjustment
- Global state variables for player position, health, score, weapons, difficulty, etc.

### Core Game Objects
- **Player ship**: Positioned at bottom, moves horizontally, fires weapons
- **Enemies**: Multiple types (grunt, interceptor, tank) spawned continuously; patrol downward, shoot at player
- **Boss ships**: One per stage, move in patterns, have high health and damaging attacks
- **Projectiles**: Player bullets and enemy shots with distinct behaviors
- **Power-ups**: Weapon upgrades and bomb refills spawned from defeated enemies
- **Explosions**: Particle effects for visual feedback on hits/destruction

### Key Systems

**Input System**: Handles keyboard (WASD/Arrows for movement, Space/X for weapons/bomb), mouse (movement targeting and click-to-fire), and multi-touch (joystick-style movement zone and action buttons). Touch detection automatically enables mobile controls.

**Audio System**: Web Audio API-based procedural sound generation. Master gain control, muting, and stereo panning. Sound effects for shooting, explosions (type-specific), pickups, boss actions, and UI. Supports frequency sweeping and noise bursts.

**Rendering**: Canvas 2D context. Base resolution 480×720, responsive scaling up to 1.6× based on viewport. Renders player/enemy ships (with flame effects), projectiles, explosions, particles, and HUD. Includes visual feedback via full-screen flash, vignette effects, and screen shake.

**Collision Detection**: Rectangle-based overlap checks between player, enemies, bullets, power-ups, and explosions.

**Difficulty System**: Three presets (easy/normal/hard) modulate enemy spawn rate, movement speed, shooting frequency, starting lives, bomb count, and boss health. Selected difficulty is stored in localStorage.

**Progression**: Stages advance after killing 18 enemies (KILLS_PER_STAGE). Each stage has a new theme (sky gradient, cloud color, sun color). Six themes cycle through stages.

**Scoring**: Base points for kills (scaled by enemy type and difficulty), multiplied by combo counter (increments on consecutive hits, resets on player damage). Score and high score persist in localStorage.

**Weapons**: Three types (Vulcan, Laser, Missiles) with upgrade levels. Weapon strength and pattern improve with level. Power-ups spawn from defeated enemies to upgrade or refill weapons.

**Bombs**: Limited-use smart bombs that clear all enemies/bullets on screen with large explosion effect and invulnerability window.

## Development

### Running and Testing
The game runs directly in a browser by opening `index.html`. No build step, dependencies, or dev server required.

- **Desktop**: Open `index.html` in a browser; use keyboard or mouse
- **Mobile/Tablet**: Open in browser; touch controls (left zone to move, right zone to fire/bomb) enable automatically

### Testing Game Features
- **Difficulty**: Click difficulty buttons on start screen to test gameplay balance
- **Weapons**: Play through stages to trigger power-ups and test weapon progression
- **Bosses**: Survive to stage-end to encounter boss patterns
- **Score mechanics**: Kill enemies in rapid succession to test combo multiplier
- **Audio**: Use mute button (top-right 🔊) to toggle; reduced-motion button (🎆) toggles animations for accessibility
- **Mobile**: Resize browser window or use device emulation to test touch controls
- **Performance**: Game adapts quality (lowered particle count, effect intensity) if frame time exceeds targets

### Key Constants and Configuration
- **BASE_W, BASE_H**: Canvas resolution (480×720)
- **KILLS_PER_STAGE**: 18 — enemies defeated before stage advance
- **DIFFICULTIES**: Three presets defining spawn/speed/lives/bombs/boss HP multipliers
- **THEMES**: Six visual themes cycling per stage
- **STORAGE_* keys**: LocalStorage keys for persistence (best score, muted state, difficulty, reduced motion)

### Common Modifications
- Adjust **DIFFICULTIES** object to tune gameplay balance (spawn rates, enemy speed, starting lives/bombs)
- Modify **THEMES** array to add new visual stage appearances
- Edit **sfx** object to change sound design (frequency, duration, filter characteristics)
- Change **KILLS_PER_STAGE** to alter progression speed
- Adjust **BASE_W, BASE_H** for different aspect ratios (will auto-scale)
- Tweak weapon fire rate, damage, and patterns in fireWeapon() and weapon-related game loop code

## Browser Compatibility
- Requires modern browser with Canvas 2D, Web Audio API, localStorage
- Tested on desktop and mobile (iOS Safari, Android Chrome)
- Gracefully degrades reduced-motion support via prefers-reduced-motion media query check

## Accessibility
- Keyboard-only playable (arrows, space, X, P)
- High contrast visuals with text shadows
- Screen reader live region (aria-live) for status announcements
- Reduced-motion preference toggle respects prefers-reduced-motion and allows manual override
- Touch controls optimized for large target sizes (78px buttons)
