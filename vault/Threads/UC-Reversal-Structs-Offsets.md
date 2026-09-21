# [UC] Dune: Awakening Reversal, Structs and Offsets

> Source: unknowncheats.me/forum/other-fps-games/699887-dune-awakening-reversal-structs-offsets.html
> Thread by: Respecter (Nov 2012, 1424 posts, rep 44478)
> 79 replies, 35,171+ views, 4 pages (last checked 2026-09-20)
> Forum: Other FPS Games
> Scraped: 2026-09-19

## Game Info (from OP)

- **Engine:** Unreal Engine 5.2.1
- **AntiCheat:** BattlEye
- **DirectX:** 12
- **GameName:** DuneSandbox
- **GameVersion:** `5.2.1-1514770+Seabass_sb-1.1.0.11`

## Beta Offsets (May 8, 2025 - by Respecter)

```
GObjects:      0xBD47670
GNames:        0xBC7C380
GWorld:        0xBBD1150
PE-Index:      0x55
GetObjectName: 0x51E5C20
```

## Release Offsets (June 10, 2025 - by BlackMax97)

### Core Functions
```
StaticFindObject:    0x5518550
StaticLoadObject:    0x5518E70
ProcessEvent:        0x54E8A50
BoneMatrix:          0x71FE140
GObjects:            0xBEC79A0
GetFullName:         0x5508300
GNames (FNamePool):  0xBDFC640
FName::AppendString: 0x5307130
GWorld:              0xC02EE08
```

### UObject Layout
```
Off::UObject::Flags:  0x8
Off::UObject::Index:  0xC
Off::UObject::Class:  0x10
Off::UObject::Name:   0x18
Off::UObject::Outer:  0x20
```

### UStruct / UField
```
Off::UStruct::SuperStruct:     0x48
Off::UStruct::Children:        0x50
Off::UStruct::ChildProperties: 0x58
Off::UStruct::Size:            0x60
Off::UStruct::MinAlignments:   0x64
Off::UField::Next:             0x30
Off::UClass::CastFlags:        0xE0
Off::UClass::ClassDefaultObject: 0x130
Off::UClass::ImplementedInterfaces: 0x220
Off::UEnum::Names:             0x48
```

### FProperty System
```
Game uses FProperty system (not legacy UProperty)

Off::FField::Next:  0x20
Off::FField::Name:  0x28
Off::FField::Flags: 0x30

Off::Property::ArrayDim:        0x38
Off::Property::ElementSize:     0x3C
Off::Property::PropertyFlags:   0x40
Off::Property::Offset_Internal: 0x4C
UPropertySize:                  0x78

Off::ArrayProperty::Inner:      0x78
Off::SetProperty::ElementProp:  0x78
Off::MapProperty::Base:         0x78
```

### UFunction
```
Off::UFunction::FunctionFlags: 0xB8
Off::UFunction::ExecFunction:  0xE0
```

### InSDK Offsets
```
Off::InSDK::ULevel::Actors:              0xA0
Off::InSDK::UDataTable::RowMap:          0x38
Off::InSDK::Text::TextSize:             0x18
Off::InSDK::Text::TextDatOffset:        0x0
Off::InSDK::Text::InTextDataStringOffset: 0x30

PE-Offset: 0x54E8A50
PE-Index:  0x55
```

### Signature Patterns
```
GetBoneMatrix: E8 ? ? ? ? 48 8B 05 ? ? ? ? 80 38 ? 74 ? 48 8B 05 ? ? ? ? 0F
```

## SDK Dump

BlackMax97 uploaded an SDK dump file:
- SHA256: `1f36e910d5d2ac0d6ec149aa2b37213d18c82772a185f8808072065820adbefe`
- Filename: `5.2.1514770+Seabass_sb-1.1.0.DuneSandbox.zip`
- Download: UC file ID 50056

## Technical Discussion (Page 1)

### Launch Without BattlEye
- Execute the .exe directly (bypasses BE loader)
- `-nobe` launch option on Steam
- Useful for SDK dumping and static analysis

### Dumping the SDK
- Use Dumper-7 while BE is disabled
- CE speedhack can slow CPU clock to give time for dump before crash
- Full SDK dump achievable even with BE (with timing tricks)

### Resource Position Problem
Respecter discovered that **PickUpItems are NOT in standard Actors array** (`ULevel +0xA0`):

> "It's a resource, but I never saw this type on unreal, you can see the object, but you can't retrieve its position"

**Solution hints:**
- Check `World->TArray<ULevel*> Levels` (iterate all levels, not just PersistentLevel)
- Check `ResourceSpawner` objects
- Server functions: `ServerRequestSpawnResourceNode`, `ServerRequestScanResourceNode`
- Devs may have "erased the standard class root component for resources"
- Similar approach seen in "Will To Life" game

### Admin Functions Found in SDK
```cpp
void SetHasSpeedHax(const bool bValue);
```

### LineTraceSingle Issue
- Question about LineTraceSingle ignoring mountains (terrain)
- Relevant for visibility checks in ESP

### BattlEye Bypass Discussion
- Partial bypasses available (can find values but can't debug/script)
- Full internal injection requires proper BE bypass
- DMA approach discussed as alternative

## Thread Participants (Page 1)

| User | Rep | Contribution |
|------|-----|-------------|
| Respecter | 44478 | OP, beta offsets, resource reversing |
| BlackMax97 | 58943 | Release offsets, full SDK dump, forum moderator |
| N_FIDANzza | 18146 | UWorld level iteration advice |
| dracorx | 84282 | -nobe launch tip, former staff |
| FX142 | 235 | GetBoneMatrix signature |
| bmgr | 18 | Dumper-7 technique, SetHasSpeedHax discovery |
| RaphaelS | 1010 | SDK dumping interest |
| hzm0603 | 921 | GetBoneMatrix request, LineTrace question |

---

## Updated Offsets by Game Version

### v1.1.0.13 (June 12, 2025 - RaphaelS, post #23)
```
GObjects:            0xBECFAA0
GNames (FNamePool):  0xBE04740
FName::AppendString: 0x530BE30
GWorld:              0xBD55888
PE-Offset:           0x54EE1A0
GameVersion: 5.2.1-1519259+Seabass_sb-1.1.0.13
```
SDK dump: UC file ID 50076

### v1.1.0.14 build 1520778 (June 13, 2025 - RaphaelS, post #36)
```
GObjects:            0xBED1C20
GNames (FNamePool):  0xBE068C0
FName::AppendString: 0x530CF20
GWorld:              0xBD57A38
PE-Offset:           0x54EF290
GameVersion: 5.2.1-1520778+Seabass_sb-1.1.0.14
```
SDK dump: UC file ID 50089

### v1.1.0.14 build 1522808 (June 13, 2025 - BlackMax97, post #39)
```
GWorld:           0xBD57B78
GObjects:         0xBED1DA0
StaticFindObject: 0x551F1C0
StaticLoadObject: 0x551FAE0
ProcessEvent:     0x54EF6C0
BoneMatrix:       0x7205A70
```

## Page 2 Technical Discussion (Posts 21-40)

### Injection Methods (bmgr, post #35)
- LoadLibrary injection unexpectedly works for internal cheats
- Manual mapping gets killed immediately by BattlEye
- Hypothesis: must have launched without BE (WARBYYTE2 confirms LoadLibrary + BE = impossible)
- No hooks, scanning, detours, or inline patching used

### UWorld Reading Issue (FarmXC0de, post #22)
- Reading Base + 0xC036F38 returns 0x8000000000
- Fix: use correct GWorld offset for the current game version

### FName Reading Problem (RaphaelS, post #38)
- Finding "too many" actors but zero names
- Indicates FName reading implementation issue

### Additional Participants (Page 2)
| User | Contribution |
|------|-------------|
| laowai | Confirmed v1.1.0.13 offsets |
| goatly | Asked about internal benefits |
| FarmXC0de | UWorld reading issues |
| WARBYYTE2 | Confirmed LoadLibrary+BE impossible |
| ivanpos2010 | Disputed UWorld offset |
| bongrip69 | UEDumper issues |
| BigBombCode | Requested bone name dump |

## Complete Bone Map (557 bones - Xed0s, post #41)

### Key Aimbot Bones
```
Root:     0    pelvis:    1
spine_01: 2    spine_02:  3    spine_03: 4    spine_04: 5    spine_05: 6
neck_01:  7    neck_02:   8    head:     9
clavicle_l: 10   upperarm_l: 11   lowerarm_l: 12   hand_l: 20
clavicle_r: 140  upperarm_r: 141  lowerarm_r: 142  hand_r: 150
thigh_r: 464  calf_r: 465  foot_r: 466
thigh_l: 497  calf_l: 498  foot_l: 499
```

### Weapon / IK Bones
```
wpn_root: 538  wpn_move: 539
ik_foot_l: 541  ik_foot_r: 542
ik_hand_gun: 544  ik_hand_l: 545  ik_hand_r: 546
Interact: 547
```

### Cloth Simulation
- `cloth_upper_root` (274) through `cloth_upper_row_12_11_end` (406) - upper body cloth
- `cloth_lower_root` (407) through `cloth_lower_row_08_06_end` (463) - lower body cloth
- `hair_root` (548) through `hair_05` (553) - hair chain

Full skeleton: 557 bones total including finger joints (24-126 left, 150-256 right), toe joints, twist/corrective bones, and virtual bones (VB Curves: 554, VB root_foot_l: 555, VB root_foot_r: 556)

## Mesh/Bone Component Offsets (FarmXC0de, post #45)
```
CompToWorld = 0x160
BoneArray   = 0x06C0
Mesh        = 0x03D8
```

## Dune Actor Class Names (choky10, post #51)

```
ADunePlayerCharacter
ADuneNpcCharacter
AHarvestableInstancedMeshActor
ALootContainer
AItemContainer
ATemporaryLootContainer
AWorldItemBase
AResourceNode
AResourceField
APatrolShip
ASandwormPawn
```

## Resource Spawner Architecture (choky10, post #58)

Resources are NOT treated as Actors until scanned:
1. Each ResourceNode belongs to a **group**
2. Each group has a **RESOURCE SPAWNER**
3. Inside each spawner: 0-5/6 nodes
4. Must do math to get correct **WORLD SPACE** position from spawner data

**Tips from choky10:**
- Dump SDK **IN SERVER** before doing actions (pickup, loot, kill, drive) to catch more classes
- ProcessEvent crashes on certain game names - skip by function index
- PRF is also used (protocol)
- **NetMessenger** communication system for vehicle hijack etc.
- Since last update, vehicle hijack triggers a payload sent by net mgr (anti-cheat telemetry)
- "Don't be surprised if someone on PVE steals your base" (exploit hint)

## NPC Hostility Detection (posts #74-75)

**Problem:** No obvious hostility/team/aggression variable works (crazybloo)

**Suggested approach (suxers):**
- Faction Component / Team Component / Relationship Component
- Faction ID
- AI controller component (hostile vs passive AI behavior)
- Check interactable NPC paths

## FName Status

FName encryption NOT present as of July 2025 (confirmed by GodOfLife, post #64):
```
UWorld FName: MainMenu
Game Instance: BP_DuneGameInstance_C
Local Player: DuneLocalPlayer
Player Controller: BP_DunePlayerController_C
```

## Later Version Offsets

### June 26, 2025 (BlackMax97, post #56)
```
GObjects:            0xBED2520
GNames:              0xBE07200
FName::AppendString: 0x5310540
GWorld:              0xBD57B98
PE-Offset:           0x54E7230
```

### Early July 2025 (GodOfLife, post #61 - from Dumper-7)
```
GNames:   0xBD53580
GObjects: 0xBE1E8A0
GWorld:   0xBCA3C20
```
**Note:** These offsets from Dumper-7 caused UEDumper to fail with `"ERROR! Could not find the requested Pointer 2 in the cache!"` — Dumper-7 offsets may not be directly compatible with UEDumper.

### Mid-July 2025 (GodOfLife, post #62)
```
GWorld:   0xBCA8BE0
GNames:   0xBD58600
GObjects: 0xBE23920
```

### August 9, 2025 (WARBYYTE2, post #69)
```
UWorld: 0xBFC3430
Engine: 0xBFC05B8
```

### September 18, 2025 (Laurax64, post #70)
```
GNAMES:           0xBBB0F00
GWORLD:           0xBAE4E18
BONEARRAY:        0x6C8 / 0x6D8
COMPONENTTOWORLD: 0x300
```

## Current Game Version (as of Sept 2026)

**v1.5.3.1** is the current live version (confirmed by Obsidian420, post #79, 2026-09-20). The latest SDK dump on UC (file 57111, Aug 2026) is **outdated** — it predates v1.5.3.x (Amir Mustafa asked, Obsidian420 confirmed).

### GameVersion String Format
```
5.2.1-{BuildNumber}+Seabass_sb-{GameVersion}
```
Known versions: 1.1.0.11 → 1.1.0.13 → 1.1.0.14 → ... → 1.4.10 → ... → 1.5.3.0 → 1.5.3.1 (current)

## SDK Dump File History

| Date | Version | UC File ID | SHA256 | Uploader |
|------|---------|-----------|--------|----------|
| Jun 10, 2025 | 1.1.0.11 | 50056 | 1f36e91... | BlackMax97 |
| Jun 12, 2025 | 1.1.0.13 | 50076 | 6717e6e... | RaphaelS |
| Jun 13, 2025 | 1.1.0.14 | 50089 | 2690cf1... | RaphaelS |
| Aug 4, 2025 | unknown | 50670 | 96fbd5b... | Xed0s |
| Jul 9, 2026 | 1.4.10 | 56163 | 5498fcc... | (approved rhaym) |
| Aug 22, 2026 | pre-1.5.3 (outdated) | 57111 | f3dffa2... | (approved rhaym) |

## See Also

- [[../Offsets/UE5-Offset-Guide]]
- [[../Tools/UEDumper]]
- [[../Code-Snippets/UE5-Common-Patterns]]
- [[UC-Dupe-Exploit]]
- [[UC-Repair-Exploit]]
