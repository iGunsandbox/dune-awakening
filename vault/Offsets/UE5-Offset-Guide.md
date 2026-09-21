# UE5 Offset Research for Dune Awakening

> Compiled from UEDumper documentation and UE5 reversing resources. 2026-09-19.

## Dune Awakening Engine Details

- **Engine:** Unreal Engine 5.2.1
- **GameName:** DuneSandbox
- **GameVersion:** `5.2.1-1514770+Seabass_sb-1.1.0.11`
- **Anti-Cheat:** BattlEye (kernel-level)
- **DirectX:** 12
- **FProperty System:** Yes (not legacy UProperty)

## Live Offsets

> Source: UC Reversal thread. See [[../Threads/UC-Reversal-Structs-Offsets]] for full context.

### Beta Offsets (May 8, 2025 - Respecter)

```
GObjects:      0xBD47670
GNames:        0xBC7C380
GWorld:        0xBBD1150
PE-Index:      0x55
GetObjectName: 0x51E5C20
```

### Release Offsets (June 10, 2025 - BlackMax97)

#### Core Engine Globals
```
GObjects:            0xBEC79A0
GNames (FNamePool):  0xBDFC640
GWorld:              0xC02EE08
```

#### Core Functions
```
StaticFindObject:    0x5518550
StaticLoadObject:    0x5518E70
ProcessEvent:        0x54E8A50
BoneMatrix:          0x71FE140
GetFullName:         0x5508300
FName::AppendString: 0x5307130
```

#### Signature Patterns
```
GetBoneMatrix: E8 ? ? ? ? 48 8B 05 ? ? ? ? 80 38 ? 74 ? 48 8B 05 ? ? ? ? 0F
```

## Dune Awakening Struct Layouts

### UObject
```
+0x00 VTable
+0x08 ObjectFlags
+0x0C InternalIndex
+0x10 ClassPrivate (UClass*)
+0x18 NamePrivate (FName)
+0x20 OuterPrivate (UObject*)
```

### UField / UStruct / UClass
```
UField::Next:                    0x30
UStruct::SuperStruct:            0x48
UStruct::Children:               0x50
UStruct::ChildProperties:        0x58   (FProperty chain head)
UStruct::Size:                   0x60
UStruct::MinAlignments:          0x64
UClass::CastFlags:               0xE0
UClass::ClassDefaultObject:      0x130
UClass::ImplementedInterfaces:   0x220
UEnum::Names:                    0x48
```

### FProperty System (FField-based)
```
FField::Next:                0x20
FField::Name:                0x28
FField::Flags:               0x30
Property::ArrayDim:          0x38
Property::ElementSize:       0x3C
Property::PropertyFlags:     0x40
Property::Offset_Internal:   0x4C
UPropertySize:               0x78
ArrayProperty::Inner:        0x78
SetProperty::ElementProp:    0x78
MapProperty::Base:           0x78
```

### UFunction
```
UFunction::FunctionFlags:    0xB8
UFunction::ExecFunction:     0xE0
```

### InSDK
```
ULevel::Actors:                  0xA0
UDataTable::RowMap:              0x38
Text::TextSize:                  0x18
Text::TextDatOffset:             0x0
Text::InTextDataStringOffset:    0x30
PE-Offset:                       0x54E8A50
PE-Index:                        0x55
```

### Key Types for Cheats
| Structure | Use Case |
|-----------|----------|
| `UWorld` | World instance, level access |
| `ULevel` | Level/map data, Actors array at +0xA0 |
| `APlayerState` | Player info, names, scores |
| `ACharacter` | Player/NPC character data |
| `USkeletalMeshComponent` | Bone positions for aimbot |
| `APlayerCameraManager` | Camera position/rotation for W2S |
| `ResourceSpawner` | Resource node positions (NOT in Actors) |

## Dune-Specific Gotchas

- **Resources NOT in Actors array**: PickUpItems/resources are not in `ULevel::Actors` (+0xA0). Must iterate `World->TArray<ULevel*> Levels` or find `ResourceSpawner` objects
- **Server functions for resources**: `ServerRequestSpawnResourceNode`, `ServerRequestScanResourceNode`
- **Admin speed hack**: SDK contains `void SetHasSpeedHax(const bool bValue);`
- **LineTraceSingle**: May ignore terrain/mountains - relevant for visibility checks

## SDK Dump Available

- UC file ID 50056
- Filename: `5.2.1514770+Seabass_sb-1.1.0.DuneSandbox.zip`
- SHA256: `1f36e910d5d2ac0d6ec149aa2b37213d18c82772a185f8808072065820adbefe`
- Uploaded by BlackMax97

## Finding Offsets Methodology

### Pattern Scanning
1. Use IDA Pro, Ghidra, or x64dbg to analyze the game binary
2. Search for known UE5 patterns/signatures
3. Cross-reference with UEDumper's built-in patterns

### Runtime Scanning
1. Launch game without BattlEye (execute .exe directly or `-nobe` Steam launch)
2. Attach UEDumper or Dumper-7
3. CE speedhack trick: slow CPU clock to buy dump time before crash
4. Tool generates full SDK with all class layouts

### DMA Approach
- If BattlEye blocks direct access, DMA hardware can read game memory
- UEDumperDMA variant designed for this use case
- Requires DMA hardware (e.g., PCILeech-compatible FPGA)

## FName Encryption

Dune Awakening does NOT appear to use FName encryption based on successful SDK dumps with standard tools. If future patches add it:
- Need to reverse the decryption algorithm
- Implement in `FName_decryption.h`
- Enable `USE_FNAME_ENCRYPTION` macro

## See Also

- [[../Tools/UEDumper]]
- [[../Tools/Cheat-Engine-Tables]]
- [[../Guides/UE5-Reversing-Methodology]]
- [[../References/Anti-Cheat-BattlEye]]
