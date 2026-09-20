# Sky Fighter — Codebase Documentation

## Project Overview

**Sky Fighter** is a Raiden-style vertical shoot-'em-up game built as a single-file HTML5 canvas application. The game features airplane combat with weapon upgrades, smart bombs, stage-based progression with boss battles, and a production-quality presentation including mobile controls, audio synthesis, accessibility features, and adaptive performance optimization.

**Location**: `index.html` (1669 lines of HTML, CSS, and embedded JavaScript)

**Key Technologies**:
- HTML5 Canvas for rendering
- Web Audio API for synthesized sound effects
- Keyboard, touch, and mouse input handling
- LocalStorage for game state persistence

---

## Game Features

### Core Gameplay
- **Player Aircraft**: A shaded airplane sprite with weapon slots and bomb capacity
- **Weapon System**: Vulcan (rapid-fire) and Laser weapons with 3 power levels
- **Smart Bombs**: Limited-use screen-clear ability that damages bosses
- **Score Multiplier**: Kill-chain combo system for consecutive enemy kills
- **Power-ups**: Weapon level, bombs, and extra lives

### Enemies & Bosses
**Enemy Types** (3 archetypes):
- **Grunt**: Basic weak enemy
- **Weaving Interceptor**: Mobile enemy with diagonal movement
- **Tank**: High-health multi-hit enemy with damage indicators

**Boss Designs** (6 rotating bosses):
- Vanguard, Behemoth, Specter (initial 3)
- Dreadnought, Wraith, Titan (expanded roster)

Each boss has:
- Unique visual design and color scheme
- 3-4 distinct bullet-pattern attack sets
- Pre-attack telegraph flash for fair gameplay
- Multi-phase health bar with stage-based difficulty scaling

### Stage Progression
- **6-stage structure** with escalating enemy waves
- **3 difficulty modes**: Normal, Hard, Insane (affects wave intensity, boss patterns, power-up frequency)
- **Visual themes**: 6 rotating backdrop themes (daylight, sunset, night variants)
- **Progression**: Each stage culminates in a boss battle; clearing it advances to the next stage

### Audio System
Synthesized audio engine with:
- **Master compression** to prevent clipping during dense explosions
- **Per-sound biquad filtering** and pitch sweeps for weight
- **Stereo panning** driven by on-screen position (spatial audio)
- **Layered sound sequences**:
  - Enemy explosions (distinct per type: grunt/interceptor/tank)
  - Hit/bomb/power-up pickup cues
  - Boss telegraph and death stingers
  - Ascending stage-clear fanfare
  - Game-over stinger
  - UI click feedback

### Accessibility
- **Reduced-motion toggle** (respects OS `prefers-reduced-motion` setting)
  - Dampens screen shake
  - Disables flash/pulse animations (photosensitive-friendly)
- **ARIA Live Region** announces:
  - Life loss
  - Boss defeats
  - Game-over results
- **Screen reader support** for HUD text

### Mobile & Control Systems
- **Touch Joystick**: On-screen directional pad in the bottom-left corner
- **Keyboard Controls**: Arrow keys or WASD for movement, Space/Enter to fire
- **Mouse Steering**: Pointer movement glides the plane toward cursor (capped at 1.7x speed, non-teleporting)
  - Mouse button click to fire
  - Crosshair reticle tracks cursor
  - Keyboard and touch input take priority when active
- **Mobile Viewport**: Handles safe areas (notch/punch-hole) and orientation changes

### Visual Polish
- **Shaded Plane Sprites**: Cockpit, flame exhaust, and tail detail
- **Glowing Particles**: Bullets and power-ups with glow effects
- **Parallax Cloud Layers**: Scrolling background depth
- **Sun Glow**: Atmospheric lighting
- **Low-Health Vignette**: Red pulse animation when health is critical
- **Hit Flash**: White screen flash on damage
- **Screen Shake**: On player hit (respects reduced-motion setting)
- **Animated Title Overlay**: Entrance and stat displays
- **Combo Popups**: Centered "COMBO x#" popup with scaling animation
- **Stage Banner**: Large animated stage-name display on progression
- **Boss Health Bar**: Top-center health indicator with smooth transition

### Persistence & UI
- **LocalStorage**: Saves high score and game settings (volume, reduced-motion, difficulty)
- **HUD Stats** (top-center):
  - Score with multiplier
  - Health (heart icons: full/critical show beating animation)
  - Bombs remaining (bomb icons)
  - Current weapon and level
- **Top-Right Controls**: Mute toggle, reduce-motion toggle, pause button
- **Pause Menu**: Glass-morphic overlay card with centered text
- **Settings Persistence**: Carries difficulty mode and audio mute state across sessions

### Performance Optimization
- **Rolling Frame-Time Monitor**: Tracks performance per frame
- **3-Tier Quality System**:
  - **High**: Full particles, shadow-blur glows (target 60 fps)
  - **Medium**: Reduced particles, no shadow blur
  - **Low**: Minimal particles, minimal effects
- **Automatic Tier Adjustment**: Steps down if frame time exceeds threshold, steps back up when recovered
- **Responsive Design**: Scales HUD text and UI elements by viewport size

---

## Code Architecture

### Single-File Structure
Everything is embedded in `index.html`:

```
<head>
  - Metadata & favicon
  - Embedded CSS (inline in <style> tag)
</head>
<body>
  - Game container & canvas
  - HUD elements (stats, overlay, boss bar)
  - Flash & vignette effects
  - Popup feedback elements
  - Embedded JavaScript (inline in <script> tag)
</body>
```

### Major JavaScript Subsystems

1. **Game State (`game` object)**
   - `state`: Enum for MENU, PLAYING, GAME_OVER
   - `difficulty`: 0 (Normal), 1 (Hard), 2 (Insane)
   - `stage`: Current stage number (0-5 loop)
   - `score`, `combo`, `health`, `bombs`, `weaponLevel`
   - `particles[]`, `enemies[]`, `bosses[]`, `bullets[]`, `effects[]`
   - `isPaused`, `isMuted`, `prefersReducedMotion`

2. **Player Aircraft**
   - Position, velocity, hitbox
   - Current weapon type and level
   - Bomb count
   - Firing logic and cooldown

3. **Weapon System**
   - Vulcan: Rapid-fire spread pattern
   - Laser: Slower, tighter pattern with homing hints
   - Power-up system: Level 1–3 upgrades

4. **Enemy Spawner**
   - Wave-based spawning by stage
   - Difficulty-based enemy type distribution
   - Teleport edges for wrapping behavior

5. **Enemy AI**
   - **Grunt**: Straight-line descent
   - **Interceptor**: Weaving side-to-side
   - **Tank**: Slow descent with health bar

6. **Boss Engine**
   - Boss roster cycling (6 designs)
   - Attack pattern scheduler (3–4 patterns per boss)
   - Health tracking and phase detection
   - Telegraph flash before attacks
   - Collision with bombs (smart-bomb damage)

7. **Particle & Effect System**
   - Bullet particles (with velocity & lifetime)
   - Explosion particles (trails and debris)
   - Power-up icons (colored floating sprites)
   - Screen shake and vignette pulse effects

8. **Audio Synthesis**
   - Master AudioContext and compressor
   - Per-sound biquad filtering and gain nodes
   - Pitch-sweep envelopes
   - Stereo panning by x-position
   - Layered explosion sequences

9. **Rendering**
   - Canvas context with requestAnimationFrame loop
   - Z-layer ordering (particles, effects, HUD)
   - Adaptive quality tier rendering
   - Mobile safe-area and viewport fit handling

10. **Input Handling**
    - Keyboard listener (arrow keys, WASD, Space, Enter)
    - Touch joystick with analog stick emulation
    - Mouse move and click listeners
    - Priority system: keyboard > touch > mouse steering

11. **UI & HUD**
    - Dynamic HUD text (score, health, bombs, weapon)
    - Pause overlay with resume/quit options
    - Boss health bar visibility toggle
    - Stage banner animation
    - Combo popup scaling

12. **Settings & Persistence**
    - LocalStorage read/write for high score
    - Mute toggle with visual indicator
    - Reduced-motion preference
    - Difficulty selector
    - ARIA announcements

---

## How to Run

### Play the Game
1. Open `index.html` directly in a modern web browser
2. Press a key or click to start
3. Use arrow keys/WASD/joystick to move
4. Click/Space/Enter to fire
5. Destroy enemies and bosses to progress through stages

### Debug & Development
- **Open DevTools Console** (F12) to see any warnings/errors
- **Mute/Reduce-Motion Toggles**: Top-right corner for accessibility testing
- **Pause Button**: Space key or the ⏸ button to pause mid-game
- **LocalStorage**: High scores and settings persist; clear to reset

### Headless Testing (Playwright Example)
```javascript
await page.goto('file:///path/to/index.html');
await page.click('#gameCanvas'); // Start game
// Verify firing, collisions, boss patterns, etc.
```

---

## Key Implementation Details

### Collision Detection
- **Bounding-box AABB** for bullets vs. enemies and bombs vs. bosses
- **Circle collision** option for power-ups (configurable)
- **Player hitbox**: Smaller than visual sprite to give margin

### Difficulty Scaling
- **Normal**: Standard wave frequency, single-pattern boss attacks, frequent power-ups
- **Hard**: Denser waves, 2-pattern boss attacks, fewer power-ups
- **Insane**: Rapid waves, 3-4-pattern boss attacks, rare power-ups

### Boss AI Pattern Selection
- Each boss has a pattern queue that cycles
- Telegraph flash plays before each pattern fires
- Patterns include:
  - Radial burst (all directions)
  - Aimed fan (at player)
  - Bullet wall (column sweep)
  - Homing spread (tracking projectiles)
  - Spiral (rotating rings)
  - Twin aimed shots (pair tracking player)

### Screen Shake & Vignette
- Screen shake triggered on player hit (translates canvas context)
- Vignette pulse (red gradient opacity animation) when health is low
- Both animations respect `prefersReducedMotion` setting

### Audio Note
The synth engine uses a master compressor to handle overlapping explosion sounds without clipping. Enemy deaths trigger layered sound sequences (pitched and filtered) to give each explosion type (grunt/interceptor/tank) a distinct auditory signature.

---

## Testing Checklist

- [ ] Player movement (keyboard, touch, mouse steering)
- [ ] Weapon firing and power-up pickups
- [ ] Enemy spawning and collision detection
- [ ] Boss patterns and health tracking
- [ ] Bomb detonation and screen clear
- [ ] Score and combo multiplier
- [ ] Difficulty mode switching
- [ ] Pause/resume functionality
- [ ] Audio synthesis (no clipping, proper panning)
- [ ] Mobile viewport and safe-area handling
- [ ] Accessibility (reduced-motion, ARIA announcements)
- [ ] Performance under high particle load
- [ ] LocalStorage persistence (high score, settings)
- [ ] Stage progression and boss rotation

---

## Future Enhancement Ideas

- Leaderboard backend integration
- Additional enemy types or boss designs
- Procedural wave generation
- Online multiplayer cooperative mode
- Mobile app wrapper (Electron/Cordova)
- Replay system recording and playback
- Cheat codes (invulnerability, weapon spawn)
- Tutorial overlay for new players
- More audio tracks (dynamic music)
- Vibration feedback for mobile devices

---

## Notes for Claude Code Agents

**File Complexity**: This is a large single-file game (1669 lines). Edits should be carefully reviewed to avoid disrupting the intertwined game loop, rendering, and physics logic.

**Search Tips**:
- Search for `function` to find major subsystems
- Look for comments marked with `//` to find key decision points
- Use "Ctrl+F" in the browser to find game state references

**Commits**: This codebase has a clean linear history. When making changes, follow the existing pattern and commit with descriptive messages citing the feature or bugfix.

**Testing**: Always verify in the browser after changes, especially around collision detection, audio, and input handling.
