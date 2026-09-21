# UE5 Reversing Methodology for Dune Awakening

> General methodology for reversing UE5 games. 2026-09-19.

## Prerequisites

### Tools Required
- **Disassembler:** IDA Pro, Ghidra, or Binary Ninja
- **Debugger:** x64dbg, WinDbg
- **UE Dumper:** UEDumper (see [[../Tools/UEDumper]])
- **Memory scanner:** Cheat Engine (see [[../Tools/Cheat-Engine-Tables]])
- **DMA hardware** (optional, for BattlEye bypass): PCILeech-compatible FPGA

### Knowledge Required
- x86-64 assembly
- C++ reverse engineering
- Unreal Engine architecture (UObject, UWorld, actors)
- Windows internals (PE format, virtual memory, kernel drivers)

## Phase 1: Reconnaissance

### Identify Game Binary
1. Find the shipping executable (e.g., DuneAwakening-Win64-Shipping.exe)
2. Check PE headers for compiler info
3. Identify linked libraries (UE modules)
4. Check for obfuscation/packing

### Determine UE5 Version
1. Check game files for engine version strings
2. Look in Engine/Config/ or DefaultEngine.ini
3. Compare binary signatures against known UE versions
4. UEDumper can auto-detect some versions

### Anti-Cheat Assessment
- Dune Awakening uses BattlEye (kernel-level)
- BattlEye blocks: direct process memory access from usermode, known debugging tools, unsigned driver loading, common injection techniques
- Potential approaches: DMA hardware, kernel driver (signed or exploited), hypervisor-based reading

## Phase 2: SDK Dumping

### Using UEDumper
1. Find GNames offset (pattern scan in binary)
2. Find GObjects offset (pattern scan)
3. Configure UEdefinitions.h with UE version
4. Configure Offsets.h with found offsets
5. Run dumper against game process
6. SDK generated with all class layouts

### Manual Approach
1. Find GNames via string references ("None", "ByteProperty", etc.)
2. Find GObjects via cross-references from GNames usage
3. Walk UObject chain to enumerate classes
4. Extract property offsets from UField/FProperty chains

## Phase 3: Identify Key Structures

### For ESP
UWorld -> ULevel -> Actors array -> Filter by class (ACharacter, APawn, etc.) -> Read position from RootComponent->RelativeLocation -> Project to screen using camera matrix

### For Aimbot
ACharacter -> USkeletalMeshComponent -> GetBoneMatrix() -> Target bone (head, chest, etc.) -> Calculate aim angles from local player camera -> Apply smoothing

### For Resource/Item ESP
UWorld -> ULevel -> Actors -> Filter by resource/item classes -> Read item type, quantity, position -> Render overlay markers

## Phase 4: Implementation Approaches

### External Overlay
- Separate process renders overlay on top of game
- Reads game memory for positions
- Uses Direct3D/Vulkan for rendering
- Less invasive but still detectable via memory reads

### Internal Hook
- Inject DLL into game process
- Hook D3D11/D3D12 Present for rendering
- Direct access to game memory (no RPM overhead)
- More features possible but higher detection risk

### DMA-Based
- Hardware reads physical memory
- Second PC processes and renders
- Game-side is read-only (no writes)
- Hardest to detect but requires hardware

## Dune Awakening Specific Considerations

1. **BattlEye** makes usermode approaches risky
2. **Always-online** means server may validate some values
3. **Client-trusted architecture** means many values ARE client-authoritative
4. **UE5** is well-documented, reducing reversing difficulty
5. **Regular patches** mean offsets break frequently (use AoB patterns)

## See Also

- [[../Tools/UEDumper]]
- [[../Offsets/UE5-Offset-Guide]]
- [[../References/Anti-Cheat-BattlEye]]
- [[../Code-Snippets/UE5-Common-Patterns]]
