<div align="center">
  <img src="https://capsule-render.vercel.app/api?type=waving&color=gradient&customColorList=12&height=220&section=header&text=Hazedotx&fontSize=70&fontAlignY=35&desc=Roblox%20Gameplay%20%26%20Systems%20Engineer&descSize=20&descAlignY=58&animation=fadeIn" alt="Hazedotx banner" />
</div>

<p align="center">
Lead programmer on Roblox games with <b>5M+ visits</b> and up to <b>3,000 concurrent players</b>.<br>
I build server-authoritative gameplay, networking, and data systems in Luau that hold up against exploiters at scale.
</p>

<p align="center">
<img src="https://img.shields.io/badge/Visits-5M%2B-2ea44f?style=for-the-badge"/>
<img src="https://img.shields.io/badge/Peak_CCU-3%2C000%2B-1f6feb?style=for-the-badge"/>
<img src="https://img.shields.io/badge/Revenue-Low_Five_Figures-8957e5?style=for-the-badge"/>
<img src="https://img.shields.io/badge/Reported_Dupes-0-d73a49?style=for-the-badge"/>
</p>

<p align="center">
<img src="https://img.shields.io/badge/Luau-2C2D72?style=for-the-badge&logo=lua&logoColor=white"/>
<img src="https://img.shields.io/badge/Roblox_Studio-00A2FF?style=for-the-badge&logo=robloxstudio&logoColor=white"/>
<img src="https://img.shields.io/badge/Python-3776AB?style=for-the-badge&logo=python&logoColor=white"/>
<img src="https://img.shields.io/badge/Java-ED8B00?style=for-the-badge&logo=openjdk&logoColor=white"/>
<img src="https://img.shields.io/badge/C%23-512BD4?style=for-the-badge"/>
<img src="https://img.shields.io/badge/Unity-222222?style=for-the-badge&logo=unity&logoColor=white"/>
<img src="https://img.shields.io/badge/Git-F05032?style=for-the-badge&logo=git&logoColor=white"/>
</p>

<p align="center">
<a href="#into-the-backrooms-tower-defense">Backrooms TD</a> •
<a href="#killspree">Killspree</a> •
<a href="#other-roblox-work">Other Roblox Work</a> •
<a href="#other-projects">Other Projects</a> •
<a href="#skills">Skills</a>
</p>

---

## Featured Projects

| Into the Backrooms Tower Defense | Killspree |
| :---: | :---: |
| [![Backrooms TD showcase](https://img.youtube.com/vi/yrDP3b7QeC4/mqdefault.jpg)](https://www.youtube.com/watch?v=yrDP3b7QeC4) | [![Killspree showcase](https://img.youtube.com/vi/0ZLOrN9r6tU/mqdefault.jpg)](https://www.youtube.com/watch?v=0ZLOrN9r6tU) |
| Tower defense · Lead Programmer<br>5M+ visits · Top 250 earning game · Front page 1+ month | Multiplayer killer game · Lead Programmer<br>Killer abilities · Hitboxes · Rollback netcode |
| [![Play](https://img.shields.io/badge/Play-00A2FF?style=flat-square&logo=roblox&logoColor=white)](https://www.roblox.com/games/123446017566246/Into-the-Backrooms-TD) [![Writeup](https://img.shields.io/badge/Writeup-8957e5?style=flat-square)](#into-the-backrooms-tower-defense) | [![Play](https://img.shields.io/badge/Play-00A2FF?style=flat-square&logo=roblox&logoColor=white)](https://www.roblox.com/games/82851129405631/KILLSPREE) [![Writeup](https://img.shields.io/badge/Writeup-8957e5?style=flat-square)](#killspree) |

> Production source code for these games is private. These writeups and showcase videos describe the systems I designed and built.

---

## Into the Backrooms Tower Defense

**Role:** Lead Programmer · [Play on Roblox](https://www.roblox.com/games/123446017566246/Into-the-Backrooms-TD)

**5M+ visits** · **Low five-figure revenue** · **Top 250 earning game on Roblox** · **Front page for over a month** · **600K+ hours played** · **130K-member community**

[![Backrooms TD showcase](https://img.youtube.com/vi/yrDP3b7QeC4/0.jpg)](https://www.youtube.com/watch?v=yrDP3b7QeC4)

*9-minute showcase covering gameplay, tower and enemy logic, and networking.*

Aside from two or three minor exceptions, I built everything under the hood: all backend systems, all networking architecture, and all frontend UI code, across a lobby place and a match place.

### Architecture

- **Two-place structure.** The lobby handles progression (summoning, trading, quests, clans, rewards) and the match place handles gameplay. Players queue on elevator pads and teleport into a match, and a join-data service carries their lobby context into the match server.
- **Services vs. Systems.** Services each own one domain of player state (currency, tower inventory, item inventory, datastore) and expose a clean API. Systems implement features by composing services; trading, for example, works through the inventory and currency services rather than touching player data directly. This keeps ownership of data clear as the codebase grows.
- **Shared packages.** Core modules are shared as packages across both places, so the lobby and match servers always run the same data and networking code.
- **Behavior modules.** Every tower, enemy, map, and in-match summon is its own behavior module plugged into a shared interface, with example templates for new content. Adding a tower means writing one module, not editing the core tower system.
- **Datastore health gating.** The data layer tracks datastore health, and critical operations (like finalizing a trade) check it before committing.

### Trading System

The piece I'm proudest of. Item duplication ("duping"), where a trade leaves someone with a copy of an item, is the most common exploit in games like this. The trading system prevents it through several layers:

- **State-gated trade objects.** Each trade tracks `Active`, `Processing`, or `Complete`, and every client action (offer, confirm, cancel) is gated by that state, so nothing can change mid-settlement.
- **Confirmation re-arming.** Any offer change after confirming un-confirms both sides, preventing last-second swaps.
- **Atomic settlement with rollback.** Both inventories are snapshotted before anything moves. Items are removed from both sides before anything is granted; if any step fails, both snapshots are restored and related counters are diffed back to their prior state.
- **Unique item IDs.** Every item carries its own ID, so any duplicate would be detectable and traceable.
- **Session locking during trades.** Player data stays locked until a trade fully settles, even if a player leaves mid-trade, so a settlement can't be cut off by a save or a server hop.
- **Datastore-health gating.** Trades refuse to finalize if the datastore is in a critical or closing state.
- **Shutdown safety.** In-progress trades get a grace window on server shutdown and player leave, so a settlement isn't interrupted mid-write.

**Result:** backed by a **$300 bounty** at launch for anyone who could produce a working dupe. Five experienced exploiters tried and failed, and there have been **zero reported dupes across hundreds of thousands of trades** since release.

### Distributed Leaderboard System

A cross-server leaderboard for endless mode that tracks up to **20,000 ranked players**, resets on a season timer, and pays out rewards to top ranks at season end. The core challenge: many servers run in parallel with no shared memory, and only the datastore ties them together.

- **Config-driven leaderboards.** Each leaderboard defines its own player cap, season length, lock duration, and which server types are allowed to perform the expensive rebuild.
- **Single-writer locking.** A server must claim a lock in the datastore before rebuilding. If it stalls, the lock expires and another server takes over; if the original server finishes late anyway, its write is rejected because it no longer holds the lock.
- **Failed reads never overwrite good data.** A partial or failed datastore read is discarded instead of saved, so a transient failure can't blank the board for everyone.
- **Reads decoupled from builds.** Any server can serve a cached leaderboard regardless of which server built it, and concurrent requests share a single read instead of hitting the datastore once per player.
- **Rate-limit-aware pacing.** Rebuilds check the live request budget instead of running on a fixed timer, staying fast under normal load and backing off only under real contention.

### Quest System

Manages daily, weekly, lifetime, and clan quests per player. Every quest type shares one interface (`isComplete`, `title`, `textGoal`, `textProgress`, `percentProgress`), so new quest types drop into the pool without touching the core service.

- **Refresh-in-progress guards.** Overlapping refresh calls for the same player (from the async data layer) are blocked from running concurrently, preventing duplicate quest batches.
- **Self-healing corruption detection.** A player missing a daily or weekly quest gets that category regenerated automatically.
- **Anti-farming caps.** Clan quests draw from a shared pool with per-type caps, so a player's active slots can't all roll the same low-effort quest.
- **Periodic sweep.** A background loop re-checks players and tops up missing slots, so one dropped event can't leave a player without quests for a session.
- **Data-driven reward scaling.** Each category has its own difficulty range and reward rate, so harder rolls pay out proportionally more. Lifetime quests add a small, similarly scaled chance at rare items.
- **Decoupled reward calculation.** What a quest *will* reward is computed separately from granting it, so the UI can preview rewards without an active quest handler.

### Other Systems

<details>
<summary><b>Tower & enemy gameplay</b></summary>

- **10+ towers** built as behavior modules on a shared interface, including Commander, Necromancer, Pyromancer, Cryomancer, Electrician, Trapper, King, and Construction Worker
- **Enemy system** with special-ability behaviors, such as enemies that spawn others on death and disguised enemies
- **Targeting utilities** for tower targeting modes
- **Attack shapes** defining the areas towers hit
- **Pathing utilities** for enemy movement along lanes
- **Status effects** such as burns, slows, and freezes
- **Traps, walls, and tower-summoned units** placed during a match
- **Waves and Endless mode**, with wave analytics used for balancing
- **Map behaviors** adding mechanics unique to each level
- **Match flow:** game run lifecycle, adjustable game speed, in-match cash, base health, and end-of-match results and rewards

</details>

<details>
<summary><b>Summoning</b></summary>

- **Deterministic cross-server banners.** The rotating banner is generated from an RNG seeded by a time bucket, so every server produces the identical banner and rotates at the same moment with no cross-server messaging.
- **Odds verification harness.** A debug tool simulates hundreds of thousands of rolls and reports the real rarity and per-tower distribution, confirming the actual odds match the configured ones.
- Weighted rarity rolls, pity, stacking luck (server events, player boosts, gamepasses), scripted first rolls for new players, and auto-delete for unwanted rarities
- Server-authoritative validation on every summon: debounce, trade lock, allowed summon amounts, affordability, and inventory space

</details>

<details>
<summary><b>Economy & progression</b></summary>

- Currency, rewards, item and tower inventories, and selling
- Mutations and potions
- Daily and playtime rewards
- Redeemable codes
- Gamepasses and Robux purchases

</details>

<details>
<summary><b>Social</b></summary>

- Clans with clan quests
- Trading (see above)
- Cross-server leaderboards (see above)

</details>

<details>
<summary><b>Lobby & player experience</b></summary>

- Elevator matchmaking and lobby-to-match data handoff
- Onboarding in both the lobby and the match
- Dialogue, notifications, sound, and settings
- Interactive lobby objects

</details>

<details>
<summary><b>Data & tooling</b></summary>

- Datastore layer with health states
- Wave analytics for balancing
- Playtime tracking
- Developer tools for testing

</details>

---

## Killspree

**Role:** Lead Programmer · [Play on Roblox](https://www.roblox.com/games/82851129405631/KILLSPREE)

[![Killspree showcase](https://img.youtube.com/vi/0ZLOrN9r6tU/0.jpg)](https://www.youtube.com/watch?v=0ZLOrN9r6tU)

*23-minute showcase covering gameplay, Killer abilities, combat, and networking.*

I was the lead programmer on Killspree and built the Killers and their abilities shown throughout the showcase, along with the core gameplay and networking systems underneath them.

- **Killer abilities.** Programmed each Killer character and its ability kit.
- **Hitbox system.** Built the hitbox system used for combat and gameplay interactions.
- **Rollback netcode.** Implemented rollback netcode using interpolated snapshot buffers for lag compensation and state reconciliation, making gameplay responsive and smooth, especially when playing as the Killer.
- **Networked gameplay.** Built the underlying systems that keep the showcased gameplay consistent across clients in a multiplayer environment.

---

## Other Roblox Work

- **Server-authoritative codebases** for two studios supporting **1,000+ and 3,000+ concurrent players**, including server-client communication layers built from the ground up.
- **Systems and gameplay features** for Star Studios IX, Critical Tower Defense, and The Collective Experiment.
- **Project management:** coordinated development timelines and feature releases for the teams I led.
- **Code reviewer:** one of 10 script rankers in a 119K+ member Roblox development community, auditing code and flagging edge cases in live-service systems. Recognized as a top performer and offered the lead ranker role.
- **Growth:** helped grow games through a content network with 20M+ followers, contributing to 175M+ total visits.

---

## Other Projects

### Procedural Dungeon Crawler (Python)

[View the repo](https://github.com/Hazedotx/My-Python-Project)

A 2,000+ line dungeon crawler built as a term project for Carnegie Mellon's 15-112.

- **Procedural generation.** Dungeons are generated with binary space partitioning, which recursively splits the map into chunks, places rooms, and connects them with corridors.
- **Exploration.** A fog-of-war overworld reveals the map as you explore, with a chance of stumbling into combat dungeons of varying difficulty.
- **Combat.** Multiple weapons with their own logic, enemies with idle, walk, attack, damage, and death animations, and health bars.
- **Rendering performance.** Layered tile sprites are baked into single pre-rendered images with Pillow, cutting 200+ draw calls down to one so the game stays stable.

### VR Firefighter Training Simulation (Unity / C#)

Built from scratch during a university summer research program: a VR training scenario for firefighters, designed around a literature review on VR safety training.

---

## Skills

| Category | |
| :--- | :--- |
| **Languages** | ![Luau](https://img.shields.io/badge/Lua%2FLuau-2C2D72?style=flat-square&logo=lua&logoColor=white) ![Python](https://img.shields.io/badge/Python-3776AB?style=flat-square&logo=python&logoColor=white) ![Java](https://img.shields.io/badge/Java-ED8B00?style=flat-square&logo=openjdk&logoColor=white) ![C#](https://img.shields.io/badge/C%23-512BD4?style=flat-square) |
| **Tools** | ![Roblox Studio](https://img.shields.io/badge/Roblox_Studio-00A2FF?style=flat-square&logo=robloxstudio&logoColor=white) ![Unity](https://img.shields.io/badge/Unity-222222?style=flat-square&logo=unity&logoColor=white) ![Git](https://img.shields.io/badge/Git-F05032?style=flat-square&logo=git&logoColor=white) ![Linux](https://img.shields.io/badge/Linux-FCC624?style=flat-square&logo=linux&logoColor=black) |
| **Systems** | Server-authoritative architecture · Real-time networking & rollback netcode · Datastore design & data integrity · Cross-server coordination · Anti-exploit design |

<div align="center">
  <img src="https://capsule-render.vercel.app/api?type=waving&color=gradient&customColorList=12&height=120&section=footer" alt="footer" />
</div>
