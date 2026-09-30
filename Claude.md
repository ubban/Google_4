# Sky Fighter - Claude Development Guide

## Project Overview

**Sky Fighter** is a Raiden-style vertical shoot-'em-up game built as a single-file HTML application. It features:
- Canvas-based 2D rendering
- Procedural sound generation (no audio files)
- Multiple enemy types and boss encounters
- Difficulty levels and weapon progression
- Responsive touch and keyboard controls
- Accessibility features (reduced motion, screen reader support)

**Tech Stack**: Vanilla HTML5 + CSS + JavaScript (no external dependencies)

**Entry Point**: `/index.html` - The entire game is embedded in this single file.

---

## Code Architecture

The game code is organized within a single `<script>` tag (lines 422-1666) in an IIFE (Immediately Invoked Function Expression).

### Key Sections (in order of appearance)

1. **DOM and State** (lines 423-527)
   - Canvas and UI element references
   - Game state initialization
   - Local storage keys for persistence

2. **Performance Tracking** (lines 473-483)
   - Adaptive quality system (`quality` variable)
   - Frame time monitoring for dynamic difficulty adjustment

3. **Audio System** (lines 542-676)
   - WebAudio API setup with synth sounds
   - `beep()`, `noiseBurst()` functions for sound effects
   - `sfx` object containing all game sounds
   - No external audio files - all procedural

4. **Input Handling** (lines 767-861)
   - Keyboard and mouse input
   - Touch controls and virtual joystick for mobile
   - Input state tracked in `keys` object and `joyVec`

5. **Game State & Definitions** (lines 863-527)
   - Difficulty settings (`DIFFICULTIES`)
   - Enemy types (`ENEMY_TYPES`)
   - Boss designs (`BOSSES`)
   - Screen themes (`THEMES`)
   - Game state variables (player, bullets, enemies, etc.)

6. **Entity Management** (lines 875-1085)
   - Spawning: `spawnEnemy()`, `spawnBoss()`, `spawnExplosion()`
   - Drawing: `drawPlane()`, `drawBossShip()`, `drawClouds()`
   - Collision detection: `rectsOverlap()`

7. **Gameplay Systems** (lines 1087-1322)
   - Score and chain multipliers (`addScore()`, `breakChain()`)
   - Weapon system (`fireWeapon()` with Vulcan/Laser variants)
   - Power-ups (`applyPowerUp()`)
   - Bombs (`useBomb()`)

8. **Boss AI** (lines 1324-1434)
   - Boss attack patterns (`BOSS_PATTERNS` array)
   - Pattern-specific firing functions (radial, aimed, spiral, etc.)
   - `updateBoss()` for boss movement and attack timing

9. **Game Loop** (lines 1640-1665)
   - Main loop: `loop()` with delta-time calculation
   - Update phase: `update(dt)`
   - Render phase: `draw()`

---

## Key Game Mechanics

### Difficulty Settings
Located in `DIFFICULTIES` object (line 485):
- **Easy**: Slower spawn, weaker enemies, more lives, 0.8x boss HP
- **Normal**: Balanced difficulty, 3 lives, standard boss HP
- **Hard**: Fast spawn, tough enemies, fewer lives, 1.3x boss HP

### Enemy Types
From `ENEMY_TYPES` (line 502):
- **Grunt**: Fast, low HP, common
- **Interceptor**: Very fast, dodging pattern (weaving)
- **Tank**: Slow, high HP (4), appears at stage 2+

### Weapon System
Two weapons with 3 levels each:
- **Vulcan**: Spread shot (levels 1-3), fires bullets
- **Laser**: Piercing beam weapon, higher rate of fire

Power-ups unlock higher levels and can swap weapons.

### Boss Encounters
- Triggered after 18 enemy kills per stage
- 6 unique boss designs that cycle
- Each boss has 3 attack patterns with unique firing behavior
- Patterns include: radial bursts, aimed spreads, homing projectiles, teleportation

### Score Multiplier System
- **Chain**: Breaks after 2.2 seconds without a kill
- **Multiplier**: Increases from 1x to 10x (1 point per 5 chain kills)
- Encourages sustained combat without gaps

---

## Common Modifications

### Adjusting Difficulty
Edit the `DIFFICULTIES` object (line 485):
```javascript
const DIFFICULTIES = {
  easy:   { spawnBase: 75, spawnMin: 40, enemySpeed: 1.0, ... },
  normal: { spawnBase: 58, spawnMin: 26, enemySpeed: 1.4, ... },
  hard:   { spawnBase: 40, spawnMin: 18, enemySpeed: 1.9, ... }
};
```
- `spawnBase`/`spawnMin`: Control enemy spawn rate (lower = faster)
- `enemySpeed`: Enemy movement speed multiplier
- `enemyShootChance`: Enemy shooting probability (0-1)
- `lives`, `bombs`, `bossHpMul`: Self-explanatory

### Adding Enemy Types
1. Add to `ENEMY_TYPES` object (line 502)
2. Set `minStage` to control when it appears
3. Update `spawnEnemy()` (line 1022) to include the new type in random selection

### Creating Boss Patterns
1. Add pattern functions to the boss patterns section (line 1324)
   - Use helpers: `fireRadial()`, `fireAimedSpread()`, `fireCross()`, etc.
   - All fire enemy bullets into the `enemyBullets` array
2. Add to `BOSS_PATTERNS` array (line 1372) as a new attack sequence

### Tweaking Visual Theme
Edit `THEMES` array (line 493):
- `sky`: Array of 3 gradient colors [top, mid, bottom]
- `cloud`: RGB values for cloud color
- `sun`: RGB values for sun glow
- `name`: Stage name displayed in banner

### Adjusting SFX
Edit the `sfx` object (line 624):
- All sounds use `beep()` (for tones) and `noiseBurst()` (for noise)
- Sound design parameters: frequency, duration, volume, sweep, filters, pan

---

## Testing the Game

### Manual Testing
1. Open `/index.html` in a browser
2. Test core gameplay:
   - **Keyboard**: Arrow keys/WASD to move, Space to fire, X/Shift for bombs, P/Esc to pause
   - **Mouse**: Move to steer (when over canvas), click to fire
   - **Touch**: Drag left half to move, tap right buttons
3. Test difficulty levels: Click Easy/Normal/Hard before starting
4. Test feature toggles:
   - Mute button (🔊) for audio
   - Motion button (🎆) for reduced motion / animations
   - Pause button (⏸)
5. Reach a boss stage to verify boss behaviors

### Edge Cases to Check
- **Stage progression**: Verify stage counter and theme changes after boss defeat
- **Chain multiplier**: Confirm multiplier resets if there's a gap in kills
- **Power-ups**: Collect weapon/bomb/life power-ups and verify effects
- **Invulnerability**: Get hit and confirm temporary invulnerability period
- **Touch controls**: Test on mobile or with touch emulation
- **Reduced motion**: Verify animations disable correctly
- **High scores**: Close and reopen to verify persistence

---

## Development Workflow

### When Making Changes
1. **Understand the scope**: Is this a mechanic change, balance tweak, visual effect, or sound?
2. **Identify the section**: Most changes fall into one of the key sections above
3. **Make minimal edits**: Avoid refactoring unrelated code
4. **Test in browser**: Open `/index.html` and play through the change
5. **Check edge cases**: Consider interactions with other systems

### Common Workflows

**Balancing Difficulty**:
- Adjust `DIFFICULTIES` object values
- Test at each difficulty level
- Verify boss HP scaling with `bossHpMul`

**Adding Content**:
- New enemy? Add to `ENEMY_TYPES`, update `spawnEnemy()`
- New boss? Add to `BOSSES`, add pattern array to `BOSS_PATTERNS`
- New theme? Add to `THEMES` array

**Fixing Bugs**:
- Collision issues: Check `rectsOverlap()` and entity dimensions
- Game logic: Debug in the `update()` function
- Rendering: Check `draw()` and entity drawing functions
- Audio: Verify `ensureAudio()` is called before sound triggers

---

## Important Notes for Claude Agents

### Code Style
- No external dependencies (keep it that way)
- Vanilla JavaScript, ES6+ syntax (arrow functions, const/let, template literals)
- Comments are minimal; code is self-documenting
- Magic numbers are defined as constants at the top (e.g., BASE_W, BASE_H)

### State Management
- **Game state**: Stored in module-level variables (player, bullets, enemies, etc.)
- **Persistence**: Uses localStorage for best score, difficulty, mute state, reduced motion
- **Frame-based timing**: Uses `frame` counter for animations; update uses delta-time

### Performance Considerations
- **Quality scaling**: Adaptive system reduces particle count and shadow effects on low FPS
- **Collision detection**: Simple rect-based, O(n²) for bullets vs enemies
- **Canvas sizing**: Responsive design scales to window size with max zoom

### Testing Accessibility
- **Keyboard**: Full keyboard support (no mouse required)
- **Touch**: Full touch support on mobile (virtual joystick + buttons)
- **Reduced motion**: Disables animations if system prefers reduced motion
- **Screen reader**: Uses `<div id="srLive">` for announcements

### File Size Considerations
- Single-file design means all changes affect the total size
- Avoid external assets or dependencies
- Procedural audio keeps file size small

---

## Quick Reference: Key Variables

| Variable | Type | Purpose |
|----------|------|---------|
| `player` | Object | Player ship state (x, y, hp, weapon, etc.) |
| `bullets` | Array | Player projectiles |
| `enemies` | Array | Enemy ships |
| `enemyBullets` | Array | Enemy projectiles |
| `boss` | Object\|null | Current boss (null if no boss) |
| `score` | Number | Current game score |
| `lives` | Number | Remaining player lives |
| `chain` | Number | Current kill chain counter |
| `multiplier` | Number | Current score multiplier (1-10) |
| `frame` | Number | Game frame counter (increments each update) |
| `running` | Boolean | Is game active? |
| `paused` | Boolean | Is game paused? |
| `difficulty` | String | Current difficulty: 'easy', 'normal', or 'hard' |
| `quality` | Number | Adaptive quality setting (0=full, 1=medium, 2=low) |

---

## Debugging Tips

**If the game won't start**:
- Check browser console for errors
- Verify `ensureAudio()` is called on user interaction
- Check that DOM elements exist (use `getElementById` in console)

**If collisions seem off**:
- Log `rectsOverlap()` results
- Verify entity dimensions (w, h) are set correctly
- Check coordinate system (y increases downward)

**If performance is poor**:
- Check `quality` variable in console
- Reduce `frameTimes.length` threshold (line 479) to trigger scaling sooner
- Profile in DevTools Performance tab

**If sound won't play**:
- Browser may require user gesture before audio context resumes
- Check `muted` variable
- Verify `ensureAudio()` was called successfully

---

## Resources & References

- **Canvas API**: https://developer.mozilla.org/en-US/docs/Web/API/Canvas_API
- **WebAudio API**: https://developer.mozilla.org/en-US/docs/Web/API/Web_Audio_API
- **Touch Events**: https://developer.mozilla.org/en-US/docs/Web/API/Touch_events
- **localStorage**: https://developer.mozilla.org/en-US/docs/Web/API/Window/localStorage

---

## Summary

Sky Fighter is a compact, self-contained game that demonstrates:
- Responsive canvas-based rendering
- Complex game logic in a single file
- Procedural audio generation
- Mobile-friendly input handling
- Accessibility best practices

When working on this code, maintain the simplicity and cohesion of the single-file design while making targeted, well-tested improvements.
