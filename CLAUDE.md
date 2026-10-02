# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project Overview

**Sky Fighter** is a Raiden-style vertical shoot-'em-up game implemented as a single HTML5 file (~1670 lines). It's a complete, self-contained game with no external dependencies — all audio is synthesized via WebAudio API, graphics are drawn with Canvas 2D, and game logic runs in a browser.

## Quick Start & Common Commands

Since this is a single HTML file:

- **Run locally**: Open `index.html` in a browser (double-click or drag to browser)
- **No build step required**: The file is production-ready as-is
- **Testing gameplay**: Load in browser, click "Start Game", select difficulty, play
- **Development**: Edit `index.html` directly, refresh browser to see changes

## Architecture & Code Organization

The entire game runs in a single `<script>` tag (lines 422-1666). The structure follows a functional architecture organized in logical sections:

### Core Systems

1. **Input Handling** (lines 767-861)
   - Keyboard input: Arrow keys, WASD, Space (fire), X/Shift (bomb), P/Esc (pause)
   - Mouse: Move to steer (glide toward pointer), click to fire
   - Touch: Joystick in left half, FIRE/BOMB buttons in right half
   - All input modes coexist; keyboard/joystick take priority over mouse

2. **Audio Subsystem** (lines 542-676)
   - No external audio files — all SFX are synthesized using WebAudio API
   - `ensureAudio()`: Lazy-initializes AudioContext on first user interaction (required for browser autoplay policy)
   - `beep()`: Creates pitched tones with optional frequency sweep, filters, stereo panning
   - `noiseBurst()`: Generates noise for explosion and bomb sounds
   - `sfx` object: Named sound effects (shoot, laser, explode, hit, bomb, powerup, etc.)
   - Audio respects mute setting stored in localStorage

3. **Game State** (lines 864-869)
   - `player`: Position, weapon, level, health, invulnerability
   - `bullets`, `enemies`, `enemyBullets`, `powerUps`: Arrays of entities
   - `boss`: Active boss object (null when no boss)
   - `score`, `lives`, `stage`, `chain`, `multiplier`: Game metrics
   - Persisted: best score, mute state, difficulty, reduced motion preference

4. **Game Loop & Rendering** (lines 1640-1652 game loop, 1520-1638 draw)
   - `requestAnimationFrame(loop)` at 60fps target
   - Delta-time based updates for frame-rate independence
   - Adaptive quality system (lines 473-483): Tracks frame times, reduces particle/shadow quality on slower devices
   - Shake effect applied via canvas translate during render
   - Screen is drawn each frame: sky gradient → sun glow → clouds → player → bullets → enemies → boss → effects

5. **Canvas & Sizing** (lines 753-765)
   - Base resolution: 480×720 (portrait)
   - Auto-scales to fit window while maintaining aspect ratio (max 1.6× scale)
   - Responsive to resize and orientation change events

### Game Mechanics

1. **Player** (lines 876-910 init, 1148-1172 movement, 1124-1146 weapons)
   - Two weapons: Vulcan (spread shot) and Laser (piercing beam)
   - Weapon levels 1–3, then cycles back to level 1 of opposite weapon
   - Cooldown between shots differs by weapon
   - Invulnerability frames after taking damage

2. **Enemies** (lines 1022-1041 spawning, 1210-1227 update)
   - Three types: Grunt (fast, weak), Interceptor (very fast, weaves), Tank (slow, tough, hp bar)
   - Min stage unlock for tank (stage 2+)
   - Spawning rate scales with difficulty and game progression
   - Enemies shoot aimed or spread patterns toward player
   - Each type has distinct color and explosion sound

3. **Bosses** (lines 1043-1056 spawn, 1411-1439 update, 1372-1409 patterns)
   - 6 boss designs (Vanguard, Behemoth, Specter, Dreadnought, Wraith, Titan) cycling by stage
   - Each boss has unique visual design and 3 distinct attack patterns
   - Attack patterns include: radial bursts, aimed spreads, spirals, homing shots, teleport barrages
   - Boss telegraphs 22 frames before attacking (visual + audio cue)
   - Boss enters screen, bounces side-to-side, teleports (some) during combat

4. **Scoring & Chain System** (lines 1087-1101, 1244-1248)
   - Base score per enemy kill (10–35 depending on type)
   - Chain multiplier: 1× base, up to 10× after 50 consecutive kills
   - Chain resets on any non-kill (dodge timeout) after 2.2 seconds
   - Visual feedback: combo popups at 10/20/30 kills, stage banners, new best score alert

5. **Power-Ups** (lines 1058-1067, 1289-1295, 1304-1322)
   - Life (6% spawn rate), Bomb (8% spawn rate), Weapon (20% spawn rate)
   - Spawn on enemy death, fall downward
   - Weapon powerup upgrades level or toggles weapon type
   - All shown with rotating glow and collision detection

### Difficulty System

Three preset difficulties (lines 485-489):
- **Easy**: Slow spawn (75–40), slow enemies (1.0×), low fire rate (0.45), 4 lives, 3 bombs, 0.8× boss HP
- **Normal**: Default (58–26), moderate (1.4×), 0.7 fire rate, 3 lives, 2 bombs, 1.0× boss HP
- **Hard**: Aggressive (40–18), fast (1.9×), high fire rate (0.95), 3 lives, 2 bombs, 1.3× boss HP

All parameters scale dynamically: spawn rate decreases over time, enemy speed is procedural.

### Visual & Audio Effects

- **Screen shake**: Triggered on player hit, bomb, boss hit; reduced motion respects user preference
- **Flash**: White flash on hit, bomb explosion (yellow)
- **Vignette**: Red pulse during critical health (1 life left)
- **Combo popup**: Yellow text at center, scales and fades
- **Stage banner**: Large text notification on stage transitions and boss entry
- **Reduced motion**: Toggle respected; animations disabled for prefers-reduced-motion media query

### Theme System

Six sky themes cycle by stage (lines 493-500):
- Daylight Front, Sunset Corridor, Night Approach, Storm Front, Arctic Wastes, Volcanic Ridge
- Each includes sky gradient colors, cloud fill/sun colors, used in procedural sky/cloud rendering

## Key Data Structures

### Player Object
```javascript
{
  x, y,           // Screen position (center)
  w, h,           // Dimensions (40×40)
  speed,          // Movement speed (260 px/s)
  cooldown,       // Fire rate limiter
  rapid,          // Unused field (can be removed)
  weapon,         // 'vulcan' | 'laser'
  level,          // 1–3
  invuln          // Invulnerability timer (seconds)
}
```

### Enemy Object
```javascript
{
  x, y, w, h,     // Position & dimensions (34×34 grunt, 26×26 interceptor, 46×46 tank)
  speed,          // Vertical speed (frames)
  type,           // 'grunt' | 'interceptor' | 'tank'
  hp,             // Hit points (1 for most, 4 for tank)
  color, dark,    // Fill and shadow colors
  score,          // Points on kill
  canShoot,       // Boolean
  pattern,        // 'spread' | 'aimed'
  shootTimer,     // Countdown to next shot
  weave,          // Boolean (interceptor only)
  weaveSeed, baseX // Weave offset & origin
}
```

### Boss Object
```javascript
{
  x, y, w, h,     // Position & dimensions (90×80)
  hp, maxHp,      // Health tracking
  design,         // Reference to BOSSES[index] with colors and name
  entering,       // Boolean (descending into arena)
  vx,             // Horizontal velocity (bounces)
  attackTimer,    // Countdown to next attack
  telegraph,      // Visual/audio cue before pattern fires
  patternIndex,   // Cycles through 3 patterns
  phaseTimer      // Unused (can be removed)
}
```

### Bullet Object (Player)
```javascript
{
  x, y, w, h,     // Position & dimensions (4×12 vulcan, 5×20 laser)
  speed,          // Upward speed (500–640 px/s)
  vx,             // Horizontal velocity (vulcan spread)
  pierce,         // Piercing counter (laser only, 1–2)
  laser           // Boolean
}
```

### Bullet Object (Enemy/Boss)
```javascript
{
  x, y, w, h,     // Position & dimensions
  speed,          // Downward speed (200–230 px/s, vertical movement)
  vx, vy,         // For radial/homing patterns (dx/dy overwrite these)
  dx, dy,         // For boss patterns (radial, homing, etc.)
  radial          // Boolean (boss bullets only)
}
```

## Collision Detection

Simple AABB overlap test (line 1069-1071):
```javascript
function rectsOverlap(a, b) {
  return Math.abs(a.x - b.x) < (a.w + b.w) / 2 &&
         Math.abs(a.y - b.y) < (a.h + b.h) / 2;
}
```

All entity-vs-entity collisions use this:
- Player bullets vs enemies
- Player bullets vs boss
- Enemy bullets vs player
- Boss vs player
- Power-ups vs player

## Performance Considerations

1. **Particle count**: Adapts based on frame time (quality 0=full, 1=60%, 2=35%). Limits explosion particles to smooth gameplay on low-end devices.
2. **Canvas blur effects**: Shadow glow only drawn at quality level 0; disabled on slow devices.
3. **Enemy spawn rate**: Decreases over time (hardcoded ramp every 300 frames = 5s).
4. **Collision checks**: Nested loops over arrays; acceptable due to small entity counts (<50 typically).

## Common Modifications

### Adding a New Enemy Type
1. Add entry to `ENEMY_TYPES` object (line 502–506) with `w`, `h`, `hp`, `speedMul`, `color`, `dark`, `score`, `minStage`
2. Update `spawnEnemy()` probability check (lines 1028–1030)
3. Add explosion sound to `EXPLODE_SFX` map (line 678) if different from grunt
4. Add matching sound function in `sfx` object if needed

### Adding a New Boss
1. Add entry to `BOSSES` array (line 508–515) with hull colors, wing color, core color, name, `hpMul`
2. Define 3 attack patterns in `BOSS_PATTERNS` array (line 1372–1409)
3. Patterns use helper functions like `fireRadial()`, `fireAimedSpread()`, etc.
4. Boss automatically cycles; no other code changes needed

### Tweaking Difficulty
Edit `DIFFICULTIES` object (lines 485–489). All spawn logic and enemy behavior keys off these values.

### Adding a Visual Theme
Add entry to `THEMES` array (line 493–500) with `sky` (3 gradient stops), `cloud`, `sun` RGB strings, and `name`.

## Debugging & Testing

- **Console logs**: Add `console.log()` in update/draw loop; won't show in game overlay
- **Visual inspection**: Toggle reduced motion button (🎆) to see shake/flash more clearly
- **Pause mid-game**: Press P or Esc to freeze state and inspect
- **Difficulty cycling**: Difficulty selection persists across sessions via localStorage
- **Performance**: Frame time tracking logs to `frameTimes` array; check avg in console
- **Audio**: Mute toggle stored; inspect `audioCtx.state` to debug WebAudio issues

## Storage & Persistence

LocalStorage keys (lines 463–466):
- `skyfighter.bestScore`: Best score ever
- `skyfighter.muted`: User mute preference (1 or 0)
- `skyfighter.difficulty`: Last selected difficulty
- `skyfighter.reducedMotion`: Reduced motion toggle override

All persist across sessions.

## Accessibility Features

- Reduced motion support: Detects `prefers-reduced-motion` OS setting, disables animations/shake/flash
- Screen reader support: `aria-live` region (#srLive) announces state changes (kills, lives lost, stage transitions)
- Semantic HTML: Button roles, labels where needed
- Safe-area insets: Layout respects notches/status bars on mobile

## Browser Compatibility

- Canvas 2D rendering: IE11+ (but no modern defaults)
- WebAudio API: Chrome, Firefox, Safari, Edge (not supported in IE)
- Touch events: Modern browsers (iOS Safari 13+, Android Chrome/Firefox)
- LocalStorage: IE8+
- Responsive viewport: Assumes `<meta name="viewport">` is set (it is, line 5)

Target: Modern browsers (last 2 years), degrades gracefully on older browsers (game runs, audio missing if no WebAudio).

## File Structure

Single `index.html` contains:
- Lines 1–359: HTML + embedded CSS (inline `<style>`)
- Lines 361–420: HTML markup (canvas, HUD, overlay, buttons, touch controls)
- Lines 422–1666: JavaScript game logic (IIFE to avoid global scope pollution)

## Future Improvements (Optional)

- **Mobile: Implement haptic feedback (Vibration API) on hits/bombs**
- **Leaderboard: Sync best scores to a cloud backend**
- **Replays: Record input sequence, replay locally**
- **Accessibility: Add in-game audio descriptions for boss telegraph**
- **Graphics: Implement WebGL for higher-quality particle effects**
