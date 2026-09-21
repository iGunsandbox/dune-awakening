# Dune Awakening Internal Cheat Architecture

> Decision document. Captures architecture choices based on research from UC, GitHub, ESP source analysis, and cross-project (Argus/sLix/Pred) infrastructure.
> Created: 2026-09-20

## Target

- **Game:** Dune: Awakening (Funcom)
- **Engine:** Unreal Engine 5.2.1
- **Anti-Cheat:** BattlEye (kernel-level, boot-start via BEDaisy.sys)
- **Process:** `DuneSandbox-Win64-Shipping.exe` (raw), `DuneSandbox_BE.exe` (BE wrapper)
- **Platform:** Windows (native laptop, not VM)
- **Game Version:** v1.5.3.1 (as of Sept 2026)

## Architecture: Argus-Style Internal

### Layer 1: BYOVD Kernel Driver
- **Driver:** AmdTools64.sys (shared with Argus/sLix/Pred)
- **Capabilities:** Physical memory R/W via IOCTLs, CR3 read
- **Risk:** Check if AmdTools64 is on BattlEye's driver blocklist (sLix may have been detected here)
- **Source:** `iGunsandbox/Argus/src/loader/byovd.rs`
- **Alternative drivers:** RamCaptureDriver64.sys (Argus also uses this)

### Layer 2: Mapped Driver (PIC Kernel Code)
- Position-independent code manually mapped into NonPagedPool
- No DriverEntry, no device object, no imports
- Communicates via shared memory page
- 4-level page table walk for VA-to-PA translation
- **Source:** `iGunsandbox/Argus/driver/mapped_driver.c`, `src/loader/mapper.rs`

### Layer 3: Anti-Forensics
- MmUnloadedDrivers cleanup
- PiDDBCache scrubbing
- BYOVD trace removal
- **Source:** `iGunsandbox/Argus/src/loader/cleanup.rs`

### Layer 4: Module Stomping Injection
- Overwrite .text section of a non-critical DLL in game process
- Candidate DLLs: enumerate loaded modules in DuneSandbox, exclude critical/AC DLLs
- Port from sLix (BattlEye-proven): `iGunsandbox/sLix/src/loader/stomp.rs`

### Layer 5: D3D11 Present Hook
- Shadow VMT on IDXGISwapChain::Present (vtable index 8)
- Temporal unhooking: restore original vtable between frames
- Runtime constant encryption (XOR with RDTSC-derived key)
- XOR-encoded string table (key=0x5A)
- Behavioral randomization (render interval 2-5 frames)
- **Source:** `iGunsandbox/Argus/driver/d3d11_hook_common.c` (~4000 lines)

### Layer 6: UE5 Entity Resolution
- Chain: Base (0x140000000) + UWORLD -> GWorld -> PersistentLevel -> Actors
- GNames chunked name pool for actor type identification
- FName NOT encrypted (confirmed Jul 2025)
- Type system from GodOfLife ESP: FName, TArray, UObject, FVector, TMap, FString
- **Offsets:** Need Dumper-7 run against v1.5.3.1

### Layer 7: ESP Rendering (via D3D hook)
- Player ESP: bounding box + name + alive/dead + distance
- NPC ESP: bounding box + type
- Spice Field ESP: type + status + distance
- Damage modifier: write f32 to weapon damage offset
- World-to-screen: standard UE rotation matrix projection

### Layer 8: HWID Protection
- SMBIOS spoofer: `iGunsandbox/Argus/src/loader/smbios_spoof.rs`
- BattlEye confirmed issuing hardware bans for Dune (OwnedCore threads bundle HWID spoofers)

### Layer 9: Launcher Integration
- Add Dune as target in `iGunsandbox/Launcher`
- Cleaner already handles BattlEye traces (`src/cleaner.rs`)

## Key Differences from Sister Projects

| Aspect | Argus (Rust) | sLix (R6 Siege) | Dune Awakening |
|--------|-------------|-----------------|----------------|
| Engine | Unity/IL2CPP | AnvilNext 2.0 | UE5 |
| Anti-cheat | EAC (demand-start) | BattlEye (boot-start) | BattlEye (boot-start) |
| Entity encryption | Heavy (XOR/ADD/SUB/ROL/ROR) | TLS-based XOR + DEC macro | None confirmed |
| Graphics API | D3D11 | D3D11 | D3D11 or D3D12 (verify) |
| Offset approach | Hardcoded | Sig-only | Start hardcoded, migrate to sigs |

## Open Questions

1. Is AmdTools64.sys blocklisted by BattlEye for Dune?
2. D3D11 or D3D12? If DX12, port from Pred instead of Argus
3. Module stomp candidates for DuneSandbox process
4. Does BE screenshot Dune? Affects rendering approach
5. Fresh offsets needed via Dumper-7 run with `-nobe`

## Development Phases

1. **Setup + Offsets** -- Windows laptop, Dumper-7 run, enumerate DLLs, verify D3D version
2. **Loader** -- Port BYOVD + manual mapper + module stomping + cleanup from Argus/sLix
3. **D3D Hook** -- Port Present hook from Argus (or D3D12 from Pred)
4. **UE5 Entity Pipeline** -- GWorld chain, GNames resolver, actor filtering (new code)
5. **ESP** -- Player/NPC/Spice boxes + names + distance via D3D rendering
6. **Damage Modifier** -- f32 write to weapon damage offset
7. **Hardening** -- SMBIOS spoofer, launcher integration, anti-screenshot, sig migration

## Reusable Component Map

| Component | Source | Path |
|-----------|--------|------|
| BYOVD driver | Argus | `src/loader/byovd.rs` |
| Physical memory R/W | Argus | `src/loader/physmem.rs` |
| Page table walk | Argus | `src/loader/pagetable.rs` |
| Manual mapper | Argus | `src/loader/mapper.rs` |
| Module stomping | sLix | `src/loader/stomp.rs` |
| Driver cleanup | Argus | `src/loader/cleanup.rs` |
| D3D11 hook | Argus | `driver/d3d11_hook_common.c` |
| D3D12 hook (if needed) | Pred | `pred-payload/src/hook/d3d12.rs` |
| SMBIOS spoofer | Argus | `src/loader/smbios_spoof.rs` |
| Thread hijack | Argus | `src/loader/threadhijack.rs` |
| Shared memory | Argus | `src/loader/shmem.rs` |
| Launcher + cleaner | Launcher | Full project |
| UE5 types (reference) | ESP Analysis | `vault/Code-Snippets/ESP-Source-Analysis.md` |
| BE detection research | sLix | `brain/Research/battleye-detection-surface-audit.md` |

## See Also

- [[../Code-Snippets/ESP-Source-Analysis]] -- GodOfLife ESP full source analysis
- [[../Threads/UC-Reversal-Structs-Offsets]] -- Offset history across game versions
- [[../Threads/UC-ESP-External-Linux]] -- External ESP thread + known issues
- [[../References/Anti-Cheat-BattlEye]] -- BattlEye reference
- [[../References/GitHub-Repos]] -- snapetech's DuneAwakeningSelfHost + UE4SSLinux
- [[../Tools/Cheat-Engine-Tables]] -- prometheu5Studios CE table (v1.5.3.1 AoB patterns)
