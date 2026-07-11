# Warren Engine

- **Team Name:** Kodair (Farwalker3)
- **Payment Details:**
  - **DOT**: [POLKADOT_ADDRESS_PENDING — will be added before marking ready for review]
  - **Payment**: [ASSETHUB_ADDRESS_PENDING] (USDC)
- **Level:** 1

## Project Overview :page_facing_up:

### Overview

**Warren Engine is an open-source Substrate pallet suite for autonomous
idle worlds — game worlds whose state advances every block, inside the
runtime itself, with zero transactions.**

On EVM platforms, "idle" game worlds are inert between transactions: they
either lazy-evaluate elapsed time when a user finally acts, or pay
off-chain keeper networks (Chainlink Automation, Gelato) to poke contracts
on a schedule. FRAME's `on_initialize`/`on_idle` hooks make autonomous
world-advancement a native primitive — a capability unique to the
Substrate stack. Warren Engine packages it as reusable infrastructure:
a weight-bounded tick engine, stateful creature NFTs with trait
inheritance, and timed expedition resolution.

**Why we're building it:** we revive dead NFT projects as stories. Our
first revival is The Rabbit Project — an abandoned NFT idle game whose
surviving canon and artwork we recovered and archived
(https://github.com/Farwalker3/therabbitproject), now being developed into
an animated series. We are generalizing that methodology into a public
revival platform (The Revival Project), and Warren Engine is its on-chain
substrate: a revived world should not depend on anyone remembering to keep
it alive. The engine's flagship demo is the premise made executable — **a
world that keeps running after everyone logs off.**

### Project Details

Full design document:
https://github.com/Farwalker3/therabbitproject/blob/main/engine/DESIGN.md

**Architecture — three pallets plus a demo chain:**

- **`pallet-warren`** — generic tick engine. Storage: task queue keyed by
  block number (`TasksByBlock: StorageMap<BlockNumber, BoundedVec<TaskId>>`
  + `Tasks: StorageMap<TaskId, TaskInfo>`); tasks scheduled by downstream
  pallets via a `ScheduleTick` trait. Execution: `on_initialize` drains
  due tasks under a configurable weight budget (`MaxTickWeight`),
  spillover consumes spare weight in `on_idle`, remainder rolls forward —
  the world never stalls, it stretches. Downstream logic plugs in through
  a `TickHandler` associated type; the pallet knows nothing about any
  specific game.
- **`pallet-critters`** — creature NFTs with genomes (fixed-length trait
  vectors; deterministic inheritance over two parent genomes + a
  randomness input) and a time-based state machine
  (`Idle → OnExpedition → Injured → Recovering`, breeding cooldowns)
  driven by warren tasks — healing completes because the chain ticks, not
  because a user claims it. Ships with a `pallet-nfts` adapter for
  wallet/marketplace interoperability.
- **`pallet-expeditions`** — timed raids: validate and lock a party,
  schedule resolution N blocks ahead, resolve inside the tick (outcome
  from party stats + destination difficulty + randomness), mint rewards
  via `fungibles` traits, apply injuries with scheduled recovery, and a
  heal-with-burn extrinsic that shortens recovery.
- **Randomness, stated honestly:** dev/demo uses
  `insecure-randomness-collective-flip`, clearly labeled. The production
  design combines BABE epoch randomness with a commit–reveal delay: the
  randomness applied to an action is drawn from an epoch after the action
  was committed, so neither players nor block authors can grind outcomes.
  Epoch-granularity limitations are documented, and the `Randomness`
  config type is pluggable for chains with stronger beacons.
- **Demo:** a solochain built from the polkadot-sdk solochain template
  plus a minimal web dashboard (TypeScript, polkadot.js) showing
  expeditions resolving and critters healing **with zero transactions
  submitted** — the empty warren, running anyway.

**Technology stack:** Rust, FRAME / polkadot-sdk, TypeScript + polkadot.js
(demo UI), Docker.

**What Warren Engine is not:** not a token launch (no token in any
milestone; the demo currency is a plain `pallet-assets` asset), not a game
economy design, not a story/media production — those belong to the
separately funded product layer (disclosed under Additional Information).

### Ecosystem Fit

- **The gap:** no general-purpose autonomous-execution layer for games
  exists as reusable FRAME pallets. `pallet-scheduler` dispatches
  individual calls at target blocks; a game needs thousands of small
  recurring tasks resolved in weight-bounded batches with graceful
  overflow. Every Substrate game team currently hand-rolls this.
- **Who benefits:** Substrate game developers, the autonomous-worlds
  community, and agent-simulation projects that need on-chain time.
  Comparable EVM work (MUD) still requires off-chain tick drivers.
- **The bigger audience — the graveyard:** most NFT games shipped during
  the 2021–2023 boom are dead: domains lapsed, teams vanished, communities
  holding orphaned assets. Our revival methodology (archive → retell →
  optionally re-run the world) is being productized as a public platform,
  and Warren Engine is the piece that lets any revived world run
  autonomously and cheaply (coretime economics fit small persistent
  worlds well). Each revival is a potential new Substrate world and
  community — an onboarding channel no other ecosystem is targeting.
- **Why Polkadot specifically:** the core primitive (runtime hooks) does
  not exist on contract platforms; forkless upgrades double as game
  live-ops; runtime-level fee abstraction lets players play without gas
  tokens; XCM opens critters and currencies to other parachains later.

## Team :busts_in_silhouette:

### Team members

- John C. Barr (Farwalker3) — founder / project lead

### Contact

- **Contact Name:** John C. Barr
- **Contact Email:** farwalker4@gmail.com
- **Website:** https://github.com/Farwalker3

### Legal Structure

- **Registered Address:** None (individual)
- **Registered Legal Entity:** None

### Team's experience

Independent builder; nine years of public GitHub output (63+ repos):
consumer mobile apps (Flutter/Dart — Allowance Ally / Screentime Rewards,
Fire TV apps), web games, AI/automation tooling (MCP integrations, Python
media pipelines), and web3 onboarding (Nearzy — free NEAR wallets).
Executed The Rabbit Project recovery end-to-end: complete archives of the
last surviving listings, reconstructed game canon, original artwork
recovery, revival site, and a six-episode series treatment — all public.

**Honest disclosure:** this is the team's first Substrate/Rust project.
Development is done in partnership with AI engineering tooling (Claude,
Anthropic), and we have scoped this application at Level 1 with
conservative milestones precisely because of that: W3F's milestone review
means payment only against working, tested, benchmarked code.

### Team Code Repos

- https://github.com/Farwalker3/therabbitproject
- https://github.com/kodarize
- Engine repository on grant start: https://github.com/Farwalker3/warren-engine

## Development Status :open_book:

- Complete design document: [engine/DESIGN.md](https://github.com/Farwalker3/therabbitproject/blob/main/engine/DESIGN.md)
- Recovered game canon that defines the reference implementation's
  mechanics: [research/RESEARCH.md](https://github.com/Farwalker3/therabbitproject/blob/main/research/RESEARCH.md)
- The revival methodology proven once end-to-end (The Rabbit Project) and
  being generalized into a public registry/platform.

## Development Roadmap :nut_and_bolt:

### Overview

- **Total Estimated Duration:** 3.5 months
- **Full-Time Equivalent (FTE):** 1
- **Total Costs:** 10,000 USD
- **DOT %:** 50%

### Milestone 1 — `pallet-warren` (tick engine)

- **Estimated duration:** 1.5 months
- **FTE:** 1
- **Costs:** 4,000 USD

| Number | Deliverable | Specification |
| -----: | ----------- | ------------- |
| **0a.** | License | Apache 2.0 |
| **0b.** | Documentation | Inline rustdoc + a tutorial: "add an autonomous tick to your runtime in 30 minutes" |
| **0c.** | Testing and Testing Guide | Unit tests covering scheduling, weight-budget overflow, ordering guarantees, and roll-forward liveness under flood (property tests); guide to run them |
| **0d.** | Docker | Dockerfile running a node with the pallet and executing the test suite |
| 1. | `pallet-warren` | `ScheduleTick` API, weight-bounded `on_initialize` drain, `on_idle` spillover, roll-forward under load, `TickHandler` extension point |
| 2. | Benchmarks | FRAME benchmarks + generated `WeightInfo` for all dispatchables and hooks |

### Milestone 2 — `pallet-critters`

- **Estimated duration:** 1 month
- **FTE:** 1
- **Costs:** 3,000 USD

| Number | Deliverable | Specification |
| -----: | ----------- | ------------- |
| **0a.** | License | Apache 2.0 |
| **0b.** | Documentation | rustdoc + tutorial: defining a genome and breeding rules |
| **0c.** | Testing and Testing Guide | Unit tests: inheritance determinism, state-machine transitions, cooldown enforcement, randomness commit–reveal flow |
| **0d.** | Docker | Updated image incl. pallet-critters |
| 1. | `pallet-critters` | Genome storage, deterministic trait inheritance, time-based state machine driven by warren tasks |
| 2. | Randomness module | Commit–reveal breeding over a pluggable `Randomness` type; written threat model documenting manipulation resistance and limitations |
| 3. | `pallet-nfts` adapter | Critters exposed through `pallet-nfts` interfaces for wallet/marketplace interop |

### Milestone 3 — `pallet-expeditions` + autonomous world demo

- **Estimated duration:** 1 month
- **FTE:** 1
- **Costs:** 3,000 USD

| Number | Deliverable | Specification |
| -----: | ----------- | ------------- |
| **0a.** | License | Apache 2.0 |
| **0b.** | Documentation | rustdoc + end-to-end tutorial: "build an idle game on Substrate with Warren Engine" |
| **0c.** | Testing and Testing Guide | Unit + integration tests across all three pallets: full expedition lifecycle resolved with zero user transactions |
| **0d.** | Docker | Compose file: demo chain + dashboard |
| **0e.** | Article | Public write-up: "Worlds that keep running: autonomous idle games on Substrate" — the technical thesis, the engine, and the revival use case |
| 1. | `pallet-expeditions` | Party lock, scheduled resolution, rewards via `fungibles`, injury/recovery loop, heal-with-burn extrinsic |
| 2. | Demo chain + dashboard | Solochain runtime integrating all three pallets + web dashboard visualizing the world advancing autonomously |

## Future Plans

- **Immediate:** deploy the reference world as the revived Rabbit Project
  (game product and community funded separately; see disclosure), and
  evaluate an on-demand coretime deployment for it.
- **The Revival Project platform:** a public registry where communities
  submit dead NFT projects for revival — archive, story, and (where the
  world's mechanics fit) an autonomous Warren Engine world. The engine is
  the reusable substrate for every such revival.
- **Engine roadmap:** `pallet-clans` (clan treasuries, arena matchmaking)
  as a follow-up grant or community contribution; XCM asset exposure.
- **Maintenance:** the engine underpins our own product; it stays
  maintained because we ship on it.

## Additional Information :heavy_plus_sign:

**How did you hear about the Grants Program?** Web3 Foundation website and
the Grants-Program repository.

**Work already done:** the design document, the recovered canon defining
the reference implementation, and the revival site/treatment are public in
https://github.com/Farwalker3/therabbitproject.

**Disclosures:** The Rabbit Project revival (game product, community,
animated series) is separately seeking ecosystem funding (a SKALE gaming
grant application has been submitted; a Sei Creator Fund application is
planned, reflecting the original game's Sei heritage). **No deliverable in
this application overlaps with that work:** this grant funds only the
open-source Substrate engine described above. No other team has financially
contributed to this project; we have not applied for other grants *for the
Warren Engine* itself.
