---
icon: door-open
---

# Basic Setup

## Getting Started with RNGEngine

### Introduction

RNGEngine is an advanced weighted random selection system for RedM VORP. It provides a robust framework for implementing randomized mechanics in your RedM server with fine-grained control over probabilities, item selection, and persistence.

### Installation

1. Download the resource and place it in your server's resources folder
2. Add `ensure alentexas_rng` to your server.cfg (make sure it loads before any resources that depend on it)
3. Use it in your resources by adding `alentexas_rng` to the dependencies in your fxmanifest.lua

```lua
-- In your fxmanifest.lua
dependencies {
    'alentexas_rng'
}
```

### Accessing the RNG System

You can access the RNG functions in two ways:

#### Client-Side

```lua
local RNG = exports.alentexas_rng
-- Now use RNG functions, e.g., RNG.CreateProbabilityList()
```

#### Server-Side

```lua
local RNG = exports.alentexas_rng
-- Access both core RNG and persistence functions
local Persistence = RNG.Persistence
```

### Key Features

* **Weighted Random Selection**: Assign different probabilities to items
* **Multiple Selection Methods**: Choose from Cumulative, Linear, or Random algorithms
* **Advanced Depletion System**:
  * Simple binary depletion (available/depleted)
  * Advanced gradual recharge system with multiple units that recharge over time
* **History Tracking**: Prevent repetition in selections
* **Collections**: Organize multiple probability lists with their own weights
* **Persistence**: Save and load RNG states to/from database with configurable reset timers

### Quick Start Example

#### Basic Item Selection

```lua
-- In your resource script
local RNG = exports.alentexas_rng

-- Create a new probability list
local lootTable = RNG.CreateProbabilityList()

-- Add items with their weights
lootTable:Add("Gold Nugget", 10) -- 10% chance
lootTable:Add("Silver Watch", 20) -- 20% chance
lootTable:Add("Tobacco", 30) -- 30% chance
lootTable:Add("Whiskey", 40) -- 40% chance

-- Get a random item
local item = RNG.SelectRandomItem(lootTable)
print("Player found: " .. item)
```

#### Using Structured Data

```lua
local RNG = exports.alentexas_rng

-- Create a list with structured data
local lootTable = RNG.CreateProbabilityList()

-- Add items with data tables
lootTable:Add({name = "Gold Nugget", value = 50, model = "p_goldnugget01x"}, 10)
lootTable:Add({name = "Silver Watch", value = 25, model = "p_watch01x"}, 20)
lootTable:Add({name = "Tobacco", value = 5, model = "p_tobaccopouch01x"}, 30)
lootTable:Add({name = "Whiskey", value = 10, model = "p_bottleJD01x"}, 40)

-- Get a random item WITH data (pass true as second parameter)
local itemData = RNG.SelectRandomItem(lootTable, true)
print("Player found: " .. itemData.name .. " worth $" .. itemData.value)
```

#### Basic Persistence (Server-Side)

```lua
local RNG = exports.alentexas_rng
local Persistence = RNG.Persistence

-- Initialize persistence for your resource (server-side only)
Persistence.Initialize("my_resource", 24) -- 24-hour reset time

-- Create and set up a loot table
local lootTable = RNG.CreateProbabilityList()
lootTable:Add("Gold Nugget", 10)
lootTable:Add("Silver Watch", 20)

-- Save the state with a unique identifier
Persistence.SaveList("chest_valentine_1", lootTable)

-- Later, load the saved state
Persistence.LoadList("chest_valentine_1", lootTable)
```

#### Advanced Depletion (Server-Side)

```lua
local RNG = exports.alentexas_rng

-- Create a mining node that recharges over time
local miningNode = RNG.CreateProbabilityList()
miningNode:Add("Iron Ore", 0.6)
miningNode:Add("Gold Ore", 0.4)

-- Enable advanced depletion: 3 uses, 10 minutes per recharge
RNG.EnableAdvancedDepletion(miningNode, 3, 10, true)

-- Use the mining node
local success, message = RNG.UseItemUnits(miningNode, 0, 1)
if success then
    local ore = RNG.SelectRandomItem(miningNode)
    print("Mined: " .. ore)
    
    -- Check remaining uses
    local units, maxUnits, timeToNext = RNG.GetItemUnitsInfo(miningNode, 0)
    print(string.format("Node: %d/%d uses left, recharges in %.1f min", 
        units, maxUnits, timeToNext/60))
end
```

### Troubleshooting Common Issues

#### Selection Always Returns the Same Item

* Check if your probability weights are set correctly
* Make sure you're not accidentally depleting all items
* Verify history tracking is set up correctly if using repeat prevention

#### Server Error When Using Persistence

* Ensure you're only using persistence on the server side
* Check that the Initialize function was called before other persistence functions
* Verify the resource name matches your resource's folder name exactly

#### Random Selection Not Working as Expected

* Remember that weights are relative (10, 20, 30, 40 equals 10%, 20%, 30%, 40%)
* Try a different selection method if you need a different distribution pattern
* Ensure total probability is not zero (which can happen if all items are depleted)
