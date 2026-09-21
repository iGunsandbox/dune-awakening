# UE5 Common Code Patterns

> Common UE5 code patterns for game reversing. From UEDumper docs, UC Reversal thread, and UE5 community. 2026-09-19.
> Updated 2026-09-20 with Dune Awakening-specific data from UC.

## GNames Pattern Scanning

### Common GNames Signatures (UE5)
```
// Generic UE5 patterns - actual Dune Awakening sigs need to be found
// Pattern varies by UE5 sub-version

// UE 5.1+ common pattern for GNames:
// 48 8D 0D ?? ?? ?? ?? E8 ?? ?? ?? ?? C6 05 ?? ?? ?? ?? 01 0F 10 03
// The 48 8D 0D (LEA RCX, [rip+offset]) loads the GNames address
```

### GObjects Common Pattern
```
// UE 5.x GObjects pattern:
// 48 8B 05 ?? ?? ?? ?? 48 8B 0C C8 48 8D 04 D1 EB
// 48 8B 05 loads the GObjects pointer
```

### GWorld Common Pattern
```
// UE 5.x GWorld:
// 48 8B 1D ?? ?? ?? ?? 48 85 DB 74
// 48 8B 1D (MOV RBX, [rip+offset]) loads UWorld pointer
```

## Actor Iteration (Dune Awakening)

### Standard PersistentLevel Actors
```cpp
UWorld* World = *(UWorld**)GWorldAddress;  // GWorld: 0xC02EE08 (v1.1.0.11)
ULevel* Level = World->PersistentLevel;
TArray<AActor*>& Actors = *(TArray<AActor*>*)((uintptr_t)Level + 0xA0);

for (int i = 0; i < Actors.Num(); i++) {
    AActor* Actor = Actors[i];
    if (!Actor) continue;
    
    FName ClassName = Actor->ClassPrivate->NamePrivate;
    const char* Name = GetNameFromFName(ClassName);
    
    // Dune actor class matching
    if (strstr(Name, "DunePlayerCharacter") ||
        strstr(Name, "DuneNpcCharacter") ||
        strstr(Name, "SandwormPawn") ||
        strstr(Name, "PatrolShip")) {
        // ESP rendering...
    }
}
```

### Multi-Level Iteration (REQUIRED for resources)
```cpp
// Resources are NOT in PersistentLevel. Must iterate ALL levels.
TArray<ULevel*>& Levels = World->Levels;
for (int l = 0; l < Levels.Num(); l++) {
    ULevel* Level = Levels[l];
    if (!Level) continue;
    TArray<AActor*>& Actors = *(TArray<AActor*>*)((uintptr_t)Level + 0xA0);
    for (int i = 0; i < Actors.Num(); i++) {
        AActor* Actor = Actors[i];
        if (!Actor) continue;
        // Check for: AResourceNode, AResourceField, AHarvestableInstancedMeshActor,
        //            ALootContainer, AItemContainer, ATemporaryLootContainer, AWorldItemBase
    }
}
```

## World-to-Screen (W2S) Projection

```cpp
bool WorldToScreen(FVector WorldLocation, FVector2D& ScreenLocation) {
    APlayerCameraManager* CameraManager = GetLocalPlayerCameraManager();
    FVector CameraLocation = CameraManager->GetCameraLocation();
    FRotator CameraRotation = CameraManager->GetCameraRotation();
    float FOV = CameraManager->GetFOVAngle();
    
    FMatrix RotationMatrix = FRotationMatrix(CameraRotation);
    FVector Forward = RotationMatrix.GetUnitAxis(EAxis::X);
    FVector Right = RotationMatrix.GetUnitAxis(EAxis::Y);
    FVector Up = RotationMatrix.GetUnitAxis(EAxis::Z);
    
    FVector Delta = WorldLocation - CameraLocation;
    FVector Transformed;
    Transformed.X = FVector::DotProduct(Delta, Forward);
    Transformed.Y = FVector::DotProduct(Delta, Right);
    Transformed.Z = FVector::DotProduct(Delta, Up);
    
    if (Transformed.X <= 0.f) return false;
    
    float FOVRad = FOV * PI / 360.f;
    float ScreenCenterX = ViewportWidth / 2.f;
    float ScreenCenterY = ViewportHeight / 2.f;
    
    ScreenLocation.X = ScreenCenterX + (Transformed.Y / Transformed.X) 
                       * (ScreenCenterX / tanf(FOVRad));
    ScreenLocation.Y = ScreenCenterY - (Transformed.Z / Transformed.X) 
                       * (ScreenCenterX / tanf(FOVRad));
    return true;
}
```

## Bone Access for Aimbot

```cpp
FVector GetBonePosition(ACharacter* Character, int BoneIndex) {
    USkeletalMeshComponent* Mesh = Character->Mesh;
    if (!Mesh) return FVector::ZeroVector;
    
    TArray<FTransform>& BoneTransforms = Mesh->BoneSpaceTransforms;
    if (BoneIndex >= 0 && BoneIndex < BoneTransforms.Num()) {
        FTransform& BoneTransform = BoneTransforms[BoneIndex];
        return BoneTransform.GetLocation();
    }
    return FVector::ZeroVector;
}

// Dune Awakening bone indices (557 total bones):
// 0   = Root
// 1   = pelvis         (center mass)
// 4   = spine_03       (body shot)
// 8   = neck_02        (aimbot secondary)
// 9   = head           (aimbot primary)
// 10  = clavicle_l
// 11  = upperarm_l
// 140 = clavicle_r
// 141 = upperarm_r
// 150 = hand_r         (weapon hand)
// 464 = thigh_r
// 497 = thigh_l
// 538 = wpn_root
// 544 = ik_hand_gun
//
// Component offsets:
// Mesh:          0x03D8
// CompToWorld:   0x160 (or 0x300 in later patches)
// BoneArray:     0x6C0 (or 0x6C8/0x6D8 in later patches)
```

## FName Reading

```cpp
const char* GetNameFromFName(FName Name) {
    uint32 ComparisonIndex = Name.ComparisonIndex;
    uint32 BlockIndex = ComparisonIndex >> 16;
    uint32 EntryIndex = ComparisonIndex & 0xFFFF;
    
    FNameEntry* Entry = GNames->Blocks[BlockIndex] + EntryIndex;
    return Entry->AnsiName;
}
```

## Core UE5 Structures

```cpp
// TArray layout
template<typename T>
struct TArray {
    T* Data;           // +0x00
    int32 Count;       // +0x08
    int32 Max;         // +0x0C
};

struct FVector { float X, Y, Z; };
struct FRotator { float Pitch, Yaw, Roll; };

struct FTransform {
    FQuat Rotation;      // +0x00 (16 bytes)
    FVector Translation; // +0x10 (12 bytes + 4 pad)
    FVector Scale3D;     // +0x20 (12 bytes + 4 pad)
};

// UObject base layout
// +0x00 VTable
// +0x08 ObjectFlags
// +0x0C InternalIndex
// +0x10 ClassPrivate (UClass*)
// +0x18 NamePrivate (FName)
// +0x20 OuterPrivate (UObject*)
```

## GEngine Chain (Alternative UWorld Access)

```cpp
// Instead of direct GWorld read (which changes every patch),
// traverse through GEngine for more stability:
//   Module Base -> GEngine -> GameViewport (+0xB70) -> World (+0x78)

uintptr_t GetWorldFromGEngine() {
    uintptr_t moduleBase = GetModuleBase();
    uintptr_t engine = Read<uintptr_t>(moduleBase + ENGINE_OFFSET);  // 0xBFC05B8 (Aug 2025)
    uintptr_t viewport = Read<uintptr_t>(engine + 0xB70);
    return Read<uintptr_t>(viewport + 0x78);
}
```

## Notes

Offsets change with every game update. The values above are from the UC Reversal thread across versions 1.1.0.11 through 1.4.10. Always verify against the current game version. See [[../Threads/UC-Reversal-Structs-Offsets]] for version-tracked offset history.

## See Also

- [[../Offsets/UE5-Offset-Guide]]
- [[../Threads/UC-Reversal-Structs-Offsets]]
- [[../Guides/UC-General-UE-Coding-Tips]]
- [[../Tools/UEDumper]]
- [[../Guides/UE5-Reversing-Methodology]]
