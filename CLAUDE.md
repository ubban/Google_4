# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project Overview

**Sky Fighter** is a single-file HTML5/Canvas vertical shoot-'em-up game (Raiden-style) with approximately 1,670 lines of code. The entire game logic, UI, and styling is contained in `index.html` with no external dependencies.

## Getting Started

### Running the Game
Simply open `index.html` in any modern web browser. The game works on desktop and mobile (with touch controls).

**Common workflows:**
- **Edit and reload**: Make changes to `index.html`, then refresh the browser (Ctrl+R or Cmd+R)
- **Quick testing**: Open DevTools (F12) to check console for any errors
- **Mobile testing**: Use browser device emulation or test on an actual device with `file://` protocol or via a local server

### Game Development Workflow

**Making changes:**
1. Edit `index.html` directly
2. Reload the browser tab (no build step needed)
3. Test gameplay, collision detection, animations, and audio

**Testing specific features:**
- Use the **Difficulty selector** (Easy/Normal/Hard) to test different game states
- Use **mute/reduced motion** buttons to verify accessibility features
- Test **touch controls** on mobile or via DevTools device emulation
- Monitor browser console for accessibility announcements and warnings

## Architecture

The game is a single IIFE (Immediately Invoked Function Expression) with these core systems:

### Game State
- `player`: Position, health, weapon type, bombs, score multiplier
- `bullets`: Player projectiles with position, velocity, damage
- `enemies`: Spawned enemies (grunt/interceptor/tank) with type-specific properties
- `boss`: Current stage boss with unique attack patterns and health
- `stage`: Current stage number (1-6+)
- `score`, `bestScore`: Game score tracking
- `running`, `gameOver`: Game state flags
- `difficulty`: Game difficulty affecting enemy stats and spawn rates

### Core Systems

**1. Enemy & Boss System**
- `ENEMY_TYPES`: Defines 3 enemy types with HP, speed, color, scoring
- `BOSSES`: 6 stage bosses with unique names and attack patterns
- `spawnEnemy()`: Spawns enemies based on stage and kill count
- `spawnBoss()`: Triggers at stage completion; each boss has specific bullet patterns
- `updateBoss(dt)`: Updates boss position and attack timers

**2. Weapon System**
- Multiple weapon types (Vulcan, Laser, etc.) with upgrade levels
- `fireWeapon()`: Spawns player bullets based on current weapon type and level
- Each weapon fires different bullet patterns (straight, spread, homing, etc.)
- Weapon upgrades increase fire rate and visual effect intensity

**3. Physics & Collision**
- `rectsOverlap()`: AABB collision detection for rectangles
- Collision checks between: player vs enemy bullets, player bullets vs enemies, bombs vs enemies
- Enemy and player bullets use simple velocity-based movement
- Particles use acceleration for explosion effects

**4. Audio System**
- Web Audio API for synthesized sound effects (no pre-recorded audio files)
- `beep()`: Generates sine/square wave tones for various sounds
- `noiseBurst()`: White noise for explosion and bomb effects
- `sfx` object: Maps game events to sound generation functions
- LocalStorage persistence for mute state

**5. Rendering**
- Canvas 2D for all game graphics
- Sprite drawing using fillRect for rectangles and circles
- Particle system for explosions and visual feedback
- HUD rendered with DOM elements (score, weapons, health, boss bar)
- Gradients for background gradient effect

**6. Input Handling**
- **Keyboard**: Arrow keys or WASD for movement, Space to fire, Shift+Space for bombs
- **Mouse/Touch**: Crosshair movement and tap-to-fire on desktop; on-screen joystick on mobile
- Touch controls: Virtual joystick with dynamic zones and separate fire/bomb buttons
- Pause functionality accessible via pause button

**7. Persistence & Settings**
- LocalStorage keys: `skyfighter.bestScore`, `skyfighter.muted`, `skyfighter.difficulty`, `skyfighter.reducedMotion`
- Settings persist across sessions (best score, audio mute, difficulty, motion preferences)

### Difficulty Settings
```javascript
DIFFICULTIES = {
  easy:   { spawnRate: 2.5, enemyHPMul: 0.7,  enemySpeedMul: 0.8,  baseMult: 0.5 },
  normal: { spawnRate: 2.0, enemyHPMul: 1.0,  enemySpeedMul: 1.0,  baseMult: 1.0 },
  hard:   { spawnRate: 1.5, enemyHPMul: 1.3,  enemySpeedMul: 1.2,  baseMult: 1.5 }
}
```

### Boss Attack Patterns
Each of the 6 bosses has unique bullet spawning patterns in `updateBoss()`. Study the patterns by finding the boss name in that function—each boss fires bullets at different angles, speeds, and frequencies.

## Key Code Sections

**Main game loop** (line ~1165-1245): Updates player, enemies, bullets, collision detection, rendering
**Boss system** (line ~1043-1450): Boss spawning, attack patterns, health tracking
**Weapon firing** (line ~1124-1160): Bullet generation based on weapon type
**Input handling** (line ~1540-1570): Keyboard, mouse, touch event listeners
**Rendering** (line ~1590-1630): Canvas drawing and HUD updates
**Sound effects** (line ~550-680): Web Audio synthesis

## Common Tasks

### Adding a New Enemy Type
1. Add entry to `ENEMY_TYPES` object with properties: `w`, `h`, `hp`, `speedMul`, `color`, `dark`, `score`, `minStage`, optional `weave`
2. Update `spawnEnemy()` to include the new type in spawn choices
3. Optional: Add unique behavior in the enemy update loop

### Adding a New Boss
1. Add boss config to `BOSSES` array with: `name`, `attackPattern` function
2. Define unique bullet spawning in `updateBoss()` function
3. Test the attack pattern by reaching that stage

### Balancing Difficulty
- Adjust `DIFFICULTIES` multipliers for spawn rate, enemy HP, and speed
- Modify `spawnBoss()` timing (1200ms default between stage clear and boss appearance)
- Tweak weapon damage and fire rates in `fireWeapon()`

### Adding Visual Effects
- Use the `particles` array and `renderParticles()` for explosions
- Vignette effect controlled by red damage indicator
- Flash effects on hit/bomb already implemented

## Accessibility Features

- **Screen reader support**: Game announces stage changes and boss defeats to `#srLive` element
- **Reduced motion**: Toggle via button; disables screen shake, flashes, and animations
- **Touch controls**: On-screen joystick and buttons for mobile
- **Keyboard controls**: Full keyboard support (arrow keys + space)
- **ARIA labels**: Buttons have proper ARIA attributes

## Performance Considerations

- Canvas rendering is the primary performance bottleneck on low-end devices
- `frameTimes` array tracks frame timing for debugging (see `trackPerf()`)
- Particle cleanup happens automatically as particles fall off-screen
- Bullet cleanup uses `.filter()` to remove off-screen bullets each frame

## Browser Compatibility

Requires modern browser with support for:
- Canvas 2D API
- Web Audio API
- LocalStorage
- ES6 (arrow functions, const/let, template literals)
- Touch Events API (for mobile)

## Debugging Tips

1. **Check console** for accessibility announcements and errors
2. **Enable frame timing**: Look for `frameTimes` array in DevTools console
3. **Test collision** by adding debug rectangles around hitboxes
4. **Audio debugging**: Verify `ensureAudio()` is called before any sound plays (audio context requires user interaction first)
5. **Mobile testing**: Use DevTools device emulation or physically test on a device
