# Sky Fighter — Claude Code Development Guide

## Project Overview

**Sky Fighter** is a Raiden-style vertical shoot-'em-up game implemented as a single-file HTML5 application. The entire game engine, rendering, audio synthesis, input handling, and UI are contained in `index.html` as a self-contained, feature-complete arcade game.

**Key Features:**
- Single-file, no external dependencies or assets
- Procedurally generated audio using Web Audio API
- Responsive canvas scaling for desktop and mobile
- Touch controls with joystick for mobile devices
- Multiple difficulty levels (Easy, Normal, Hard)
- 6 unique boss types with procedural attack patterns
- 3 enemy types with distinct behaviors
- Weapon upgrades (Vulcan gun, Laser weapon)
- Score multiplier system based on kill chains
- Local storage for persistent high scores

**Technology Stack:**
- Pure HTML5 Canvas for rendering
- Web Audio API for sound effects and music
- No external frameworks or asset files
- Vanilla JavaScript with no dependencies

## How to Run the Project

1. **Open in browser:** Simply open `index.html` in any modern web browser
2. **Development server (optional):** For development, use any local HTTP server:
   ```bash
   python3 -m http.server 8000
   # or
   npx http-server
   ```
3. **Mobile testing:** Use responsive design mode or test on actual devices

## Controls

- **Desktop:**
  - Mouse: Move to steer, click to fire
  - Arrows/WASD: Move
  - Space: Fire
  - X or Shift: Bomb
  - P/Esc: Pause
- **Mobile:** Virtual joystick and buttons in bottom corners

## Code Structure

### Game State Variables
- `player`: Player ship object with position, weapon, health, cooldown
- `enemies`: Array of active enemy ships
- `boss`: Current boss object (if stage boss is active)
- `bullets`, `enemyBullets`: Player and enemy projectiles
- `particles`: Explosion particles
- `powerUps`: Collectible items (life, bombs, weapons)
- Game flow: `running`, `paused`, `gameOver`, `score`, `lives`, `stage`

### Key Systems

#### 1. **Rendering** (canvas 2D context)
- Scene: Sky gradient background with parallax clouds
- Entities: Player, enemies, boss, bullets, particles, power-ups
- HUD: Score, best score, lives, bombs, weapon level, multiplier, boss health bar

#### 2. **Physics & Collision**
- `rectsOverlap()`: AABB collision detection for all game objects
- Movement: Position updates per frame with delta time (dt)
- Screen boundaries: Clamping player position; off-screen cleanup for bullets

#### 3. **Audio** (Web Audio synthesis)
- `beep()`: Tone generation with optional frequency sweep and filter
- `noiseBurst()`: Burst noise for explosive effects
- `sfx` object: Individual sound effect generators (shoot, laser, explode, bomb, etc.)
- Master gain and dynamics compressor for audio balance

#### 4. **Input Handling**
- Keyboard: Direct key state tracking; preventDefault for Space
- Mouse: Pointer tracking on canvas; player follows smoothly when hovering
- Touch: Virtual joystick (left zone) and buttons (right zone) for mobile
- Priority: Keyboard/joystick input takes precedence over mouse steering

#### 5. **Enemy Behavior**
- Types: Grunt (fast, weak), Interceptor (swift, weaving), Tank (slow, tough)
- Spawning: Difficulty-based timing with progressive ramp
- Movement: Straight descent, some weave; Y-based off-screen culling
- Firing patterns: Spread (multiple angles) or aimed (at player)

#### 6. **Boss Battles**
- Spawn condition: After KILLS_PER_STAGE (18) normal enemies
- 6 boss designs with unique visual style and attack patterns:
  1. VANGUARD: Radial bursts, aimed fans, wide V-fans
  2. BEHEMOTH: Heavy columns, slow homing, dense radials
  3. SPECTER: Teleport + dash, tight spirals, dual aimed shots
  4. DREADNOUGHT: 8-way cross fire, homing missiles, radial walls
  5. WRAITH: Teleport barrage, double rings, cross-spreads
  6. TITAN: Heavy columns, full rings, relentless homing
- Boss enters from top, moves side-to-side
- Telegraph phase (visual + sound warning) before attack pattern
- Attack patterns: Procedural bullet spawning with multiple geometry types

#### 7. **Scoring & Progression**
- Chain mechanic: Consecutive kills without breaks; breaks after 2.2s idle
- Multiplier: x1 base, increases by 1 for every 5 chain kills (max x10)
- Stage progression: 18 kills → boss fight → next stage
- Best score: Persisted in localStorage with new best notification

#### 8. **Difficulty Scaling**
- **Easy:** Longer spawn intervals, slower enemies, fewer shots, 4 lives, 3 bombs, 0.8× boss HP
- **Normal:** Baseline spawn intervals, 1.4× enemy speed, 3 lives, 2 bombs, 1.0× boss HP
- **Hard:** Aggressive spawning, 1.9× speed, frequent fire, 3 lives, 2 bombs, 1.3× boss HP
- Adaptive difficulty: Spawn timer decreases every 300 frames (difficulty ramp)

#### 9. **Visual Effects**
- Screen shake: Magnitude and duration on major events (hit, bomb, boss death)
- Flash overlay: White flash on hit, full screen flash on bomb
- Vignette: Red pulsing overlay when health is critical (1 life)
- Combo pop: Large, centered text animation for milestone kills
- Stage banner: Announcement animation for stage transitions
- Particle system: Collision and explosion sparks with fade-out

#### 10. **Accessibility**
- Reduced motion support: Detects system preference; toggle in settings
- Screen reader live region: Announcements for score, stage, game state
- ARIA attributes: Buttons labeled with title attributes

## Development Guidelines for Claude Code Agents

### When Modifying This Project

#### 1. **Single-File Architecture**
- All code must remain in `index.html` — do NOT split into multiple files
- This is intentional: the game is a self-contained, deployable artifact with zero dependencies
- External assets are forbidden; use procedural/algorithmic generation for audio and visuals

#### 2. **No Breaking Changes to Core Mechanics**
- Do not alter the input system without preserving all existing control schemes
- Do not remove or disable accessibility features (reduced motion, ARIA, screen reader support)
- Do not modify localStorage keys without migration logic

#### 3. **Preserve Game Balance**
- Before adjusting difficulty values, test the impact across all three difficulty levels
- Maintain the feel of weapon progression (Vulcan → Laser should feel meaningfully different)
- Boss attack patterns are intentionally challenging; only adjust if game becomes unplayable

#### 4. **Code Clarity Over Cleverness**
- Prefer explicit variable names over shorthand (e.g., `enemyBullets` not `eb`)
- Keep functions focused; complex procedural patterns (boss attacks) can be extracted to their own generators
- Comment only WHY for non-obvious logic (e.g., "screen shake disabled in reduced-motion mode")

#### 5. **Performance Considerations**
- Use adaptive quality mode (tracked in `quality` variable) to skip effects on low-end devices
- Particle counts scale with quality; do NOT add thousands of always-on particles
- Canvas resize happens on window resize/orientationchange; test responsiveness
- Frame time tracking is built-in; use for adaptive rendering decisions

#### 6. **Testing Approach**
- Always test in the browser: open `index.html`, play through multiple stages
- Test on mobile: Responsive design or physical device with touch
- Verify all three difficulty levels work correctly
- Check that high score persists after page reload

#### 7. **When Adding Features**
- Keep feature scope small (e.g., "add a new enemy type" not "redesign the entire AI")
- Respect the procedural-generation philosophy: no external assets
- New sounds must use the `beep()` or `noiseBurst()` API
- New visuals must use Canvas 2D context (no WebGL or DOM overlays)

#### 8. **Git Commit Messages**
- Be specific: "Add homing missile enemy type with curve-following AI" not "Update game"
- Reference the stage or system affected: "Improve boss telegraph feedback with pitch sweep"
- Include reasoning when non-obvious: "Reduce tank spawn rate on hard; was blocking stage progression"

## Common Tasks

### Add a New Enemy Type
1. Add entry to `ENEMY_TYPES` object with `w`, `h`, `hp`, `speedMul`, `color`, `dark`, `score`, `minStage`
2. Add optional `weave: true` for sine-wave movement pattern
3. Update `spawnEnemy()` logic to include the new type based on stage/difficulty
4. Add explosion sound effect to `EXPLODE_SFX` if needed (call appropriate `sfx.explode*()`)
5. Test spawning and collision behavior

### Add a New Boss Pattern
1. Create a pattern function in the attack-pattern section that calls enemy-bullet spawners:
   - `fireRadial(b, count, speed, offset)`: Circular spread
   - `fireAimedSpread(b, count, speed, spreadStep)`: Aimed fan
   - `fireColumns(b, speed)`: Vertical lines
   - `fireCross(b, speed)`: 8-directional cross
   - `fireHomingSeeded(b, count, speed)`: Aimed with randomness
   - `fireSpiralBurst(b, speed)`: Rotating spiral
2. Add pattern function to new or existing boss's entry in `BOSS_PATTERNS`
3. Test on all difficulty levels; adjust speed/count as needed

### Tweak Difficulty
1. Edit values in `DIFFICULTIES` object (spawnBase, enemySpeed, enemyShootChance, lives, bombs, bossHpMul)
2. Test on each mode: easy should feel accessible, hard should require skill
3. Ensure boss fights remain challenging but not impossible

## Accessibility & Responsiveness

- **Reduced Motion:** Detect with `window.matchMedia('(prefers-reduced-motion: reduce)')`, respect in animations
- **Touch-Friendly:** Default virtual controls on touch devices; test on 16:9 and 9:16 aspect ratios
- **High DPI:** Canvas stretching is automatic; font sizes use `clamp()` for fluid scaling
- **Color Contrast:** HUD text uses white on dark semi-transparent backgrounds; accessible on all themes

## Performance Notes

- Particle count scales with `quality` (0=full, 1=medium, 2=low)
- Frame time averaging (45-frame window) drives quality adaptation
- Shadow effects and glow are disabled on low quality
- Cloud rendering is constant cost; enemy count varies by difficulty
- Audio context is lazy-initialized on first user interaction

## Browser Compatibility

- Target: Modern browsers (Chrome, Firefox, Safari, Edge)
- Fallback: Audio context with webkit prefix for older Safari
- Mobile: iOS Safari, Android Chrome with touch and orientation events
- Unsupported: IE11, older mobile browsers (no Canvas 2D or Web Audio)

---

**Last updated:** 2026-09-29  
This document describes the codebase state as of the latest commit. Update it when adding new systems or changing core behaviors.
