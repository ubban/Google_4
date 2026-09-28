# Sky Fighter — Codebase Guide

**Project**: Sky Fighter — A Raiden-style vertical shoot-em-up game
**Repository**: ubban/Google_4
**Branch**: claude/inspiring-planck-n8ip9k
**Last Updated**: 2026-09-28

## Overview

Sky Fighter is a complete browser-based shoot-em-up arcade game built in a single HTML file (`index.html`). The game features:

- **Single player arcade gameplay**: Vertical scrolling with 6 stages, boss battles, and score tracking
- **Weapon system**: Multiple weapon types (Vulcan, Laser, Spread) with level progression
- **Enemy variety**: 3 enemy types (Grunt, Interceptor, Tank) with different behaviors
- **Boss encounters**: 6 unique boss designs with individual visual styles
- **Difficulty levels**: Easy, Normal, Hard with configurable gameplay parameters
- **Touch + keyboard controls**: Desktop mouse/keyboard and mobile touch support
- **Procedural audio**: All sound effects generated via Web Audio API (no external assets)
- **Responsive design**: Works on desktop, tablet, and mobile (respects safe-area-insets)
- **Accessibility**: Screen reader support, reduced-motion preference, semantic HTML

## Architecture

### Single-File Design
The entire game runs in `index.html`:
- **CSS**: Embedded styles (lines 10–358) handle all UI, animations, and responsive layout
- **HTML**: Semantic structure with canvas, HUD, overlay screens, and touch controls (lines 361–420)
- **JavaScript**: Game loop, entity management, physics, collision, audio, and UI logic (line 422 onwards)

### Code Structure (JavaScript)

1. **Initialization** (lines 423–527)
   - DOM element caching for performance
   - LocalStorage keys for persistence (best score, mute, difficulty, reduced-motion)
   - Initial state setup

2. **Performance Tracking** (lines 473–483)
   - Adaptive quality system (full → medium → low quality) based on frame time
   - Tracks 45-frame rolling average; adjusts `quality` if avg > 1/40ms (throttle) or < 1/56ms (boost)

3. **Game Configuration** (lines 485–523)
   - `DIFFICULTIES`: Easy/Normal/Hard spawn rates, enemy speed, lives, bombs, boss HP multiplier
   - `KILLS_PER_STAGE`: 18 enemies per stage before advancing
   - `THEMES`: 6 parallax background themes (sky gradients, clouds, sun)
   - `ENEMY_TYPES`: Grunt, Interceptor, Tank with HP, speed, color, scoring
   - `BOSSES`: 6 unique bosses (Vanguard, Behemoth, Specter, Dreadnought, Wraith, Titan)

4. **Audio System** (lines 542–595+)
   - Web Audio Context management (`ensureAudio()`)
   - `beep(freq, dur, type, vol, opts)`: Oscillator-based tone generation with frequency sweep, filter, panning
   - `noiseBurst(dur, vol, opts)`: Noise burst generator for explosions
   - Stereo panning based on entity position (`panFor()`)
   - Mute toggle with localStorage persistence

5. **Game State** (declared throughout)
   - `gameRunning`: Boolean for game active state
   - `paused`: Boolean for pause state
   - `player`: Player entity (aircraft)
   - `enemies`: Array of active enemies
   - `bosses`: Array of active bosses
   - `bullets`: Array of player bullets
   - `particles`: Array of visual effect particles
   - `explosions`: Array of explosion effects
   - `stage`, `kills`, `lives`, `score`, `mult` (combo multiplier): Current game metrics

6. **Entity System**
   - **Player**: x, y, width, height, vx, vy, health, weapon (type/level), bombs
   - **Enemy**: x, y, type, hp, shootTimer, shootChance
   - **Boss**: x, y, name, hp, maxHp, bulletPattern, shootTimer
   - **Bullet**: x, y, vx, vy, type (player or enemy), damage
   - **Particle**: x, y, vx, vy, ttl (time-to-live), color

7. **Main Game Loop** (animation frame)
   - Update: Enemy/player/bullet/particle physics, collision detection, spawning
   - Render: Clear canvas, draw parallax background, entities, effects, HUD
   - Performance tracking and quality adjustment

8. **Input Handling**
   - **Keyboard**: Arrow keys or WASD for movement, Space/X/Shift for fire/bomb, P/Esc for pause
   - **Mouse**: Move to steer, click to fire
   - **Touch**: Left drag zone for movement, FIRE/BOMB buttons, joystick feedback

9. **UI & Overlays**
   - Start/end screen with difficulty selection, score display, best score
   - In-game HUD: Score, best, multiplier, weapon name/level, hearts, bombs, boss health bar
   - Combo popup, stage banner, vignette (damage), flash (hit/bomb)
   - Mute, motion, pause buttons in top-right

10. **Persistence**
    - LocalStorage keys: `skyfighter.bestScore`, `skyfighter.muted`, `skyfighter.difficulty`, `skyfighter.reducedMotion`
    - Best score persists across sessions
    - User preferences (mute, motion, difficulty) remembered

## Key Files & Locations

- **`index.html`** (only file)
  - Lines 1–359: CSS (structure, animations, responsive utilities)
  - Lines 360–420: HTML structure
  - Lines 422+: JavaScript game engine

## Key Constants & Configuration

### Difficulty Presets
```javascript
const DIFFICULTIES = {
  easy:   { spawnBase: 75, spawnMin: 40, enemySpeed: 1.0, enemyShootChance: 0.45, lives: 4, bombs: 3, bossHpMul: 0.8 },
  normal: { spawnBase: 58, spawnMin: 26, enemySpeed: 1.4, enemyShootChance: 0.7,  lives: 3, bombs: 2, bossHpMul: 1.0 },
  hard:   { spawnBase: 40, spawnMin: 18, enemySpeed: 1.9, enemyShootChance: 0.95, lives: 3, bombs: 2, bossHpMul: 1.3 }
};
```

### Enemy Types
- **Grunt**: Basic enemy, 1 HP, slow
- **Interceptor**: Fast weaving enemy, 1 HP
- **Tank**: Heavy armored, 4 HP, appears from stage 2+

### Bosses
Each of 6 bosses has unique hull colors, core color, and HP multiplier. Bosses appear sequentially (one per stage).

## Gameplay Flow

1. **Initialization**: Load best score, display start screen with difficulty selection
2. **Stage Start**: Spawn enemies at configured rate based on difficulty and stage
3. **Combat**: Player dodges/shoots enemies, collects weapon upgrades, manages bombs
4. **Progression**: After 18 kills per stage, boss spawns; defeating boss advances to next stage
5. **Game Over**: When lives reach 0, show end screen with score comparison
6. **Restart**: Player selects difficulty and restarts from stage 1

## Development Notes

### Adding a New Enemy Type
1. Add entry to `ENEMY_TYPES` with `w`, `h`, `hp`, `speedMul`, `color`, `dark`, `score`, `minStage`, optional `weave`
2. Add rendering code in enemy draw section (search `drawEnemy` or similar)
3. Update spawn selection logic to include new type at appropriate stages

### Adding a New Boss
1. Add to `BOSSES` array with `name`, hull colors (`hullTop`, `hullBottom`, `wing`, `core`), and `hpMul`
2. Implement boss bullet pattern in boss update/shoot logic
3. Adjust `hpMul` for difficulty balance

### Adding a New Weapon Type
1. Add weapon config (name, fireRate, bulletSpeed, damage, visual)
2. Update player draw code to show weapon visually
3. Implement firing pattern in player bullet generation

### Tuning Difficulty
Modify `DIFFICULTIES` constants:
- `spawnBase`/`spawnMin`: Enemy spawn frequency (lower = more enemies)
- `enemySpeed`: Scales all enemy movement
- `enemyShootChance`: Probability enemies fire per frame
- `lives`/`bombs`: Starting resources
- `bossHpMul`: Boss health multiplier

### Performance Optimization
- Adaptive quality system runs continuously: if frame time > 25ms, reduce quality (fewer particles, less detail)
- Quality levels: 0 (full), 1 (medium), 2 (low)
- Framerate tracking: 45-frame rolling average triggers quality change
- Particle pooling/recycling recommended for high-enemy scenarios

## Git Workflow

### Branch Management
- **Development branch**: `claude/inspiring-planck-n8ip9k`
- All changes should be committed and pushed to this branch
- Commit format:
  ```
  Brief description of change
  
  Co-Authored-By: Claude Haiku 4.5 <noreply@anthropic.com>
  Claude-Session: https://claude.ai/code/session_01ShjLHevYWTAKt1ZknJWbNu
  ```

### Commit Guidelines
- One logical change per commit
- Descriptive messages explaining the "why" (e.g., "Reduce boss spawn difficulty to balance stage 3" not just "adjust boss")
- Include any tuning constants changed in commit message if significant

## Testing & Validation

### Manual Testing Checklist
- [ ] Game starts on all 3 difficulty levels
- [ ] Player aircraft renders and responds to keyboard/mouse/touch
- [ ] Enemies spawn at correct rates, move, and shoot
- [ ] Collision detection works (bullets hit enemies, enemies hit player)
- [ ] Weapon upgrades drop and change player weapon
- [ ] Bombs detonate and clear enemies
- [ ] Score multiplier increases on consecutive kills
- [ ] All 6 bosses appear in sequence
- [ ] Best score persists across page reload
- [ ] Mute/motion/pause buttons work
- [ ] UI HUD displays correctly on mobile (respects safe-area-insets)
- [ ] Audio works on first interaction (WebAudio resume policy)

### Performance Validation
- Inspect frame rate on low-end devices (target 30+ FPS on medium difficulty)
- Check memory usage doesn't grow over 60MB
- Verify quality scaling activates under load

## Common Modifications

### Adjust Boss Difficulty
- Edit `bossHpMul` in `BOSSES` array entry
- Or modify `DIFFICULTIES[difficulty].bossHpMul` multiplier

### Add Stage Progression Milestone
- Modify `KILLS_PER_STAGE` constant
- Add stage theme/enemy type gating if needed

### Change Weapon Progression
- Locate weapon upgrade drop logic
- Modify weapon type selection or fire rate progression

### UI Layout Tweaks
- Modify CSS in `<style>` block (lines 10–358)
- Test responsive breakpoints with `clamp()` values
- Verify touch controls don't overlap on mobile

## Audio Implementation

All sounds are generated via Web Audio API:
- `beep()`: Frequency sweep oscillators (weapon fire, power-ups)
- `noiseBurst()`: White noise burst (explosions, impacts)
- Panning: Based on entity screen position for spatial audio
- Master gain: Compressor for dynamic range control

The audio context requires user interaction to start (click/touch), then resumes on demand.

## Accessibility

- **Screen reader support**: `#srLive` div with `aria-live="polite"` for announcements
- **Reduced motion**: Respects `prefers-reduced-motion` system preference; disables animations if toggled
- **Color contrast**: HUD uses bright colors on dark backgrounds
- **Touch controls**: Large 78px buttons, safe-area-inset margins
- **Keyboard nav**: All buttons and controls keyboard accessible

## Known Limitations & Future Enhancements

- Single-file architecture limits code organization (consider bundling if it grows)
- No persistent leaderboard (only best score per device)
- No mobile app distribution (web-only currently)
- Boss patterns are relatively simple (room for more complexity)
- No multiplayer or network features
- Audio works best after initial user interaction (WebAudio context suspend policy)

## Resources

- Canvas 2D API: https://developer.mozilla.org/en-US/docs/Web/API/CanvasRenderingContext2D
- Web Audio API: https://developer.mozilla.org/en-US/docs/Web/API/Web_Audio_API
- Game loop pattern: requestAnimationFrame with delta time
- LocalStorage API: Browser storage for persistence

## Attribution

This codebase was developed and maintained via Claude Code (AI-assisted development).
