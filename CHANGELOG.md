# Changelog

All notable changes to TinySki will be documented in this file.

## [Unreleased]

### Fixed
- **Fixed: player incorrectly triggers game over at the boundary instead of colliding with boundary markers.** The boundary collision check in `checkPlayerCollision()` used `<=` / `>=` against `TERRAIN_MIN_X` / `TERRAIN_MAX_X`, which fired game over the instant the player reached the navigable edge (since `playerX` is clamped to those exact values in `update()`). This prevented the player from ever colliding with the boundary marker tiles (`tile_0032.png` / `tile_0033.png`). Removed the boundary check entirely — the boundary markers are already in the `obstacles` array, so the existing obstacle collision detection now correctly triggers game over only when the player's circle actually overlaps a boundary marker tile.

### Added
- Boundary markers at the furthest player navigable edges: `tile_032.png` (right edge) and `tile_033.png` (left edge), repeating every 20m starting at 20m

### Changed
- Adjusted game canvas dimensions from 800×560 to 480×720 (portrait orientation)
- Re-centered player to the new canvas center (240, 360)
- Increased player navigable terrain width by 50% (expanded lateral movement bounds)
- Reduced snow particle spawn density behind the player

### Fixed
- **Fixed: boundary objects (`tile_0032.png` / `tile_0033.png`) did not trigger game over on collision.** The boundary collision check in `checkPlayerCollision()` used strict `<` and `>` operators (`playerX < TERRAIN_MIN_X || playerX > TERRAIN_MAX_X`), but `playerX` is clamped to `TERRAIN_MIN_X` / `TERRAIN_MAX_X` in `update()` before the check fires. Since the clamping prevents `playerX` from ever being strictly less than/greater than those bounds, the collision check was dead code. Fixed by changing the boundary check to use `<=` and `>=` so game over triggers when the player reaches the boundary edge where the boundary markers are drawn.
- **Fixed: boundary markers (`tile_0032.png` / `tile_0033.png`) appeared on the same tile row after distance trigger.** Boundary markers were spawned with a fixed `worldY = interval * 20 * PPM` and their screen Y was calculated once at spawn time but never updated. When the player reached a new 20m interval, the marker appeared at the player's screen position (center) instead of the bottom row. Fixed by: (1) storing `screenY` in each marker object, (2) updating `screenY` each frame in the update loop so markers scroll up with the terrain, (3) spawning markers at `CANVAS_H` (bottom of screen) with `worldY = distance * PPM + CANVAS_H`, and (4) removing markers when `screenY < -TILE * 4` (off-screen above).
- **Fixed: collision objects did not extend to the left/right navigable boundaries.** The `spawnObstacle()` function placed obstacles within screen bounds (`TILE` to `CANVAS_W - TILE`), but the player can navigate 6 tiles beyond each screen edge (`TERRAIN_MIN_X = -96`, `TERRAIN_MAX_X = 576`). This left a 5-6 tile gap on each side with no obstacles. Fixed by expanding the spawn range to cover the full navigable terrain bounds (`TERRAIN_MIN_X` to `TERRAIN_MAX_X`), accounting for the obstacle draw size. Pre-placed obstacles in `initGame()` were updated to use the same range.
- **Fixed: player incorrectly triggers game over when hitting the boundary.** The boundary collision check in `checkPlayerCollision()` used `<=` and `>=` operators, which fired game over the moment the player reached the boundary edge while the clamping in `update()` allowed `playerX` to equal those bounds. This caused an immediate death loop. The fix (using strict `<` and `>`) was later corrected again — see the boundary objects fix above — to use `<=` and `>=` so game over triggers when the player reaches the boundary edge where the boundary markers are drawn.
- Boundary markers (`tile_0032.png` / `tile_0033.png`) now render correctly at screen edges every 20m. The tile paths in `TILE_PATHS` were missing leading zeros (`tile_032.png` → `tile_0032.png`), so the images never loaded. Fixed the paths to match the actual asset filenames.

- **Fixed: game had no movement and no collision objects after starting.** Three root causes were identified and fixed:
  1. `randomTerrainTile()` could return `'snowE'` which doesn't exist in `TILE_PATHS`, causing ~8% of terrain tiles to be invisible (gaps in the ground). Fixed by removing the unreachable `return 'snowE'` fallback and adjusting the probability ranges.
  2. `pickObstacleType()` could return `'tree'` which doesn't exist in `TILE_PATHS`, causing some collision objects to be invisible. Fixed by returning the last obstacle type as a safe fallback instead of a non-existent key.
  3. Steering (A/D keys) was forcibly setting `currentSpeed = PLAYER_STEER_SPEED` (1.0 px/frame), which is the lateral movement speed constant, not a forward speed. This made the game feel like it barely moved while steering. Fixed by removing the forced speed override — forward speed is now independent of steering direction, and lateral movement is applied separately via `playerX += steerDir * steerSpeed`.
