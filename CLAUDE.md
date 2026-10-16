# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project Overview

**Sky Fighter** is a Raiden-style vertical shoot-'em-up game. It's a single-file HTML5 game (~1670 lines) with no build process—just open `index.html` in a browser to play. The entire game is self-contained: HTML structure, embedded CSS, and inline JavaScript (no external dependencies).

## How to Run

- **Local Testing**: Open `index.html` directly in a web browser
- **Live Server**: If using a local development server: `python -m http.server 8000`, then navigate to `http://localhost:8000/index.html`
- **Desktop Testing**: The game is fully responsive and works on desktop browsers (Chrome, Firefox, Safari, Edge)
- **Mobile Testing**: Works on iOS Safari and Android Chrome with touch controls

No build, test, or installation steps are needed.

## Architecture Overview

The game follows a classic game loop architecture with a single global IIFE (Immediately Invoked Function Expression) containing all game logic. The structure is:

```
┌─ Canvas Setup & Rendering (Canvas 2D API)
├─ Input Handling (Keyboard, Mouse, Touch)
├─ Game State Management (player, enemies, bullets, boss, score, etc.)
├─ Update Loop (frame-by-frame game logic)
├─ Collision Detection (rectangle overlap)
├─ WebAudio Synthesis (procedural sound generation)
├─ Difficulty Tuning & Adaptive Performance
└─ UI/UX Overlays (menus, HUD, pause screen)
```

### Key Systems

#### 1. **Game Loop & Frame Management** (lines 1640–1666)
- Uses `requestAnimationFrame` for 60 FPS targeting
- Delta-time-based updates (clamped at 50ms to prevent spiral-of-death)
- Performance tracking via frame times array; quality auto-adjusted (lines 474–483)

#### 2. **Player & Input Handling** (lines 768–861)
- **Keyboard**: Arrow keys / WASD for movement, Space/click for fire, X/Shift for bomb, P/Esc for pause
- **Mouse**: Moves to point and fires; mouse input has lower priority than keyboard/joystick
- **Touch**: Dual-zone touch controls—left zone is movement joystick, right zone has fire and bomb buttons
- Player object: position, speed, weapon type, level, cooldown, invulnerability timer

#### 3. **Weapon System** (lines 871–1146)
- Two weapons: **Vulcan** (rapid spread shots) and **Laser** (piercing shots with wider lanes at higher levels)
- Three upgrade levels per weapon; reaching level 3 switches to the other weapon
- Upgrade priorities: weapon level → weapon switch
- Different fire rates: Vulcan (0.15s), Laser (0.1s)

#### 4. **Enemy & Boss Spawning** (lines 1022–1056, 1372–1409)
- **Enemy Types**: Grunt (weak, common), Interceptor (fast, weaving), Tank (armored, high-HP)
- Stage progression: Each stage requires 18 kills (`KILLS_PER_STAGE`), then boss appears
- **Boss System**: Six boss designs (VANGUARD, BEHEMOTH, SPECTER, DREADNOUGHT, WRAITH, TITAN), each with unique attack patterns
- Boss telegraph mechanic: 22-frame delay before attack gives player reaction time

#### 5. **Collision Detection** (lines 1069–1302)
- Simple rectangle-based overlap (`rectsOverlap`): checks all bullets vs enemies, bullets vs boss, enemies vs player, bullets vs player, boss vs player, powerups vs player
- Collision checks happen in a tight loop after each update to ensure no missed frames

#### 6. **Difficulty Settings** (lines 485–489)
- Three modes: Easy, Normal, Hard
- Tuned via spawn rate, enemy speed, shoot chance, lives, bomb count, boss HP multiplier
- Persisted to localStorage

#### 7. **Score & Combo System** (lines 1087–1101)
- Chain mechanic: Each kill increments combo count; combo resets after 2.2 seconds of inactivity
- Multiplier scales with chain: 1–10x (1x at 0 kills, 10x at 45+ kills)
- Bomb kills and boss kills add bonus points (5 and 500 respectively)

#### 8. **WebAudio Synthesis** (lines 542–676)
- No external audio files; all sounds are synthesized procedurally using Web Audio API
- Master gain + compressor for polished sound
- Separate SFX functions for shoot, laser, explosions, hit, bomb, powerup, boss telegraph, stage clear, gameover
- Spatial audio: pan effect based on screen position

#### 9. **Adaptive Performance** (lines 474–483)
- Tracks frame times over 45 frames; adjusts quality level (0=full, 1=medium, 2=low)
- High frame time → reduce particle count & glow effects
- Lower frame time → restore effects
- Quality affects explosion particle count and shadow rendering

#### 10. **Visual Theme & Stage Design** (lines 493–500)
- Six rotating sky themes (Daylight Front, Sunset Corridor, Night Approach, etc.)
- Theme changes every stage
- Procedurally drawn clouds, gradient backgrounds, particle effects

#### 11. **Accessibility Features** (lines 520–540, 349–358)
- Reduced motion mode: disables animations if system prefers it or user toggles it
- Respects `prefers-reduced-motion` media query
- Screen reader announcements via ARIA live region (`srLive`)

#### 12. **Data Persistence** (lines 463–466, 517–527)
- localStorage keys: `skyfighter.bestScore`, `skyfighter.muted`, `skyfighter.difficulty`, `skyfighter.reducedMotion`
- Best score persists across sessions
- User preferences (mute, difficulty, motion) remembered

### Important File Structure Notes

**The entire codebase is in `index.html`.** No separation into modules or external files. All HTML, CSS, and JavaScript are inline. The logical separation is by section:

- **Lines 10–359**: CSS (styles for canvas, HUD, overlays, animations)
- **Lines 362–420**: HTML structure (canvas, UI elements, touch controls)
- **Lines 422–1666**: JavaScript game code (wrapped in IIFE)

## Performance Considerations

1. **Canvas Rendering**: Direct 2D drawing; no WebGL or sprite sheets
2. **Particle Scaling**: Explosion particles scale down at medium/low quality to maintain 60 FPS
3. **Glow Effects**: Shadow blur only rendered at quality level 0 to save GPU cycles
4. **Enemy Limiting**: No hard cap on enemy count, but spawn rate adapts to difficulty; expect 10–40 active enemies at any time
5. **Screen Shake**: Reduces to 10% amplitude on reduced-motion mode; skipped entirely if user has animations disabled

## Common Development Tasks

- **Change difficulty balance**: Edit `DIFFICULTIES` object (lines 485–489)
- **Add/modify boss attack patterns**: Edit `BOSS_PATTERNS` array (lines 1372–1409); each pattern is a function that calls enemy bullet spawners
- **Adjust weapon tuning**: Modify `fireWeapon()` function (lines 1124–1146)
- **Tweak visual themes**: Edit `THEMES` array (lines 493–500)
- **Add new enemy types**: Add to `ENEMY_TYPES` object (lines 502–506) and handle spawning in `spawnEnemy()` (lines 1022–1041)
- **Change audio**: Modify `beep()`, `noiseBurst()`, or SFX functions (lines 566–676)

## Important Design Decisions

1. **Single-File Architecture**: Keeps deployment simple (one file to ship), but limits code organization. Good for a jam or prototype; would need refactoring if feature set grows significantly.
2. **Procedural Audio**: Avoids external asset dependencies and loading overhead; trades audio fidelity for smaller file size and instant startup.
3. **Canvas 2D**: Simpler than WebGL for a 2D game; sufficient performance with adaptive quality scaling.
4. **Delta-Time Updates**: Frame-rate independent; physics and game logic scale with elapsed time, not frame count.
5. **Boss Telegraph System**: 22-frame delay before boss attacks makes the game fair and telegraphs danger to the player.
6. **Touch Controls Dual-Zone**: Left is movement (joystick), right is action buttons; mirrors classic arcade layout adapted for mobile.

## Debugging Notes

- **Performance issues**: Check frame time tracking (`trackPerf` function). If avg > 1/40, quality degrades. Lower quality = fewer particles, no shadows.
- **Collision issues**: Verify `rectsOverlap` logic; uses half-width/half-height from center. Collision detection runs after update, so stale frame data is possible if framerate drops.
- **Audio not playing**: Check mute state and audio context. Audio requires user interaction to initialize (`ensureAudio`). Some browsers block autoplay audio until user gesture.
- **Touch controls not working**: Verify `isTouchDevice` detection (line 778). Some browsers report maxTouchPoints = 0 even on touch devices.

## Browser Compatibility

- **Modern browsers**: Chrome 90+, Firefox 88+, Safari 14+, Edge 90+
- **Mobile**: iOS Safari 14+, Android Chrome 90+
- **Features used**: Canvas 2D, Web Audio API, localStorage, TouchEvent, requestAnimationFrame, arrow functions, template literals, destructuring
- **No polyfills needed** for target browsers.
