# Cheat Engine Tables for Dune Awakening

> Compiled from CE table sites, WeMod, Cheat Happens, FLiNG, and forum threads. Updated 2026-09-20.

## Overview

Cheat Engine tables (.CT files) exist for Dune Awakening's single-player mode. The primary table by prometheu5Studios offers 20+ options across combat, mobility, progression, and miscellaneous categories. These use Lua-based activation checks, AoB pattern scanning, and Auto Assemble code injection to modify game values at runtime.

Single-player mode launched September 17, 2026. The game executable `DuneSandbox-Win64-Shipping.exe` can be started directly (bypassing BattlEye) from `DuneAwakening/DuneSandbox/Binaries/Win64/` instead of the BattlEye-wrapped `DuneSandbox_BE.exe`.

---

## 1. prometheu5Studios Table (FearLess Revolution) -- PRIMARY

The most complete and actively maintained free CE table.

| Field | Value |
|-------|-------|
| Author | prometheu5Studios (joined FearLess June 2026) |
| Thread | `fearlessrevolution.com/viewtopic.php?t=40849` |
| Request thread | `fearlessrevolution.com/viewtopic.php?t=40825` (by Sxsxarael, Sept 18 2026) |
| Current version | v1.1 |
| Game build targeted | Steam v1.5.3.1 (v1.0 targeted v1.5.3.0) |
| CE required | 7.7 or newer |
| File size | v1.1: 264.85 KiB (file ID 81777) / v1.0: 237.85 KiB (file ID 81747) |
| Downloads | v1.0: 855 / v1.1: 217 (as of Sept 20 2026) |
| Cost | Free |
| SP only | Yes -- launches via Shipping.exe to bypass BattlEye |

### Complete Feature List

**Combat (9 options):**
- God Mode (includes radiation protection in v1.1)
- Infinite Ammo
- Rapid Fire (4x multiplier)
- Instant Missile Lock-On
- Instant Ability Cooldowns
- Infinite Stamina
- Infinite Water (added v1.1)
- Third Ability Slot / Full Spice (added v1.1)
- Infinite Power

**Mobility (4 options):**
- Adjustable Run Speed
- Infinite Vehicle Fuel
- No Vehicle Overheating (added v1.1)
- Adjustable Ornithopter Speed

**Progression (7 options):**
- Free Building
- Free Crafting
- Set Nearby Base Power to 99,999 (added v1.1)
- Unlock All Research (includes unique-item recipes and story-locked schematics)
- Set Skill Points
- Repair Gear
- Freeze Day/Night cycle
- Set Time of Day

**Miscellaneous (3 options):**
- Fast Scanner with Extended Range (testing/experimental)
- Add Item to Backpack (with item name and quantity selection)
- Sandworms Ignore Player (limited -- thumpers and compactors still attract worms)

### v1.1 Changelog (Sept 20 2026)

Fixed for game build 1.5.3.1:
- Research unlock updated and tested on fresh characters
- Item spawning (Add Item to Backpack) restored
- Free Building mechanics updated
- Ammo/Rapid Fire systems updated
- Time controls updated
- Added: Infinite Water, radiation protection in God Mode, Third Ability Slot, No Vehicle Overheating, 99,999 Base Power

### Known Issues

- Free Crafting does NOT eliminate power requirements for fabricators
- Research menu must be closed and reopened after activating Unlock All Research
- Item changes, research unlocks, and skill modifications persist in save files -- backup saves first
- Sandworm avoidance does not protect against thumper/compactor attraction
- Fast Scanner requires scanner equipment fitted; extended range may miss targets
- Free Building may still require materials for some items despite activation
- Some unique items may retain permit requirements

### Technical Details

- Uses Lua-based activation checks with version validation
- Displays error "actions need Dune 1.5.3.0" on version mismatch
- .CT files contain AoB patterns and Auto Assemble injection scripts
- Must run CE as Administrator if attachment fails
- Load game world BEFORE attaching CE to process
- Base power changes may require disabling/re-enabling sub-fiefs to take effect

### Setup Instructions

1. Start `DuneSandbox-Win64-Shipping.exe` directly from `DuneAwakening/DuneSandbox/Binaries/Win64/` (NOT `DuneSandbox_BE.exe`)
2. Load your single-player world
3. Open the .CT file in Cheat Engine 7.7+
4. Click the computer icon and select `DuneSandbox-Win64-Shipping.exe` process
5. If CE cannot attach, right-click Cheat Engine and Run as Administrator
6. Tick "Activate Cheats" and enable desired options

---

## 2. WeMod -- NOT AVAILABLE

| Field | Value |
|-------|-------|
| URL | `wemod.com/cheats/dune-awakening-trainers` |
| Status | "Coming soon" on page, but actually declined |
| Cost | N/A |

WeMod has a page for Dune: Awakening that displays "Coming soon" and prompts users to download WeMod to be notified. However, the game is actually **unsupported**.

**Official WeMod response** (community.wemod.com/t/dune-awakening/378581): Patrick from WeMod stated: "Dune still has an active anticheat system in place. Therefore we are currently unable to offer support for it."

The playtest version (`wemod.com/cheats/dune-awakening-playtest-trainers`) is also marked "Unsupported."

WeMod does not support games with active anti-cheat (BattlEye) even when a single-player mode exists, because the anti-cheat remains active in the executable. The prometheu5Studios CE table works around this by launching the non-BE executable directly.

---

## 3. Cheat Happens -- RETIRED

| Field | Value |
|-------|-------|
| URL | `cheathappens.com/81773-PC-Dune-Awakening-trainer` |
| Status | RETIRED |
| Features | None |

Cheat Happens evaluated Dune: Awakening and marked it as RETIRED: "a trainer was not possible or the game is multiplayer/online only."

As of September 2026, a forum post noted "Single Player mode dropped today," potentially reopening evaluation. Cheat Happens offers their free CoSMOS memory-scanning tool as a self-service alternative.

---

## 4. FLiNG Trainer

| Field | Value |
|-------|-------|
| URL | `flingtrainer.us/dune-awakening-trainer/` |
| Options | 6+ (labeled "+10" on page title) |
| Game version | 1.5.3.0 |
| Cost | Free |
| Updated | September 20, 2026 |
| SP only | Yes |

**Listed features:**
- Unlimited HP/MP
- One-Hit Kill
- Mega EXP
- Max Gold
- No Skill Cooldown
- Infinite Items

Hotkeys: Numpad 0-5, F4 for options menu. File size 70-352 KB depending on version. Windows only, run as Administrator.

> **Reliability note:** FLiNG trainer sites often use generic feature descriptions that may not accurately reflect the specific game's mechanics. The feature names (e.g., "Max Gold," "Mega EXP") do not map to known Dune Awakening systems. Treat with skepticism compared to the prometheu5Studios table which uses game-accurate terminology.

---

## 5. FearLess Cheat Engine (fearlesscheatengine.net)

| Field | Value |
|-------|-------|
| URL | `fearlesscheatengine.net/dune-awakening-cheat-engine/` |
| Options | "9+ options" (unspecified) |
| Trainer version | v2.051411753883878 |
| Hotkeys | F1-F12 for cheats, F6 for settings |
| Downloads | 11,628 |
| Release date | 16.09.2026 |
| Claimed compatibility | "all game versions" |

This site does NOT list specific features despite claiming 9+ options. The version string `v2.051411753883878` appears auto-generated. No anti-cheat warnings provided.

> **Reliability note:** This appears to be a content-farm mirror site, not an original source. The download count (11,628) seems inflated for a table released days prior.

---

## 6. MehTrainer

| Field | Value |
|-------|-------|
| URL | `mehtrainer.com/dune-awakening-trainer/` |
| Options | 20 |
| Version | 1.5.3.0 |
| Format | .CT file by prometheu5Studios |
| File size | 38.1 KB |
| Cost | Free (Google Drive download) |

This is a redistribution of the prometheu5Studios v1.0 table (see section 1). Same 20 features. Single-player only.

---

## 7. ForgeCheats (paid, online-targeting)

| Field | Value |
|-------|-------|
| URL | `forgecheats.com/en/game/dune-awakening/` |
| Type | Paid subscription cheat software |
| Cost | $30 USD / 390-400 RUB per day |
| Anti-cheat | Claims "Complete bypass of BattlEye" with HWID protection |
| Status | Claims "Undetected" as of Dec 30, 2025 |

Four products listed: SHACK ($30), SMG (400 RUB), DULLWAVE (390 RUB), PUSSYCAT (400 RUB).

**Claimed features:**
- Aimbot / Silent Aim with movement prediction and bone selection
- ESP / Wallhack (enemies, bases, vehicles with name/faction/HP)
- Loot ESP (resources, spice, water, rare minerals)
- Vehicle hacks (ornithopter speed, no-clip, sandworm tracking)
- Survival bypasses (no thirst/heat debuffs)
- Auto-craft, camera distance manipulation, config save/load

> **Warning:** This is a commercial cheat vendor. Claims of being "undetected" and "BattlEye bypass" are unverified promotional material. These target online multiplayer, not single-player. High ban risk.

---

## 8. Other Sources (inaccessible or minimal)

| Site | URL | Status |
|------|-----|--------|
| Escape Dynamics | `escapedynamics.com/viewtopic.php?t=96545` | HTTP 403 -- Cloudflare blocked |
| Cheat Engine Net | `cheatengine.net/viewtopic.php?t=97508` | HTTP 403 -- Cloudflare blocked |
| Games-Manuals | `games-manuals.com/cheat-table-cheat-engine/dune-awakening-1172710` | HTTP 503 -- service unavailable |
| CheatBook.de | `cheatbook.de/files/duneawakening.htm` | Game guides/tips only, no trainers |

---

## How CE Tables Work with UE5

### Scannable Values
- Health, stamina, water
- Resource counts
- Skill points
- Ammunition / durability

### Data Types
- Most numerical values: 4 Bytes
- Floating point values (health %): Float
- Some values may use Double or 8 Bytes

### Memory Techniques
1. **Value Scanning:** Search for known value, change in-game, scan for new value
2. **Pointers:** Stable references that survive game restarts
3. **Array of Bytes (AoB):** Pattern scanning for code injection points -- more resilient across updates
4. **Auto Assemble Scripts:** Code injection to freeze or modify values
5. **Lua Scripts:** Version validation, conditional activation, UI integration within CE

### Why the Non-BE Executable Matters

The game ships two executables:
- `DuneSandbox_BE.exe` -- launches BattlEye then the game; CE cannot attach
- `DuneSandbox-Win64-Shipping.exe` -- the raw game binary without anti-cheat wrapper

CE tables work by launching the Shipping.exe directly for single-player. This is NOT an anti-cheat bypass -- it simply runs the game without the anti-cheat layer, which single-player mode permits.

---

## Anti-Cheat Warnings

> "Funcom is actively on the hunt for people running scripts like these, so use it at your own peril."

- BattlEye actively scans for Cheat Engine in the online executable
- CE tables are detectable when used in online mode
- Server-side validation may catch impossible values
- Ban risk is high for online use
- WeMod explicitly declined support due to active anti-cheat
- Cheat Happens retired the title for the same reason
- Item/research/skill modifications persist in save files -- changes made with CE carry into saves

---

## Limitations

- Addresses break with every game update; AoB patterns more resilient but still version-dependent
- Technical details (addresses, pointers, scripts) are inside the .CT files themselves
- Many CE table mirror/redistribution sites are content farms with inflated stats
- FLiNG and similar generic trainer sites may use feature names that do not match actual game mechanics
- The prometheu5Studios table on FearLess Revolution is currently the only verified, actively-maintained table with game-accurate feature descriptions

---

## See Also

- [[UEDumper]]
- [[../Offsets/UE5-Offset-Guide]]
- [[../References/Anti-Cheat-BattlEye]]
