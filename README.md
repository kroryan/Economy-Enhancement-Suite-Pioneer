# Economy Enhancement Suite for Pioneer v2.1

A comprehensive economic simulation mod for [Pioneer Space Simulator](https://pioneerspacesim.net/). All modules use **real Pioneer APIs** (`SetCommodityPrice`, `SetCommodityStock`, `AddCommodityStock`, `CargoManager`, `BulletinBoard`) to produce visible in-game effects. Every system is designed to **complement** Pioneer's base game without duplication or conflicts.

## What's New in v2.1

- **Supply Chain Bonuses now apply real price changes** — chain completion modifies station prices via `SetCommodityPrice` on dock, restored on undock
- **Station Services have real economic effects** — fuel discounts, trade bonuses, repair/exploration/bounty bonuses modify actual prices and integrate with other modules
- **Dynamic crew morale system** — morale starts at 75, rises +5 on dock, drops -2 per combat hit; affects dialogue and UI display
- **Cross-module integration** — ExplorationRewards uses StationServices sensor bonus; BountyBoard uses weapon refit bonus
- **NPC cargo destruction fix** — PersistentNPCTrade now correctly detects cargo type via `GetCargoType()` with fallback
- **No global function leaks** — SupplyChainNetwork functions are properly scoped as local
- **Price layering system** — all price-modifying modules (DynamicSystemEvents, SupplyChainNetwork, StationServices) use cache-and-restore pattern with correct ordering
- **UI improvements** — morale color indicators (green/yellow/red), active service bonus display in both Economy tabs

## Features

### Core Economy Modules

#### Dynamic System Events (`DynamicSystemEvents.lua`)
8 event types that **actually modify station prices and stock** while docked:

| Event | Affected Commodities | Trigger |
|-------|---------------------|---------|
| CIVIL_WAR | hand_weapons, battle_weapons, medicines, military_fuel | High lawlessness |
| FAMINE | grain, fruit_and_veg, animal_meat, fertilizer | Low population |
| ECONOMIC_BOOM | consumer_goods, liquor, computers, robots | High population |
| NATURAL_DISASTER | industrial_machinery, metal_alloys, air_processors | Any system |
| PLAGUE | medicines, chemicals, air_processors | Any system |
| PIRATE_RAIDS | hand_weapons, narcotics, slaves, military_fuel | High lawlessness |
| MINING_BOOM | metal_ore, carbon_ore, precious_metals, mining_machinery | Any system |
| TECH_REVOLUTION | computers, robots, plastics, industrial_machinery | Any system |

- Events apply severity-scaled price/stock multipliers on `onPlayerDocked`
- Originals restored on `onPlayerUndocked`
- News adverts appear on the BulletinBoard showing affected commodities and price directions
- Max 2 events per system, generated on `onEnterSystem` and every 30 minutes

#### Persistent NPC Trade (`PersistentNPCTrade.lua`)
Tracks **real NPC cargo destruction** via `CargoManager:CountCommodity()`:
- Hooks `onShipDestroyed` and `onCargoDestroyed` events
- Records supply deficits per system per commodity (with 2%/hour decay)
- On player dock: reduces station stock via `AddCommodityStock(-reduction)`
- BulletinBoard warnings list commodity shortages at station

#### Supply Chain Network (`SupplyChainNetwork.lua`)
Detects **real player trades** by comparing cargo on dock vs undock:

| Chain | Nodes | Base Bonus | Per Node |
|-------|-------|-----------|----------|
| Mining to Manufacturing | metal_ore → metal_alloys → industrial_machinery → robots | 15% | +8% |
| Agriculture to Luxury | grain → animal_meat → liquor → consumer_goods | 12% | +6% |
| Electronics Production | metal_alloys → plastics → computers → robots | 18% | +9% |
| Medical Supply | chemicals → medicines → air_processors → fertilizer | 14% | +7% |
| Industrial Base | carbon_ore → plastics → industrial_machinery → mining_machinery | 16% | +8% |

**How bonuses work:**
- Trade ≥10 tonnes of a chain commodity to activate that node
- Bonus formula: `base_bonus × (active_nodes / total_nodes) + per_node_bonus × active_nodes`
- Example: Mining chain with 3/4 nodes active → 0.15×0.75 + 0.08×3 = **35.25% bonus**
- Bonuses **modify real station prices** via `SetCommodityPrice` on dock (better sell prices for chain commodities)
- Original prices cached and restored on undock — no permanent price corruption
- Maximum bonus cap: 50%

#### Master Coordinator (`EconomyEnhancements.lua`)
v2.0 coordinator — pcall loading, unified API pass-through.

### Expansion Modules

#### Exploration Rewards (`ExplorationRewards.lua`)
- 250 cr bonus per new system explored
- Sell scan data at stations via Bulletin Board ("EXPLORERS' GUILD")
- 6 milestones: Pathfinder (10) → Scout (25) → Explorer (50) → Trailblazer (100) → Vanguard (250) → Pioneer (500)
- **Integrates with StationServices**: Sensor Calibration bonus increases data sale value

#### System News Feed (`SystemNewsFeed.lua`)
- Procedural galactic news from system properties + DynamicSystemEvents integration
- 8 event-reactive templates, 5 filler article categories
- Posted as BB adverts at every station

#### Bounty Hunter Board (`BountyBoard.lua`)
- 4 target types: PIRATE_LORD, SMUGGLER, MURDERER, DESERTER
- Targets spawn via `ShipBuilder.MakeShipOrbit` with appropriate threat levels
- Reputation gate: rep ≥ 4, killcount ≥ 2
- Mission type "BountyHunt" in Missions panel
- **Integrates with StationServices**: Weapon Refit bonus increases bounty reward payouts

#### Smuggling Contracts (`SmugglingContracts.lua`)
- 5 contraband types with risk-scaled rewards
- Only in systems with lawlessness > 15%
- 25% police scan chance on delivery
- Mission type "Smuggling"

#### Passenger Missions (`PassengerMissions.lua`)
- 5 flavours: BUSINESS, VIP (3× pay), FAMILY (3-6 pax), SCIENTIST (2-4), REFUGEE (lawless only)
- Real Passengers API: `EmbarkPassenger`/`DisembarkPassenger` per Character
- Early bonus +15%/day (max 5 days); late penalty 50%
- Mission type "PassengerTransport"

#### Station Services (`StationServices.lua`)
- 6 services with **real economic effects**:

| Service | Effect | Value |
|---------|--------|-------|
| Hull Reinforcement | Repair cost discount | 10% |
| Engine Tuning | Hydrogen fuel price discount | 15% |
| Sensor Calibration | Exploration data bonus | 20% |
| Cargo Optimization | Trade goods price bonus | 15% |
| Weapon Refit | Bounty reward bonus | 25% |
| Full Service | All bonuses combined | 30% |

- Tech level filtering, engineering skill checks, duration-based effects
- Active bonuses **modify real station prices** (fuel discount, trade markup) via `SetCommodityPrice`
- Cross-module integration: ExplorationRewards and BountyBoard query active bonuses via `GetExplorationBonus()` / `GetBountyBonus()`
- All price changes cached and restored on undock

#### Crew Interactions (`CrewInteractions.lua`)
- Timer-based events (900s, 35% chance)
- Categories: idle, danger, trade suggestions, engineering, crew conflicts
- **Dynamic morale system**: starts at 75, +5 on dock, -2 per combat hit, -1/-3 for danger/conflicts
- Morale influences dialogue: low morale → complaints, high morale → confident comments
- Color-coded morale display in Economy UI tabs (green ≥70, yellow ≥40, red <40)

### UI Integration

#### Economy Tab — Station View
Visible **when docked** as a tab alongside Lobby, Bulletin Board, etc. Shows all economy data at a glance.

File: `data/pigui/modules/station-view/08-economy.lua`

#### Economy Tab — Info View (F2)
Accessible **anytime** via F2. Same comprehensive overview available in flight.

File: `data/pigui/modules/info-view/07-economy-dashboard.lua`

#### Bulletin Board Adverts
All modules post interactive adverts when docked: bounty contracts, smuggling jobs, passenger transport, ship services, exploration data sales, news, economic alerts, supply chain progress.

## Installation

Copy the contents into your Pioneer `data/` directory:

```
Pioneer/
└── data/
    ├── Modules/
    │   ├── DynamicSystemEvents.lua
    │   ├── PersistentNPCTrade.lua
    │   ├── SupplyChainNetwork.lua
    │   ├── EconomyEnhancements.lua
    │   ├── ExplorationRewards.lua
    │   ├── SystemNewsFeed.lua
    │   ├── BountyBoard.lua
    │   ├── SmugglingContracts.lua
    │   ├── PassengerMissions.lua
    │   ├── StationServices.lua
    │   ├── CrewInteractions.lua
    │   └── QuickTest.lua          (optional - testing)
    └── pigui/
        └── modules/
            ├── info-view/
            │   └── 07-economy-dashboard.lua
            └── station-view/
                └── 08-economy.lua
```

Modules auto-initialize via Pioneer's event system. No autoload.lua changes needed.

## API Reference

```lua
-- Core Economy
local E = require('modules.EconomyEnhancements')
E.GetVersion()                         --> "2.0.0"
E.GetStatus()                          --> {enabled, version, system_events, npc_trade, supply_chains}
E.GetSystemEvents()                    --> table of active events
E.GetEventTypes()                      --> all event type definitions
E.GetNPCTradeStatus()                  --> {ships_destroyed, total_cargo_lost, total_value_lost}
E.GetSupplyDeficits()                  --> {system -> {commodity -> deficit}}
E.GetSupplyChains()                    --> chain definitions
E.GetChainOpportunities()              --> sorted by bonus potential
E.GetChainProgress()                   --> {chain -> {commodity -> tonnes}}
E.GetCommodityBonus(commodity)         --> 1.0 + bonus multiplier

-- Expansion Modules
require('modules.ExplorationRewards').GetExploredCount()
require('modules.BountyBoard').GetActiveBounties()
require('modules.SmugglingContracts').GetActiveContracts()
require('modules.PassengerMissions').GetActiveTransports()
require('modules.StationServices').GetActiveEffects()
require('modules.CrewInteractions').GetMorale()
require('modules.SystemNewsFeed').GetCurrentNews()
```

## Testing

```lua
require('modules.QuickTest').Run()     -- Comprehensive verification (all 11 modules)
require('modules.QuickTest').Watch()   -- Real-time monitoring
require('modules.QuickTest').Inspect() -- Detailed data dump
require('modules.QuickTest').Stress()  -- Stability test (100 iterations × all APIs)
```

## How Modules Work Together

All price-modifying modules use a **cache-and-restore layering** system on dock/undock:

1. **Base game** `NewsEventCommodity` applies price multipliers (registered first)
2. **DynamicSystemEvents** caches NEC prices, applies crisis multipliers on top
3. **SupplyChainNetwork** caches DSE prices, applies chain bonuses
4. **StationServices** caches chain prices, applies service discounts/bonuses

All restore in reverse order on undock — no price corruption across systems.

### Cross-Module Links
- `ExplorationRewards` → queries `StationServices.GetExplorationBonus()` for data sale multiplier
- `BountyBoard` → queries `StationServices.GetBountyBonus()` for reward multiplier
- `SystemNewsFeed` → reads `DynamicSystemEvents` for event-reactive news articles
- `07-economy-dashboard` / `08-economy` → display morale from `CrewInteractions`, bonuses from `StationServices`

## Compatibility

- **Pioneer version**: 20260203-dev and later
- **Save games**: Full `Serializer:Register` support for all modules
- **Other mods**: Complements base game `NewsEventCommodity`, `Assassination`, `Taxi`, `Scout`, `TradeShips`, `CrewContracts` — no overlaps or conflicts

## Credits

- **Economy Enhancement Suite v2.0**: Designed and implemented by **kroryan**

## License

GPL-3.0 — consistent with Pioneer's license.
