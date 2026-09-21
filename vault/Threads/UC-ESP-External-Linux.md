# [UC] Dune Awakening Simple External ESP Base (Linux)

> Source: unknowncheats.me/forum/other-fps-games/710801-linux-dune-awakening-simple-external-esp-base.html
> Thread by: GodOfLife (Feb 2022, 31 posts, rep 284)
> 10 replies, 2,269 views
> Forum: Other FPS Games
> Scraped: 2026-09-20

## Overview

External ESP cheat for Dune Awakening, Linux only (Proton/Wine). Written in Rust.

### Features
- Spice Field ESP — shows active spice fields
- NPC ESP
- Player ESP
- Weapon Damage Modifier — one-hit kill all NPCs

### Requirements
- Linux with Proton/Wine
- Root access (runs `./Protect` first to hide root processes and change MAC address)

## Download

- **SHA256:** `39f98fbaf2ff592b8c3a524eca136006e4b4b0d88b50e3ea1f0e7b438eeb1b96`
- **Filename:** `dune-external.zip`
- **UC File ID:** 50549
- **Approved by:** electrolux (Former Staff, rep 41630)

## Credits

- `Gerosity/Apex-Protection` — process hiding
- Wine/Proton game injection technique

## Known Issues (as of Mar 2026)

The tool is **outdated** — offsets have changed since Jul 2025. Two users report the same crash:

```
GObjects FChunkedFixedUObjectArray { objects: 0x42dda63042dda5d, ... }
Processing 70114572 chunks with 181963148 total elements

thread 'main' panicked at src/ui/overlay.rs:63:71:
called `Result::unwrap()` on an `Err` value: Errno(14)
```

**Root cause (S4g3l0rd55, #10):** The UWorld offset `self.process.read(self.uworld + 0x2c8)` is no longer valid after game updates. Needs updated offsets from [[UC-Reversal-Structs-Offsets]].

## Technical Notes

- External approach — reads process memory from outside, no injection
- Uses `/proc/` filesystem for memory access (Linux-specific)
- Rust implementation with overlay rendering
- Protontricks used for launching alongside game

### Questions from Thread
- **twiztedkarrtoon (#8):** "what constant people are using to find the uworld, or if there is an easier way" — no answer provided
- **mrcarpetabr (#9):** Asked how to dump the game with ProcessHacker — no answer provided

## See Also

- [[../Code-Snippets/ESP-Source-Analysis]] — Full source code analysis (architecture, memory chain, offsets, bugs, reusability)
- [[UC-Reversal-Structs-Offsets]] — Offsets needed to update this tool
- [[UC-Dupe-Exploit]] — GodOfLife also posted auction house dupe method
- [[../Tools/UEDumper]] — SDK dumping tools
- [[../References/Anti-Cheat-BattlEye]] — BattlEye bypass context
