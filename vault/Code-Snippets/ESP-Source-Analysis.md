# GodOfLife ESP Source Code Analysis

> Analyzed from: `dune-external.zip` (UC File ID 50549)
> Author: GodOfLife (UC, Feb 2022, rep 284)
> Language: Rust (edition 2024)
> Target: Linux (Proton/Wine) external memory reader
> SHA256: `39f98fbaf2ff592b8c3a524eca136006e4b4b0d88b50e3ea1f0e7b438eeb1b96`
> Analysis date: 2026-09-20

## Architecture Overview

External process memory reader that attaches to Dune Awakening running under Proton/Wine on Linux. Reads game memory via `/proc/` filesystem using the `memflex` crate, renders an overlay via `egui_overlay` with a GLFW passthrough window.

### File Structure

```
src/
  main.rs              Entry point (root + hidepid checks)
  app.rs               App init, process finding, config struct
  ui/
    mod.rs             Module declarations
    main.rs            Menu UI + weapon damage modifier write
    overlay.rs         EguiOverlay impl, main render loop (CRASH POINT)
    esps.rs            ESP drawing (player, NPC, spice field)
  unreal/
    mod.rs             GObjects chunked array loader
    global.rs          Global state (OnceCell singletons)
    offsets.rs         Hardcoded game offsets (OUTDATED)
    screen.rs          World-to-screen projection
    types/
      mod.rs           Module declarations
      enums.rs         UE5 flag enums (EObjectFlags, EClassFlags, etc.)
      structs.rs       UE5 struct defs (FName, TArray, UObject, FVector, FProperty, TMap, FString)
```

### Dependencies

| Crate | Version | Purpose |
|-------|---------|---------|
| memflex | 0.5 (external feature) | Cross-process memory R/W via /proc |
| egui | 0.29 | Immediate-mode GUI framework |
| egui_overlay | 0.9.0 | Transparent overlay window |
| egui_render_three_d | 0.9.0 | OpenGL backend for egui |
| egui_window_glfw_passthrough | 0.9 | Click-through GLFW window |
| sysinfo | 0.36.1 | Process enumeration |
| once_cell | 1.21.3 | Lazy/OnceCell global singletons |
| libc | 0.2.174 | Low-level Linux syscalls |
| egui-keybind | 0.4.1 | Hotkey binding (F1 toggle) |
| fastcontains | 1.0.1 | Fast string contains check |

## Startup Sequence

```
main()
  1. Check running as root (sudo)
  2. Verify /proc mounted with hidepid=invisible
  3. App::init()
     a. Find process "DuneSandbox-Win64-Shipping.exe" (process name: "GameThread")
     b. Set base_address = 0x140000000 (Windows PE base under Proton)
     c. Read UWorld from base + UWORLD offset
     d. Read GNames from base + GNAMES offset
     e. Load GObjects (chunked array, 64K elements/chunk)
     f. Set defaults: ESP on, weapon_damage = 250.0, F1 = menu toggle
  4. egui_overlay::start(app)  -- enters render loop
```

## Memory Read Chain

```
Base Address: 0x140000000
  + UWORLD (0xbcb2cb0) --> UWorld*
    |
    +-- 0x2c8 --> GameState                    [CRASH POINT: offset outdated]
    |     +-- 0x368 --> PlayerArray (TArray<PlayerState*>)
    |           +-- each PlayerState:
    |                 0x620 --> LifeState (u8: 0=alive)
    |                 0x3d8 --> Pawn (usize)
    |                 0x460 --> PlayerName (FString)
    |
    +-- 0x38  --> PersistentLevel
    |     +-- 0xA0 --> Actors (TArray<AActor*>)
    |           +-- each Actor:
    |                 UObject.name --> class name filter
    |                 0x240 --> RootComponent
    |                     0x1b8 --> WorldLocation (FVector: f64 x,y,z)
    |                 0x690 --> SpiceField type (FName)
    |                 0x750 --> SpiceField status (u32)
    |
    +-- OWNING_GAME_INSTANCE (0x330) --> GameInstance
          +-- LOCALPLAYERS (0x40) --> LocalPlayers TArray
                [0] --> LocalPlayer
                  +-- PLAYER_CONTROLLER_OFFSET (0x38) --> PlayerController
                        +-- PLAYER_CAMERA_MANAGER (0x408) --> CameraManager
                        |     +-- 0x13a0 + 0x10 --> FMinimalViewInfo
                        |           .location (FVector)
                        |           .rotation (FVector -- Euler pitch/yaw/roll)
                        |           .fov (f32)
                        +-- PLAYER_CHARACTER (0x3a0) --> Character
                              +-- 0xfd8 --> WeaponActor
                                    +-- 0x4e0 + 0x160 --> damage (f32, WRITE target)
```

## ESP Features

### Player ESP (draw_player_esp)

- Source: `src/ui/esps.rs:137`
- Reads all players from GameState PlayerArray
- Per player: life state, pawn pointer, root component location, name (FString)
- Head position = pawn.z + 85.0 (constant offset)
- Draws corner-style bounding box (box_mode 2) with proportional sizing
- Labels: `"{player_name} : Alive/Dead"`
- No distance limit, no visibility check

### NPC ESP (draw_npc_esp, draw_esp_test)

- Source: `src/ui/esps.rs:241`
- Filters PersistentLevel actors where `name.starts_with("BP_Npc_SoldierBase")`
- Draws bounding box only (no name, no distance label)
- Same proportional sizing as player ESP

### Spice Field ESP (draw_spice_field)

- Source: `src/ui/esps.rs:194`
- Filters actors where `name.starts_with("BP_SpiceField")`
- Reads spice type (FName at +0x690) and status (u32 at +0x750)
- Status: 0=Soon, 1/2=Active, 3=hidden (skipped)
- Displays: `"{type} Spice Field | {status} | {distance}m"`

### Weapon Damage Modifier (change_weapon_damage)

- Source: `src/ui/main.rs:46`
- **WRITE operation** -- writes f32 to `weapon_actor + 0x4e0 + 0x160`
- Default: 250.0, slider range: 0.0 - 100,000.0
- Labeled "To NPCs Only" but actually modifies the local character's weapon damage value globally
- Executes every frame when enabled

## World-to-Screen Projection

- Source: `src/unreal/screen.rs:49`
- Standard UE rotation matrix from Euler angles (pitch=X, yaw=Y, roll=Z in degrees)
- Camera-space transform: delta.dot(axisY), delta.dot(axisZ), delta.dot(axisX)
- Behind-camera check: returns (0,0) if z < 1.0
- Perspective division with FOV tangent
- **Hardcoded 1920x1080** resolution

## FName Resolution System

- Source: `src/unreal/types/structs.rs:15`
- UE5 chunked name pool format:
  - `chunk_offset = comparison_index >> 16`
  - `name_offset = comparison_index & 0xFFFF`
  - Length in upper 10 bits of u16 header (`>> 6`)
- Chunk pointer: `gnames + 8 * (chunk_offset + 2)`
- Name data at: `chunk_ptr + 2 * name_offset + 2` (skip 2-byte header)
- **Cache:** `Box::leak()` creates `&'static str` references (intentional memory leak for perf)
- Thread-safe via `Lazy<Mutex<Vec<Option<&'static str>>>>`

## GObjects Loader

- Source: `src/unreal/mod.rs:11`
- UE 4.20+ chunked format: `FChunkedFixedUObjectArray`
- 64K (65536) elements per chunk
- Each `FUObjectItem` = 24 bytes: `object: usize, flags: i32, cluster_root_index: i32, serial_number: i32`
- Reads base at hardcoded `0x140000000 + GOBJECTS`
- Stores all items in `Mutex<Vec<FUObjectItem>>`

## UE5 Type System Implementation

### Fully Implemented Types
- `FName` -- with chunked name pool resolution and caching
- `TArray<T>` -- with get, read_all, for_each, validation
- `UObject` -- with get_name() and get_fullname() (walks outer chain)
- `FVector` -- with math ops (distance, dot, cross, normalize, to_matrix)
- `FMinimalViewInfo` -- location + rotation + FOV
- `FString` -- UTF-16 to UTF-8 conversion (also leaks via Box::leak)
- `FUObjectItem` -- GObjects array element
- `FChunkedFixedUObjectArray` -- GObjects container
- `TMap<K,V>` -- via TSetElement<TPair<K,V>> (read_all, for_each)
- `FProperty` -- full UE5 property struct with reflection metadata
- `FField`, `FFieldClass`, `FFieldVariant` -- UE5 field system

### Enum Definitions
- `EObjectFlags` -- 28 flags
- `EClassFlags` -- 32 flags
- `EClassCastFlags` -- 57 flags (u64)
- `EPropertyFlags` -- 47 flags (u64)
- `EFunctionFlags` -- 33 flags
- `EArrayPropertyFlags` -- 2 variants

## Anti-Detection Measures

### Protect.sh
```bash
sudo mount -o remount,rw,hidepid=2 /proc     # Hide /proc entries from non-root
sudo sysctl -w kernel.yama.ptrace_scope=2     # Restrict ptrace
macchanger -r <interface>                      # Randomize MAC on all interfaces
```

### Assessment
- **Weak:** hidepid=2 only hides /proc from non-root; BattlEye runs as root/kernel
- **Irrelevant:** MAC randomization has no bearing on game anti-cheat
- **Missing:** No memory read pattern obfuscation, no timing randomization
- **Fingerprint risk:** Package name "dune" in Cargo.toml, process creates visible GLFW window

## Hardcoded Offsets (ALL from Jul 2025, OUTDATED for v1.5.3.1)

### From offsets.rs
| Name | Value | Status |
|------|-------|--------|
| UWORLD | 0xbcb2cb0 | OUTDATED (causes crash) |
| GNAMES | 0xbd62700 | OUTDATED |
| GOBJECTS | 0xbe2da20 | OUTDATED |
| OWNING_GAME_INSTANCE | 0x330 | Likely stable (engine offset) |
| LOCALPLAYERS | 0x40 | Likely stable (engine offset) |
| PLAYER_CONTROLLER_OFFSET | 0x38 | Likely stable (engine offset) |
| PLAYER_CAMERA_MANAGER | 0x408 | Likely stable (engine offset) |
| PLAYER_CHARACTER | 0x3a0 | Likely stable (engine offset) |

### Scattered in esps.rs / overlay.rs / main.rs
| Offset | Context | Notes |
|--------|---------|-------|
| GameState + 0x368 | PlayerArray | Game-specific, likely outdated |
| PlayerState + 0x620 | LifeState | Game-specific |
| PlayerState + 0x3d8 | Pawn | Game-specific |
| PlayerState + 0x460 | PlayerName (FString) | Game-specific |
| Actor + 0x240 | RootComponent | Engine offset, likely stable |
| SceneComponent + 0x1b8 | RelativeLocation | Engine offset, likely stable |
| CameraManager + 0x13a0 + 0x10 | MinimalViewInfo | Engine offset with game delta |
| SpiceField + 0x690 | Type (FName) | Game-specific |
| SpiceField + 0x750 | Status (u32) | Game-specific |
| Character + 0xfd8 | WeaponActor | Game-specific |
| WeaponActor + 0x4e0 + 0x160 | Damage (f32) | Game-specific |

## Known Bugs & Issues

1. **Fatal crash at overlay.rs:63** -- `self.process.read(self.uworld + 0x2c8).unwrap()` panics because UWORLD offset (0xbcb2cb0) is outdated. Two UC users confirmed this crash (S4g3l0rd55 diagnosed root cause).

2. **Hardcoded 1920x1080** -- Both overlay.rs (window size) and screen.rs (projection) assume 1080p. Non-1080p displays get broken ESP positioning.

3. **unwrap() in render loop** -- Multiple `.unwrap()` calls in the hot path (overlay.rs:63, esps.rs:143/146/153/159/162) will panic on any transient read failure instead of gracefully skipping.

4. **Memory leaks** -- `Box::leak()` used for FName cache and FString conversion. Grows unbounded over time. Acceptable for short sessions but problematic for long ones.

5. **No distance filtering** -- Draws every actor in the level regardless of distance, wasting draw calls on entities thousands of meters away.

6. **Per-frame string comparison** -- `actor_name.starts_with("BP_...")` runs on every actor every frame. Should cache actor types after first identification.

7. **Weapon damage label misleading** -- UI says "To NPCs Only" but the write goes to the local character's weapon, affecting all damage output including PvP.

8. **box_mode hardcoded** -- `let box_mode = 2;` with a TODO to make configurable, never done.

## Code Quality Assessment

### Strengths
- Clean Rust with idiomatic patterns (Result chaining, pattern matching)
- Good separation: types/offsets/rendering/UI in distinct modules
- OnceCell for global state is correct for this use case
- TArray/TMap implementations are reusable for any UE5 external reader
- Proper `repr(C)` on structs that map to game memory
- FName cache is a smart optimization (despite the leak)

### Weaknesses
- No config file -- all offsets hardcoded, requires recompilation to update
- No graceful degradation -- single bad read crashes the whole overlay
- No actor type caching -- resolves FName strings every frame for every actor
- Resolution hardcoded -- should read from display or make configurable
- GObjects loaded but barely used -- only actor name resolution uses FName, the full GObjects dump is underutilized
- FProperty/FField types defined but never used (leftover from SDK dump plans?)

## Reusability for Custom Development

### Directly Reusable Components
- `src/unreal/types/structs.rs` -- Full UE5 type definitions (FName, TArray, UObject, FVector, TMap, FString)
- `src/unreal/types/enums.rs` -- Complete UE5 flag definitions
- `src/unreal/screen.rs` -- World-to-screen projection (fix hardcoded resolution)
- `src/unreal/mod.rs` -- GObjects loader (chunked array reader)
- FName resolution algorithm (chunked name pool with caching)

### Needs Modification
- `src/unreal/offsets.rs` -- All game-specific offsets need updating for v1.5.3.1
- `src/ui/overlay.rs` -- Add error handling, dynamic resolution
- `src/ui/esps.rs` -- Add distance filtering, actor type caching
- `src/app.rs` -- Add config file loading for offsets

### Architecture Patterns Worth Adopting
- External `/proc/` reader approach (BattlEye doesn't monitor this well on Linux)
- egui_overlay for transparent window (lightweight, good performance)
- OnceCell globals for process handle and name cache
- memflex crate for typed cross-process reads

## Updating for v1.5.3.1

To make this work with current game version:

1. **Get new offsets** from [[../Threads/UC-Reversal-Structs-Offsets]] or run Dumper-7 with `-nobe` Steam launch option
2. Update `offsets.rs` with new UWORLD, GNAMES, GOBJECTS values
3. Verify all game-specific offsets in esps.rs (PlayerArray, LifeState, Pawn, etc.) against new SDK dump
4. Fix `overlay.rs:63` error handling (replace `.unwrap()` with match/if-let)
5. Make resolution configurable

## See Also

- [[../Threads/UC-ESP-External-Linux]] -- UC thread discussion and known issues
- [[../Threads/UC-Reversal-Structs-Offsets]] -- Updated offsets needed
- [[UE5-Common-Patterns]] -- General UE5 patterns
- [[../Offsets/UE5-Offset-Guide]] -- Offset documentation
- [[../Tools/UEDumper]] -- SDK dumping for fresh offsets
