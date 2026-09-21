# Dune Awakening Research Vault

> Scraped 2026-09-19/20 from UnknownCheats, OwnedCore, Steam Community, GitHub, Cheat Engine forums, and gaming news outlets.

## Game Overview

- **Title:** Dune: Awakening
- **Developer:** Funcom
- **Engine:** Unreal Engine 5
- **Anti-Cheat:** BattlEye (kernel-level)
- **Platform:** Windows (released June 10, 2025), PS5/Xbox Series X|S (Sept 22, 2026)
- **Steam App ID:** 1172710
- **Genre:** Survival MMO (open-world, PvP/PvE)

## Vault Map

| Folder | Contents |
|--------|----------|
| [[Threads/]] | Forum threads from Steam, OwnedCore, and community discussions |
| [[Code-Snippets/]] | Technical code, CE scripts, and implementation notes |
| [[Offsets/]] | UE5 offset information and SDK dumping |
| [[Guides/]] | Reverse engineering guides and methodology |
| [[Tools/]] | Tools for UE5 reversing, CE tables, and SDK dumpers |
| [[References/]] | Developer statements, patch notes, and anti-cheat research |

## Key Findings

### UnknownCheats Status
- Dune Awakening threads are in **Other FPS Games** section on UC
- **Primary thread:** "Dune: Awakening Reversal, Structs and Offsets" (79 replies, 35k+ views, 4 pages) - see [[Threads/UC-Reversal-Structs-Offsets]]
- 167 total search results for "dune awakening" on UC (all 9 pages checked — only 9 are actual Dune threads, rest are false positives from other games)
- UC search is gated behind authentication - guest browsing shows thread titles only
- [Glitch]/[Help]/[Information]-tagged threads are access-restricted for low-rep accounts (3 threads inaccessible)

### UC-Scraped Technical Data
- **Current game version:** v1.5.3.1 (confirmed Sept 2026); latest UC SDK dump is outdated
- **Full release offsets** (GObjects, GNames, GWorld, ProcessEvent, BoneMatrix, etc.) across 8+ game versions
- **Complete 557-bone skeleton map** with key aimbot bones (head:9, neck_02:8, spine_03:4, pelvis:1)
- **10+ Dune actor class names** (ADunePlayerCharacter, ADuneNpcCharacter, AResourceNode, ASandwormPawn, etc.)
- **Resource spawner architecture** documentation (resources NOT in standard Actors array)
- **6 SDK dumps** with SHA256 hashes and UC file IDs
- **FName encryption status:** NOT encrypted as of July 2025
- **GEngine chain approach** as alternative to GWorld direct read
- **Anti-cheat telemetry warnings** (NetMessenger payloads on vehicle hijack)
- **External ESP source code** (Rust, Linux/Proton) with Spice Field, NPC, Player ESP + damage modifier — see [[Threads/UC-ESP-External-Linux]], full source analysis at [[Code-Snippets/ESP-Source-Analysis]]
- See [[Guides/UC-General-UE-Coding-Tips]] for compiled coding practices

### Anti-Cheat
- BattlEye kernel-level anti-cheat
- Linux/SteamOS/Proton compatible (Proton BattlEye Runtime required)
- Server trusts client for performance reasons - fundamental architecture weakness
- See [[References/Anti-Cheat-BattlEye]]

### Known Exploit Categories
1. **Resource Duplication** - Multiple dupe exploits found and patched (see [[Threads/Duplication-Exploits]], [[Threads/UC-Dupe-After-Patch]])
2. **Base Raiding in PvE** - NPC luring, permission spoofing (see [[Threads/Base-Exploits]])
3. **Vehicle Theft** - Permission system manipulation
4. **Combat Exploits** - Increased damage, insta-kill, rapid fire
5. **Repair Exploit** - Full durability restoration (see [[Threads/UC-Repair-Exploit]] — 83-post UC thread with full patch timeline)
6. **Deployable Duplication** - Massive UC thread, 1,381 replies, 8 exploit methods documented (see [[Threads/UC-Dupe-Exploit]])
7. **Solaris Currency Dupe** - Bank NPC respawn loophole (see [[Threads/UC-Currency-Loophole]])
8. **ESP/Damage Mod** - Linux external ESP + one-hit kill, Rust source code (see [[Threads/UC-ESP-External-Linux]])

### OwnedCore Status
- Dedicated Dune: Awakening forum with General + Buy/Sell/Trade subforums
- **26 threads found** (13 cheat software, 4 dupe exploits, 8 RMT/boosting, 1 announcement)
- Cloudflare Turnstile blocks automated scraping; content reconstructed from search index
- 10+ distinct cheat vendors identified (DEXAiMCHEATS, Pussycat, Dullwave, SMG, Arcane, BetterCheats, etc.)
- 4 commercial dupe exploit threads (Oct-Dec 2025) confirm exploits persisted post-patch
- HWID spoofer bundling confirms BattlEye hardware bans active
- See [[Threads/OC-Dune-Threads]] for full compendium

### Cheat Types Available
- ESP (player, NPC, loot, resource overlay)
- Aimbot (bone selection, smoothing)
- Speedhack
- Ghost mode / No clip
- Resource radar
- Auto-farm
- Teleport / TPKill (teleport-kill)
- HWID spoofer (hardware ban evasion)
- Vehicle hack
- Damage modifier / instant kill
- Cheat Engine tables (+28 options)

### Architecture Vulnerability
> "There is 0 security in the game as the server always trust the client for performance reasons."
> -- Steam Community discussion

## Sources

- Steam Community: `steamcommunity.com/app/1172710/`
- OwnedCore: `ownedcore.com/forums/dune-awakening/`
- Funcom Official: `duneawakening.com/news/developer-update/`
- UEDumper: `github.com/Spuckwaffel/UEDumper`
- Various cheat engine table sites
