# Roblox Gameplay & Systems Engineer

Lead programmer on Roblox games with **5M+ visits**, **low five-figure revenue**, and up to **3,000 concurrent players**.
I build server-authoritative gameplay, networking, and data systems in Luau, with a focus on systems that hold up against exploiters at scale.

**Jump to:** [Into the Backrooms Tower Defense](#into-the-backrooms-tower-defense) · [Killspree](#killspree)

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
