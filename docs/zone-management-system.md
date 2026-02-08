---
icon: map-location-dot
---

# Zone Management System

The Zone Management System provides PolyZone integration for area-specific loot tables, allowing you to create dynamic zones with different loot distributions, priority handling, and zone-specific systems.

### Overview

The Zone Management System allows you to:

* Create area-specific loot tables that only activate when players are in certain zones
* Set up different mining areas with unique ore distributions
* Create treasure hunting zones with special loot
* Implement territory-based gameplay mechanics
* Handle overlapping zones with priority systems
* Apply zone-specific influence modifiers to loot probabilities

**Requirements**: This system requires the PolyZone resource to be installed and started.

### Zone Types

The system supports four types of zones:

#### Circle Zones

Simple circular areas defined by a center point and radius.

```lua
local RNG = exports.alentexas_rng

local zone = RNG.CreateZone("mining_area", "Valentine Mine", "circle", 
    vector3(-1823.0, -433.0, 160.0), {
        radius = 50.0,
        priority = 2
    })
```

#### Box Zones

Rectangular areas with configurable dimensions and rotation.

```lua
local zone = RNG.CreateZone("hunting_grounds", "Big Valley Hunting", "box",
    vector3(-1200.0, 300.0, 95.0), {
        length = 200.0,
        width = 150.0,
        heading = 45.0,
        minZ = 80.0,
        maxZ = 120.0,
        priority = 2
    })
```

#### Polygon Zones

Complex shapes defined by multiple points.

```lua
local zone = RNG.CreateZone("treasure_beach", "Treasure Beach", "poly", nil, {
    points = {
        vector2(1234.0, -2890.0),
        vector2(1456.0, -2890.0),
        vector2(1456.0, -2650.0),
        vector2(1234.0, -2650.0)
    },
    minZ = 40.0,
    maxZ = 60.0,
    priority = 1
})
```

#### Combo Zones

Combinations of multiple zone types.

```lua
local zone = RNG.CreateZone("complex_area", "Complex Zone", "combo", nil, {
    zones = {
        -- Array of other zone definitions
    },
    priority = 3
})
```

### Creating Zones

#### Basic Zone Creation

```lua
local RNG = exports.alentexas_rng

-- Create a simple mining zone
local miningZone = RNG.CreateZone(
    "valentine_mine",           -- Unique ID
    "Valentine Mining Area",    -- Display name
    "circle",                   -- Zone type
    vector3(-1823.0, -433.0, 160.0), -- Coordinates
    {
        radius = 50.0,          -- Zone-specific options
        priority = 2,
        debug = false
    }
)
```

#### Zone Options

| Option               | Type    | Description                               |
| -------------------- | ------- | ----------------------------------------- |
| `radius`             | number  | Radius for circle zones                   |
| `length`             | number  | Length for box zones                      |
| `width`              | number  | Width for box zones                       |
| `heading`            | number  | Rotation for box zones                    |
| `minZ`               | number  | Minimum Z coordinate                      |
| `maxZ`               | number  | Maximum Z coordinate                      |
| `points`             | table   | Array of vector2 points for polygon zones |
| `priority`           | number  | Zone priority (higher = more important)   |
| `debug`              | boolean | Show debug visualization                  |
| `influenceModifiers` | table   | Item-specific probability modifiers       |
| `chargeSettings`     | table   | Zone-specific charge configurations       |
| `depletionSettings`  | table   | Zone-specific depletion configurations    |

### Zone Properties

Each zone has the following properties:

* **ID**: Unique identifier
* **Name**: Display name
* **Type**: Zone type (circle, box, poly, combo)
* **Priority**: Determines which zone takes precedence in overlapping areas
* **Loot Tables**: Collection of probability lists assigned to the zone
* **Influence Modifiers**: Probability adjustments for specific items
* **Player Tracking**: List of players currently in the zone
* **Settings**: Zone-specific charge and depletion configurations

### Loot Table Management

#### Adding Loot Tables to Zones

```lua
-- Create loot tables
local miningLoot = RNG.CreateProbabilityList()
miningLoot:Add("iron_ore", 0.4)
miningLoot:Add("coal", 0.3)
miningLoot:Add("silver_ore", 0.2)
miningLoot:Add("gold_ore", 0.1)

local commonLoot = RNG.CreateProbabilityList()
commonLoot:Add("rock", 0.6)
commonLoot:Add("stick", 0.4)

-- Add to zone
local zone = RNG.GetZone("valentine_mine")
zone:AddLootTable("mining", miningLoot, 1.0)
zone:AddLootTable("common", commonLoot, 0.3)
```

#### Selecting Loot from Zones

```lua
-- Get loot from a specific table in player's zones
local item, zone = RNG.SelectLootFromPlayerZones(playerId, "mining")

-- Get loot from any table in player's zones (weighted by priority)
local item, zone, tableName = RNG.SelectRandomLootTableFromPlayerZones(playerId)
```

### Priority System

When a player is in multiple overlapping zones, the system uses priority to determine which zone's loot tables to use:

```lua
-- Zone A: Priority 1
local zoneA = RNG.CreateZone("zone_a", "Low Priority Zone", "circle", coords, {
    priority = 1
})

-- Zone B: Priority 3  
local zoneB = RNG.CreateZone("zone_b", "High Priority Zone", "circle", coords, {
    priority = 3
})

-- Player in both zones will get loot from Zone B (higher priority)
local highestZone = RNG.GetHighestPriorityZone(playerId)
```

### Influence Modifiers

Zones can modify the probability of specific items:

```lua
local zone = RNG.CreateZone("premium_mine", "Premium Mining Area", "circle", coords, {
    priority = 3,
    influenceModifiers = {
        ["gold_ore"] = 3.0,    -- Triple gold ore chance
        ["silver_ore"] = 2.0,  -- Double silver ore chance
        ["iron_ore"] = 0.5,    -- Half iron ore chance
        ["*"] = 1.2            -- 20% bonus to all other items
    }
})
```

The `"*"` key applies to all items that don't have specific modifiers.

### Zone-specific Settings

#### Charge Settings

```lua
local zone = RNG.CreateZone("mining_zone", "Mining Zone", "circle", coords, {
    chargeSettings = {
        mining = {
            maxCharges = 10,
            rechargeTime = 30,  -- minutes
            autoRecharge = true
        }
    }
})

-- Get zone-specific charge settings for a player
local chargeSettings = RNG.GetZoneChargeSettings(playerId, "mining")
```

#### Depletion Settings

```lua
local zone = RNG.CreateZone("treasure_zone", "Treasure Zone", "circle", coords, {
    depletionSettings = {
        treasure_hunting = {
            maxUnits = 5,
            rechargeTime = 60,  -- minutes
            autoRecharge = true
        }
    }
})

-- Get zone-specific depletion settings for a player
local depletionSettings = RNG.GetZoneDepletionSettings(playerId, "treasure_hunting")
```

### Player Tracking

The system automatically tracks which players are in which zones:

```lua
-- Check if player is in any zone
local inZone = RNG.IsPlayerInAnyZone(playerId)

-- Get all zones a player is currently in
local playerZones = RNG.GetPlayerZones(playerId)

-- Get the highest priority zone for a player
local highestZone = RNG.GetHighestPriorityZone(playerId)

-- Get player count in a specific zone
local playerCount = RNG.GetZonePlayerCount("valentine_mine")
```

#### Zone Events

The system triggers events when players enter or exit zones:

```lua
-- Player entered zone
AddEventHandler("rngengine:zone:playerEntered", function(playerId, zoneId, zoneName)
    print("Player " .. playerId .. " entered " .. zoneName)
    TriggerClientEvent("vorp:Tip", playerId, "Entered: " .. zoneName, 3000)
end)

-- Player exited zone
AddEventHandler("rngengine:zone:playerExited", function(playerId, zoneId, zoneName)
    print("Player " .. playerId .. " left " .. zoneName)
    TriggerClientEvent("vorp:Tip", playerId, "Left: " .. zoneName, 3000)
end)
```

### API Reference

#### Zone Creation and Management

| Function                                      | Description       |
| --------------------------------------------- | ----------------- |
| `CreateZone(id, name, type, coords, options)` | Create a new zone |
| `GetZone(id)`                                 | Get zone by ID    |
| `GetAllZones()`                               | Get all zones     |
| `DestroyZone(id)`                             | Destroy a zone    |
| `DestroyAllZones()`                           | Destroy all zones |
| `GetZonesByType(type)`                        | Get zones by type |

#### Player Zone Functions

| Function                           | Description                          |
| ---------------------------------- | ------------------------------------ |
| `GetPlayerZones(playerId)`         | Get zones player is in               |
| `GetHighestPriorityZone(playerId)` | Get highest priority zone for player |
| `IsPlayerInAnyZone(playerId)`      | Check if player is in any zone       |
| `GetZonePlayerCount(zoneId)`       | Get player count in zone             |

#### Loot Selection Functions

| Function                                         | Description                     |
| ------------------------------------------------ | ------------------------------- |
| `SelectLootFromPlayerZones(playerId, tableName)` | Select from specific loot table |
| `SelectRandomLootTableFromPlayerZones(playerId)` | Select from any loot table      |

#### Zone-specific Settings

| Function                                           | Description                              |
| -------------------------------------------------- | ---------------------------------------- |
| `GetZoneChargeSettings(playerId, resourceType)`    | Get charge settings for player's zone    |
| `GetZoneDepletionSettings(playerId, resourceType)` | Get depletion settings for player's zone |

### Examples

#### Mining System with Zone-specific Ore

```lua
local RNG = exports.alentexas_rng

-- Create different mining zones
local valentineMine = RNG.CreateZone("valentine_mine", "Valentine Mine", "circle",
    vector3(-1823.0, -433.0, 160.0), {
        radius = 50.0,
        priority = 2,
        influenceModifiers = {
            ["iron_ore"] = 2.0,  -- Double iron ore
            ["coal"] = 1.5       -- 50% more coal
        }
    })

local strawberryMine = RNG.CreateZone("strawberry_mine", "Strawberry Mine", "circle",
    vector3(-1760.0, -224.0, 181.0), {
        radius = 40.0,
        priority = 3,
        influenceModifiers = {
            ["gold_ore"] = 4.0,    -- Quadruple gold ore
            ["silver_ore"] = 2.5   -- 2.5x silver ore
        }
    })

-- Create mining loot table
local miningLoot = RNG.CreateProbabilityList()
miningLoot:Add("iron_ore", 0.4)
miningLoot:Add("coal", 0.3)
miningLoot:Add("silver_ore", 0.2)
miningLoot:Add("gold_ore", 0.1)

-- Add loot table to zones
valentineMine:AddLootTable("mining", miningLoot, 1.0)
strawberryMine:AddLootTable("mining", miningLoot, 1.0)

-- Mining event handler
RegisterNetEvent("mining:mine")
AddEventHandler("mining:mine", function()
    local _source = source
    
    -- Check if player is in a mining zone
    if not RNG.IsPlayerInAnyZone(_source) then
        TriggerClientEvent("vorp:Tip", _source, "You need to be in a mining area.", 4000)
        return
    end
    
    -- Get ore from player's zone (with zone-specific modifiers applied)
    local ore, zone = RNG.SelectLootFromPlayerZones(_source, "mining")
    
    if ore and zone then
        -- Give ore to player
        TriggerServerEvent("vorp_inventory:giveItem", _source, ore, 1)
        TriggerClientEvent("vorp:TipRight", _source, 
            string.format("Mined %s in %s", ore, zone.name), 5000)
    else
        TriggerClientEvent("vorp:Tip", _source, "Nothing found.", 4000)
    end
end)
```

#### Dynamic Event Zones

```lua
-- Create temporary event zone
function CreateEventZone(eventType, coords, duration)
    local zoneId = "event_" .. eventType .. "_" .. os.time()
    local zoneName = "Event: " .. eventType
    
    local zone = RNG.CreateZone(zoneId, zoneName, "circle", coords, {
        radius = 25.0,
        priority = 10, -- Highest priority
        debug = true,
        influenceModifiers = {
            ["*"] = 3.0 -- Triple all loot
        }
    })
    
    if zone then
        -- Add special event loot
        local eventLoot = RNG.CreateProbabilityList()
        eventLoot:Add("event_token", 0.5)
        eventLoot:Add("rare_collectible", 0.3)
        eventLoot:Add("event_exclusive", 0.2)
        
        zone:AddLootTable("event", eventLoot, 2.0)
        
        -- Auto-destroy after duration
        Citizen.SetTimeout(duration * 1000, function()
            RNG.DestroyZone(zoneId)
            print("Event zone expired: " .. zoneName)
        end)
    end
end

-- Trigger event zone creation
RegisterNetEvent("admin:createEventZone")
AddEventHandler("admin:createEventZone", function(eventType, duration)
    local _source = source
    local coords = GetEntityCoords(GetPlayerPed(_source))
    CreateEventZone(eventType, coords, duration or 300) -- 5 minutes default
end)
```

#### Territory Control System

```lua
-- Different loot tables for different gang territories
local territories = {
    {
        id = "gang_a_territory",
        name = "Gang A Territory", 
        coords = vector3(100.0, 200.0, 50.0),
        radius = 100.0,
        priority = 2,
        lootBonus = {
            ["weapon_parts"] = 2.0,
            ["ammunition"] = 1.5
        }
    },
    {
        id = "gang_b_territory",
        name = "Gang B Territory",
        coords = vector3(500.0, 600.0, 75.0), 
        radius = 120.0,
        priority = 2,
        lootBonus = {
            ["drugs"] = 3.0,
            ["money"] = 1.8
        }
    }
}

-- Initialize territory zones
for _, territory in ipairs(territories) do
    local zone = RNG.CreateZone(territory.id, territory.name, "circle", 
        territory.coords, {
            radius = territory.radius,
            priority = territory.priority,
            influenceModifiers = territory.lootBonus
        })
    
    if zone then
        -- Add territory-specific loot tables
        local territoryLoot = RNG.CreateProbabilityList()
        -- ... add items based on territory type
        zone:AddLootTable("territory", territoryLoot, 1.0)
    end
end
```

### Best Practices

#### Performance Optimization

1. **Limit Zone Count**: Don't create too many zones as each one requires PolyZone processing
2. **Use Appropriate Zone Types**: Circle zones are most efficient, polygon zones are most expensive
3. **Set Reasonable Priorities**: Use a limited range (1-10) for priorities
4. **Clean Up Temporary Zones**: Always destroy event or temporary zones when done

#### Zone Design

1. **Avoid Excessive Overlap**: Too many overlapping zones can be confusing
2. **Use Clear Priorities**: Higher priority zones should be more specific/valuable
3. **Provide Visual Feedback**: Use debug mode during development
4. **Test Zone Boundaries**: Ensure zones cover the intended areas

#### Loot Table Management

1. **Share Common Tables**: Reuse loot tables across similar zones
2. **Use Influence Modifiers**: Prefer modifiers over separate loot tables for variations
3. **Balance Probabilities**: Consider how zone modifiers affect overall game balance
4. **Document Zone Purposes**: Keep track of what each zone is intended for

#### Error Handling

```lua
-- Always check if zone creation succeeded
local zone = RNG.CreateZone(id, name, type, coords, options)
if not zone then
    print("Failed to create zone: " .. name)
    return
end

-- Check if player is in zones before selecting loot
if not RNG.IsPlayerInAnyZone(playerId) then
    -- Handle player not in any zone
    return
end

-- Validate loot selection results
local item, zone = RNG.SelectLootFromPlayerZones(playerId, "mining")
if not item or not zone then
    -- Handle no loot available
    return
end
```

The Zone Management System provides powerful tools for creating location-based gameplay mechanics while maintaining performance and flexibility. Use it to create immersive area-specific experiences that enhance your server's gameplay.
