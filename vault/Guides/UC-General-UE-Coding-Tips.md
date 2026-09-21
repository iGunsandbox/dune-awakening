# UC General UE Coding Tips & Practices

> Scraped from UnknownCheats Unreal Engine 4 forum and related threads. 2026-09-20.
> These are general UE4/UE5 practices directly applicable to Dune Awakening.

## GEngine Chain: Alternative to GWorld Direct Read

> Source: UC UE4 forum, "[Information] Stop Using UWorld Decrypt" by SnowSable (July 2026)

Instead of using a direct GWorld pointer (which changes every patch), traverse the engine's object chain:

```
Module Base -> GEngine -> GameViewport -> World
```

### Implementation

```cpp
namespace offsets {
    constexpr uintptr_t GEngine      = 0x???; // game-specific
    constexpr uintptr_t GameViewport = 0xB70;
    constexpr uintptr_t World        = 0x78;
}

uintptr_t GetWorldFromGEngine() {
    uintptr_t moduleBase = GetModuleBase();
    if (!moduleBase) return 0;

    uintptr_t engine = Read<uintptr_t>(moduleBase + offsets::GEngine);
    if (!engine) return 0;

    uintptr_t viewport = Read<uintptr_t>(engine + offsets::GameViewport);
    if (!viewport) return 0;

    return Read<uintptr_t>(viewport + offsets::World);
}
```

### Why This Is Better
- GWorld decrypt changes every patch (key rotation, XOR/shift changes)
- GEngine chain only needs a few offset updates
- Easier to debug: check each pointer in chain individually
- GEngine offset is more stable than GWorld decryption routines

### Dune Awakening Note
WARBYYTE2 posted an Engine offset (0xBFC05B8) alongside UWorld in the Reversal thread (Aug 2025). This confirms the GEngine approach works for Dune. Dune does NOT use FName encryption, so the standard GWorld pointer works too, but GEngine is a good fallback if GWorld changes.

## DLSS / Upscaling W2S Fix

> Source: UC EFT forum, "[Discuss] DLSS broke my w2s" (April 2022)

DLSS renders at lower resolution then upscales. This breaks World-to-Screen if you use screen dimensions directly.

**Key findings:**
- DLSS Performance mode = 2x upscale (renders at half res)
- DLSS Balanced = 1.72x upscale
- DLSS Quality = 1.5x upscale
- W2S coordinates need to account for render resolution vs display resolution
- ESP jittering with TAA/DLSS: try enabling Z-Blur option (slight DOF effect)
- When hooking Present: delay rendering until fully in-game (transforms invalid during countdown)

**Relevance to Dune:** Uses DX12, may have DLSS/FSR. If implementing ESP overlay, account for upscaling.

## SDK Dumping Best Practices

> Compiled from UC Reversal threads and UE4 forum

### Dumper-7 (Recommended)
- Universal SDK generator for UE4 and UE5
- 136 replies, 98k views on UC, by Fischsalat
- Auto-detects GObjects, GNames, FNamePool
- Generates full SDK with class layouts
- Tested on UE 5.2.1 (same version as Dune)
- Source: github.com/Fischsalat/Dumper-7
- UnrealContainers (TArray, TMap): github.com/Fischsalat/UnrealContainers

#### SDK Usage After Dump (from Dumper-7 thread)
```cpp
// Initialize
SDK::InitGObjects();

// Method 1: Direct GWorld pointer
SDK::UWorld** GWorld = (SDK::UWorld**)(ModuleBase + Offsets::GWorld);
auto GameInstance = (*GWorld)->OwningGameInstance;
auto LocalPlayer = GameInstance->LocalPlayers[0];

// Method 2: Find GEngine through GObjects iteration
UEngine* GEngine = nullptr;
for (int i = 0; i < UObject::GObjects->Num(); i++) {
    UObject* Obj = UObject::GObjects->GetByIndex(i);
    if (!Obj) continue;
    if (Obj->IsA(UEngine::StaticClass()) && !Obj->IsDefaultObject()) {
        GEngine = reinterpret_cast<UEngine*>(Obj);
    }
}
auto World = GEngine->GameViewport->World;
```

#### Common Dumper-7 Issues
- Name duplication in generated SDK (manual fix needed)
- Enum size defaults to uint8 unless EnumProperty provides size
- File handle limit: increase `_setmaxstdio` in Generator.cpp line 410
- Some games need hardcoded `Struct::Next` offset

### Dump Timing for BE Games
1. Launch game without BattlEye (`-nobe` or execute .exe directly)
2. OR use CE speedhack to slow CPU clock, run Dumper-7 before detection
3. Dump SDK IN SERVER (not just menu) to capture runtime-only classes
4. Dump before/after actions (pickup, loot, kill, drive) for more class coverage

### UEDumper (Alternative)
- All-in-one tool, UE 4.19-5.4 support
- Requires manual GNames/GObjects offsets
- Has live editor mode for runtime browsing
- DMA variant available (UEDumperDMA)

## Actor Iteration Patterns

### Standard Level Actors
```
UWorld -> PersistentLevel -> Actors (at +0xA0)
```

### Multi-Level Iteration (Required for Dune)
Resources and many objects are NOT in PersistentLevel. Must iterate ALL levels:
```
UWorld -> TArray<ULevel*> Levels -> each Level -> Actors
```

### Dune-Specific Actor Classes
```
ADunePlayerCharacter    - Player characters
ADuneNpcCharacter       - NPCs
AHarvestableInstancedMeshActor - Harvestable items
ALootContainer          - Loot containers
AItemContainer          - Item storage
ATemporaryLootContainer - Temp loot
AWorldItemBase          - World items
AResourceNode           - Resource nodes (in spawner groups)
AResourceField          - Resource fields
APatrolShip             - Patrol ships
ASandwormPawn           - Sandworms
```

## Resource Node Architecture

Resources use a spawner pattern (NOT standard actors):
1. Each `AResourceNode` belongs to a **group**
2. Each group has a **ResourceSpawner**
3. Each spawner contains 0-5/6 nodes
4. Position requires math from spawner world-space data
5. Nodes appear only after scanning (server-side spawn)
6. Server functions: `ServerRequestSpawnResourceNode`, `ServerRequestScanResourceNode`

## ProcessEvent Hooking Gotchas

- Some function names in Dune crash when processed by hooked ProcessEvent
- Fix: skip by function index instead of processing all
- PE crash particularly happens during server switching
- PE-Index for Dune: 0x55

## Injection Methods for BattlEye Games

| Method | Status | Notes |
|--------|--------|-------|
| LoadLibrary | Works WITHOUT BE | Surprisingly works when BE disabled |
| Manual Mapping | Killed immediately | BE detects it right away |
| DMA Read | Works | Hardware memory read, undetectable |
| Internal + BE | Requires bypass | Full kernel bypass needed |
| External overlay | Medium risk | Separate process, RPM-based |

## BoneMatrix / ESP Essentials

### Key Dune Bone Indices
```
head: 9         (aimbot primary target)
neck_02: 8      (aimbot secondary)
spine_03: 4     (body shot target)
pelvis: 1       (center mass)
hand_r: 150     (weapon hand)
```

### Component Offsets (Dune-specific)
```
Mesh:            0x03D8
CompToWorld:     0x160 (or 0x300 in later versions)
BoneArray:       0x6C0 (or 0x6C8/0x6D8 in later versions)
```

### GetBoneMatrix Signature
```
E8 ? ? ? ? 48 8B 05 ? ? ? ? 80 38 ? 74 ? 48 8B 05 ? ? ? ? 0F
```

## NPC Hostility Detection

No simple flag found. Suggested approach:
- Faction Component / Team Component / Relationship Component
- AI Controller component (hostile vs passive behavior)
- Interactable surface / quest giver / dialogue component

## Health Reading

`m_DuneCharacterAttributeSet` returns 0 via direct read. May need:
- Attribute system uses Gameplay Ability System (GAS)
- Health stored in attribute set but accessed via delegates
- Try reading through the character's AbilitySystemComponent instead

## Anti-Cheat Telemetry Warning

As of mid-2025 update, certain game actions trigger NetMessenger payloads:
- Vehicle hijack sends telemetry to server
- Likely used for cheat detection / ban waves
- Base permission exploits on PvE servers still possible despite this

## IDA Offset-Finding Methodology

> Source: UC UE4 forum, "[Tutorial] Finding Offset In UE4 - Any Game" by Gen_MJ (June 2018, 93k views, sticky)
> and "[Help] How to find GWorld, GObjects and FNamePool in UE5" (March 2026)

### Prerequisites
- IDA Pro (with decompiler)
- UE source code (GitHub)
- PE dumper: Explorer Suite (ntcore) / Scylla Imports Reconstructor
- NOT Process Explorer (creates .dmp, not useful)

### Workflow
1. Launch game → load into server/map
2. Dump PE with Task Explorer / Scylla → get `.exe`
3. Open dumped exe in IDA
4. Re-launch game → attach IDA debugger → rebase → save → detach
5. Let IDA auto-analysis finish
6. **Strings window:** View → Open Subviews → Strings (Shift+F12), enable Unicode

### Finding GWorld
**UE4 method (Gen_MJ):**
1. In UE source: find `GWorld` reference in `LaunchEngineLoop.cpp` → `FEngineLoop::Tick()`
2. Note nearby strings: `r.OneFrameThreadLag`
3. In IDA strings: search `r.OneFrameThreadLag` → xref to function → decompile (F5)
4. Look for `GNewWorldToMetersScale != 0.0` pattern:
```
if ( *(float *)&dword_XXXX != 0.0 ) {
    v35 = (UWorld *)GWorld;  // ← THIS IS YOUR TARGET
```
5. Double-click GWorld label → get address → subtract module base = offset

**UE5 method (RushenteHD):**
1. Search for string `"SeamlessTravel FlushLevelStreaming"`
2. Xref → decompile (F5)
3. Look for `qword_14XXXXXXX` being assigned
4. Strip `qword_14` prefix → that's GWorld offset

### Finding GNames / FNamePool
**UE4 method (Gen_MJ):**
- Search for `"ByteProperty"` in IDA strings
- Above the string, find the function containing the GNames pointer
- That function is `GetNames()`

**UE5 method (space785 / RushenteHD):**
1. Search for string `"%d ansi FNames"`
2. Xref → decompile
3. Look for `unk_` variable in the function
4. Strip prefix → that's FNamePool offset

### Finding GObjects
**String method (Gen_MJ):**
- Search for `"norhithread"` → leads to `FEngineLoop::PreInit` → `GUObjectArray`
- Also try `"SHOWDEFAULTS"` or `"SHOWPENDINGKILLS"` or `"DETAILED"`

**CE byte scanning (Stridemann2):**
- GNames byte pattern (UE 4.25):
  `2A 01 4E 6F 6E 65 08 03 42 79 74 65 50 72 6F 70 65 72 74 79...`
- For GObjects: use UniversalUE4Unlocker dump → invert byte addresses → search

### FUObjectArray Layout (UE 4.11+)
```cpp
class FUObjectArray {
    int32_t ObjFirstGCIndex;
    int32_t ObjLastNonGCIndex;
    int32_t MaxObjectsNotConsideredByGC;
    bool OpenForDisregardForGC;
    FChunkedFixedUObjectArray ObjObjects;
};
```

### UEngine String (for GEngine chain)
- Search string: `"engine-ini:/Script/Engine.Engine.GameEngine"` → leads to UEngine pointer
- From UEngine → UGameViewportClient → UWorld

### GetObjectById Formula
```cpp
*(__int64*)GlobalObjects + 8 * (id / 0x10400) + 24 * (id % 0x10400)
```

## See Also

- [[../Threads/UC-Reversal-Structs-Offsets]]
- [[../Offsets/UE5-Offset-Guide]]
- [[../Tools/UEDumper]]
- [[../Code-Snippets/UE5-Common-Patterns]]
- [[../References/Anti-Cheat-BattlEye]]
