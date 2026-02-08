---
icon: database
---

# Persistence System

The persistence module in RNGEngine allows you to save and restore the state of your RNG systems, including depletion states, across server restarts or after specific time intervals.

### Overview

The persistence system provides:

* Database storage for RNG state data
* Automatic time-based reset of depleted items
* Resource-specific storage to avoid conflicts
* Simple API for saving and loading states
* Support for per-user depletion states

### Setting Up Persistence

#### Initialize in Your Resource

```lua
-- In your server script
local RNG = exports.alentexas_rng
local Persistence = RNG.Persistence

-- Initialize persistence for your resource with a custom reset time (hours)
Persistence.Initialize("my_resource_name", 24) -- Reset after 24 hours
```

#### Configuration Options

You can configure the persistence system with these parameters:

```lua
-- Initialize with custom settings
Persistence.Initialize("my_resource_name", {
    resetTime = 12, -- Reset after 12 hours
    saveInterval = 15, -- Save changes every 15 minutes
    debugMode = false -- Disable debug messages
})
```

### Saving RNG States

#### Save a Probability List

```lua
-- Create and populate a probability list
local lootTable = RNG.CreateProbabilityList()
lootTable:Add("Gold Nugget", 10)
lootTable:Add("Silver Watch", 20)
lootTable:Add("Tobacco", 30)
lootTable:Add("Whiskey", 40)

-- Mark some items as depleted
local goldItem = lootTable:GetItemByIndex(1)
RNG.SetDepletion(goldItem, true)

-- Save the state of the loot table with a unique identifier
Persistence.SaveList("town_chest_1", lootTable)
```

#### Save Custom Data

```lua
-- Save any custom data with the persistence system
local customData = {
    lastOpened = os.time(),
    openedBy = "player123",
    timesLooted = 5
}

Persistence.SaveData("town_chest_1_metadata", customData)
```

### Loading RNG States

#### Load a Probability List

```lua
-- Create a new probability list
local lootTable = RNG.CreateProbabilityList()
lootTable:Add("Gold Nugget", 10)
lootTable:Add("Silver Watch", 20)
lootTable:Add("Tobacco", 30)
lootTable:Add("Whiskey", 40)

-- Load saved state (depletion flags) from database
Persistence.LoadList("town_chest_1", lootTable)

-- Now the lootTable has the same depletion states as when it was saved
```

#### Load Custom Data

```lua
-- Load custom data
local metadata = Persistence.LoadData("town_chest_1_metadata")

if metadata then
    print("Chest was last opened by: " .. metadata.openedBy)
    print("Times looted: " .. metadata.timesLooted)
end
```

### Time-Based Reset

The persistence system automatically resets depleted items after the configured time has elapsed.

```lua
-- Initialize with a 3-hour reset time
Persistence.Initialize("my_resource_name", 3)

-- Items will automatically become undepleted after 3 hours
-- The system compares the current time with the time when items were depleted
```

#### Manual Reset

```lua
-- Force a reset of a specific list
Persistence.ResetList("town_chest_1")

-- Or reset all data for your resource
Persistence.ResetAll()
```

### Resource-Specific Storage

Each resource using the persistence system has its own namespace in the database to avoid conflicts.

```lua
-- In resource A
local Persistence = exports.alentexas_rng.Persistence
Persistence.Initialize("resourceA", 24)
Persistence.SaveList("chest_1", chestList) -- Saved as "resourceA:chest_1"

-- In resource B
local Persistence = exports.alentexas_rng.Persistence
Persistence.Initialize("resourceB", 24)
Persistence.SaveList("chest_1", anotherList) -- Saved as "resourceB:chest_1"

-- No conflict between the two resources
```

### Per-User Depletion

You can implement per-user depletion by incorporating the player's identifier into your persistence keys. This allows different players to have their own individual depletion states for the same game object.

#### Creating Per-User Identifiers

```lua
-- In your server script
local RNG = exports.alentexas_rng
local Persistence = RNG.Persistence

-- Initialize persistence
Persistence.Initialize("my_loot_system", 6) -- 6 hour reset

-- When a player interacts with a lootable container
RegisterNetEvent("loot:openContainer")
AddEventHandler("loot:openContainer", function(containerId)
    local source = source
    local playerCharId = exports.vorp_core:getUser(source).getUsedCharacter.charIdentifier
    
    -- Create a player-specific identifier for this container
    local playerContainerId = string.format("container_%s_player_%s", containerId, playerCharId)
    
    -- The rest of your loot code...
end)
```

#### Implementing Per-User Looting

```lua
-- Complete example for a chest that tracks depletion separately for each player
local RNG = exports.alentexas_rng
local Persistence = RNG.Persistence

-- Initialize persistence
Persistence.Initialize("my_loot_system", 6)

-- Create loot table
local treasureChest = RNG.CreateProbabilityList()
treasureChest:Add("Gold Nugget", 10)
treasureChest:Add("Silver Pocket Watch", 20)
treasureChest:Add("Valuable Document", 30)
treasureChest:Add("Money Clip", 40)

-- When player interacts with a chest
RegisterNetEvent("loot:openChest")
AddEventHandler("loot:openChest", function(chestId)
    local source = source
    local playerCharId = exports.vorp_core:getUser(source).getUsedCharacter.charIdentifier
    
    -- Create a player-specific identifier for this chest
    local playerChestId = string.format("chest_%s_player_%s", chestId, playerCharId)
    
    -- Make a copy of the base loot table for this player
    local playerLoot = RNG.CreateProbabilityList()
    for i = 1, treasureChest:GetItemCount() do
        local item = treasureChest:GetItemByIndex(i)
        playerLoot:Add(item.Data, item.Probability)
    end
    
    -- Load this player's saved state for this chest
    Persistence.LoadList(playerChestId, playerLoot)
    
    -- Select a random item
    local item = RNG.SelectRandomItem(playerLoot)
    
    if item then
        -- Mark the item as depleted for this player
        local selectedIndex = RNG.GetChosenItemIndex(playerLoot)
        local selectedItem = playerLoot:GetItemByIndex(selectedIndex)
        RNG.SetDepletion(selectedItem, true)
        
        -- Save this player's updated state for this chest
        Persistence.SaveList(playerChestId, playerLoot)
        
        -- Give item to player
        TriggerClientEvent("loot:receiveItem", source, item)
    else
        -- All items depleted for this player
        TriggerClientEvent("loot:empty", source)
    end
end)
```

#### Checking Global vs. Per-User Depletion

You can implement both global and per-user depletion by checking both states:

```lua
-- Example showing both global and per-user depletion
local RNG = exports.alentexas_rng
local Persistence = RNG.Persistence

-- Initialize persistence
Persistence.Initialize("my_resource", 24)

-- Function to handle resource gathering with both global and per-user limits
function GatherResource(playerId, resourceId, resourceType)
    local playerCharId = exports.vorp_core:getUser(playerId).getUsedCharacter.charIdentifier
    
    -- Create identifiers
    local globalResourceId = string.format("%s_%s", resourceType, resourceId)
    local playerResourceId = string.format("%s_%s_player_%s", resourceType, resourceId, playerCharId)
    
    -- Create and load global resource state (affects all players)
    local globalResource = RNG.CreateProbabilityList()
    globalResource:Add("Resource Available", 1)
    Persistence.LoadList(globalResourceId, globalResource)
    
    -- Check if the resource is globally depleted
    local globalItem = globalResource:GetItemByIndex(1)
    local isGloballyDepleted = globalItem and globalItem.IsDepleted
    
    if isGloballyDepleted then
        return "This resource has been depleted and needs time to replenish."
    end
    
    -- Create and load player-specific resource state
    local playerResource = RNG.CreateProbabilityList()
    playerResource:Add("Resource Available", 1)
    Persistence.LoadList(playerResourceId, playerResource)
    
    -- Check if the player has already gathered this resource
    local playerItem = playerResource:GetItemByIndex(1)
    local isPlayerDepleted = playerItem and playerItem.IsDepleted
    
    if isPlayerDepleted then
        return "You've already gathered this resource recently."
    end
    
    -- Resource is available to this player
    -- Mark as depleted for this player
    RNG.SetDepletion(playerItem, true)
    Persistence.SaveList(playerResourceId, playerResource)
    
    -- Track global gathering count
    local metadata = Persistence.LoadData(globalResourceId .. "_metadata") or {
        timesGathered = 0,
        lastGatheredTime = 0
    }
    
    metadata.timesGathered = metadata.timesGathered + 1
    metadata.lastGatheredTime = os.time()
    Persistence.SaveData(globalResourceId .. "_metadata", metadata)
    
    -- If resource has been gathered X times, deplete it globally
    if metadata.timesGathered >= 10 then -- For example, deplete after 10 gatherings
        RNG.SetDepletion(globalItem, true)
        Persistence.SaveList(globalResourceId, globalResource)
    end
    
    return "You successfully gathered the resource!"
end

-- Event handler
RegisterNetEvent("gather:resource")
AddEventHandler("gather:resource", function(resourceId, resourceType)
    local source = source
    local result = GatherResource(source, resourceId, resourceType)
    TriggerClientEvent("gather:result", source, result)
end)
```

This approach allows for sophisticated resource management where:

* Each player can gather a resource once (per-user depletion)
* After a certain number of players have gathered the resource, it becomes depleted for everyone (global depletion)
* Both depletion states reset based on their configured timers

### Practical Examples

#### Lootable Containers with Respawn

```lua
local RNG = exports.alentexas_rng
local Persistence = RNG.Persistence

-- Initialize with 6 hour reset
Persistence.Initialize("my_loot_system", 6)

-- Create loot table
local cabinetLoot = RNG.CreateProbabilityList()
cabinetLoot:Add("Money Clip", 30)
cabinetLoot:Add("Pocket Watch", 25)
cabinetLoot:Add("Cigarette Card", 35)
cabinetLoot:Add("Gold Pocket Watch", 10)

-- When player interacts with a lootable cabinet
RegisterNetEvent("loot:openCabinet")
AddEventHandler("loot:openCabinet", function(cabinetId)
    local source = source
    local containerId = "cabinet_" .. cabinetId
    
    -- Load saved state
    Persistence.LoadList(containerId, cabinetLoot)
    
    -- Select a random item
    local item = RNG.SelectRandomItem(cabinetLoot)
    
    if item then
        -- Mark the item as depleted
        local selectedIndex = RNG.GetChosenItemIndex(cabinetLoot)
        local selectedItem = cabinetLoot:GetItemByIndex(selectedIndex)
        RNG.SetDepletion(selectedItem, true)
        
        -- Save the updated state
        Persistence.SaveList(containerId, cabinetLoot)
        
        -- Give item to player
        TriggerClientEvent("loot:receiveItem", source, item)
    else
        -- All items depleted
        TriggerClientEvent("loot:empty", source)
    end
end)
```

#### Mineable Resources with Respawn Timer

```lua
local RNG = exports.alentexas_rng
local Persistence = RNG.Persistence

-- Initialize with 8 hour reset
Persistence.Initialize("mining_system", 8)

-- Create ore probability list
local oreTypes = RNG.CreateProbabilityList()
oreTypes:Add({name = "Iron Ore", quantity = {min = 1, max = 3}}, 50)
oreTypes:Add({name = "Copper Ore", quantity = {min = 1, max = 2}}, 30)
oreTypes:Add({name = "Silver Ore", quantity = {min = 1, max = 2}}, 15)
oreTypes:Add({name = "Gold Ore", quantity = {min = 1, max = 1}}, 5)

-- When player mines a rock
RegisterNetEvent("mining:mineRock")
AddEventHandler("mining:mineRock", function(rockId)
    local source = source
    local rockResourceId = "rock_" .. rockId
    
    -- Load saved state
    Persistence.LoadList(rockResourceId, oreTypes)
    
    -- Select a random ore
    local ore = RNG.SelectRandomItem(oreTypes, true)
    
    if ore then
        -- Mark the ore as depleted
        local selectedIndex = RNG.GetChosenItemIndex(oreTypes)
        local selectedOre = oreTypes:GetItemByIndex(selectedIndex)
        RNG.SetDepletion(selectedOre, true)
        
        -- Save the updated state
        Persistence.SaveList(rockResourceId, oreTypes)
        
        -- Calculate random quantity
        local quantity = math.random(ore.quantity.min, ore.quantity.max)
        
        -- Give ore to player
        TriggerClientEvent("mining:receiveOre", source, ore.name, quantity)
    else
        -- All ores depleted
        TriggerClientEvent("mining:depleted", source)
    end
end)
```

#### Gradual Replenishment System

The standard depletion system uses a binary state (available/depleted) with a hard reset timer. The gradual replenishment system offers a more sophisticated approach where resources have multiple "charges" that regenerate over time.

```lua
local RNG = exports.alentexas_rng
local Persistence = RNG.Persistence

-- Initialize with settings for gradual replenishment
Persistence.Initialize("resource_management", {
    resetTime = 24,       -- Default reset time in hours
    saveInterval = 30     -- Save changes every 30 minutes
})

-- Settings for gradual replenishment
local ReplenishmentSettings = {
    enabled = true,
    default_max_charges = 4,        -- Default maximum charges for a resource
    default_recharge_time = 10,     -- Default time in minutes to recharge 1 unit
    check_interval = 5              -- How often to check for recharges (in minutes)
}

-- Initialize a resource with charges
function InitializeResource(resourceId, resourceType, maxCharges, rechargeTime)
    if not ReplenishmentSettings.enabled then
        return false
    end
    
    local key = "resource_charge_" .. resourceType .. "_" .. resourceId
    local resourceData = Persistence.LoadData(key)
    
    -- Use defaults if not specified
    maxCharges = maxCharges or ReplenishmentSettings.default_max_charges
    rechargeTime = rechargeTime or ReplenishmentSettings.default_recharge_time
    
    if not resourceData then
        -- Create new resource data with full charges
        resourceData = {
            resourceId = resourceId,
            resourceType = resourceType,
            maxCharges = maxCharges,
            currentCharges = maxCharges,
            rechargeTime = rechargeTime, -- Minutes per charge
            lastUpdate = os.time()
        }
    else
        -- Update existing resource with new settings if needed
        resourceData.maxCharges = maxCharges
        resourceData.rechargeTime = rechargeTime
    end
    
    -- Save the resource data
    Persistence.SaveData(key, resourceData)
    return true
end

-- Use a charge from a resource
function UseResourceCharge(resourceId, resourceType, charges)
    if not ReplenishmentSettings.enabled then
        return false, "Replenishment system not enabled"
    end
    
    charges = charges or 1 -- Default to using 1 charge
    
    local key = "resource_charge_" .. resourceType .. "_" .. resourceId
    local resourceData = Persistence.LoadData(key)
    
    if not resourceData then
        -- Resource doesn't exist, initialize it first
        InitializeResource(resourceId, resourceType)
        resourceData = Persistence.LoadData(key)
    end
    
    -- Check if there are enough charges
    if resourceData.currentCharges < charges then
        return false, "Not enough charges"
    end
    
    -- Update current charges and last update time
    resourceData.currentCharges = resourceData.currentCharges - charges
    resourceData.lastUpdate = os.time()
    
    -- Save updated resource state
    Persistence.SaveData(key, resourceData)
    
    return true, "Used " .. charges .. " charges. Remaining: " .. resourceData.currentCharges .. "/" .. resourceData.maxCharges
end

-- Get current charges for a resource
function GetResourceCharges(resourceId, resourceType)
    if not ReplenishmentSettings.enabled then
        return 0, 0
    end
    
    local key = "resource_charge_" .. resourceType .. "_" .. resourceId
    local resourceData = Persistence.LoadData(key)
    
    if not resourceData then
        return 0, 0
    end
    
    -- Process any recharges that might have happened since last update
    local currentTime = os.time()
    local timeSinceUpdate = currentTime - resourceData.lastUpdate
    local minutesSinceUpdate = timeSinceUpdate / 60
    local rechargeTime = resourceData.rechargeTime or ReplenishmentSettings.default_recharge_time
    local rechargesToAdd = math.floor(minutesSinceUpdate / rechargeTime)
    
    if rechargesToAdd > 0 then
        resourceData.currentCharges = math.min(
            resourceData.maxCharges, 
            resourceData.currentCharges + rechargesToAdd
        )
        resourceData.lastUpdate = currentTime
        
        -- Save updated resource state
        Persistence.SaveData(key, resourceData)
    end
    
    -- Calculate time to next charge
    local timeToNextCharge = 0
    if resourceData.currentCharges < resourceData.maxCharges then
        local minutesFraction = (minutesSinceUpdate % rechargeTime)
        timeToNextCharge = (rechargeTime - minutesFraction) * 60 -- in seconds
    end
    
    return resourceData.currentCharges, resourceData.maxCharges, timeToNextCharge
end

-- Reset a resource to full charges
function ResetResourceCharges(resourceId, resourceType)
    if not ReplenishmentSettings.enabled then
        return false
    end
    
    local key = "resource_charge_" .. resourceType .. "_" .. resourceId
    local resourceData = Persistence.LoadData(key)
    
    if not resourceData then
        return false
    end
    
    -- Reset to full charges
    resourceData.currentCharges = resourceData.maxCharges
    resourceData.lastUpdate = os.time()
    
    -- Save updated resource state
    Persistence.SaveData(key, resourceData)
    
    return true
end

-- Process recharges for all tracked resources
function ProcessRecharges()
    -- Get all keys with the "resource_charge" prefix
    local keys = Persistence.GetAllKeys("resource_charge")
    local currentTime = os.time()
    
    for _, key in ipairs(keys) do
        local resourceData = Persistence.LoadData(key)
        
        if resourceData then
            -- Calculate how many charges should have been replenished since last update
            local timeSinceUpdate = currentTime - resourceData.lastUpdate
            local minutesSinceUpdate = timeSinceUpdate / 60
            local rechargeTime = resourceData.rechargeTime or ReplenishmentSettings.default_recharge_time
            local rechargesToAdd = math.floor(minutesSinceUpdate / rechargeTime)
            
            if rechargesToAdd > 0 then
                resourceData.currentCharges = math.min(
                    resourceData.maxCharges, 
                    resourceData.currentCharges + rechargesToAdd
                )
                resourceData.lastUpdate = currentTime
                
                -- Save updated resource state
                Persistence.SaveData(key, resourceData)
            end
        end
    end
end

-- Initialize periodic recharge check
Citizen.CreateThread(function()
    while ReplenishmentSettings.enabled do
        Wait(ReplenishmentSettings.check_interval * 60 * 1000) -- Convert minutes to ms
        ProcessRecharges()
    end
end)

-- Example of using the gradual replenishment system
RegisterNetEvent("resource:gather")
AddEventHandler("resource:gather", function(resourceId, resourceType)
    local source = source
    
    -- Get current resource charges
    local currentCharges, maxCharges, timeToNextCharge = GetResourceCharges(resourceId, resourceType)
    
    if currentCharges > 0 then
        -- Use a charge
        local success, message = UseResourceCharge(resourceId, resourceType, 1)
        
        if success then
            -- Give appropriate rewards based on resource type
            if resourceType == "tree" then
                -- Wood gathering example
                GiveItemToPlayer(source, "wood", math.random(2, 4))
            elseif resourceType == "mine" then
                -- Ore gathering example
                GiveItemToPlayer(source, "ore", math.random(1, 3))
            end
            
            -- Send update to client
            TriggerClientEvent("resource:updateState", source, {
                resourceId = resourceId,
                resourceType = resourceType,
                currentCharges = currentCharges - 1, -- Reflect the charge we just used
                maxCharges = maxCharges,
                timeToNextCharge = timeToNextCharge
            })
        else
            TriggerClientEvent("resource:error", source, message)
        end
    else
        -- No charges available
        local minutes = math.ceil(timeToNextCharge / 60)
        TriggerClientEvent("resource:error", source, "This resource is depleted. Next charge in " .. minutes .. " minutes.")
    end
end)
```

**Use Cases for Gradual Replenishment**

The gradual replenishment system is ideal for:

1. **Trees with multiple harvests** - A tree can be harvested multiple times before being fully depleted
2. **Mining nodes with quality variation** - Ore quality decreases with each harvest, but the node regenerates over time
3. **Seasonal resources** - Resources that regenerate faster during certain seasons or times of day
4. **Fishing spots** - Fishing locations that have limited catches before needing time to repopulate

**Choosing Between Systems**

When implementing resource gathering in your server:

1.  **Traditional Depletion**: Use when you want simple on/off resource states with a full reset after a fixed time

    ```lua
    -- Mark a tree as depleted
    Persistence.SaveList("tree_oak_123", treeList)

    -- Check if depleted
    Persistence.LoadList("tree_oak_123", treeList)
    if RNG.GetItemByIndex(treeList, 1).IsDepleted then
        -- Tree is depleted
    end
    ```
2.  **Gradual Replenishment**: Use when you want resources with multiple uses that recharge over time

    ```lua
    -- Setup a resource with multiple charges
    InitializeResource("oak_123", "tree", 4, 15)

    -- Use the resource and get remaining charges
    UseResourceCharge("oak_123", "tree", 1)
    local charges, max = GetResourceCharges("oak_123", "tree")
    ```

The persistence system makes it easy to create realistic resource management, lootable containers, and other systems that need to maintain state across server restarts or reset based on elapsed time.
