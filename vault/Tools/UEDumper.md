# UEDumper - Unreal Engine SDK Dumper

> From github.com/Spuckwaffel/UEDumper. 2026-09-19.

## Overview

UEDumper is an all-in-one Unreal Engine Dumper and Editor supporting UE 4.19 - 5.4.0 without requiring modification of internal structures. Relevant because Dune Awakening runs on UE5.

- **Repository:** `github.com/Spuckwaffel/UEDumper`
- **DMA variant:** `github.com/dvGrab/UEDumperDMA` (for DMA-based memory reading)

## Supported UE Versions

- Unreal Engine 4.19 through 5.4.0
- No internal structure modifications needed
- Manual reverse engineering required per-game for offsets

## Core Dumping Requirements

To dump any UE game, you need three critical offsets:

| Offset | Macro Name | Required |
|--------|-----------|----------|
| GObjects | `OFFSET_GOBJECTS` | Yes |
| GNames | `OFFSET_GNAMES` | Yes |
| GWorld | `OFFSET_GWORLD` | For live editor |

> "You still have to reverse it on your own to find the information" for each game.

## Configuration Files

### UEdefinitions.h
```cpp
// Key settings per-game:
UE_VERSION           // Set to target UE version
WITH_CASE_PRESERVING_NAME   // Game-specific flag
UE_BLUEPRINT_EVENTGRAPH_FASTCALLS  // Game-specific flag
```

### Offsets.h
- Stores all game offsets with unique names
- `OFFSET_GNAMES` and `OFFSET_GOBJECTS` are mandatory
- Contains offset signatures and pointers

## FName Decryption

Some UE games encrypt FNames (function/class names):

### Implementation
- Add decryption logic to `FName_decryption.h`
- Enable `USE_FNAME_ENCRYPTION` macro
- Caller expects decrypted name in `inputBuf` parameter

```cpp
// Pseudocode for FName decryption
void decrypt_fname(char* inputBuf, int len) {
    // Game-specific decryption logic here
    // XOR, rolling key, custom algorithm, etc.
}
```

## SDK Generation

- Generates SDK and MDK (Mod Development Kit) formats
- Can save project and generate SDK at any time
- Changes stored in StructDefinitions files for regeneration

## Live Editor

- Read/write game memory at runtime
- Browse UWorld and derived objects
- Configurable refresh rate (default 500ms)
- Custom struct visualization via LiveEditor.cpp

## Related UE5 Dumping Tools

| Tool | URL | Notes |
|------|-----|-------|
| UEDumper | `github.com/Spuckwaffel/UEDumper` | Primary, UE 4.19-5.4 |
| UEDumperDMA | `github.com/dvGrab/UEDumperDMA` | DMA variant |
| UE5Dumper | `github.com/Jiang-Night/UE5Dumper` | UE5 specific |
| UE4-Dumper | `github.com/Device1337/UE4-Dumper` | Older UE4 focus |
| Unreal Finder Tool | `github.com/uuaing/Unreal-Finder-Tool` | UE4 info fetcher |

## Dune Awakening Status

- **No specific Dune Awakening offsets found publicly** as of 2026-09-19
- Game uses UE5 so UEDumper should be compatible
- Would need to reverse engineer GNames, GObjects, GWorld offsets
- BattlEye may interfere with direct memory access (DMA variant may be needed)

## See Also

- [[../References/Anti-Cheat-BattlEye]]
- [[../Offsets/UE5-Offset-Guide]]
- [[../Guides/UE5-Reversing-Methodology]]
- [[Cheat-Engine-Tables]]
