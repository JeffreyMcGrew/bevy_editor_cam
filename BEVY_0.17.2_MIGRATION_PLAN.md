# Bevy 0.17.2 Migration Plan for bevy_editor_cam

## Overview
This document outlines a plan to update `bevy_editor_cam` from Bevy 0.16.1 to Bevy 0.17.2 **without changing any core functionality code**. The goal is to update dependencies and ensure compatibility while maintaining the existing implementation.

## Current State
- **Current Bevy Version**: 0.16.1
- **Target Bevy Version**: 0.17.2
- **Crate Version**: 0.6.0

## Dependency Analysis

### Main Dependencies (Cargo.toml lines 18-37)
All of these need to be updated from `0.16.1` to `0.17.2`:

#### Core Dependencies:
- `bevy_app`
- `bevy_color`
- `bevy_derive`
- `bevy_ecs`
- `bevy_image`
- `bevy_input`
- `bevy_log`
- `bevy_math`
- `bevy_picking`
- `bevy_reflect`
- `bevy_render`
- `bevy_time`
- `bevy_transform`
- `bevy_utils`
- `bevy_window`
- `bevy_platform`

#### Optional Dependencies:
- `bevy_asset` (optional, used by `extension_independent_skybox`)
- `bevy_core_pipeline` (optional, used by `extension_independent_skybox`)
- `bevy_gizmos` (optional, used by `extension_anchor_indicator`)

### Dev Dependencies (Cargo.toml lines 39-61)
These need to be checked for Bevy 0.17 compatibility:

1. **bevy** (main crate): `0.16.0` → `0.17.2`
2. **bevy_framepace**: Currently `0.19.1` 
   - **Action Required**: Check crates.io for Bevy 0.17 compatible version
   - Likely needs update to 0.20.x or 0.21.x
3. **big_space**: Currently `0.10.0`
   - **Action Required**: Check crates.io for Bevy 0.17 compatible version
   - May need update to 0.11.x or later
4. **indoc**: `2.0.5` (no change needed - not Bevy-dependent)
5. **rand**: `0.9` (no change needed - not Bevy-dependent)

## Critical API Areas to Test

Based on code review, these are the key Bevy APIs used by the crate:

### 1. Camera Components
**Files affected**: 
- `src/extensions/independent_skybox.rs` (line 144)
- Examples (minimal.rs, cad.rs, etc.)

**Current usage**:
```rust
Camera3d::default()
```

**Potential Impact**: LOW
- Camera3d is a well-established component, unlikely to have breaking changes
- May have new fields added with defaults

### 2. Projection Types
**Files affected**:
- `src/extensions/independent_skybox.rs` (lines 151, 213)
- `src/extensions/dolly_zoom.rs` (lines 65, 101, 226)
- `src/controller/component.rs` (line 368)

**Current usage**:
```rust
Projection::Perspective(PerspectiveProjection { ... })
PerspectiveProjection::default()
```

**Potential Impact**: LOW-MEDIUM
- Core projection API is stable
- Field names and defaults might have minor changes

### 3. Picking System
**Files affected**:
- `src/input.rs` (all pointer/picking related code)
- `src/controller/component.rs` (anchor detection)

**Current usage**:
- `bevy_picking::pointer::*`
- `bevy_picking::PickSet::Last`

**Potential Impact**: MEDIUM
- Picking system had improvements in 0.17
- May have new event types or API refinements
- The "Improved Observer/Events" feature in 0.17 may affect this

### 4. Transform & GlobalTransform
**Files affected**: Throughout codebase

**Potential Impact**: LOW
- Core transform system is very stable

### 5. Gizmos System
**Files affected**:
- `src/extensions/anchor_indicator.rs`

**Current usage**:
```rust
gizmos.circle(Isometry3d::new(...), ...)
gizmos.ray(...)
```

**Potential Impact**: LOW
- Gizmos API is relatively stable

### 6. Skybox & Environment
**Files affected**:
- `src/extensions/independent_skybox.rs`

**Current usage**:
```rust
Skybox { image, brightness, rotation }
```

**Potential Impact**: LOW
- Skybox API is stable

### 7. Time & Platform
**Files affected**:
- `src/controller/component.rs`
- `src/controller/motion.rs`

**Current usage**:
```rust
bevy_platform::time::Instant
bevy_time::prelude::*
Duration
```

**Potential Impact**: LOW
- Time APIs are very stable

## Migration Steps

### Phase 1: Dependency Updates (Cargo.toml only)

1. **Update all Bevy core dependencies** (lines 18-37):
   ```toml
   bevy_app = "0.17.2"
   bevy_color = "0.17.2"
   bevy_derive = "0.17.2"
   bevy_ecs = "0.17.2"
   bevy_image = "0.17.2"
   bevy_input = "0.17.2"
   bevy_log = "0.17.2"
   bevy_math = "0.17.2"
   bevy_picking = "0.17.2"
   bevy_reflect = "0.17.2"
   bevy_render = "0.17.2"
   bevy_time = "0.17.2"
   bevy_transform = "0.17.2"
   bevy_utils = "0.17.2"
   bevy_window = "0.17.2"
   bevy_platform = "0.17.2"
   # Optional
   bevy_asset = { version = "0.17.2", optional = true }
   bevy_core_pipeline = { version = "0.17.2", optional = true }
   bevy_gizmos = { version = "0.17.2", optional = true }
   ```

2. **Update main Bevy dev-dependency** (line 46):
   ```toml
   version = "0.17.2"
   ```

3. **Research and update bevy_framepace**:
   - Visit https://crates.io/crates/bevy_framepace
   - Find the version compatible with Bevy 0.17
   - Update accordingly (likely 0.20.x or 0.21.x)

4. **Research and update big_space**:
   - Visit https://crates.io/crates/big_space
   - Find the version compatible with Bevy 0.17
   - Update accordingly (likely 0.11.x or higher)

### Phase 2: Compilation & Testing

5. **Clean build**:
   ```bash
   cargo clean
   cargo check
   ```

6. **Address any compilation errors**:
   - Most likely errors will be in:
     - Input handling (if picking API changed)
     - Projection handling (if new required fields added)
   - Check for deprecation warnings
   - Consult Bevy 0.17 migration guide for any breaking changes

7. **Build all features**:
   ```bash
   cargo build --all-features
   cargo build --no-default-features
   cargo build --features extension_anchor_indicator
   cargo build --features extension_independent_skybox
   ```

8. **Build all examples**:
   ```bash
   cargo build --examples
   ```

### Phase 3: Runtime Testing

9. **Test each example**:
   - `cargo run --example minimal`
   - `cargo run --example cad`
   - `cargo run --example floating_origin`
   - `cargo run --example map`
   - `cargo run --example ortho`
   - `cargo run --example split_screen`
   - `cargo run --example zoom_limits`

10. **Test matrix** - For each example, verify:
    - ✓ Camera orbits correctly (right-click + drag)
    - ✓ Camera pans correctly (Shift+right-click + drag, based on your recent change)
    - ✓ Mouse wheel zoom works
    - ✓ Anchor indicator displays (if feature enabled)
    - ✓ Skybox renders correctly (if feature enabled)
    - ✓ No visual glitches or unexpected behavior
    - ✓ Performance is acceptable

### Phase 4: Documentation & Finalization

11. **Update version references**:
    - Update any documentation that mentions Bevy version compatibility
    - Update README.md if it specifies Bevy version requirements
    - Consider updating crate version (0.6.0 → 0.7.0) to reflect the Bevy version bump

12. **Run linter and formatter**:
    ```bash
    cargo clippy --all-features
    cargo fmt --check
    ```

13. **Final verification**:
    - Run tests if any exist
    - Check that no warnings were introduced
    - Verify examples still work correctly

## Risk Assessment

### Low Risk Areas (unlikely to need code changes):
- Transform/GlobalTransform system
- Time and Duration APIs
- Math operations (DVec3, DMat4, etc.)
- Reflection system
- Basic ECS (Component, Query, System)

### Medium Risk Areas (may need minor adjustments):
- Picking system (improvements in 0.17)
- Input handling (keyboard, mouse)
- Event system (Observer/Event API improvements)

### Known Bevy 0.17 Features (won't affect this crate):
- Raytraced lighting (bevy_solari) - new feature
- DLSS support - new feature
- HTTP asset loading - new feature
- Automatic reflection registration - may simplify some code in the future
- UI gradients - not used by this crate
- Rust hotpatching - dev feature only

## Rollback Plan

If critical issues are encountered:
1. Keep a git branch at the current 0.16.1 version
2. Document any blockers found
3. Consider a phased approach:
   - Release 0.6.x series for Bevy 0.16
   - Release 0.7.x series for Bevy 0.17 (with compatibility notes)

## Success Criteria

The migration is successful when:
- ✓ All code compiles without errors on Bevy 0.17.2
- ✓ No new warnings introduced
- ✓ All examples run correctly
- ✓ All features (default and optional) work
- ✓ Camera behavior is unchanged from 0.16.1 version
- ✓ Performance is similar or better
- ✓ No visual regressions

## Timeline Estimate

- Phase 1 (Dependency Updates): 15-30 minutes
- Phase 2 (Compilation): 30-60 minutes
- Phase 3 (Testing): 1-2 hours
- Phase 4 (Documentation): 30 minutes

**Total Estimated Time**: 2.5-4 hours

## Notes

1. **No Code Changes Expected**: Based on Bevy's commitment to stability and the analysis of this crate's API usage, it's likely that only dependency version numbers need updating in Cargo.toml. The code itself should compile and work without modifications.

2. **Potential Minor Adjustments**: If any code changes are needed, they will likely be:
   - Adding `.unwrap_or_default()` if new optional fields were added to components
   - Updating deprecated method calls if any exist
   - These would be mechanical changes, not logic changes

3. **Breaking Changes**: Check the official Bevy 0.17 migration guide at: https://bevyengine.org/learn/migration-guides/0-16-to-0-17/

4. **Community Support**: If issues arise, the Bevy Discord and GitHub discussions are excellent resources.

## Additional Recommendations

### Post-Migration Opportunities (Optional):
1. Consider using new automatic reflection registration (removes some boilerplate)
2. Explore new Observer/Event APIs for more efficient event handling
3. Consider DLSS support for examples (if targeting RTX hardware)

### Maintenance Strategy:
1. Set up CI to test against Bevy main branch (early warning system)
2. Subscribe to Bevy release notifications
3. Consider supporting multiple Bevy versions if user base requests it

