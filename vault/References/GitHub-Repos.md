# GitHub Repositories Related to Dune Awakening

> Compiled from GitHub search. Last updated 2026-09-20.
> Internal project name: **DuneSandbox**
> Binaries: `DuneSandbox-Win64-Shipping.exe`, `DuneSandbox_BE.exe` (BattlEye), `DuneSandboxServer-Linux-Shipping`
> Engine: UE5.2 (CUE4Parse enum `GAME_UE5_2 + 6`)

---

## Repos with ACTUAL CODE

### Asset Parsing / Pak Reading

| Repo | Stars | Language | What It Does | Last Updated |
|------|-------|----------|--------------|--------------|
| `FabianFG/CUE4Parse` | 639 | C# | Canonical UE4/UE5 asset parser. Full `GAME_DuneAwakening` support: custom pak header reader (261-byte trailer with magic `0xA590ED1E`; standard UE magic `0x5A6F12E1` present but index offsets intentionally corrupted), Dune-specific struct deserialization (`DuneStructs.cs`), quirks in WorldComposition, StaticMesh, HISM, Materials, and RenderData parsing. Used by FModel. | 2026-09-20 |
| `adainrivers/uread2` | 0 | C# | Standalone pak reader. Dedicated `DuneAwakeningProfile` + `DunePakReader` implementing the same custom pak header format. Documents the 261-byte custom header layout: 40-byte custom block (magic + correct IndexOffset/IndexSize + hash), followed by 221-byte standard UE header (where IndexOffset/IndexSize are corrupted). Same author as `dune-dedicated-server-manager`. | 2026-01-25 |
| `the4rchangel/dune-awakening-server-manager` | 10 | JS/C# | Server manager that includes `tools/Cue4ParsePatents/` -- a CUE4Parse-based scanner that reads Dune paks via `GAME_DuneAwakening` to discover inventory template IDs not in the wiki catalog. | 2026-08-27 |

### Binary Analysis / Runtime Instrumentation

| Repo | Stars | Language | What It Does | Last Updated |
|------|-------|----------|--------------|--------------|
| `snapetech/DuneAwakeningSelfHost` | 9 | Python | Self-hosted server tooling with **extensive binary-level work**: binary patching (`patch-subfief-cap-binary.py` patches `je` to NOP in the totem placement path, signature-based, idempotent), PE signature analysis (`validate-client-pe-signatures.py`, `export-client-pe-signature-manifest.py`), client loader xref scanning (`summarize-client-loader-xrefs.py`), UE4SS porting (dozens of `ue4ss-package-*` scripts for source ABI recovery, vtable scanning, call-frame recovery, runtime trace), ELF analysis tools (`summarize-elf-ue-function-neighborhoods.py`, `summarize-elf-ue-relocation-surface.py`, writable root shape scanning). Also documents `DuneSandbox.*` UE class namespaces, DB schema, binary candidate offset recovery, and includes a `version.dll` Windows client probe system. ~38MB repo. | 2026-09-19 |
| `snapetech/UE4SSLinux` | 1 | C | Generic UE4SS-style Linux/Wine/Proton runtime loader framework. Ships a **Dune-specific profile** (`profiles/dune/profile.json`) with process filters (`DuneSandbox-Linux-Shipping`, `DuneSandboxServer-Linux-Shipping`, `DuneSandbox-Win64-Shipping.exe`), scan presets (`building`, `brt`, `deep-desert`, `gm`, `cheat`, `ue`), and Windows proxy DLL candidates (`version.dll`, `winmm.dll`, `dxgi.dll`; `version.dll` is the confirmed working proxy). | 2026-07-15 |

### GPU / Driver Quirks

| Repo | Stars | Language | What It Does | Last Updated |
|------|-------|----------|--------------|--------------|
| `HansKristian-Work/vkd3d-proton` | -- | C | Vulkan D3D12 translation layer. Has Dune-specific GPU quirks in `device_workarounds.c`: `dune_quirks` applied when process starts with `DuneSandbox`. Uses `VKD3D_STRING_COMPARE_STARTS_WITH` matching. | Active |

### Server Admin Tools (with technical reversing value)

These repos are primarily server administration tools but contain useful technical details: UE class paths under `/Script/DuneSandbox.*`, config key enumerations, database schemas, binary process names, and server architecture details.

| Repo | Stars | Language | Notable Technical Content |
|------|-------|----------|--------------------------|
| `Red-Blink/dune-awakening-selfhost-docker` | 49 | JS | `DuneSandbox/Saved/` paths, permission settings parser, Coriolis seed resolver, DLL override management |
| `adainrivers/dune-dedicated-server-manager` | 46 | Rust | Server management with config parsing |
| `coastal-ms/DST-DuneServerTool` | 26 | PS/C# | `DuneSoloDb` tool (interacts with game process), game config editor, solo-mode process detection |
| `AlphaNineGaming/AlphaNine-Dune-Suite` | 9 | JS | Full `DuneSandbox.*` settings enumeration (PvP, XP, fame multipliers, security zones, building, hydration) |
| `Manaiakalani/arrakis-command-nexus` | 6 | Python | Deep desert knobs, Coriolis storm config, resource respawn config sections |
| `n0logic/LastSietch` | 6 | Python | Canonical config documentation, PvP partition pinning, security zone config |
| `Icehunter/dune-admin` | -- | Go | k3s pod exec for `DuneSandbox/Saved/UserSettings`, server settings TypeScript constants |

---

## Process Name / Binary References (not game-specific tools)

These repos reference Dune binaries in their own tooling (overlays, performance tools, Linux compatibility):

| Repo | Purpose | Detail |
|------|---------|--------|
| `benjamimgois/goverlay` | Linux overlay manager | Launcher rewrite rule: `FuncomLauncher.exe` -> `DuneSandbox-Win64-Shipping.exe` |
| `CachyOS/ananicy-rules` | CPU scheduler rules | Process entries for `DuneSandbox-Win64-Shipping.exe`, `DuneSandbox_BE.exe`, `DuneSandbox.exe` |
| `CXWorld/CapFrameX` | Frame time capture | Process name `DuneSandbox-Win64-Shipping` -> display name "Dune: Awakening" |
| `N3oRay/proton-autogen` | Proton auto-config | Maps exe to "Dune Awakening, dx12, Unreal Engine 5" |
| `The-Hidden-Gaming-Lair/thgl-web-components` | Gaming overlay | Lists both `DuneSandbox-Win64-Shipping.exe` and `DuneSandbox-WinGDK-Shipping.exe` |
| `CubeCoders/AMPTemplates` | Server hosting panel | `duneawakeningconfig.json` with full `/Script/DuneSandbox.*` config schema |

---

## Confirmed STUBS (No Real Code)

Marketing pages, README-only repos, or placeholder files. No functional cheat/hack/tool code.

### Previously Confirmed

| Repo | Claimed Features | Reality |
|------|-----------------|---------|
| `eatchi69/dune-awakening-boost-toolkit` | ESP, automation, AI | Single README, no code |
| `swatcat-hub/dune-awakening-toolbox` | Cheats guide | Marketing page only |
| `Zainmalik242/DuneAwakeningToolkit` | Cheats guide 2025 | No implementation |
| `dune-awakening-cheat/.github` | Cheat toolkit | README redirect |

### Verified as Stubs (from unverified list)

| Repo | Claimed Features | Reality |
|------|-----------------|---------|
| `lk591-Dune-Awakening-ESP-hack/.github` | Wallhack, loot finder | **404 -- repo deleted or never existed** |
| `Dune-Awakening-ESP-imagal0/.github` | Player, loot, AI tracker | React SPA marketing page (minified JS/CSS/HTML). Zero game code. |
| `Dune-Awakening-Hack-dimples/.github` | ESP, aimbot, auto farm | **404 -- repo deleted or never existed** |
| `everguy-Dune-Awakening-Hack` | Full ESP, aimbot, ghost mode | **404 -- repo deleted or never existed** |
| `Dune-Awakening-Trainer/Trainer-Software` | Trainer software | README only, no code |

### Newly Found Stubs

| Repo | Claimed Features | Reality |
|------|-----------------|---------|
| `Stonedescode/Dune-Awakening-Helper-AI` | C++ hack: health, speed, ESP | `main.cpp` contains only `git add`/`git commit`/`git push` commands (language marker) |
| `ThrasherDevastate/Dune-Awakening-Max-Cheats-Hacks-Trainers` | C# trainer pack | `Project.cs` is `Console.WriteLine("Game Loaded"); Console.ReadKey();` -- trivial placeholder |
| `iTobagorMesugol/Dune-Awakening-SavageX` | C# aimbot + ESP | `.cs` file is a single `var` -- truncated placeholder. Plus pictures and README. |
| `hmongcodequest/Dune-Awakening-Trainer-Singleplayer` | God mode, infinite resources | README + 1 JPEG image only |
| `Dune-Awakening-Aimbot-c158/.github` | Silent aim, triggerbot | CSS/HTML marketing page |
| `eg752-Dune-Awakening-Aimbot/.github` | Silent aim, triggerbot | HTML marketing page |
| `nonamehorse8/DuneAwakening-ShadowTools` | Python/Java cheats | No language detected, guide-only |
| `mrshivam-in/DuneAwakening-ShadowTools` | Python/Java cheats | Fork of above, same empty content |
| `TRArcx/Dune-Awakening-v12.06` | Aimbot, ESP, unlimited spice | No language detected, README-only |
| `Distracted88/Dune-Awakening-adventure--v3` | God mode, ESP, fly hack | No language, README redirect |
| `SummitKhanOscillate/DuneSift` | 2026 hack | No language, README-only |
| `MrLediv/dune-awakening-enhancer-toolkit` | Cheats and hacks | No language, guide-only |
| `Fr1styy/dune-awakening-advantage-tools` | Tips and strategies | No language, guide-only |
| `taekgigi/dune-awakening-enhanced-menu` | Cheat menu | No language, guide-only |
| `natnat1230/dune-awakening-unlock-menu` | Unlock features | No language, guide-only |
| `Qawertus-spec/dune-awakening-toolset` | Hack tool guide | No language, guide-only |
| `tom4iks/dune-awakening-trainer-hub` | Trainer hub | No language, guide-only |

> **Pattern**: Every GitHub repo claiming Dune Awakening cheats, hacks, aimbots, or trainers is a stub.
> They contain either: (a) README marketing text with download links (likely malware), (b) a trivial
> placeholder file to set the repo language, or (c) a minified React/HTML marketing page. None contain
> functional game-interacting code.

---

## Negative Search Results

| Search | Result |
|--------|--------|
| UEDumper forks mentioning Dune | **None found**. No fork of `Spuckwaffel/UEDumper` references Dune Awakening. |
| Cheat Engine tables (.CT files) | **None found** on GitHub for DuneSandbox or Dune Awakening. |
| Hex offset patterns (e.g. `0xbcb2cb0`) | **None found** on GitHub. |
| `DuneSandbox` in C++ code | **None found** beyond vkd3d-proton device workarounds. |
| `dune awakening` in Lua scripts | **None found**. |
| UEDumper + Dune repos | **None found**. |
| Dune Awakening BattlEye bypass repos | **None found**. |

---

## UE5 SDK Dumping Tools (Applicable)

| Tool | Repo | Notes |
|------|------|-------|
| CUE4Parse | `FabianFG/CUE4Parse` | **Has explicit `GAME_DuneAwakening` support** -- custom pak parsing, struct quirks |
| UEDumper | `Spuckwaffel/UEDumper` | Best all-in-one, UE 4.19-5.4. No Dune-specific fork found. |
| UEDumperDMA | `dvGrab/UEDumperDMA` | DMA variant for AC games |
| UE5Dumper | `Jiang-Night/UE5Dumper` | UE5 specific |
| UE4-Dumper | `Device1337/UE4-Dumper` | Older but established |
| Unreal Finder Tool | `uuaing/Unreal-Finder-Tool` | UE4 info fetcher |
| AndUE4Dumper | `MJx0/AndUE4Dumper` | Android UE dumper |
| URead2 | `adainrivers/uread2` | **Has Dune-specific pak reader profile** |

---

## Technical Details Extracted

### Pak Header Format (from CUE4Parse + uread2)

```
261 bytes from end of .pak file:

-261: Custom Dune Header (40 bytes)
  - CustomMagic:  4 bytes  = 0xA590ED1E
  - IndexOffset:  8 bytes  (CORRECT -- use these)
  - IndexSize:    8 bytes  (CORRECT -- use these)
  - IndexHash:    20 bytes (SHA1)

-221: Standard UE Header (221 bytes)
  - EncryptionKeyGuid: 16 bytes
  - IsIndexEncrypted:  1 byte
  - StandardMagic:     4 bytes  = 0x5A6F12E1
  - Version:           4 bytes
  - IndexOffset:       8 bytes  (CORRUPTED -- don't use)
  - IndexSize:         8 bytes  (CORRUPTED -- don't use)
  - IndexHash:         20 bytes
  - CompressionMethods: 5 * 32 = 160 bytes
```

### Known DuneSandbox UE Class Sections

From server config analysis across multiple repos:

- `/Script/DuneSandbox.DuneGameMode` -- XP, fame, progression multipliers
- `/Script/DuneSandbox.DuneGameSettings` -- loot drop on death, etc.
- `/Script/DuneSandbox.PvpPveSettings` -- PvP partition settings
- `/Script/DuneSandbox.SecurityZonesSubsystem` -- Security zone enable/disable
- `/Script/DuneSandbox.BuildingSettings` -- Landclaim, foundation, staking
- `/Script/DuneSandbox.SandStormConfig` -- Coriolis storm settings
- `/Script/DuneSandbox.SandwormSettings` -- Sandworm behavior
- `/Script/DuneSandbox.SpiceHarvestingSystem` -- Spice fields, deep desert
- `/Script/DuneSandbox.HydrationSubsystem` -- Water/hydration system
- `/Script/DuneSandbox.ResourceLocationSystem` -- Resource spawning
- `/Script/DuneSandbox.PermissionSettings` -- Base permissions
- `/Script/DuneSandbox.InventorySystemSettings` -- Inventory
- `/Script/DuneSandbox.EncountersSubsystem` -- NPC encounters
- `/Script/DuneSandbox.TimeOfDaySettings` -- Day/night cycle
- `/Script/DuneSandbox.FlourSandSubsystem` -- Flour sand mechanics
- `/Script/DuneSandbox.DuneSandboxGameModeBase` -- Base game mode

### Known Process Names

| Binary | Platform | Purpose |
|--------|----------|---------|
| `DuneSandbox-Win64-Shipping.exe` | Windows | Main game client |
| `DuneSandbox-WinGDK-Shipping.exe` | Windows (GDK) | Xbox/Microsoft Store variant |
| `DuneSandbox-Win64-Test.exe` | Windows | Test client |
| `DuneSandbox_BE.exe` | Windows | BattlEye launcher wrapper |
| `DuneSandbox.exe` | Windows | Direct launch |
| `DuneSandboxServer-Linux-Shipping` | Linux | Dedicated server binary |
| `DuneSandbox-Linux-Shipping` | Linux | Linux client |
| `FuncomLauncher.exe` | Windows | Funcom launcher |

---

## See Also

- [[../Tools/UEDumper]]
- [[../Guides/UE5-Reversing-Methodology]]
