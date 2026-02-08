---
icon: webhook
---

# API Reference

This document provides a complete reference for all functions and methods available in the RNGEngine system.

### Core Functions

#### Creating Objects

| Function                                   | Description                                                         | Parameters                                                                                | Returns                |
| ------------------------------------------ | ------------------------------------------------------------------- | ----------------------------------------------------------------------------------------- | ---------------------- |
| `CreateProbabilityList()`                  | Creates a new probability list for storing items with their weights | None                                                                                      | ProbabilityList object |
| `CreateProbabilityItem(data, probability)` | Creates a standalone probability item that can be added to a list   | <p><code>data</code>: Any value or table<br><code>probability</code>: Number (weight)</p> | ProbabilityItem object |
| `CreateCollection()`                       | Creates a new collection for managing multiple probability lists    | None                                                                                      | Collection object      |

#### Selection Functions

| Function                                                  | Description                                                                 | Parameters                                                                                                | Returns                                                                                         |
| --------------------------------------------------------- | --------------------------------------------------------------------------- | --------------------------------------------------------------------------------------------------------- | ----------------------------------------------------------------------------------------------- |
| `SelectRandomItem(list, includeData)`                     | Selects a random item from a list based on probability weights              | <p><code>list</code>: ProbabilityList<br><code>includeData</code>: Boolean (optional, default false)</p>  | <p>When includeData=false: Data value only<br>When includeData=true: ProbabilityItem object</p> |
| `SelectRandomItemFromCollection(collection, includeData)` | Selects a random item from any list in the collection based on list weights | <p><code>collection</code>: Collection<br><code>includeData</code>: Boolean (optional, default false)</p> | Selected data or ProbabilityItem                                                                |

#### Selection Method Configuration

| Function                                                  | Description                                                                                      | Parameters                                                                                                                                            | Returns |
| --------------------------------------------------------- | ------------------------------------------------------------------------------------------------ | ----------------------------------------------------------------------------------------------------------------------------------------------------- | ------- |
| `UseCumulativeMethod(list)`                               | Sets selection method to Cumulative (default) - fastest but may have slight bias for first items | `list`: ProbabilityList                                                                                                                               | None    |
| `UseLinearMethod(list)`                                   | Sets selection method to Linear - balanced performance with no bias                              | `list`: ProbabilityList                                                                                                                               | None    |
| `UseRandomMethod(list)`                                   | Sets selection method to Random - most random but slower for large lists                         | `list`: ProbabilityList                                                                                                                               | None    |
| `UsePreventRepeatMethod(list, method, shuffleIterations)` | Sets the method for preventing repeated selections                                               | <p><code>list</code>: ProbabilityList<br><code>method</code>: PreventRepeatMethod enum (0-3)<br><code>shuffleIterations</code>: Number (optional)</p> | None    |

#### Item Management Functions

| Function                         | Description                                                                  | Parameters                                                                    | Returns         |
| -------------------------------- | ---------------------------------------------------------------------------- | ----------------------------------------------------------------------------- | --------------- |
| `SetDepletion(item, isDepleted)` | Marks an item as depleted (temporarily removed from selection) or undepleted | <p><code>item</code>: ProbabilityItem<br><code>isDepleted</code>: Boolean</p> | None            |
| `ResetAllDepletions(list)`       | Resets all depletion flags in a list (makes all items available)             | `list`: ProbabilityList                                                       | None            |
| `GetChosenItemIndex(list)`       | Gets the index of the most recently selected item                            | `list`: ProbabilityList                                                       | Number (index)  |
| `GetChosenItem(list)`            | Gets the most recently selected item object                                  | `list`: ProbabilityList                                                       | ProbabilityItem |

#### Advanced Depletion Functions

| Function                                                                         | Description                                                             | Parameters                                                                                                                                                                                                                      | Returns                                                                                                      |
| -------------------------------------------------------------------------------- | ----------------------------------------------------------------------- | ------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- | ------------------------------------------------------------------------------------------------------------ |
| `EnableAdvancedDepletion(list, maxUnits, rechargeTime, autoRecharge)`            | Enables advanced depletion for all items in a list                      | <p><code>list</code>: ProbabilityList<br><code>maxUnits</code>: Number (max units per item)<br><code>rechargeTime</code>: Number (minutes per unit recharge)<br><code>autoRecharge</code>: Boolean (optional, default true)</p> | None                                                                                                         |
| `EnableItemAdvancedDepletion(list, index, maxUnits, rechargeTime, autoRecharge)` | Enables advanced depletion for a specific item                          | <p><code>list</code>: ProbabilityList<br><code>index</code>: Number (item index)<br><code>maxUnits</code>: Number<br><code>rechargeTime</code>: Number<br><code>autoRecharge</code>: Boolean</p>                                | Boolean (success)                                                                                            |
| `DisableAdvancedDepletion(list)`                                                 | Disables advanced depletion for all items (reverts to simple depletion) | `list`: ProbabilityList                                                                                                                                                                                                         | None                                                                                                         |
| `ProcessRecharges(list)`                                                         | Manually processes recharges for all items with advanced depletion      | `list`: ProbabilityList                                                                                                                                                                                                         | Number (items recharged)                                                                                     |
| `UseItemUnits(list, index, amount)`                                              | Uses units from a specific item by index                                | <p><code>list</code>: ProbabilityList<br><code>index</code>: Number<br><code>amount</code>: Number (optional, default 1)</p>                                                                                                    | Boolean (success), String (message)                                                                          |
| `UseUnitsFromValue(list, value, amount)`                                         | Uses units from a specific item by value                                | <p><code>list</code>: ProbabilityList<br><code>value</code>: Any (item value)<br><code>amount</code>: Number (optional, default 1)</p>                                                                                          | Boolean (success), String (message)                                                                          |
| `GetItemUnitsInfo(list, index)`                                                  | Gets current units info for a specific item by index                    | <p><code>list</code>: ProbabilityList<br><code>index</code>: Number</p>                                                                                                                                                         | Number (current units), Number (max units), Number (time to next recharge in seconds), Boolean (is depleted) |
| `GetUnitsInfoFromValue(list, value)`                                             | Gets current units info for a specific item by value                    | <p><code>list</code>: ProbabilityList<br><code>value</code>: Any (item value)</p>                                                                                                                                               | Number (current units), Number (max units), Number (time to next recharge in seconds), Boolean (is depleted) |
| `ResetAllUnits(list)`                                                            | Resets all items to maximum units                                       | `list`: ProbabilityList                                                                                                                                                                                                         | None                                                                                                         |
| `SetAllRechargeTime(list, minutes)`                                              | Sets recharge time for all items with advanced depletion                | <p><code>list</code>: ProbabilityList<br><code>minutes</code>: Number</p>                                                                                                                                                       | None                                                                                                         |

#### History Functions

| Function                                   | Description                                                                 | Parameters                                                                                         | Returns                 |
| ------------------------------------------ | --------------------------------------------------------------------------- | -------------------------------------------------------------------------------------------------- | ----------------------- |
| `EnableHistoryTracking(list, historySize)` | Enables tracking of selection history to prevent repetition                 | <p><code>list</code>: ProbabilityList<br><code>historySize</code>: Number (max items to track)</p> | None                    |
| `DisableHistoryTracking(list)`             | Disables history tracking for a list                                        | `list`: ProbabilityList                                                                            | None                    |
| `AvoidRepeats(list, count)`                | Prevents recent selections from repeating for the specified number of picks | <p><code>list</code>: ProbabilityList<br><code>count</code>: Number (items to avoid)</p>           | None                    |
| `GetLastPickedItem(list)`                  | Gets the most recently selected item                                        | `list`: ProbabilityList                                                                            | Data or ProbabilityItem |

#### Collection Management

| Function                                          | Description                                                       | Parameters                                                                                                        | Returns |
| ------------------------------------------------- | ----------------------------------------------------------------- | ----------------------------------------------------------------------------------------------------------------- | ------- |
| `AddListToCollection(collection, listName, list)` | Adds a probability list to a collection with a unique name        | <p><code>collection</code>: Collection<br><code>listName</code>: String<br><code>list</code>: ProbabilityList</p> | None    |
| `NormalizeWeights(collection)`                    | Normalizes the weights of all lists in a collection to sum to 1.0 | `collection`: Collection                                                                                          | None    |

#### Zone Management Functions

| Function                                          | Description                                  | Parameters                                                                                                                                                                                                                                                               | Returns            |
| ------------------------------------------------- | -------------------------------------------- | ------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------ | ------------------ |
| `CreateZone(id, name, zoneType, coords, options)` | Creates a new zone with PolyZone integration | <p><code>id</code>: String (unique zone ID)<br><code>name</code>: String (display name)<br><code>zoneType</code>: String ("circle", "box", "poly", "combo")<br><code>coords</code>: Vector3 (center coordinates)<br><code>options</code>: Table (zone configuration)</p> | Zone object or nil |
| `GetZone(id)`                                     | Gets a zone by its ID                        | `id`: String (zone ID)                                                                                                                                                                                                                                                   | Zone object or nil |
| `GetAllZones()`                                   | Gets all active zones                        | None                                                                                                                                                                                                                                                                     | Table of zones     |
| `DestroyZone(id)`                                 | Destroys a zone and removes all players      | `id`: String (zone ID)                                                                                                                                                                                                                                                   | Boolean (success)  |
| `DestroyAllZones()`                               | Destroys all zones                           | None                                                                                                                                                                                                                                                                     | None               |
| `GetZonesByType(zoneType)`                        | Gets all zones of a specific type            | `zoneType`: String                                                                                                                                                                                                                                                       | Table of zones     |

#### Player Zone Functions

| Function                           | Description                                 | Parameters                     | Returns            |
| ---------------------------------- | ------------------------------------------- | ------------------------------ | ------------------ |
| `GetPlayerZones(playerId)`         | Gets all zones a player is currently in     | `playerId`: Number (server ID) | Table of zones     |
| `GetHighestPriorityZone(playerId)` | Gets the highest priority zone for a player | `playerId`: Number (server ID) | Zone object or nil |
| `IsPlayerInAnyZone(playerId)`      | Checks if player is in any zone             | `playerId`: Number (server ID) | Boolean            |
| `GetZonePlayerCount(zoneId)`       | Gets number of players in a zone            | `zoneId`: String (zone ID)     | Number             |

#### Zone Loot Selection Functions

| Function                                             | Description                                          | Parameters                                                                                          | Returns                                     |
| ---------------------------------------------------- | ---------------------------------------------------- | --------------------------------------------------------------------------------------------------- | ------------------------------------------- |
| `SelectLootFromPlayerZones(playerId, lootTableName)` | Selects loot from a specific table in player's zones | <p><code>playerId</code>: Number (server ID)<br><code>lootTableName</code>: String (table name)</p> | Item data, Zone object                      |
| `SelectRandomLootTableFromPlayerZones(playerId)`     | Selects loot from any table in player's zones        | `playerId`: Number (server ID)                                                                      | Item data, Zone object, String (table name) |

### ProbabilityList Methods

| Method                     | Description                                                                   | Parameters                                                                                | Returns         |
| -------------------------- | ----------------------------------------------------------------------------- | ----------------------------------------------------------------------------------------- | --------------- |
| `Add(data, probability)`   | Adds an item to the list with the specified data and probability weight       | <p><code>data</code>: Any value or table<br><code>probability</code>: Number (weight)</p> | None            |
| `GetItemByIndex(index)`    | Gets an item at the specified index (0-based)                                 | `index`: Number                                                                           | ProbabilityItem |
| `RemoveItem(index)`        | Removes an item from the list at the specified index                          | `index`: Number                                                                           | None            |
| `Clear()`                  | Removes all items from the list                                               | None                                                                                      | None            |
| `GetTotalProbability()`    | Gets the sum of all item probabilities in the list                            | None                                                                                      | Number          |
| `ItemCount()`              | Gets the number of items in the list                                          | None                                                                                      | Number          |
| `NormalizeProbabilities()` | Normalizes all probabilities to sum to 1.0 for easier percentage calculations | None                                                                                      | None            |

### Collection Methods

| Method                            | Description                                           | Parameters                                                          | Returns         |
| --------------------------------- | ----------------------------------------------------- | ------------------------------------------------------------------- | --------------- |
| `GetList(listName)`               | Gets a probability list from the collection by name   | `listName`: String                                                  | ProbabilityList |
| `SetListWeight(listName, weight)` | Sets the selection weight of a list in the collection | <p><code>listName</code>: String<br><code>weight</code>: Number</p> | None            |
| `GetListWeight(listName)`         | Gets the weight of a list in the collection           | `listName`: String                                                  | Number          |
| `RemoveList(listName)`            | Removes a list from the collection                    | `listName`: String                                                  | None            |
| `GetTotalWeight()`                | Gets sum of all list weights in the collection        | None                                                                | Number          |
| `GetListCount()`                  | Gets number of lists in the collection                | None                                                                | Number          |
| `HasList(listName)`               | Checks if a list exists in the collection             | `listName`: String                                                  | Boolean         |
| `NormalizeWeights()`              | Normalizes all list weights to sum to 1.0             | None                                                                | None            |

### Zone Methods

| Method                                     | Description                               | Parameters                                                                                        | Returns                |
| ------------------------------------------ | ----------------------------------------- | ------------------------------------------------------------------------------------------------- | ---------------------- |
| `AddLootTable(tableName, probabilityList)` | Adds a loot table to the zone             | <p><code>tableName</code>: String<br><code>probabilityList</code>: ProbabilityList</p>            | None                   |
| `GetLootTable(tableName)`                  | Gets a loot table from the zone           | `tableName`: String                                                                               | ProbabilityList or nil |
| `RemoveLootTable(tableName)`               | Removes a loot table from the zone        | `tableName`: String                                                                               | Boolean (success)      |
| `SetLootTableWeight(tableName, weight)`    | Sets the weight of a loot table           | <p><code>tableName</code>: String<br><code>weight</code>: Number</p>                              | None                   |
| `GetLootTableWeight(tableName)`            | Gets the weight of a loot table           | `tableName`: String                                                                               | Number                 |
| `SetInfluenceModifier(itemData, modifier)` | Sets influence modifier for specific item | <p><code>itemData</code>: Any (item identifier)<br><code>modifier</code>: Number (multiplier)</p> | None                   |
| `GetInfluenceModifier(itemData)`           | Gets influence modifier for item          | `itemData`: Any (item identifier)                                                                 | Number                 |
| `SetPriority(priority)`                    | Sets zone priority for overlapping zones  | `priority`: Number                                                                                | None                   |
| `GetPriority()`                            | Gets zone priority                        | None                                                                                              | Number                 |
| `IsPlayerInZone(playerId)`                 | Checks if player is in this zone          | `playerId`: Number (server ID)                                                                    | Boolean                |
| `GetPlayersInZone()`                       | Gets all players currently in zone        | None                                                                                              | Table of player IDs    |
| `GetPlayerCount()`                         | Gets number of players in zone            | None                                                                                              | Number                 |
| `SetChargeSettings(settings)`              | Sets zone-specific charge settings        | `settings`: Table (charge configuration)                                                          | None                   |
| `GetChargeSettings()`                      | Gets zone charge settings                 | None                                                                                              | Table                  |

### Persistence System

The persistence system allows saving and loading RNG states to/from the database. All persistence functions are accessed through the `Persistence` export:

```lua
local Persistence = exports.alentexas_rng.Persistence
```

#### Initialization

| Function                            | Description                               | Parameters                                                                                                                                           | Returns |
| ----------------------------------- | ----------------------------------------- | ---------------------------------------------------------------------------------------------------------------------------------------------------- | ------- |
| `Initialize(resourceName, options)` | Initializes persistence for your resource | <p><code>resourceName</code>: String (your resource folder name)<br><code>options</code>: Number (hours until reset) or Table with configuration</p> | None    |

#### Basic Persistence Functions

| Function                                | Description                                  | Parameters                                                                                                                                               | Returns           |
| --------------------------------------- | -------------------------------------------- | -------------------------------------------------------------------------------------------------------------------------------------------------------- | ----------------- |
| `SaveList(identifier, list, resetTime)` | Saves a probability list's state to database | <p><code>identifier</code>: String (unique ID)<br><code>list</code>: ProbabilityList<br><code>resetTime</code>: Number (optional, hours until reset)</p> | None              |
| `LoadList(identifier, list)`            | Loads a list's state from database           | <p><code>identifier</code>: String<br><code>list</code>: ProbabilityList</p>                                                                             | Boolean (success) |
| `SaveData(identifier, data, resetTime)` | Saves custom data to database                | <p><code>identifier</code>: String<br><code>data</code>: Any serializable value<br><code>resetTime</code>: Number (optional, hours)</p>                  | None              |
| `LoadData(identifier)`                  | Loads custom data from database              | `identifier`: String                                                                                                                                     | Data or nil       |

#### Management Functions

| Function                | Description                                      | Parameters           | Returns |
| ----------------------- | ------------------------------------------------ | -------------------- | ------- |
| `ResetList(identifier)` | Resets a specific list's data (clears depletion) | `identifier`: String | None    |
| `ResetAll()`            | Resets all data for the resource                 | None                 | None    |
| `SetDebugMode(enabled)` | Enables or disables debug messages               | `enabled`: Boolean   | None    |

### Enums

#### PreventRepeatMethod

| Name      | Value | Description                    |
| --------- | ----- | ------------------------------ |
| `Off`     | 0     | No prevention of repeats       |
| `Spread`  | 1     | Fast prevention with some bias |
| `Repick`  | 2     | Moderate speed, low bias       |
| `Shuffle` | 3     | Slowest but most accurate      |

### Specialized Systems

RNGEngine includes specialized systems built on top of the core functionality:

| System           | Description                                                                 | Documentation    |
| ---------------- | --------------------------------------------------------------------------- | ---------------- |
| Zone Management  | PolyZone integration for area-specific loot tables with influence modifiers | Zone Management  |
| Lootbox System   | Advanced tiered loot system with rarity, prevention, and time-based resets  | Lootbox System   |
| Lumbering System | Complete woodcutting mechanic with tree types, influencers, and persistence | Lumbering System |

### Events

#### Client-Side Events

| Event                                  | Description                             | Parameters                                                                                                                                                      |
| -------------------------------------- | --------------------------------------- | --------------------------------------------------------------------------------------------------------------------------------------------------------------- |
| `alentexas_rng:requestRandomSelection` | Requests a server-side random selection | <p><code>listData</code>: Table with RNG configuration<br><code>includeData</code>: Boolean<br><code>callbackEvent</code>: String (event name for response)</p> |

#### Zone Events

| Event                             | Description                          | Parameters                                                                         |
| --------------------------------- | ------------------------------------ | ---------------------------------------------------------------------------------- |
| `alentexas_rng:playerEnteredZone` | Triggered when player enters a zone  | <p><code>playerId</code>: Number (server ID)<br><code>zone</code>: Zone object</p> |
| `alentexas_rng:playerExitedZone`  | Triggered when player exits a zone   | <p><code>playerId</code>: Number (server ID)<br><code>zone</code>: Zone object</p> |
| `alentexas_rng:zoneCreated`       | Triggered when a new zone is created | `zone`: Zone object                                                                |
| `alentexas_rng:zoneDestroyed`     | Triggered when a zone is destroyed   | `zoneId`: String (zone ID)                                                         |

#### Server-Side Events

The system automatically handles client requests without requiring manual event registration.

### Common Usage Patterns

#### Basic Item Selection

```lua
local RNG = exports.alentexas_rng

-- Create and populate a list
local lootTable = RNG.CreateProbabilityList()
lootTable:Add("Gold Nugget", 10)
lootTable:Add("Silver Watch", 20)
lootTable:Add("Tobacco", 30)
lootTable:Add("Whiskey", 40)

-- Select a random item
local item = RNG.SelectRandomItem(lootTable)
```

#### Working with Collections

```lua
local RNG = exports.alentexas_rng

-- Create a collection
local treasureCollection = RNG.CreateCollection()

-- Add lists to the collection
local commonList = RNG.CreateProbabilityList()
commonList:Add("Common Item 1", 1)
commonList:Add("Common Item 2", 1)

local rareList = RNG.CreateProbabilityList()
rareList:Add("Rare Item 1", 1)
rareList:Add("Rare Item 2", 1)

RNG.AddListToCollection(treasureCollection, "common", commonList)
treasureCollection:SetListWeight("common", 80)

RNG.AddListToCollection(treasureCollection, "rare", rareList)
treasureCollection:SetListWeight("rare", 20)

-- Select from the collection
local item = RNG.SelectRandomItemFromCollection(treasureCollection)
```

#### Using Persistence

```lua
local RNG = exports.alentexas_rng
local Persistence = RNG.Persistence

-- Initialize persistence
Persistence.Initialize("my_resource", {
    resetTime = 24,     -- Default reset time in hours
    saveInterval = 30,  -- Save to database every 30 minutes
    debugMode = false   -- Disable debug messages
})

-- Create and save a list
local lootTable = RNG.CreateProbabilityList()
lootTable:Add("Item 1", 10)
lootTable:Add("Item 2", 20)

-- Mark an item as depleted
local item = lootTable:GetItemByIndex(1)
RNG.SetDepletion(item, true)

-- Save to database with custom reset time
Persistence.SaveList("chest_1", lootTable, 48) -- 48 hour reset

-- Later, load from database
Persistence.LoadList("chest_1", lootTable)
```

#### Using Zone Management

```lua
local RNG = exports.alentexas_rng

-- Create a mining zone
local miningZone = RNG.CreateZone("mine_1", "Gold Mine", "circle", vector3(100.0, 200.0, 30.0), {
    radius = 50.0,
    priority = 1,
    debugPoly = false
})

if miningZone then
    -- Create loot tables for the zone
    local commonOres = RNG.CreateProbabilityList()
    commonOres:Add("iron_ore", 50)
    commonOres:Add("copper_ore", 30)
    commonOres:Add("coal", 20)
    
    local rareOres = RNG.CreateProbabilityList()
    rareOres:Add("gold_ore", 60)
    rareOres:Add("silver_ore", 30)
    rareOres:Add("diamond", 10)
    
    -- Add loot tables to zone
    miningZone:AddLootTable("common", commonOres)
    miningZone:SetLootTableWeight("common", 80)
    
    miningZone:AddLootTable("rare", rareOres)
    miningZone:SetLootTableWeight("rare", 20)
    
    -- Set influence modifiers for specific items
    miningZone:SetInfluenceModifier("gold_ore", 1.5) -- 50% bonus for gold in this zone
    miningZone:SetInfluenceModifier("diamond", 0.5)  -- 50% penalty for diamond
    
    -- Listen for zone events
    AddEventHandler('alentexas_rng:playerEnteredZone', function(playerId, zone)
        if zone.id == "mine_1" then
            print("Player " .. playerId .. " entered the gold mine!")
        end
    end)
    
    -- Select loot when player mines
    local function onPlayerMine(playerId)
        local item, zone = RNG.SelectLootFromPlayerZones(playerId, "common")
        if item and zone then
            print("Player found: " .. item .. " in zone: " .. zone.name)
        end
    end
end
```
