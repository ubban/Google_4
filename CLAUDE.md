# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project Overview

**Sky Fighter** is a vertical shoot-'em-up game (Raiden-style) built as a single HTML file. It's a fully self-contained game with no build process or external assets — all rendering, audio, and logic are implemented in embedded CSS and JavaScript using Canvas 2D and Web Audio APIs.

## File Structure

- **index.html** — The entire game (1670 lines)
  - Lines 1-359: HTML structure and CSS styling
  - Lines 422-676: Audio synthesis system (SFX generation)
  - Lines 752-862: Canvas sizing and input handling (keyboard, mouse, touch)
  - Lines 863-1303: Core game state, update loop, entity management, collision detection
  - Lines 1304-1439: Boss AI and attack patterns
  - Lines 1440-1638: Rendering functions and game UI
  - Lines 1640-1666: Main game loop and event listeners

## Game Architecture

### State Management
- **Player**: Position, weapon type (vulcan/laser), level (1-3), health, invulnerability timer
- **Entities**: Arrays for bullets, enemies, enemy bullets, particles, power-ups
- **Game Flow**: Spawning timer, stage progression, boss state, difficulty settings
- **Scoring**: Chain multiplier system (kills reset timer, damage to time window increases multiplier)

### Entity Systems

**Difficulty Presets** (line 485-489):
- Easy, Normal, Hard with separate spawn rates, enemy speed, health multipliers
- Boss HP scales by difficulty × boss design multiplier
- Settings persist in localStorage

**Enemy Types** (line 502-506):
- **Grunt**: Fast, weak, aimed or spread shot pattern
- **Interceptor**: Faster, weaker grunt, weave movement
- **Tank**: Slow, high HP, low spawn rate (stage 2+)
- Enemies spawn based on stage, difficulty, and frame-based progression

**Boss System** (line 508-1409):
- 6 boss designs (VANGUARD, BEHEMOTH, SPECTER, DREADNOUGHT, WRAITH, TITAN) cycling by stage
- Each has unique attack pattern set (radial, spread, homing, columns, telegraph warning)
- Boss enters from top, telegraphs before attacks, moves left-right
- 18 kills per stage triggers boss spawn after 1.2s delay

### Rendering Pipeline
- **Canvas**: 480×720 base (scales to fit window, max 1.6× zoom)
- **Sky themes**: 6 gradient themes with parallax cloud layers
- **Visual effects**: Screen shake, white flash on hit, red vignette pulsing on low health
- **Glowing**: Conditionally applied based on adaptive quality (0=full, 1=medium, 2=low)
- **Particles**: Explosion particles with velocity, fade, per-frame life countdown

### Input System
- **Keyboard**: Arrows/WASD to move, Space to fire, X/Shift or P/Esc for bomb/pause
- **Mouse**: Hovering canvas steers player, clicking fires (steered input has lower priority than keyboard)
- **Touch**: Left zone (55% width) for joystick movement, right zone for FIRE/BOMB buttons
- **Audio initialization**: Triggered on first user input

### Audio System (Web Audio Synthesis)
- No external sound files; all SFX generated via oscillators, noise, filters
- **beep()**: Frequency sweep with optional filter and panning
- **noiseBurst()**: White noise shaped by ADSR envelope
- Named SFX: `shoot`, `laser`, `explode*`, `hit`, `bomb`, `powerup`, `bossTelegraph`, `bossDown`, `stageClear`, `uiClick`, `gameover`
- Mute state persists in localStorage
- Master compressor on output

### Performance Optimization
- Adaptive quality system (lines 474-483): Tracks frame times over 45 frames
  - High FPS (>56) → reduce shadow effects, particle count
  - Low FPS (<40) → reduce particle count further, disable shadow blur
- Particle count scales with quality (line 1074)
- Reduced motion support via `prefers-reduced-motion` media query

### UI & Accessibility
- HUD shows score, best score, multiplier, weapon, lives (hearts), bombs
- Boss health bar appears during boss fight
- Combo popup at 10, 20+ kills
- Stage banners on transition
- Screen reader announcements for score milestones and game state changes
- Settings: Audio mute, motion effects toggle, difficulty selection

## Development Workflow

### Local Testing
Since there's no build process, open `index.html` directly in a browser:
```bash
# Option 1: Direct file open (file:// protocol)
open index.html

# Option 2: Simple HTTP server for testing
python3 -m http.server 8000
# Then visit http://localhost:8000
```

### Code Changes

**Modifying Gameplay**:
- Enemy spawn rates: `DIFFICULTIES` object (line 485)
- Enemy types and stats: `ENEMY_TYPES` (line 502)
- Boss designs and patterns: `BOSSES` (line 508), `BOSS_PATTERNS` (line 1372)
- Weapon behavior: `fireWeapon()` (line 1124), weapon stats in player object
- Difficulty-dependent values scale through `DIFFICULTIES[difficulty]` throughout update logic

**Visual Tweaks**:
- Colors: Inline `fillStyle`/`strokeStyle` assignments in draw functions
- Canvas size: `BASE_W`, `BASE_H` (line 753)
- Scaling limits: Line 757
- Themes: `THEMES` array (line 493)

**Audio**:
- SFX frequencies and durations: Individual `sfx.*` function definitions (line 624+)
- Master gain: `masterGain.gain.value` (line 556)
- Filter frequencies and types: Passed as `opts` to `beep()` and `noiseBurst()`

**Controls & Input**:
- Keyboard codes: Update key listeners (line 769+)
- Mouse steering smoothness: `maxStep` calculation (line 1167)
- Touch button sizes: `#fireBtn`, `#bombBtn` CSS (line 310+)

### Testing & Validation

**Manual testing checklist**:
1. Start game at each difficulty (Easy, Normal, Hard)
2. Verify weapon progression: Vulcan 1→2→3→Laser 1→2→3
3. Test bomb mechanic and count limit
4. Verify chain multiplier builds and resets after 2.2s
5. Test boss spawns at stage progression boundaries
6. Check all SFX play (mute/unmute toggle)
7. Verify screen shake and flash on hits (toggle motion effects)
8. Test touch controls on mobile (if available)
9. Verify high score persistence across page refreshes
10. Check pause/resume functionality

**Performance**:
- Monitor FPS in DevTools
- Quality setting should adapt automatically; check console logs if needed
- Test on low-end device simulator in DevTools

## Common Development Tasks

### Add a New Enemy Type
1. Add entry to `ENEMY_TYPES` (line 502) with stats and min stage
2. Define explosion SFX variant if needed and add to `EXPLODE_SFX` map (line 678)
3. Update `spawnEnemy()` probability check (line 1028-1030) if needed

### Add a New Boss Attack Pattern
1. Create a pattern function in `BOSS_PATTERNS` section (e.g., `fireWave()`)
2. Add pattern function to the boss's array in `BOSS_PATTERNS`
3. Test telegraph timing and bullet density

### Adjust Game Balance
- Spawn timing: Modify `spawnBase`, `spawnMin` in `DIFFICULTIES`
- Enemy bullet speed/spread: Adjust values in enemy shoot code (line 1219-1223)
- Boss damage to player: Reduce to player collision check to disable or add invuln timer
- Weapon effectiveness: Adjust `pierce` values and bullet speeds in `fireWeapon()`

### Extend Visuals
- New particle color: Add to `spawnExplosion()` calls
- New UI element: Add HTML to `#overlay` or persistent overlay, style with CSS, update in relevant state handlers
- New screen effect: Add CSS animation and toggle logic similar to vignette/flash

## Key Constraints & Considerations

1. **Single File**: Entire codebase is in one HTML file. Changes should remain inline; do not extract to separate files unless critical refactoring requires it.

2. **No External Assets**: All audio is synthesized, all graphics are drawn. Keep it that way for portability and simplicity.

3. **Canvas Sizing**: Base 480×720 is intentional (vertical mobile aspect ratio). Scaling caps at 1.6× to avoid overwhelming large screens.

4. **LocalStorage Keys**: Prefixed with `skyfighter.` (line 463-466) to avoid conflicts.

5. **Frame-Based Timing**: Enemy spawning, animations, and boss attacks use frame counter (not delta time) for consistent progression. Player movement and bullets use delta time for frame-rate independence.

6. **Quality Adaptive**: Shadow effects, particle count, and glow are conditionally applied. Disabling quality effects is preferable to reducing frame rate.

7. **Audio Context**: Web Audio context is lazy-initialized on first user input and may be suspended (line 545-561).

8. **Collision Detection**: Simple AABB (axis-aligned bounding box) via `rectsOverlap()` (line 1069).

## Performance Profiles by Change Type

- **Safe (no performance impact)**: Color tweaks, sound frequencies, text strings
- **Moderate (test FPS)**: Particle counts, enemy spawn rates, new SFX
- **High impact (benchmark)**: New boss patterns with dense bullet patterns, adding particle systems, changing canvas size

## Accessibility Features

- Reduced motion mode: CSS animations disabled, screen effects minimized
- Screen reader announcements: All state changes announced via `announce()` function
- Color contrast: HUD text uses gold on dark background, boss bar uses orange gradient
- Font sizing: Uses `clamp()` for responsive font scaling

---

**Last Updated**: 2026-09-18  
**Game Loop**: ~60 FPS target (adaptive quality maintains playability)  
**Browser Support**: Modern browsers with Canvas 2D, Web Audio, and ES6 support
