# Bug Fixes

This document summarizes the bugs identified and resolved in TinySki, as tracked in [CHANGELOG.md](CHANGELOG.md).

---

## Critical Bugs

### 1. Game Would Not Start — No Movement or Collision Objects

**Severity:** Critical  
**Symptom:** After starting the game, the screen showed no terrain movement and no collision objects appeared.

**Root Causes & Fixes:**

| # | Cause | Fix |
|---|-------|-----|
| 1a | `randomTerrainTile()` could return `'snowE'`, which does not exist in `TILE_PATHS`. This caused ~8% of terrain tiles to be invisible, creating gaps in the ground. | Removed the unreachable `return 'snowE'` fallback and adjusted the probability ranges so all return values map to valid tile keys. |
| 1b | `pickObstacleType()` could return `'tree'`, which does not exist in `TILE_PATHS`. Some collision objects were invisible. | Changed the fallback to return the last obstacle type in `OBSTACLE_TYPES` as a safe default instead of a non-existent key. |
| 1c | Steering (A/D keys) forcibly set `currentSpeed = PLAYER_STEER_SPEED` (1.0 px/frame), overriding the player's forward speed. This made the game feel like it barely moved while steering. | Removed the forced speed override. Forward speed is now independent of steering direction; lateral movement is applied separately via `playerX += steerDir * steerSpeed`. |

---

## Boundary & Collision Bugs

### 2. Player Incorrectly Triggered Game Over at Boundary

**Severity:** High  
**Symptom:** The player triggered game over the instant they reached the navigable edge of the terrain, even without hitting any obstacle.

**Root Cause:** The boundary collision check in `checkPlayerCollision()` used `<=` / `>=` against `TERRAIN_MIN_X` / `TERRAIN_MAX_X`, but `playerX` is clamped to those exact values in `update()`. This meant the condition fired immediately upon reaching the edge.

**Fix:** Removed the boundary check entirely — the boundary markers (`tile_0032.png` / `tile_0033.png`) are already included in the `obstacles` array, so the existing obstacle collision detection correctly triggers game over only when the player's circle actually overlaps a boundary marker tile.

---

### 3. Boundary Objects Did Not Trigger Game Over

**Severity:** High  
**Symptom:** The player could pass through boundary markers without triggering game over.

**Root Cause:** The boundary collision check used strict `<` and `>` operators (`playerX < TERRAIN_MIN_X || playerX > TERRAIN_MAX_X`), but `playerX` is clamped to `TERRAIN_MIN_X` / `TERRAIN_MAX_X` in `update()` before the check fires. Since the clamping prevents `playerX` from ever being strictly less than/greater than those bounds, the collision check was dead code.

**Fix:** Changed the boundary check to use `<=` and `>=` so game over triggers when the player reaches the boundary edge where the boundary markers are drawn. (This was later superseded by Fix #2 — removing the boundary check entirely in favor of obstacle-based collision.)

---

### 4. Boundary Markers Appeared on Wrong Tile Row

**Severity:** Medium  
**Symptom:** When the player reached a new 20m interval, boundary markers appeared at the player's screen position (center) instead of at the bottom row.

**Root Cause:** Boundary markers were spawned with a fixed `worldY = interval * 20 * PPM`, and their screen Y was calculated once at spawn time but never updated. As the terrain scrolled, the markers did not move with it.

**Fix:**
1. Stored `screenY` in each marker object.
2. Updated `screenY` each frame in the update loop so markers scroll up with the terrain.
3. Spawned markers at `CANVAS_H` (bottom of screen) with `worldY = distance * PPM + CANVAS_H`.
4. Removed markers when `screenY < -TILE * 4` (off-screen above).

---

### 5. Collision Objects Did Not Extend to Navigable Boundaries

**Severity:** Medium  
**Symptom:** A 5–6 tile gap on each side of the screen had no obstacles, creating an unnavigable safe zone near the edges.

**Root Cause:** The `spawnObstacle()` function placed obstacles within screen bounds (`TILE` to `CANVAS_W - TILE`), but the player can navigate 6 tiles beyond each screen edge (`TERRAIN_MIN_X = -96`, `TERRAIN_MAX_X = 576`).

**Fix:** Expanded the spawn range to cover the full navigable terrain bounds (`TERRAIN_MIN_X` to `TERRAIN_MAX_X`), accounting for the obstacle draw size. Pre-placed obstacles in `initGame()` were updated to use the same range.

---

### 6. Boundary Marker Tile Paths Missing Leading Zeros

**Severity:** Medium  
**Symptom:** Boundary marker images never loaded — markers appeared as blank spaces at terrain edges.

**Root Cause:** The tile paths in `TILE_PATHS` used incorrect filenames (`tile_032.png` / `tile_033.png`) instead of the actual asset filenames (`tile_0032.png` / `tile_0033.png`).

**Fix:** Updated the paths in `TILE_PATHS` to match the actual asset filenames with leading zeros.

---

## Summary

| # | Bug | Severity | Status |
|---|-----|----------|--------|
| 1 | Game would not start (no movement/obstacles) | Critical | ✅ Fixed |
| 2 | Player incorrectly triggered game over at boundary | High | ✅ Fixed |
| 3 | Boundary objects did not trigger game over | High | ✅ Fixed |
| 4 | Boundary markers appeared on wrong tile row | Medium | ✅ Fixed |
| 5 | Collision objects did not extend to navigable boundaries | Medium | ✅ Fixed |
| 6 | Boundary marker tile paths missing leading zeros | Medium | ✅ Fixed |
