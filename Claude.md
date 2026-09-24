# Sky Fighter - Codebase Documentation

## Project Overview

**Sky Fighter** is a Raiden-style vertical shoot-'em-up game built with HTML5 Canvas. It's a single-page web application delivered as a self-contained `index.html` file (~1700 lines).

### Core Features

- **Game Mechanics**: Vertical scrolling shooter with weapon progression, smart bombs, stage-based bosses, and kill-chain multipliers
- **Enemy Variety**: Three enemy archetypes (Grunt, Weaving Interceptor, Multi-hit Tank) with unique behaviors
- **Boss System**: Six boss designs (Vanguard, Behemoth, Specter, Dreadnought, Wraith, Titan) with distinct attack patterns
- **Weapon System**: Vulcan/Laser dual-weapon with 3 power levels; power-ups include weapon level-up, bombs, and extra lives
- **Controls**: Keyboard (WASD/Arrow keys), touch joystick, and mouse-steering with precision aim
- **Audio**: Synthesized audio engine with spatial panning, compression, and per-sound biquad filtering
- **Accessibility**: Reduced-motion toggle (respects OS prefers-reduced-motion), aria-live announcements for screen readers
- **Performance**: Adaptive quality tier system that auto-adjusts particle counts and effects based on frame rate
- **Visual Polish**: Shaded aircraft with cockpit/flame/tail, glowing bullets/power-ups, parallax cloud layers, hit flash/screen shake, low-health vignette pulse

## File Structure

```
Google_4/
├── index.html          # Complete game implementation (1669 lines)
├── README.md           # Minimal placeholder
└── Claude.md           # This file
```

## Code Architecture

The entire game is contained in `index.html` as an inline JavaScript application within the `<script>` tag. The code is organized into logical sections:

### HTML/Head Section
- Meta tags, viewport configuration, favicon
- CSS styling for dark theme, HUD (pill-style overlays), animations, vignette effects, and responsive layout

### Game State & Constants
- Game configuration (canvas size, wave patterns, difficulty multipliers)
- Game state object (player, enemies, bullets, bosses, game flow)
- Difficulty settings with multiplier variations

### Core Game Systems

#### Rendering System
- Canvas setup and fullscreen adaptation
- HUD rendering (health, bombs, weapon level, multiplier)
- Vignette and flash overlays for visual feedback
- Particle system for explosions and effects

#### Game Loop
- `requestAnimationFrame`-based update loop
- Collision detection (bullets vs. enemies/bosses, enemies vs. player, powerups vs. player)
- Enemy and bullet updates (movement, death checks)
- Boss state management and pattern cycling

#### Input Handling
- Keyboard: WASD/Arrow keys for movement
- Touch: Virtual joystick with dynamic zone detection
- Mouse: Cursor-following plane steering (capped at 1.7x speed to remain catch-able)
- Priority: Keyboard > Touch > Mouse (last active input type wins)

#### Audio Engine
- Web Audio API with master gain and dynamics compressor
- Per-sound biquad filtering for frequency shaping
- Pitch sweeps for weight and impact
- Stereo panning driven by on-screen x-position
- Synthesized sequences for explosions, boss telegraphs, fanfares, and game-over stinger

#### Wave/Stage System
- Procedurally generated enemy waves with difficulty scaling
- Six background themes that cycle (daylight → sunset → night → repeat)
- Boss spawning at wave end with health bar and telegraph flashes before attacks
- Stage clear fanfare and boss defeat stinger

#### Boss System
Three boss designs cycle by stage:
- **Vanguard**: Radial burst, aimed fan, wide V attack patterns
- **Behemoth**: Heavy columns, homing spread, bullet wall patterns
- **Specter**: Teleport dash, spiral, twin aimed shots
- **Dreadnought**, **Wraith**, **Titan** (alternates with the above)

Each boss has its own color scheme and phase-based logic.

#### Weapon System
- **Vulcan Gun**: Rapid-fire close-range weapon (blue bullets)
- **Laser**: Charged shots with wider beam (yellow bullets)
- Three power levels with increasing damage and fire rate
- Power-up system provides level-up pickups

#### Smart Bombs
- Limited quantity (earn via power-ups)
- On activation: clears all bullets and enemies, damages bosses, triggers screen flash and audio cue

#### Difficulty Modes
- **Easy**: Slower enemy fire, weaker patterns, 1.5s power-up duration
- **Normal**: Standard rates and durations
- **Hard**: Faster fire, more aggressive patterns, power-ups last only 0.8s

### Input/Control Priority
1. **Keyboard Input**: WASD or Arrow keys override all movement
2. **Touch Input**: On-screen joystick activates when touched
3. **Mouse Steering**: Plane follows cursor when no keyboard/touch active; crosshair reticle visible

### Performance Optimization
- **Frame-time Monitor**: Measures frame duration in milliseconds
- **Quality Tiers**:
  - Tier 1 (Full): All visual effects, full particle counts, shadow blur
  - Tier 2 (Medium): Reduced particles (50%), no shadow blur
  - Tier 3 (Low): Minimal particles (25%), simplified effects
- **Auto-scaling**: Automatically downgrades if frame time exceeds threshold (≈16ms), upgrades once stable

### Accessibility Features
- **Reduced-Motion Toggle**: Respects `prefers-reduced-motion` media query; dampens shake/animations
- **Aria-live Region**: Announces life loss, boss defeats, game-over results for screen readers
- **UI Visibility Fix**: Top bar (mute/motion/pause buttons) properly layered above start overlay

### Persistence
- **LocalStorage**: Saves:
  - Player state (health, bombs, weapon level, score, multiplier)
  - Game settings (difficulty, sound mute state, motion preference)
  - Continues game on page reload if interrupted mid-game

### State Management

Key game state object fields:
- `player`: position (x, y), health, bombs, weapon level, firing state
- `enemies`: array of active enemies with position, health, type, animation state
- `bullets`: array of fired bullets with position, velocity, damage, owner
- `bosses`: array of active bosses (usually one) with position, health, attack patterns, telegraph state
- `powerups`: array of dropped power-ups with position, type, collected state
- `currentWave`: wave number and enemy spawn schedule
- `currentStage`: stage number, background theme, boss design
- `score`, `combo`: kill-chain multiplier and current count
- `paused`, `gameOver`: game flow states

## Known Constraints & Trade-offs

1. **Single File Design**: All code lives in one HTML file for portability and offline play; makes refactoring complex features non-trivial
2. **Canvas-based Rendering**: Fixed coordinate system; responsive sizing via CSS `max-width/max-height` with letterboxing
3. **Deterministic Difficulty**: Enemy spawn patterns and boss patterns are hardcoded by stage/difficulty; procedural variety is limited
4. **Audio Synthesis Only**: No sample-based audio; all sounds are generated, so some effects may feel synthetic
5. **Adaptive Quality is CPU-driven**: No GPU utilization; relies on JavaScript frame-time heuristics

## Common Tasks & How-To

### Add a New Enemy Type
1. Add archetype definition in the game configuration (e.g., in the wave setup)
2. Implement spawn logic in `createEnemy()` function
3. Add collision and update logic in the game loop's enemy update section
4. Test wave generation to ensure new type appears and takes damage correctly

### Add a New Boss Pattern
1. Define attack pattern in the boss configuration
2. Implement pattern logic in `bossBulletLogic()` and telegraph flash timing
3. Test in headless mode or manual play across all difficulty levels

### Add a New Weapon or Powerup Type
1. Add type constant to weapon/powerup definitions
2. Implement rendering in particle system or bullet rendering
3. Add pickup logic in collision detection
4. Add stat UI update for new type if needed

### Adjust Difficulty Parameters
- Edit the `difficulties` object: enemy spawn frequency, bullet fire rate, power-up duration
- Re-run full regression test on all three difficulty modes to ensure balance

### Performance Tuning
- Adjust quality tier thresholds in the frame-time monitor if devices consistently jump tiers
- Reduce particle base count if lower-end devices still struggle
- Profile with browser DevTools Performance tab during heavy wave phases

## Testing Notes

The codebase has been verified with:
- **Headless Playwright runs**: Firing, collisions, weapon pickups, boss spawn/attack/death, all work with no console errors
- **All three difficulty modes**: Multiple trials each, no crashes
- **Full stage cycles**: All three bosses spawn in order, each with distinct patterns, proper progression
- **Accessibility checks**: Reduced-motion, aria-live announcements, mute/pause controls all functional
- **Mobile & Desktop**: Keyboard, touch joystick, and mouse steering all tested

## Browser Support

- **Modern browsers**: Chrome/Chromium, Firefox, Safari (mobile and desktop)
- **Canvas API**: Full 2D context with shadow blur, composite operations
- **Web Audio API**: Dynamics compressor, biquad filters, panning
- **Touch & Pointer Events**: Android/iOS touch joystick, desktop mouse steering
- **LocalStorage**: For game state persistence
- **CSS Media Queries**: Reduced-motion, color-scheme preferences

## Future Enhancement Ideas

- Network multiplayer or leaderboard via web service
- Mobile-optimized touch controls (larger buttons, haptic feedback)
- Additional boss designs and enemy archetypes
- Customizable control remapping UI
- In-game tutorial or story mode
- Achievement/medal system
- Procedurally generated boss patterns for infinite mode
- Particle system improvements (GPU acceleration if available)

## Attribution

- **Game Design**: Raiden-style vertical shooter
- **Development**: Claude AI (with team contributions)
- **Author**: ubban <ubban@irobotx.top>

---

**Last Updated**: 2026-09-24
