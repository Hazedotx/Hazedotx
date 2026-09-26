# My Project Portfolio

## Killspree

###  Showcase

[Watch the 23-minute Killspree showcase](https://www.youtube.com/watch?v=0ZLOrN9r6tU)

> The video demonstrates the gameplay, Killer abilities, combat, and networking systems that I worked on.

### My Contributions

I was the **Lead Programmer** for Killspree. I programmed the Killers and their abilities demonstrated throughout the showcase.

I also developed several core gameplay and networking systems, including:

* **Killer abilities and gameplay systems** — programmed the Killer characters and abilities showcased in the video.
* **Hitbox system** — developed the hitbox system used for gameplay interactions and combat.
* **Networking** — implemented interpolated buffer snapshot rollback netcode to improve the responsiveness and smoothness of gameplay, particularly when playing as the Killer.
* **Gameplay programming** — worked on the underlying systems required to make the showcased gameplay function correctly in a networked environment.

### Source Code

The production source code for this project is private and cannot be publicly shared. This repository is intended to showcase my work and demonstrate the systems I contributed to through gameplay footage.


# Backrooms Tower Defense

**Showcase:** [Watch the 9-minute showcase](https://www.youtube.com/watch?v=yrDP3b7QeC4)

The video walks through the gameplay, tower logic, enemy logic, networking systems, and more.

## My Contribution

I was the lead programmer on this project. I built:
- All of the frontend UI Code
- All of the backend systems
- All of the networking architecture

Aside from two or three minor exceptions, I was responsible for essentially everything under the hood.

## Trading System

The piece I'm proudest of. Item duplication ("duping") — trading an item and ending up with a copy of it — is the most common exploit in games like this. The trading system prevents it through several layers:

- **State-gated trade objects.** Each trade tracks `Active`, `Processing`, or `Complete`, and every client action (offer, confirm, cancel) is gated by that state so nothing can change mid-settlement.
- **Confirmation re-arming.** Any offer change after confirming un-confirms both sides, preventing last-second swaps.
- **Atomic settlement with rollback.** Both inventories are snapshotted before anything moves. Items are removed from both sides before anything is granted; if any step fails, both snapshots are restored and any related counters are diffed back to their prior state.
- **Datastore-health gating.** Trades refuse to finalize if the datastore is in a critical or closing state.
- **Shutdown safety.** In-progress trades get a grace window on server shutdown and player-leaving so a settlement isn't cut off mid-write.

Backed by a **$300 bounty** at launch for anyone who could produce a working dupe. Five experienced exploiters tried and failed. Zero reported dupes across hundreds of thousands of trades since release.

## Quest System

Manages daily, weekly, lifetime, and clan quest categories per player. Every quest type shares one interface — `isComplete`, `title`, `textGoal`, `textProgress`, `percentProgress` — so new quest types drop into the pool without touching the core service.

- **Refresh-in-progress guards.** Overlapping refresh calls for the same player (from the async data layer) are blocked from running concurrently, preventing duplicate quest batches.
- **Self-healing corruption detection.** A player missing a daily or weekly quest gets that category regenerated automatically.
- **Anti-farming caps.** Clan quests draw from a shared pool with per-identifier caps, so a player's active slots can't all roll the same low-effort quest type.
- **Periodic sweep.** A background loop re-checks trackable players and tops up missing slots, so one dropped event can't leave a player quest-less for a session.
- **Data-driven reward scaling.** Each category has its own difficulty range and reward rate, so longer-requirement rolls pay out proportionally more. Lifetime quests add a small chance of rare items on top, scaled the same way.
- **Decoupled reward calculation.** What a quest *will* reward is computed separately from granting it, so the UI can preview rewards without an active quest handler.

## Distributed Leaderboard System

A cross-server leaderboard for endless mode: tracks up to 20,000 ranked players, resets on a season timer, pays out rewards to top ranks at season end. The core challenge is that many servers run in parallel with no shared memory — only the datastore ties them together.

- **Config-driven leaderboards.** Each leaderboard is defined by its own player cap, season length, lock duration, and a check for which server types are allowed to perform the (expensive) rebuild.
- **Single-writer locking.** A server must claim a lock in the datastore before rebuilding. If it stalls, the lock expires and another server takes over; if the original server finishes late anyway, its write is rejected since it no longer holds the lock.
- **Failed reads never overwrite good data.** A partial or failed read from the datastore is discarded instead of saved, so a transient failure can't blank the board for everyone.
- **Reads are decoupled from builds.** Any server can serve a cached leaderboard to players regardless of which server built it, and concurrent requests share a single read instead of hitting the datastore per player.
- **Rate-limit-aware pacing.** Rebuilds check live request budget instead of running on a fixed timer, staying fast under normal load and backing off only under real contention.
