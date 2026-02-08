---
icon: fire-burner
---

# Implementation Examples

This document provides detailed implementation examples for using RNGEngine in various gameplay systems. Each example includes complete code and explanations.

### Basic Loot System

This example demonstrates a comprehensive loot system with tiered rarity and dynamic item selection.

```lua
-- server.lua
local RNG = exports.alentexas_rng
local Persistence = RNG.Persistence

-- Initialize persistence with 12-hour reset
Persistence.Initialize("loot_system", 12)

-- Create different loot tables by rarity
local lootTables = {
    common = RNG.CreateProbabilityList(),
    uncommon = RNG.CreateProbabilityList(),
    rare = RNG.CreateProbabilityList(),
    epic = RNG.CreateProbabilityList()
}

-- Create a collection for tiered selection
local allLoot = RNG.CreateCollection()

-- Populate common items (70% chance)
lootTables.common:Add({name = "Crackers", value = 1, quantity = {1, 3}}, 20)
lootTables.common:Add({name = "Canned Beans", value = 2, quantity = {1, 2}}, 20)
lootTables.common:Add({name = "Whiskey", value = 3, quantity = {1, 1}}, 15)
lootTables.common:Add({name = "Tobacco", value = 2, quantity = {1, 2}}, 15)
lootTables.common:Add({name = "Matches", value = 1, quantity = {1, 3}}, 15)
lootTables.common:Add({name = "Bullets (Regular)", value = 2, quantity = {2, 5}}, 15)
RNG.AddListToCollection(allLoot, "common", lootTables.common)
allLoot:SetListWeight("common", 70)

-- Populate uncommon items (20% chance)
lootTables.uncommon:Add({name = "Silver Ring", value = 8, quantity = {1, 1}}, 20)
lootTables.uncommon:Add({name = "Pocket Watch", value = 10, quantity = {1, 1}}, 20)
lootTables.uncommon:Add({name = "Valuable Document", value = 12, quantity = {1, 1}}, 15)
lootTables.uncommon:Add({name = "Money Clip", value = 15, quantity = {1, 1}}, 15)
lootTables.uncommon:Add({name = "Bullets (Express)", value = 5, quantity = {2, 4}}, 15)
lootTables.uncommon:Add({name = "Tonic", value = 7, quantity = {1, 2}}, 15)
RNG.AddListToCollection(allLoot, "uncommon", lootTables.uncommon)
allLoot:SetListWeight("uncommon", 20)

-- Populate rare items (8% chance)
lootTables.rare:Add({name = "Gold Ring", value = 20, quantity = {1, 1}}, 25)
lootTables.rare:Add({name = "Silver Pocket Watch", value = 25, quantity = {1, 1}}, 25)
lootTables.rare:Add({name = "Valuable Bracelet", value = 30, quantity = {1, 1}}, 20)
lootTables.rare:Add({name = "Remedy", value = 15, quantity = {1, 1}}, 15)
lootTables.rare:Add({name = "Bullets (Express Explosive)", value = 12, quantity = {1, 3}}, 15)
RNG.AddListToCollection(allLoot, "rare", lootTables.rare)
allLoot:SetListWeight("rare", 8)

-- Populate epic items (2% chance)
lootTables.epic:Add({name = "Gold Nugget", value = 50, quantity = {1, 1}}, 30)
lootTables.epic:Add({name = "Gold Pocket Watch", value = 60, quantity = {1, 1}}, 25)
lootTables.epic:Add({name = "Treasure Map", value = 100, quantity = {1, 1}}, 20)
lootTables.epic:Add({name = "Special Miracle Tonic", value = 40, quantity = {1, 1}}, 15)
lootTables.epic:Add({name = "Valuable Collectible", value = 80, quantity = {1, 1}}, 10)
RNG.AddListToCollection(allLoot, "epic", lootTables.epic)
allLoot:SetListWeight("epic", 2)

-- Function to generate container loot
function GenerateContainerLoot(containerId, containerType)
    -- Create a unique ID for this container
    local lootId = string.format("%s_%s", containerType, containerId)
    
    -- Generate 1-5 items based on container type
    local itemCount = 1
    if containerType == "cabinet" then
        itemCount = math.random(1, 2)
    elseif containerType == "chest" then
        itemCount = math.random(2, 3)
    elseif containerType == "safe" then
        itemCount = math.random(2, 4)
    elseif containerType == "treasure" then
        itemCount = math.random(3, 5)
    end
    
    -- Select random items
    local selectedItems = {}
    for i = 1, itemCount do
        -- Select a random item from the collection
        local item = RNG.SelectRandomItemFromCollection(allLoot, true)
        
        if item then
            -- Generate quantity based on item's range
            local quantity = math.random(item.Data.quantity[1], item.Data.quantity[2])
            
            -- Add to selected items
            table.insert(selectedItems, {
                name = item.Data.name,
                value = item.Data.value,
                quantity = quantity
            })
        end
    end
    
    return selectedItems
end

-- Register server event for looting containers
RegisterNetEvent("vorp:lootContainer")
AddEventHandler("vorp:lootContainer", function(containerId, containerType)
    local source = source
    local items = GenerateContainerLoot(containerId, containerType)
    
    -- Send loot to client
    TriggerClientEvent("vorp:receiveLoot", source, items)
end)
```

### Zone Management System

This example demonstrates how to create area-specific loot tables with PolyZone integration and influence modifiers.

```lua
-- server.lua
local RNG = exports.alentexas_rng

-- Create loot tables for different activities
local miningLoot = RNG.CreateProbabilityList()
miningLoot:Add("iron_ore", 0.4)
miningLoot:Add("coal", 0.3)
miningLoot:Add("silver_ore", 0.2)
miningLoot:Add("gold_ore", 0.08)
miningLoot:Add("precious_gem", 0.02)

local treasureLoot = RNG.CreateProbabilityList()
treasureLoot:Add("old_coin", 0.35)
treasureLoot:Add("jewelry", 0.25)
treasureLoot:Add("treasure_map", 0.15)
treasureLoot:Add("antique_bottle", 0.15)
treasureLoot:Add("family_heirloom", 0.1)

-- Create mining zones with different ore qualities
local valentineMine = RNG.CreateZone("valentine_mine", "Valentine Mine", "circle", 
    vector3(-1823.0, -433.0, 160.0), {
        radius = 50.0,
        priority = 2,
        debugPoly = false
    })

if valentineMine then
    -- Add loot tables to the zone
    valentineMine:AddLootTable("mining", miningLoot)
    valentineMine:SetLootTableWeight("mining", 1.0)
    
    -- Set influence modifiers for specific items
    valentineMine:SetInfluenceModifier("iron_ore", 1.5)  -- 50% more iron ore
    valentineMine:SetInfluenceModifier("coal", 2.0)      -- Double coal chance
    
    print("Created Valentine Mine zone")
end

-- Create a premium mining zone
local strawberryMine = RNG.CreateZone("strawberry_mine", "Strawberry Mine", "circle",
    vector3(-1760.0, -224.0, 181.0), {
        radius = 40.0,
        priority = 3,
        debugPoly = false
    })

if strawberryMine then
    strawberryMine:AddLootTable("mining", miningLoot)
    strawberryMine:SetLootTableWeight("mining", 1.0)
    
    -- Premium zone with better rare ore chances
    strawberryMine:SetInfluenceModifier("gold_ore", 3.0)    -- Triple gold ore chance
    strawberryMine:SetInfluenceModifier("silver_ore", 2.0)  -- Double silver ore chance
    strawberryMine:SetInfluenceModifier("precious_gem", 5.0) -- 5x gem chance
    
    print("Created Strawberry Mine zone")
end

-- Create a treasure hunting zone using polygon
local treasureBeach = RNG.CreateZone("treasure_beach", "Treasure Beach", "poly", nil, {
    points = {
        vector2(1234.0, -2890.0),
        vector2(1456.0, -2890.0),
        vector2(1456.0, -2650.0),
        vector2(1234.0, -2650.0)
    },
    minZ = 40.0,
    maxZ = 60.0,
    priority = 1,
    debugPoly = false
})

if treasureBeach then
    treasureBeach:AddLootTable("treasure", treasureLoot)
    treasureBeach:SetLootTableWeight("treasure", 1.0)
    
    -- Beach has higher chance of treasure maps and jewelry
    treasureBeach:SetInfluenceModifier("treasure_map", 5.0)
    treasureBeach:SetInfluenceModifier("jewelry", 2.0)
    
    print("Created Treasure Beach zone")
end

-- Zone event handlers
AddEventHandler('alentexas_rng:playerEnteredZone', function(playerId, zone)
    TriggerClientEvent("vorp:Tip", playerId, 
        "Entered " .. zone.name .. " - Special loot available!", 4000)
end)

AddEventHandler('alentexas_rng:playerExitedZone', function(playerId, zone)
    TriggerClientEvent("vorp:Tip", playerId, 
        "Left " .. zone.name, 2000)
end)

-- Mining function with zone-specific loot
RegisterNetEvent("zone_example:mine")
AddEventHandler("zone_example:mine", function()
    local _source = source
    
    -- Check if player is in any zone
    if not RNG.IsPlayerInAnyZone(_source) then
        TriggerClientEvent("vorp:Tip", _source, "You need to be in a mining area to mine.", 4000)
        return
    end
    
    -- Get loot from player's current zones (prioritized by zone priority)
    local item, zone = RNG.SelectLootFromPlayerZones(_source, "mining")
    
    if item and zone then
        -- Give item to player
        TriggerClientEvent("vorp:Tip", _source, 
            string.format("Mined %s in %s!", item, zone.name), 4000)
        -- Add to inventory: VorpInventory.addItem(_source, item, 1)
    else
        TriggerClientEvent("vorp:Tip", _source, "No ore found.", 2000)
    end
end)

-- Treasure hunting function
RegisterNetEvent("zone_example:treasure_hunt")
AddEventHandler("zone_example:treasure_hunt", function()
    local _source = source
    
    local item, zone = RNG.SelectLootFromPlayerZones(_source, "treasure")
    
    if item and zone then
        TriggerClientEvent("vorp:Tip", _source, 
            string.format("Found %s while treasure hunting in %s!", item, zone.name), 4000)
    else
        TriggerClientEvent("vorp:Tip", _source, "No treasure found here.", 2000)
    end
end)

-- Admin commands for zone management
RegisterCommand("createzone", function(source, args)
    if source == 0 then return end -- Console only
    
    local zoneId = args[1]
    local zoneName = args[2] or "Test Zone"
    local zoneType = args[3] or "circle"
    
    if not zoneId then
        TriggerClientEvent("vorp:Tip", source, "Usage: /createzone <id> [name] [type]", 3000)
        return
    end
    
    -- Get player position
    local playerPed = GetPlayerPed(source)
    local coords = GetEntityCoords(playerPed)
    
    local zone = RNG.CreateZone(zoneId, zoneName, zoneType, coords, {
        radius = 25.0,
        priority = 1,
        debugPoly = true
    })
    
    if zone then
        TriggerClientEvent("vorp:Tip", source, "Zone created: " .. zoneName, 3000)
    else
        TriggerClientEvent("vorp:Tip", source, "Failed to create zone", 3000)
    end
end)

RegisterCommand("destroyzone", function(source, args)
    if source == 0 then return end -- Console only
    
    local zoneId = args[1]
    if not zoneId then
        TriggerClientEvent("vorp:Tip", source, "Usage: /destroyzone <id>", 3000)
        return
    end
    
    local success = RNG.DestroyZone(zoneId)
    if success then
        TriggerClientEvent("vorp:Tip", source, "Zone destroyed: " .. zoneId, 3000)
    else
        TriggerClientEvent("vorp:Tip", source, "Zone not found: " .. zoneId, 3000)
    end
end)

RegisterCommand("listzones", function(source, args)
    if source == 0 then return end -- Console only
    
    local zones = RNG.GetAllZones()
    local count = 0
    
    for zoneId, zone in pairs(zones) do
        count = count + 1
        TriggerClientEvent("vorp:Tip", source, 
            string.format("%d. %s (%s)", count, zone.name, zoneId), 3000)
    end
    
    if count == 0 then
        TriggerClientEvent("vorp:Tip", source, "No zones found", 3000)
    end
end)
```

### Advanced Depletion System

This example demonstrates the gradual recharge system for resources like mining nodes and fishing spots.

```lua
-- server.lua
local RNG = exports.alentexas_rng

-- Example 1: Mining Node with Quality Degradation
local miningNode = RNG.CreateProbabilityList()
miningNode:Add({name = "Iron Ore", value = 2}, 0.5)
miningNode:Add({name = "Gold Ore", value = 10}, 0.3)
miningNode:Add({name = "Diamond", value = 50}, 0.2)

-- Enable advanced depletion: 6 units, 15 minutes per recharge
RNG.EnableAdvancedDepletion(miningNode, 6, 15, true)

function MineNode(playerId, nodeId)
    -- Check current node status
    local units, maxUnits, timeToNext, isDepleted = RNG.GetItemUnitsInfo(miningNode, 0)
    
    if isDepleted then
        TriggerClientEvent("vorp:Tip", playerId, "This mining node is depleted. Try again later.", 3000)
        return
    end
    
    -- Calculate quality based on remaining units (more units = better quality)
    local qualityMultiplier = units / maxUnits
    
    -- Select ore type
    local ore = RNG.SelectRandomItem(miningNode, true)
    
    if ore then
        -- Use 1 unit from the node
        local success, message = RNG.UseItemUnits(miningNode, 0, 1)
        
        -- Calculate quantity based on quality
        local quantity = math.floor(1 + qualityMultiplier * 2) -- 1-3 ore based on quality
        
        -- Give ore to player
        TriggerClientEvent("vorp:Tip", playerId, 
            string.format("Mined %dx %s (Node quality: %d%%)", 
            quantity, ore.Data.name, math.floor(qualityMultiplier * 100)), 5000)
        
        -- Show remaining units
        units, maxUnits, timeToNext = RNG.GetItemUnitsInfo(miningNode, 0)
        if units <= 0 then
            TriggerClientEvent("vorp:Tip", playerId, 
                string.format("Node depleted, recharges in %.1f minutes.", timeToNext/60), 3000)
        end
    end
end

-- Example 2: Treasure Chest with Mixed Depletion
local treasureChest = RNG.CreateProbabilityList()
treasureChest:Add({name = "Gold Coins", amount = {10, 50}}, 0.4)
treasureChest:Add({name = "Silver Jewelry", value = 25}, 0.3)
treasureChest:Add({name = "Rare Gem", value = 100}, 0.2)
treasureChest:Add({name = "Legendary Artifact", value = 500}, 0.1)

-- Enable advanced depletion for most items (2 uses, 60 minutes recharge)
RNG.EnableAdvancedDepletion(treasureChest, 2, 60, true)

-- Make the legendary artifact one-time only
RNG.EnableItemAdvancedDepletion(treasureChest, 3, 1, 0, false) -- 1 use, no recharge

function OpenTreasureChest(playerId, chestId)
    -- Process any pending recharges
    RNG.ProcessRecharges(treasureChest)
    
    -- Check if any items are available
    local hasAvailableItems = false
    for i = 0, treasureChest:ItemCount() - 1 do
        local units, maxUnits, timeToNext, isDepleted = RNG.GetItemUnitsInfo(treasureChest, i)
        if not isDepleted then
            hasAvailableItems = true
            break
        end
    end
    
    if not hasAvailableItems then
        TriggerClientEvent("vorp:Tip", playerId, "This treasure chest is empty.", 3000)
        return
    end
    
    -- Select random treasure
    local treasure = RNG.SelectRandomItem(treasureChest, true)
    
    if treasure then
        -- Find the selected item's index
        local selectedIndex = RNG.GetChosenItemIndex(treasureChest)
        
        -- Use 1 unit from the selected item
        local success, message = RNG.UseItemUnits(treasureChest, selectedIndex, 1)
        
        if success then
            if treasure.Data.amount then
                -- Gold coins with random amount
                local amount = math.random(treasure.Data.amount[1], treasure.Data.amount[2])
                TriggerClientEvent("vorp:Tip", playerId, 
                    string.format("Found %d %s!", amount, treasure.Data.name), 4000)
            else
                -- Other items
                TriggerClientEvent("vorp:Tip", playerId, 
                    string.format("Found %s!", treasure.Data.name), 4000)
            end
            
            -- Show remaining uses for this item type
            local units, maxUnits, timeToNext, isDepleted = RNG.GetItemUnitsInfo(treasureChest, selectedIndex)
            if isDepleted then
                if timeToNext > 0 then
                    TriggerClientEvent("vorp:Tip", playerId, 
                        string.format("%s depleted. Recharges in %.1f minutes.", 
                        treasure.Data.name, timeToNext/60), 3000)
                else
                    TriggerClientEvent("vorp:Tip", playerId, 
                        string.format("%s permanently depleted.", treasure.Data.name), 3000)
                end
            end
        end
    end
end

-- Example 3: Fishing Spot with Dynamic Quality
local fishingSpot = RNG.CreateProbabilityList()
fishingSpot:Add({name = "Bass", size = "small"}, 0.5)
fishingSpot:Add({name = "Trout", size = "medium"}, 0.3)
fishingSpot:Add({name = "Salmon", size = "large"}, 0.2)

-- Enable advanced depletion: 8 uses, 12 minutes per recharge
RNG.EnableAdvancedDepletion(fishingSpot, 8, 12, true)

function FishAtSpot(playerId, spotId)
    -- Check spot status
    local units, maxUnits, timeToNext, isDepleted = RNG.GetItemUnitsInfo(fishingSpot, 0)
    
    if isDepleted then
        TriggerClientEvent("vorp:Tip", playerId, 
            string.format("This fishing spot is depleted. Fish return in %.1f minutes.", timeToNext/60), 4000)
        return
    end
    
    -- Calculate fish quality based on spot condition
    local spotQuality = units / maxUnits
    local fishQuality = "Poor"
    
    if spotQuality > 0.8 then
        fishQuality = "Excellent"
    elseif spotQuality > 0.6 then
        fishQuality = "Good"
    elseif spotQuality > 0.4 then
        fishQuality = "Fair"
    elseif spotQuality > 0.2 then
        fishQuality = "Poor"
    else
        fishQuality = "Terrible"
    end
    
    -- Fishing success chance based on spot quality
    local successChance = 0.3 + (spotQuality * 0.5) -- 30-80% success rate
    
    if math.random() <= successChance then
        -- Successfully caught a fish
        local fish = RNG.SelectRandomItem(fishingSpot, true)
        
        if fish then
            -- Use 1 unit from the fishing spot
            RNG.UseItemUnits(fishingSpot, 0, 1)
            
            -- Calculate fish weight based on quality
            local baseWeight = fish.Data.size == "small" and 1.5 or 
                              fish.Data.size == "medium" and 3.0 or 5.0
            local weight = baseWeight * (0.7 + spotQuality * 0.6) -- Quality affects fish size
            weight = math.floor(weight * 10) / 10 -- Round to 1 decimal
            
            -- Show result
            units, maxUnits = RNG.GetItemUnitsInfo(fishingSpot, 0)
            TriggerClientEvent("vorp:Tip", playerId, 
                string.format("Caught %s %s (%.1flb) - Spot quality: %s (%d/%d fish remaining)", 
                fishQuality, fish.Data.name, weight, fishQuality, units, maxUnits), 5000)
        end
    else
        -- Failed to catch fish but still use spot
        RNG.UseItemUnits(fishingSpot, 0, 1)
        
        units, maxUnits = RNG.GetItemUnitsInfo(fishingSpot, 0)
        TriggerClientEvent("vorp:Tip", playerId, 
            string.format("No fish caught. Spot quality: %s (%d/%d fish remaining)", 
            fishQuality, units, maxUnits), 3000)
    end
end

-- Register events
RegisterNetEvent("mining:mineNode")
AddEventHandler("mining:mineNode", function(nodeId)
    MineNode(source, nodeId)
end)

RegisterNetEvent("treasure:openChest")
AddEventHandler("treasure:openChest", function(chestId)
    OpenTreasureChest(source, chestId)
end)

RegisterNetEvent("fishing:castLine")
AddEventHandler("fishing:castLine", function(spotId)
    FishAtSpot(source, spotId)
end)

-- Admin commands for resource management
RegisterCommand("checkresource", function(source, args)
    if source ~= 0 then return end -- Console only
    
    local rechargedCount = RNG.ProcessRecharges(miningNode)
    print(string.format("Processed %d recharges", rechargedCount))
    
    -- Show status of all items
    for i = 0, miningNode:ItemCount() - 1 do
        local item = miningNode:GetItemByIndex(i)
        local units, maxUnits, timeToNext, isDepleted = RNG.GetItemUnitsInfo(miningNode, i)
        
        local status = string.format("%s: %d/%d units", 
            item.Data.name, units, maxUnits)
        
        if isDepleted and timeToNext > 0 then
            status = status .. string.format(" (recharges in %.1f min)", timeToNext/60)
        end
        
        print(status)
    end
end)

RegisterCommand("resetresources", function(source, args)
    if source ~= 0 then return end -- Console only
    
    RNG.ResetAllUnits(miningNode)
    RNG.ResetAllUnits(treasureChest)
    RNG.ResetAllUnits(fishingSpot)
    
    print("All resources reset to full capacity")
end)
```

### Dialog System with History Tracking

This example shows how to create an NPC dialog system that avoids repetitive conversations.

```lua
-- client.lua
local RNG = exports.alentexas_rng

-- Create dialog tables for different NPC types
local dialogOptions = {
    ["shopkeeper"] = RNG.CreateProbabilityList(),
    ["bartender"] = RNG.CreateProbabilityList(),
    ["lawman"] = RNG.CreateProbabilityList(),
    ["stranger"] = RNG.CreateProbabilityList()
}

-- Common greeting dialog
local greetings = RNG.CreateProbabilityList()
greetings:Add("Hello there, partner.", 1)
greetings:Add("Good day to you.", 1)
greetings:Add("What can I do for you?", 1)
greetings:Add("Welcome, friend.", 1)
greetings:Add("Need something?", 1)

-- Enable history tracking to avoid repetition
RNG.EnableHistoryTracking(greetings, 5)
RNG.AvoidRepeats(greetings, 3)

-- Shopkeeper dialog
dialogOptions["shopkeeper"]:Add("Looking to buy something special today?", 1)
dialogOptions["shopkeeper"]:Add("Got some fine goods in stock right now.", 1)
dialogOptions["shopkeeper"]:Add("Been a slow day. Glad to see a customer.", 1)
dialogOptions["shopkeeper"]:Add("Let me know if you need help finding anything.", 1)
dialogOptions["shopkeeper"]:Add("I just got a new shipment in yesterday.", 1)

-- Bartender dialog
dialogOptions["bartender"]:Add("What'll it be today?", 1)
dialogOptions["bartender"]:Add("Need something strong or something smooth?", 1)
dialogOptions["bartender"]:Add("You hear about that commotion last night?", 1)
dialogOptions["bartender"]:Add("Been serving drinks since sunrise. Long day.", 1)
dialogOptions["bartender"]:Add("We're running low on the good stuff, but I can still fix you up.", 1)

-- Enable history tracking for all dialog types
for npcType, dialogList in pairs(dialogOptions) do
    RNG.EnableHistoryTracking(dialogList, 5)
    RNG.AvoidRepeats(dialogList, 3)
end

-- Weather-based dialog
local weatherDialog = {
    ["sunny"] = RNG.CreateProbabilityList(),
    ["rainy"] = RNG.CreateProbabilityList(),
    ["foggy"] = RNG.CreateProbabilityList(),
    ["stormy"] = RNG.CreateProbabilityList()
}

weatherDialog["sunny"]:Add("Fine day today, isn't it?", 1)
weatherDialog["sunny"]:Add("Weather like this makes for good business.", 1)
weatherDialog["sunny"]:Add("Ain't nothing better than blue skies.", 1)

weatherDialog["rainy"]:Add("This rain ain't good for business.", 1)
weatherDialog["rainy"]:Add("Stay dry out there, partner.", 1)
weatherDialog["rainy"]:Add("Rain's good for the crops, at least.", 1)

-- Enable history tracking for weather dialog
for weather, dialogList in pairs(weatherDialog) do
    RNG.EnableHistoryTracking(dialogList, 3)
    RNG.AvoidRepeats(dialogList, 2)
end

-- Function to generate NPC dialog
function GetNPCDialog(npcType, currentWeather)
    -- Get a greeting
    local greeting = RNG.SelectRandomItem(greetings)
    
    -- Get NPC type specific dialog
    local dialog = "No dialog available."
    if dialogOptions[npcType] then
        dialog = RNG.SelectRandomItem(dialogOptions[npcType])
    end
    
    -- Add weather dialog if appropriate
    local weatherComment = ""
    if weatherDialog[currentWeather] and math.random() < 0.3 then -- 30% chance
        weatherComment = RNG.SelectRandomItem(weatherDialog[currentWeather])
    end
    
    -- Combine the dialog pieces
    local fullDialog = greeting
    
    if weatherComment ~= "" then
        fullDialog = fullDialog .. " " .. weatherComment
    end
    
    fullDialog = fullDialog .. " " .. dialog
    
    return fullDialog
end

-- Event for when player interacts with NPC
RegisterNetEvent("vorp:talkToNPC")
AddEventHandler("vorp:talkToNPC", function(npcType)
    local currentWeather = GetCurrentWeatherType() -- FiveM native
    local weatherName = "sunny" -- Default
    
    -- Convert weather to our system
    if currentWeather == 1 or currentWeather == 2 then -- Rain/drizzle
        weatherName = "rainy"
    elseif currentWeather == 3 then -- Fog
        weatherName = "foggy"
    elseif currentWeather == 4 or currentWeather == 5 then -- Thunder
        weatherName = "stormy"
    end
    
    local dialog = GetNPCDialog(npcType, weatherName)
    
    -- Display dialog to player
    TriggerEvent("vorp:showSubtitle", dialog, 5000) -- 5 seconds
end)
```

These examples demonstrate how to implement complex gameplay systems using RNGEngine's features. You can adapt and extend these examples to fit your specific needs.
