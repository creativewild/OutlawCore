---
icon: box
---

# Advanced Lootbox System

This guide demonstrates how to implement a comprehensive lootbox system using RNGEngine.

### Overview

The lootbox system provides:

* **Multi-tier rarity** system (Common through Ancient)
* **Per-player persistence** for lootable containers
* **Time-based resets** with configurable periods
* **Advanced prevention** of rare item repetition
* **Visual distinction** between different lootbox types
* **Special effects** for high-rarity drops

### Configuration Setup

First, create a configuration file for your lootbox system:

```lua
-- lootbox_config.lua
LootConfig = {}

-- Global Prevention Settings
LootConfig.Prevention = {
    -- Default method to use for rare item prevention
    default_method = 2, -- From Enums.PreventRepeatMethod: 0=Off, 1=Spread, 2=Repick, 3=Shuffle
    
    -- Default history size for tracking items
    default_history_size = 10,
    
    -- Default shuffle iterations (if using Shuffle method)
    shuffle_iterations = 3,
    
    -- Persist protection between server restarts
    persist_protection = true
}

-- LootBox quality tiers
LootConfig.BoxTypes = {
    common = {
        model = "p_chest01x",
        name = "Common Lootbox",
        color = {r = 255, g = 255, b = 255}, -- White
        items_min = 2,
        items_max = 3,
        legendary_chance = 0,
        epic_chance = 0.01,    -- 1%
        rare_chance = 0.10,    -- 10%
        reset_time = 24,       -- Hours
        prevention_enabled = false -- Common boxes don't need prevention
    },
    
    uncommon = {
        model = "p_chest02x",
        name = "Uncommon Lootbox",
        color = {r = 76, g = 175, b = 80}, -- Green
        items_min = 2,
        items_max = 4,
        legendary_chance = 0.01, -- 1%
        epic_chance = 0.05,      -- 5%
        rare_chance = 0.20,      -- 20%
        reset_time = 48,         -- Hours
        prevention_enabled = false -- Uncommon boxes don't need prevention
    },
    
    -- Add more box types all the way up to...
    
    ancient = {
        model = "p_treasurechest02x",
        name = "Ancient Lootbox",
        color = {r = 0, g = 230, b = 218}, -- Cyan/Turquoise
        items_min = 5,
        items_max = 8,
        ancient_chance = 0.10,    -- 10%
        relic_chance = 0.20,      -- 20%
        artifact_chance = 0.30,   -- 30%
        legendary_chance = 0.30,  -- 30%
        epic_chance = 0.10,       -- 10%
        reset_time = 720,         -- Hours (30 days)
        prevention_enabled = true, -- Enable prevention
        prevention_method = 2,     -- Repick method
        prevention_rolls = 15      -- Wait at least 15 rolls before repeating
    }
}

-- Item rarities
LootConfig.Rarities = {
    common = {
        name = "Common",
        color = {r = 255, g = 255, b = 255}, -- White
        prevention_enabled = false
    },
    uncommon = {
        name = "Uncommon",
        color = {r = 76, g = 175, b = 80}, -- Green
        prevention_enabled = false
    },
    -- Add all rarities up to...
    ancient = {
        name = "Ancient",
        color = {r = 0, g = 230, b = 218}, -- Cyan/Turquoise
        prevention_enabled = true,
        prevention_method = 2,  -- Repick method
        prevention_rolls = 15   -- Prevent for 15 rolls
    }
}

-- Items with their rarities and prevention settings
LootConfig.Items = {
    -- Common items
    {item = "consumable_bread", label = "Bread", min = 1, max = 3, rarity = "common", probability = 0.8},
    {item = "consumable_coffee", label = "Coffee", min = 1, max = 2, rarity = "common", probability = 0.7},
    -- More items...
    
    -- Legendary items with prevention settings
    {item = "goldbar", label = "Gold Bar", min = 1, max = 1, rarity = "legendary", probability = 0.03, 
     prevention_enabled = true, prevention_rolls = 6},
    {item = "jewelry_diamond_ring", label = "Diamond Ring", min = 1, max = 1, rarity = "legendary", 
     probability = 0.03, prevention_enabled = true, prevention_rolls = 5},
    
    -- Ancient items (highest rarity)
    {item = "ancient_mask", label = "Ceremonial Mask", min = 1, max = 1, rarity = "ancient", 
     probability = 0.01, prevention_enabled = true, prevention_rolls = 15}
}
```

### Initializing the Lootbox System

Create a core lootbox manager file that initializes the system:

```lua
-- lootbox_manager.lua (server side)
local RNG = exports.alentexas_rng
local Persistence = RNG.Persistence

-- Initialize the persistence system
Persistence.Initialize("lootbox_system", {
    resetTime = 24, -- Default reset time (hours)
    saveInterval = 30, -- Save changes every 30 minutes
    debugMode = false
})

-- Initialize our lootbox system
local LootboxManager = {}

-- Global collections for different rarity tiers
LootboxManager.Collections = {
    common = RNG.CreateCollection(),
    uncommon = RNG.CreateCollection(),
    rare = RNG.CreateCollection(),
    epic = RNG.CreateCollection(),
    legendary = RNG.CreateCollection(),
    artifact = RNG.CreateCollection(),
    relic = RNG.CreateCollection(),
    ancient = RNG.CreateCollection()
}

-- Setup loot tables for each collection
function LootboxManager.InitializeCollections()
    -- Create probability lists for each rarity
    local commonItems = RNG.CreateProbabilityList()
    local uncommonItems = RNG.CreateProbabilityList()
    local rareItems = RNG.CreateProbabilityList()
    local epicItems = RNG.CreateProbabilityList()
    local legendaryItems = RNG.CreateProbabilityList()
    local artifactItems = RNG.CreateProbabilityList()
    local relicItems = RNG.CreateProbabilityList()
    local ancientItems = RNG.CreateProbabilityList()
    
    -- Sort items by rarity
    for _, item in ipairs(LootConfig.Items) do
        local itemData = {
            name = item.item,
            label = item.label,
            min = item.min or 1,
            max = item.max or 1,
            rarity = item.rarity,
            prevention_enabled = item.prevention_enabled or false,
            prevention_rolls = item.prevention_rolls or 0
        }
        
        local probability = item.probability or 1
        
        -- Add to appropriate list
        if item.rarity == "common" then
            commonItems:Add(itemData, probability)
        elseif item.rarity == "uncommon" then
            uncommonItems:Add(itemData, probability)
        elseif item.rarity == "rare" then
            rareItems:Add(itemData, probability)
        elseif item.rarity == "epic" then
            epicItems:Add(itemData, probability)
        elseif item.rarity == "legendary" then
            legendaryItems:Add(itemData, probability)
        elseif item.rarity == "artifact" then
            artifactItems:Add(itemData, probability)
        elseif item.rarity == "relic" then
            relicItems:Add(itemData, probability)
        elseif item.rarity == "ancient" then
            ancientItems:Add(itemData, probability)
        end
    end
    
    -- Setup prevention for high-rarity lists
    if LootConfig.Rarities.legendary.prevention_enabled then
        RNG.EnableHistoryTracking(legendaryItems, LootConfig.Prevention.default_history_size)
        RNG.AvoidRepeats(legendaryItems, LootConfig.Rarities.legendary.prevention_rolls or 5)
        
        -- Set the prevention method based on config
        local method = LootConfig.Rarities.legendary.prevention_method or LootConfig.Prevention.default_method
        if method == 1 then -- Spread
            RNG.UseSpreadMethod(legendaryItems)
        elseif method == 2 then -- Repick
            RNG.UseRepickMethod(legendaryItems)
        elseif method == 3 then -- Shuffle
            RNG.UseShuffleMethod(legendaryItems, LootConfig.Prevention.shuffle_iterations)
        end
    end
    
    -- Do the same for artifact, relic, and ancient
    -- ...
    
    -- Add lists to their respective collections
    -- Common collection
    RNG.AddListToCollection(LootboxManager.Collections.common, "common", commonItems)
    LootboxManager.Collections.common:SetListWeight("common", 70)
    RNG.AddListToCollection(LootboxManager.Collections.common, "uncommon", uncommonItems)
    LootboxManager.Collections.common:SetListWeight("uncommon", 30)
    
    -- Uncommon collection
    RNG.AddListToCollection(LootboxManager.Collections.uncommon, "uncommon", uncommonItems)
    LootboxManager.Collections.uncommon:SetListWeight("uncommon", 60)
    RNG.AddListToCollection(LootboxManager.Collections.uncommon, "rare", rareItems)
    LootboxManager.Collections.uncommon:SetListWeight("rare", 40)
    
    -- Continue for other collections...
end

-- Function to generate loot from a box
function LootboxManager.GenerateLoot(boxId, boxType, playerId)
    -- Create a unique identifier for this player and box
    local playerBoxId = string.format("box_%s_player_%s", boxId, playerId)
    
    -- Get the appropriate collection for this box type
    local collection = LootboxManager.Collections[boxType]
    if not collection then
        collection = LootboxManager.Collections.common -- Default to common
    end
    
    -- Get box configuration
    local boxConfig = LootConfig.BoxTypes[boxType] or LootConfig.BoxTypes.common
    
    -- Create a tracking list for this player/box combination
    local playerBoxList = RNG.CreateProbabilityList()
    playerBoxList:Add("BoxTracker", 1) -- Just need one item to track depletion
    
    -- Load saved state (if this player has already looted this box)
    Persistence.LoadList(playerBoxId, playerBoxList)
    
    -- Check if box is already depleted
    local trackerItem = playerBoxList:GetItemByIndex(1)
    if trackerItem and trackerItem.IsDepleted then
        return {
            success = false,
            message = "You've already looted this box recently."
        }
    end
    
    -- Generate loot items
    local numItems = math.random(boxConfig.items_min, boxConfig.items_max)
    local lootItems = {}
    
    for i = 1, numItems do
        -- Select a random item from the collection
        local item = RNG.SelectRandomItemFromCollection(collection, true)
        
        if item then
            -- Generate quantity based on min/max
            local quantity = math.random(item.min, item.max)
            
            -- Add to results
            table.insert(lootItems, {
                name = item.name,
                label = item.label,
                quantity = quantity,
                rarity = item.rarity
            })
            
            -- If this is a rare item, add to global prevention
            if item.prevention_enabled then
                LootboxManager.AddToRareItemHistory(playerId, item.name, item.rarity)
            end
        end
    end
    
    -- Mark box as depleted for this player
    RNG.SetDepletion(trackerItem, true)
    
    -- Save depletion state with custom reset time
    Persistence.SaveList(playerBoxId, playerBoxList, boxConfig.reset_time)
    
    return {
        success = true,
        items = lootItems
    }
end

-- Function to add item to rare history
function LootboxManager.AddToRareItemHistory(playerId, itemName, rarity)
    -- Create a key for this player's rare item history
    local rareHistoryKey = "player_" .. playerId .. "_rare_history"
    
    -- Load existing history
    local rareHistory = Persistence.LoadData(rareHistoryKey) or {}
    
    -- Add this item
    table.insert(rareHistory, {
        item = itemName,
        rarity = rarity,
        timestamp = os.time()
    })
    
    -- Save updated history
    Persistence.SaveData(rareHistoryKey, rareHistory)
    
    -- Optional: Broadcast special notification for very rare items
    if rarity == "ancient" then
        TriggerClientEvent("vorp:NotifyAll", -1, "Ancient Discovery", 
            "A player has discovered an ancient artifact: " .. itemName, "inventory_items", itemName, 10000)
    elseif rarity == "relic" then
        -- Only notify the player with a special effect
        TriggerClientEvent("vorp:Notify", playerId, "Relic Found", 
            "You found a mythical relic: " .. itemName, "inventory_items", itemName, 8000)
    end
end

-- Initialize everything
LootboxManager.InitializeCollections()

-- Register server events
RegisterNetEvent("vorp:lootBox")
AddEventHandler("vorp:lootBox", function(boxId, boxType)
    local source = source
    
    -- Generate loot for this player and box
    local lootResult = LootboxManager.GenerateLoot(boxId, boxType, source)
    
    -- Send result back to client
    TriggerClientEvent("vorp:lootResult", source, lootResult)
end)

-- Check if box is available for player
RegisterNetEvent("vorp:checkBox")
AddEventHandler("vorp:checkBox", function(boxId, boxType)
    local source = source
    local playerBoxId = string.format("box_%s_player_%s", boxId, source)
    
    -- Create tracker list
    local playerBoxList = RNG.CreateProbabilityList()
    playerBoxList:Add("BoxTracker", 1)
    
    -- Load saved state
    Persistence.LoadList(playerBoxId, playerBoxList)
    
    -- Check if box is depleted
    local trackerItem = playerBoxList:GetItemByIndex(1)
    local isAvailable = not (trackerItem and trackerItem.IsDepleted)
    
    -- Send result to client
    TriggerClientEvent("vorp:boxCheckResult", source, boxId, isAvailable)
end)

-- Export the LootboxManager
exports("GetLootboxManager", function()
    return LootboxManager
end)
```

### Client-Side Implementation

Here's how to implement the client-side logic:

```lua
-- lootbox_client.lua
local RNG = exports.alentexas_rng

-- Track world lootboxes
local worldLootboxes = {}

-- Function to create a lootbox in the world
function CreateWorldLootbox(data)
    if not data or not data.id or not data.position then
        return false
    end
    
    -- Set defaults if not provided
    data.type = data.type or "common"
    data.reset = data.reset or "daily"
    data.respawn = data.respawn ~= false
    
    -- Get box config
    local boxConfig = LootConfig.BoxTypes[data.type] or LootConfig.BoxTypes.common
    
    -- Create the physical object
    local model = GetHashKey(boxConfig.model)
    
    -- Request the model
    RequestModel(model)
    while not HasModelLoaded(model) do
        Wait(10)
    end
    
    -- Create the object
    local obj = CreateObject(model, data.position.x, data.position.y, data.position.z, false, true, false)
    SetEntityHeading(obj, data.position.heading)
    PlaceObjectOnGroundProperly(obj)
    FreezeEntityPosition(obj, true)
    SetEntityAsMissionEntity(obj, true, true)
    
    -- Store the lootbox data
    worldLootboxes[data.id] = {
        id = data.id,
        type = data.type,
        position = data.position,
        reset = data.reset,
        respawn = data.respawn,
        object = obj,
        boxConfig = boxConfig
    }
    
    -- Check if this box is available for the player
    TriggerServerEvent("vorp:checkBox", data.id, data.type)
    
    return true
end

-- Spawn all configured lootboxes
Citizen.CreateThread(function()
    Wait(1000) -- Wait for resources to load
    
    -- Create all world lootboxes from config
    for _, boxData in ipairs(LootConfig.WorldLootboxes) do
        CreateWorldLootbox(boxData)
    end
end)

-- Handle box availability updates
RegisterNetEvent("vorp:boxCheckResult")
AddEventHandler("vorp:boxCheckResult", function(boxId, isAvailable)
    if worldLootboxes[boxId] then
        worldLootboxes[boxId].isAvailable = isAvailable
        
        -- Optional: Change appearance of depleted boxes
        if not isAvailable and worldLootboxes[boxId].object then
            -- For example, make it slightly transparent or change color
            SetEntityAlpha(worldLootboxes[boxId].object, 200, false)
        elseif isAvailable and worldLootboxes[boxId].object then
            -- Reset appearance
            SetEntityAlpha(worldLootboxes[boxId].object, 255, false)
        end
    end
end)

-- Handle interaction with lootboxes
Citizen.CreateThread(function()
    while true do
        Citizen.Wait(0)
        local playerPed = PlayerPedId()
        local coords = GetEntityCoords(playerPed)
        
        -- Check for nearby lootboxes
        for id, box in pairs(worldLootboxes) do
            local boxCoords = vector3(box.position.x, box.position.y, box.position.z)
            local dist = #(coords - boxCoords)
            
            if dist < 2.0 then
                -- Show prompt to open
                local promptText = "Open " .. box.boxConfig.name
                if not box.isAvailable then
                    promptText = "Already Looted"
                end
                
                DrawText3D(boxCoords.x, boxCoords.y, boxCoords.z + 1.0, promptText)
                
                -- Handle interaction
                if IsControlJustPressed(0, 0xE8342FF2) and box.isAvailable then -- E key
                    -- Animation for opening chest
                    TaskStartScenarioInPlace(playerPed, "WORLD_HUMAN_CROUCH_INSPECT", 0, true)
                    
                    -- Wait for animation
                    Citizen.Wait(2000)
                    
                    -- Request loot from server
                    TriggerServerEvent("vorp:lootBox", id, box.type)
                    
                    -- Exit scenario
                    ClearPedTasks(playerPed)
                end
            end
        end
    end
end)

-- Handle loot results
RegisterNetEvent("vorp:lootResult")
AddEventHandler("vorp:lootResult", function(result)
    if result.success then
        -- Show loot notification
        local itemsText = ""
        local totalItems = #result.items
        
        for i, item in ipairs(result.items) do
            local rarityConfig = LootConfig.Rarities[item.rarity] or LootConfig.Rarities.common
            local colorCode = "~w~" -- Default white
            
            -- Convert RGB to color code
            if rarityConfig.color then
                colorCode = "~COLOR_" .. string.format("%02X%02X%02X", 
                    rarityConfig.color.r, rarityConfig.color.g, rarityConfig.color.b) .. "~"
            end
            
            -- Format item text with color
            itemsText = itemsText .. colorCode .. item.label .. " x" .. item.quantity
            
            -- Add separator if not last item
            if i < totalItems then
                itemsText = itemsText .. "~w~, "
            end
        end
        
        -- Show notification with items
        TriggerEvent("vorp:TipRight", "Loot Obtained:\n" .. itemsText, 8000)
        
        -- Special effects for high rarity items
        for _, item in ipairs(result.items) do
            if item.rarity == "ancient" or item.rarity == "relic" then
                -- Play special sound
                PlaySoundFrontend(-1, "MEDAL_UP", "HUD_MINI_GAME_SOUNDSET", 1)
                
                -- Visual effect
                local playerPed = PlayerPedId()
                local coords = GetEntityCoords(playerPed)
                RequestNamedPtfxAsset("scr_mg_treasure") 
                while not HasNamedPtfxAssetLoaded("scr_mg_treasure") do
                    Wait(10)
                end
                UseParticleFxAssetNextCall("scr_mg_treasure")
                StartParticleFxNonLoopedAtCoord("scr_mg_treasure_smoke", coords.x, coords.y, coords.z, 0.0, 0.0, 0.0, 1.0, false, false, false)
            end
        end
        
        -- Mark box as unavailable locally
        if result.boxId and worldLootboxes[result.boxId] then
            worldLootboxes[result.boxId].isAvailable = false
            
            -- Update appearance
            if worldLootboxes[result.boxId].object then
                SetEntityAlpha(worldLootboxes[result.boxId].object, 200, false)
            end
        end
    else
        -- Show failure message
        TriggerEvent("vorp:Tip", result.message or "Unable to loot this box", 3000)
    end
end)

-- Helper function to draw 3D text
function DrawText3D(x, y, z, text)
    local onScreen, _x, _y = GetScreenCoordFromWorldCoord(x, y, z)
    local px, py, pz = table.unpack(GetGameplayCamCoord())
    
    SetTextScale(0.35, 0.35)
    SetTextFontForCurrentCommand(1)
    SetTextColor(255, 255, 255, 215)
    local str = CreateVarString(10, "LITERAL_STRING", text)
    SetTextCentre(1)
    DisplayText(str, _x, _y)
end

-- Export functions
exports("CreateWorldLootbox", CreateWorldLootbox)
```

### Using Advanced Prevention Methods

RNGEngine includes four prevention methods:

```lua
-- Setting up prevention using the different methods
local list = RNG.CreateProbabilityList()
-- Add items to list...

-- Method 0: Off - No prevention of repeats
-- This is the default, but you can set it explicitly
RNG.UsePreventRepeatMethod(list, 0) -- or use the enum: Enums.PreventRepeatMethod.Off

-- Method 1: Spread - Fast prevention with some bias
-- Repeats are replaced with nearest enabled items
RNG.UsePreventRepeatMethod(list, 1) -- or use the enum: Enums.PreventRepeatMethod.Spread

-- Method 2: Repick - Moderate speed, low bias
-- Repeats are prevented by re-rolling
RNG.UsePreventRepeatMethod(list, 2) -- or use the enum: Enums.PreventRepeatMethod.Repick

-- Method 3: Shuffle - Slowest but most accurate
-- Uses shuffling to preserve probabilities
local shuffleIterations = 3 -- Higher = more random but slower
RNG.UsePreventRepeatMethod(list, 3, shuffleIterations) -- or use the enum: Enums.PreventRepeatMethod.Shuffle
```

### Tracking Rare Item History

The lootbox system should track rare item history to prevent players from getting the same rare items repeatedly:

```lua
-- Function to check if an item should be prevented for a player
function ShouldPreventRareItem(playerId, itemName, itemRarity)
    local RNG = exports.alentexas_rng
    local Persistence = RNG.Persistence
    
    -- Load player's rare item history
    local historyKey = "player_" .. playerId .. "_rare_history"
    local itemHistory = Persistence.LoadData(historyKey) or {}
    
    -- Get rarity config
    local rarityConfig = LootConfig.Rarities[itemRarity]
    if not rarityConfig or not rarityConfig.prevention_enabled then
        return false -- Prevention not enabled for this rarity
    end
    
    -- Count recent acquisitions of this item
    local acquisitionCount = 0
    local currentTime = os.time()
    local preventionWindow = 86400 * 7 -- Default 7 days (in seconds)
    
    -- Check history entries
    for _, entry in ipairs(itemHistory) do
        if entry.item == itemName and (currentTime - entry.timestamp) < preventionWindow then
            acquisitionCount = acquisitionCount + 1
        end
    end
    
    -- Apply prevention based on method
    local method = rarityConfig.prevention_method or LootConfig.Prevention.default_method
    
    if method == 0 then -- Off
        return false
    elseif method == 1 then -- Spread
        -- More lenient prevention, still possible but less likely
        return acquisitionCount > 0 and math.random() < 0.8
    elseif method == 2 then -- Repick
        -- Strict prevention
        return acquisitionCount >= (rarityConfig.prevention_rolls or 1)
    elseif method == 3 then -- Shuffle
        -- Similar to repick but with some randomness
        return acquisitionCount >= (rarityConfig.prevention_rolls or 1) and math.random() < 0.9
    end
    
    return false
end
```

### Reset Timers

Using RNGEngine's persistence with custom reset times:

```lua
-- Different reset periods for lootboxes
local resetTimers = {
    hourly = 1,
    hours_3 = 3,
    hours_6 = 6,
    hours_12 = 12,
    daily = 24,
    days_3 = 72,
    weekly = 168,
    biweekly = 336,
    monthly = 720
}

-- Example of saving with a custom reset time
function SaveLootboxState(boxId, playerId, boxType)
    local playerBoxId = string.format("box_%s_player_%s", boxId, playerId)
    local boxConfig = LootConfig.BoxTypes[boxType] or LootConfig.BoxTypes.common
    local resetTime = boxConfig.reset_time or 24 -- Default 24 hours
    
    -- Create and set up tracker
    local boxTracker = RNG.CreateProbabilityList()
    boxTracker:Add("LootTracker", 1)
    
    -- Mark as depleted
    local trackerItem = boxTracker:GetItemByIndex(1)
    RNG.SetDepletion(trackerItem, true)
    
    -- Save with custom reset time
    Persistence.SaveList(playerBoxId, boxTracker, resetTime)
end
```

### Special Effects for Rare Items

Add special notifications and effects for higher rarity items:

```lua
-- Special effect function for high-rarity items
function ShowRareItemEffects(item, itemRarity)
    local rarityConfig = LootConfig.Rarities[itemRarity] or LootConfig.Rarities.common
    
    -- Base notification
    local title = "Item Found"
    local duration = 3000
    local icon = "inventory_items"
    
    -- Enhance based on rarity
    if itemRarity == "epic" then
        title = "Epic Discovery"
        duration = 5000
        PlaySoundFrontend(-1, "TENNIS_MATCH_POINT", "HUD_AWARDS", 1)
    elseif itemRarity == "legendary" then
        title = "Legendary Find"
        duration = 7000
        PlaySoundFrontend(-1, "LEGENDARY_FISH_CAUGHT", "HUD_AWARDS", 1)
        
        -- Add particle effect
        local coords = GetEntityCoords(PlayerPedId())
        UseParticleFxAsset("anm_explosion")
        StartParticleFxNonLoopedAtCoord("exp_smoke_trail", coords.x, coords.y, coords.z, 0.0, 0.0, 0.0, 1.0, false, false, false)
    elseif itemRarity == "relic" or itemRarity == "artifact" then
        title = (itemRarity == "relic") and "Ancient Relic!" or "Mystical Artifact!"
        duration = 10000
        
        -- Play sound and show effect
        PlaySoundFrontend(-1, "MEDAL_UP", "HUD_MINI_GAME_SOUNDSET", 1)
        AnimpostfxPlay("PauseMenuIn", 0, true)
        Citizen.Wait(1000)
        AnimpostfxStop("PauseMenuIn")
    elseif itemRarity == "ancient" then
        title = "ANCIENT TREASURE!"
        duration = 15000
        
        -- Server-wide notification
        TriggerServerEvent("vorp:broadcastRareFind", item.label, "ancient")
        
        -- Major visual and sound effects
        PlaySoundFrontend(-1, "LEGENDARY_FISH_CAUGHT", "HUD_AWARDS", 1)
        Citizen.Wait(500)
        PlaySoundFrontend(-1, "MEDAL_UP", "HUD_MINI_GAME_SOUNDSET", 1)
        
        -- Dramatic screen effect
        AnimpostfxPlay("PlayerDrugsPoisonWell", 0, true)
        Citizen.Wait(2000)
        AnimpostfxStop("PlayerDrugsPoisonWell")
    end
    
    -- Show notification with appropriate styling
    local r, g, b = 255, 255, 255 -- Default white
    if rarityConfig.color then
        r = rarityConfig.color.r
        g = rarityConfig.color.g
        b = rarityConfig.color.b
    end
    
    -- Format notification with color
    TriggerEvent("vorp:NotifyCustom", title, "You found: " .. item.label, "inventory_items", item.name, duration, r, g, b)
end
```

### Commands for Admin Control

Add administrative commands to manage the lootbox system:

```lua
-- Server-side command to reset a lootbox for a player
RegisterCommand("resetlootbox", function(source, args)
    -- Check admin permissions
    if not IsPlayerAceAllowed(source, "lootbox.admin") then
        return TriggerClientEvent("vorp:Tip", source, "You don't have permission to use this command", 5000)
    end
    
    local targetId = tonumber(args[1])
    local boxId = args[2]
    
    if not targetId or not boxId then
        return TriggerClientEvent("vorp:Tip", source, "Usage: /resetlootbox [playerId] [boxId]", 5000)
    end
    
    -- Create the player-specific box ID
    local playerBoxId = string.format("box_%s_player_%s", boxId, targetId)
    
    -- Delete this entry from persistence
    Persistence.ResetList(playerBoxId)
    
    TriggerClientEvent("vorp:Tip", source, "Reset lootbox " .. boxId .. " for player " .. targetId, 5000)
    TriggerClientEvent("vorp:Tip", targetId, "A lootbox has been reset for you", 5000)
end, false)

-- Command to reset all lootboxes
RegisterCommand("resetalllootboxes", function(source, args)
    -- Check admin permissions
    if not IsPlayerAceAllowed(source, "lootbox.admin") then
        return TriggerClientEvent("vorp:Tip", source, "You don't have permission to use this command", 5000)
    end
    
    local targetId = tonumber(args[1])
    
    if not targetId then
        return TriggerClientEvent("vorp:Tip", source, "Usage: /resetalllootboxes [playerId]", 5000)
    end
    
    -- This is an example of how to reset all lootboxes for a player
    -- In a real implementation, you'd need to track all lootbox IDs or use a database query
    Persistence.ResetAll()
    
    TriggerClientEvent("vorp:Tip", source, "Reset all lootboxes for player " .. targetId, 5000)
    TriggerClientEvent("vorp:Tip", targetId, "All lootboxes have been reset for you", 5000)
end, false)
```

### Integration with Other Systems

This lootbox system can work alongside other systems using the same persistence and prevention mechanisms:

```lua
-- Example of integrating with a mining system
RegisterNetEvent("vorp:mineResource")
AddEventHandler("vorp:mineResource", function(nodeId, nodeType)
    local source = source
    local playerId = source
    
    -- Create a unique ID for this node and player
    local playerNodeId = string.format("node_%s_player_%s", nodeId, playerId)
    
    -- Generate rewards similar to the lootbox system
    local lootManager = exports.vorp_lootbox:GetLootboxManager()
    
    -- Use the same prevention system for rare minerals
    local rewards = lootManager.GenerateLoot(nodeId, nodeType, playerId)
    
    -- Send results to client
    TriggerClientEvent("vorp:miningResult", source, rewards)
end)
```

This comprehensive implementation demonstrates how to recreate and enhance the RNG\_v1 lootbox system using RNGEngine's improved architecture, resulting in a more organized and maintainable codebase.
