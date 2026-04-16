# Economy Enhancement Suite for Pioneer v2.0

A comprehensive economic simulation mod for [Pioneer Space Simulator](https://pioneerspacesim.net/). All modules use **real Pioneer APIs** (`SetCommodityPrice`, `SetCommodityStock`, `AddCommodityStock`, `CargoManager`, `BulletinBoard`) to produce visible in-game effects.

## Features

### Dynamic System Events
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
- Originals restored on `onPlayerUndocked` (same pattern as NewsEventCommodity)
- News adverts appear on the BulletinBoard showing affected commodities and price directions
- Max 2 events per system, generated on `onEnterSystem` and every 30 minutes

### Persistent NPC Trade
Tracks **real NPC cargo destruction** via `CargoManager:CountCommodity()`:

- Hooks `onShipDestroyed` and `onCargoDestroyed` events
- Records supply deficits per system per commodity (with 2%/hour decay)
- On player dock: reduces station stock via `AddCommodityStock(-reduction)`
- BulletinBoard warnings list commodity shortages at station
- Tracks destruction log (last 20 events, 7-day cleanup)

### Supply Chain Network
Detects **real player trades** by comparing cargo on dock vs undock:

| Chain | Nodes (real commodity names) | Base Bonus | Per Node |
|-------|------------------------------|-----------|----------|
| Mining to Manufacturing | metal_ore → metal_alloys → industrial_machinery → robots | 15% | +8% |
| Agriculture to Luxury | grain → animal_meat → liquor → consumer_goods | 12% | +6% |
| Electronics Production | metal_alloys → plastics → computers → robots | 18% | +9% |
| Medical Supply | chemicals → medicines → air_processors → fertilizer | 14% | +7% |
| Industrial Base | carbon_ore → plastics → industrial_machinery → mining_machinery | 16% | +8% |

- 10 tonnes traded activates a chain node
- Cumulative bonuses up to 50% cap
- BulletinBoard advert shows chain progress with [OK]/[  ] markers

### Master Coordinator
`EconomyEnhancements.lua` v2.0 — clean coordinator with pcall loading, no redundant timers (sub-modules self-coordinate), unified API pass-through.

## Installation

Copy the `data/Modules/` contents into Pioneer's `data/modules/`:

```
Pioneer/
└── data/
    └── modules/
        ├── DynamicSystemEvents.lua
        ├── PersistentNPCTrade.lua
        ├── SupplyChainNetwork.lua
        ├── EconomyEnhancements.lua
        └── QuickTest.lua          (optional - testing)
```

Modules auto-initialize via Pioneer's event system. No autoload.lua changes needed.

## API Reference

```lua
local E = require('modules.EconomyEnhancements')

-- Status
E.IsEnabled()                          --> true/false
E.GetVersion()                         --> "2.0.0"
E.GetStatus()                          --> {enabled, version, system_events, npc_trade, supply_chains}

-- Events
E.GetSystemEvents()                    --> table of active events
E.GetSystemEventDescription(event)     --> string
E.GetEventTypes()                      --> table of all event type definitions

-- Trade
E.GetNPCTradeStatus()                  --> {ships_destroyed, total_cargo_lost, total_value_lost}
E.GetSupplyDeficits()                  --> {system -> {commodity -> deficit_amount}}
E.GetRegionalDependencies()            --> table
E.GetDamagedCargo()                    --> table

-- Supply Chains
E.GetSupplyChains()                    --> table of chain definitions
E.GetChainOpportunities()              --> table sorted by bonus potential
E.GetChainProgress()                   --> {chain_key -> {commodity -> tonnes_traded}}
E.GetChainEfficiency(system_path)      --> chain bonuses
E.GetCommodityBonus(commodity)         --> 1.0 + bonus multiplier
```

## Testing

```lua
require('modules.QuickTest').Run()     -- Comprehensive verification
require('modules.QuickTest').Watch()   -- Real-time monitoring
require('modules.QuickTest').Inspect() -- Detailed data dump
require('modules.QuickTest').Stress()  -- Stability test (100 iterations per function)
```

## Compatibility

- **Pioneer version**: 20260203-dev and later
- **Save games**: Full `Serializer:Register` support for all modules
- **Other mods**: Complements `NewsEventCommodity` (that mod handles single-commodity events in nearby systems; this mod handles multi-commodity systemic events in the current system)

## Credits

- **Economy Enhancement Suite**: Designed and implemented by **kroryan**

## License

GPL-3.0 — consistent with Pioneer's license.
