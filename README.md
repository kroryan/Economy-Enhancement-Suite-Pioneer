# Economy Enhancement Suite for Pioneer

A comprehensive economic simulation mod for [Pioneer Space Simulator](https://pioneerspacesim.net/), consisting of self-contained Lua modules that seamlessly integrate with Pioneer's existing economy system.

## Features

### Dynamic System Events
Creates reactive, probabilistic events that affect local economies:
- **Civil War** — Disrupts industrial sectors, increasing military goods prices
- **Food Crisis** — Shortages drive agriculture and food prices up
- **Economic Boom** — Increased demand for luxury goods and services
- **Natural Disaster** — Infrastructure damage raises construction material prices
- **Disease Outbreak** — Medical supplies and vaccines become premium commodities

Each event generates randomly (~0.05% per system check), lasts 30–72 hours in-game, and has varying severity.

### Persistent NPC Trade
Tracks inter-system trade relationships and economic dependencies:
- Monitors NPC trade shipments between stations
- Records destroyed or delayed cargo and its economic impact
- Creates regional supply dependencies
- Destroyed shipments trigger local price increases for missing commodities

### Supply Chain Network
Defines five multi-level supply chains with completion bonuses:

| Chain | Nodes | Base Bonus | Per Node |
|-------|-------|-----------|----------|
| Mining → Spacecraft | Metal Ore → Refined Metals → Machinery → Spacecraft Parts | 15% | +8% |
| Agriculture → Luxury | Raw Food → Processed Food → Beverages → Luxury Goods | 12% | +6% |
| Electronics Production | Metals → Electronic Components → Electronics → Advanced Systems | 18% | +9% |
| Medical Supply | Chemicals → Medicines → Medical Supplies → Vaccines | 14% | +7% |
| Industrial Base | Minerals → Machinery → Industrial Equipment → Construction Materials | 16% | +8% |

Complete chains to unlock cumulative price bonuses (max 50% markup).

### Master Coordinator
`EconomyEnhancements.lua` coordinates all modules with a unified API, automatic serialization, and periodic subsystem synchronization.

## Installation

1. Copy the `data/Modules/` folder contents into your Pioneer installation's `data/modules/` directory
2. The system automatically initializes on game load

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

## API Reference

```lua
local E = require('modules.EconomyEnhancements')

-- Status
E.IsEnabled()                          --> true/false
E.GetVersion()                         --> "1.0.0"
E.GetStatus()                          --> table

-- Events
E.GetSystemEvents()                    --> table of active events
E.GetSystemEventDescription(event)     --> string

-- Trade
E.GetNPCTradeStatus()                  --> {active, delivered, destroyed, total_value}
E.GetRegionalDependencies()            --> table
E.GetDamagedCargo()                    --> table

-- Supply Chains
E.GetSupplyChains()                    --> table of chain definitions
E.GetChainOpportunities()              --> table of trading opportunities
E.GetChainDescription(name)            --> string
E.GetCommodityBonus(commodity)         --> 1.0 + bonus multiplier
```

## Testing

The suite includes `QuickTest.lua` for verification:

```lua
require('modules.QuickTest').Run()     -- Comprehensive verification
require('modules.QuickTest').Watch()   -- Real-time monitoring
require('modules.QuickTest').Inspect() -- Detailed state inspection
require('modules.QuickTest').Stress()  -- Stability test (100 iterations)
require('modules.QuickTest').Debug()   -- Quick system state debug
```

## Compatibility

- **Pioneer version**: 20260203-dev and later
- **Save games**: Full serialization support — safe to install/uninstall mid-save
- **Other mods**: Non-invasive, does not modify core Pioneer systems

## Design Philosophy

- **Self-contained**: No external dependencies beyond Pioneer core libraries
- **Emergent**: Effects appear gradually through probabilistic systems
- **Subtle**: Individual changes are small; cumulative effects create a reactive economy
- **Persistent**: Full serialization for save/load compatibility
- **Non-invasive**: Coordinates without modifying core systems
- **Auditable**: Debug logging shows exactly what's happening

## Credits

- **Economy Enhancement Suite**: Designed and implemented by **kroryan**
- **Pioneer Core**: Original development by the [Pioneer Developers](https://github.com/pioneerspacesim/pioneer)

## License

GPL-3.0 — consistent with Pioneer's license.
