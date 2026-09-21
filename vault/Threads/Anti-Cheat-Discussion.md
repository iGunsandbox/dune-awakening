# Anti-Cheat Community Discussion

> Compiled from Steam Community threads. 2026-09-19.

## Steam Thread: "Hacking and Cheating is Inevitable"
- Source: `steamcommunity.com/app/1172710/discussions/0/595153277396554007/`

### Original Claim
> "Hacking and cheating are inevitable in resource-based MMOs with real-money value. Even the industry giants cannot stop it."

### Key Discussion Points

#### Exploits vs Hacking
Community distinguishes between two problems:
1. **Code exploits** (developer responsibility) - "Bad Game Code" vulnerabilities
2. **Cheat injection** (player misconduct) - actual unauthorized modifications

> "A game that is vulnerable to Cheat Injection is a far different problem than the Developers leaving gaping holes in their Game Code."

#### Impact on Endgame
> "The landsraat will now be over within seconds if they so wish and not hours as it should be, the entire point of the endgame lies in ruins."

#### Developer Awareness
> "Go on discord and look in dune-awakening-general-0 the pinned post in there says they know about it and are working on it"

---

## Steam Thread: Anti-Cheat Implementation
- Source: `steamcommunity.com/app/1172710/discussions/0/595136312034469402/`

### Anti-Cheat Identified
- "It uses BattlEye" - confirmed by community
- Skepticism about effectiveness: "BattleEye is awful"

### Architecture Debate
- Game described as "always online" from the start
- Suggestion: doesn't need "world class anticheat" but proper server-side validation
- UE5 considered inherently vulnerable: "will be ruined by cheats within a matter of hours-days"

### Key Insight
The tension between **client-side anti-cheat** limitations and **server-side security** as the preferred defense for MMOs.

---

## Steam Thread: Kernel-Level Anti-Cheat Concerns
- Source: `steamcommunity.com/app/1172710/discussions/0/595150786260905871/`

### Security Concerns
- Anti-cheat developers gain full PC access
- Unlike OS companies, anti-cheat providers lack independent audits
- Kernel-level access enables more than file inspection

### Data Collection
- EAC (different AC, referenced for comparison) reportedly saves login credentials on servers for up to 3 months
- BattlEye's specific data practices not detailed

### Mitigations Discussed
- Dedicated gaming PC
- Linux with Proton
- Accepting the tradeoff for reduced cheaters

---

## Available Cheat Types (from Commercial Providers)

Based on OwnedCore and commercial cheat sites:

| Feature | Description |
|---------|-------------|
| ESP / Wallhack | See players, NPCs, loot through terrain |
| Aimbot | Auto-aim with bone selection and smoothing |
| Speedhack | Movement speed modification |
| Ghost Mode | Phase through terrain/structures |
| Resource Radar | Overlay showing spice blooms and rare resources |
| Auto-Farm | Automated resource gathering |
| Teleport | Instant position changes |
| Damage Modification | Increased damage / insta-kill |
| Rapid Fire | Projectile weapon fire rate increase |

### OwnedCore Threads
- "DUNE AWAKENING CHEAT | Aimbot, ESP, Speedhack" - DEXAiMCHEATS
- "Dune Awakening Hack | Aimbot, ESP, Undetected" - includes spoofer
- Multiple Buy/Sell/Trade listings

## Sources

- Steam: `steamcommunity.com/app/1172710/discussions/0/595153277396554007/`
- Steam: `steamcommunity.com/app/1172710/discussions/0/595136312034469402/`
- Steam: `steamcommunity.com/app/1172710/discussions/0/595150786260905871/`
- OwnedCore: `ownedcore.com/forums/dune-awakening/`

## See Also

- [[../References/Anti-Cheat-BattlEye]]
- [[Duplication-Exploits]]
- [[Base-Exploits]]
