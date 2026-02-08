---
icon: tree
---

# Lumbering System

The lumbering system provides a comprehensive solution for harvesting wood and other resources from trees in RedM. It leverages the RNGEngine framework for randomized loot and uses a sophisticated influencer system to modify outcomes based on player skills, equipment, and other factors.

### Features

* **Tree Identification**: Automatically identifies trees in the game world using their model hashes
* **Tree Categories**: Different tree types (birch, oak, pine, etc.) provide different types of wood and resources
* **Wood Yields**: Each tree type has a base wood yield that can be modified by skills and equipment
* **Extra Item Drops**: Chance to find additional items like tree sap, moss, bird nests, etc.
* **Rarity System**: Items have common, uncommon, rare, epic, and legendary rarities
* **Influencer System**: Drops are affected by:
  * Player skills (lumberjack, botanist, naturalist, artisan)
  * Equipment (different axe types)
  * Consumables (tonics and food buffs)
  * Trinkets/perks (permanent bonuses)
  * Location (different regions have different modifiers)
* **Resource Management**: Two systems for tracking resource availability:
  * Traditional depletion with time-based resets
  * Advanced gradual replenishment with multiple charges
* **Tree Regrowth**: Trees regrow after a configurable time period based on their rarity
* **Prevention System**: Limits how often legendary items can drop
* **Persistence**: Tree states are saved between server restarts

### RNGEngine Integration

The lumbering system leverages RNGEngine's advanced features:

1. **Collections System**: Uses nested collections for organizing drops by tree type and rarity
2. **Advanced Prevention**: Implements prevention methods to control rare loot distribution
3. **Persistence**: Uses built-in persistence to track tree states
4. **Weighted Selection**: Uses probability-based selection for different item rarities

### Resource Management Systems

#### Traditional Depletion System

This system uses a binary state (available/depleted) for resources with a full reset after a fixed time.

```lua
-- Mark a tree as depleted
LumberRNG.SaveTreeState(treeId, treeType, true)

-- Check if a tree is depleted
local isDepleted = LumberRNG.IsTreeDepleted(treeId)

-- If not depleted, the tree can be harvested
if not isDepleted then
    local drops = LumberRNG.GetTreeDrops(treeType, playerData)
    -- Process drops...
    
    -- Then mark as depleted
    LumberRNG.SaveTreeState(treeId, treeType, true)
end
```

#### Gradual Replenishment System

This advanced system allows resources to have multiple uses that regenerate over time.

```lua
-- Initialize a tree with 4 charges, recharging 1 use every 15 minutes
LumberRNG.InitializeResource(treeId, "tree", 4, 15)

-- Use 1 charge from the resource
local success, message = LumberRNG.UseResourceCharge(treeId, "tree", 1)

-- Check current charges
local currentCharges, maxCharges, timeToNextCharge = LumberRNG.GetResourceCharges(treeId, "tree")

-- Reset charges (e.g., after server events)
LumberRNG.ResetResourceCharges(treeId, "tree")
```

**Key Features:**

* Resources have multiple charges (uses) that regenerate gradually
* Each resource can have customized maximum charges and recharge rates
* Time-based recharging happens automatically in the background
* Full persistence between server restarts

### Axe Types and Their Effects

Different axes provide different bonuses:

| Axe Type                       | Wood Yield | Extra Item Chance | Rare Item Chance |
| ------------------------------ | ---------- | ----------------- | ---------------- |
| Basic Hatchet                  | +0%        | +0%               | +0%              |
| Improved Hatchet               | +15%       | +5%               | +0%              |
| Refined Hatchet                | +10%       | +10%              | +5%              |
| Forester's Axe                 | +20%       | +15%              | +10%             |
| Rare Hunter's Axe              | -10%       | +20%              | +25%             |
| Woodsman's Axe                 | +30%       | +0%               | -5%              |
| Legendary Woodcutter's Hatchet | +40%       | +25%              | +20%             |

### Usage Guide

#### Basic Implementation

```lua
-- Import the lumbering system
local LumberRNG = require("lumbering_rng")

-- Player data with influencers
local playerData = {
    skills = {
        lumberjack = 5,  -- Level 5 lumberjack skill
        botanist = 2,    -- Level 2 botanist skill
    },
    equipment = {
        axe = "foresters_axe"  -- Using a forester's axe
    },
    trinkets = {
        woodcutters_charm = true  -- Has the woodcutter's charm
    },
    district = "forest"  -- In the forest region
}

-- Get drops for an oak tree
local drops = LumberRNG.GetTreeDrops("oak", playerData)

-- Process drops
for _, drop in ipairs(drops) do
    print(string.format("%dx %s (%s)", drop.quantity, drop.label, drop.rarity))
    -- Add to player's inventory
    -- GiveItemToPlayer(player, drop.item, drop.quantity)
end
```

#### Tracking Tree State with Traditional Depletion

```lua
-- Generate a unique ID for the tree
local treeId = "oak_tree_" .. GetHashKey(coords)
local treeType = "oak"

-- Check if the tree is available
if not LumberRNG.IsTreeDepleted(treeId) then
    -- Tree is available for harvesting
    local drops = LumberRNG.GetTreeDrops(treeType, playerData)
    
    -- Process drops...
    
    -- Mark tree as depleted
    LumberRNG.SaveTreeState(treeId, treeType, true)
else
    -- Tree is already depleted
    -- Show message to player
    TriggerClientEvent("ShowNotification", source, "This tree has been recently harvested.")
end
```

#### Using Gradual Replenishment System

```lua
-- Initialize a pine tree with 4 charges, recharging 1 use every 10 minutes
local treeId = "pine_tree_" .. GetHashKey(coords)
LumberRNG.InitializeResource(treeId, "tree", 4, 10)

-- Player chops the tree
local success, message = LumberRNG.UseResourceCharge(treeId, "tree", 1)
if success then
    -- Get drops for this tree type
    local drops = LumberRNG.GetTreeDrops("pine", playerData)
    
    -- Process drops...
    
    -- Check remaining charges
    local currentCharges, maxCharges, timeToNextCharge = LumberRNG.GetResourceCharges(treeId, "tree")
    
    -- Inform player
    TriggerClientEvent("ShowTreeStatus", source, currentCharges, maxCharges, timeToNextCharge)
else
    -- Tree cannot be harvested now
    TriggerClientEvent("ShowNotification", source, message)
end
```

#### Calculating Axe Damage

```lua
-- Calculate axe damage based on equipment and influencers
local damage = LumberRNG.GetAxeDamage(playerData)
```

### Advanced Implementation

#### Progressive Resource Quality

Resources can provide different quality yields based on their current charges:

```lua
-- Check current charges of a resource
local currentCharges, maxCharges = LumberRNG.GetResourceCharges(treeId, "tree")

-- Calculate quality factor (higher when resource has more charges)
local qualityFactor = currentCharges / maxCharges

-- Apply to loot generation
local baseWoodAmount = 3
local finalWoodAmount = math.ceil(baseWoodAmount * (0.75 + qualityFactor * 0.5))

-- Quality affects rare drop chances
local rareItemChance = 0.05 + (qualityFactor * 0.1)
if math.random() <= rareItemChance then
    -- Give rare item
end
```

#### Environmental Factors

You can apply additional modifiers based on game conditions:

```lua
-- Get current weather and time
local weather = GetCurrentWeather()
local gameHour = GetCurrentHour()

-- Initialize base modifiers
local woodModifier = 1.0
local rareItemModifier = 1.0

-- Apply weather effects
if weather == "rain" or weather == "thunder" then
    woodModifier = woodModifier * 0.8  -- Less wood in rain
    rareItemModifier = rareItemModifier * 1.2  -- More rare items in rain
end

-- Apply time of day effects (dawn/dusk bonus)
if (gameHour >= 5 and gameHour <= 7) or (gameHour >= 18 and gameHour <= 20) then
    rareItemModifier = rareItemModifier * 1.25  -- Bonus at dawn/dusk
end

-- Apply to loot drops
local adjustedDrops = {}
for _, drop in ipairs(LumberRNG.GetTreeDrops("oak", playerData)) do
    if drop.item == "oak_wood" then
        drop.quantity = math.ceil(drop.quantity * woodModifier)
    elseif drop.rarity == "rare" or drop.rarity == "epic" or drop.rarity == "legendary" then
        -- Apply rare item modifier logic
        if math.random() <= rareItemModifier * 0.5 then
            drop.quantity = drop.quantity + 1
        end
    end
    table.insert(adjustedDrops, drop)
end
```

### Client-Side Implementation Guide

The client-side script should handle:

* Tree detection with raycasting
* Player interaction with trees
* Animation and effects
* Sending chop requests to the server
* Updating tree visuals based on depletion state

```lua
-- Example client implementation
Citizen.CreateThread(function()
    while true do
        Citizen.Wait(0)
        
        if IsControlJustPressed(0, 0x24978A28) then -- [H] key
            -- Check if player has an axe equipped
            if HasPedGotWeapon(PlayerPedId(), GetHashKey("WEAPON_MELEE_HATCHET"), false) then
                -- Raycast to find tree
                local treeData = GetTreeInFront()
                
                if treeData then
                    -- Play chopping animation
                    TaskStartScenarioInPlace(PlayerPedId(), GetHashKey("WORLD_HUMAN_TREE_CHOP"), -1, true, false, false, false)
                    
                    -- Wait for animation
                    Citizen.Wait(5000)
                    
                    -- Send to server
                    TriggerServerEvent("lumbering:chopTree", treeData.model, treeData.coords)
                    
                    -- Clear task
                    ClearPedTasks(PlayerPedId())
                end
            else
                -- Notify player to equip an axe
                TriggerEvent("vorp:TipBottom", "You need an axe to chop trees", 3000)
            end
        end
        
        Citizen.Wait(10)
    end
end)

-- Function to detect tree in front of player
function GetTreeInFront()
    local playerPed = PlayerPedId()
    local coords = GetEntityCoords(playerPed)
    local forward = GetEntityForwardVector(playerPed)
    local raycastCoords = vec3(
        coords.x + forward.x * 2.0,
        coords.y + forward.y * 2.0,
        coords.z
    )
    
    local rayHandle = StartShapeTestRay(coords.x, coords.y, coords.z, raycastCoords.x, raycastCoords.y, raycastCoords.z, 16, playerPed, 0)
    local _, hit, hitCoords, _, entityHit = GetShapeTestResult(rayHandle)
    
    if hit and entityHit ~= 0 then
        local model = GetEntityModel(entityHit)
        
        -- Check if it's a tree (this would be your logic to identify trees)
        if IsTreeModel(model) then
            return {
                entity = entityHit,
                model = model,
                coords = hitCoords
            }
        end
    end
    
    return nil
end

-- Function to check if a model is a tree
function IsTreeModel(model)
    -- List of tree model hashes
    local treeModels = {
        GetHashKey("p_tree_pine_03"),
        GetHashKey("p_tree_oak_01"),
        GetHashKey("p_tree_maple_03"),
        -- Add more tree models as needed
    }
    
    for _, treeModel in ipairs(treeModels) do
        if model == treeModel then
            return true
        end
    end
    
    return false
end
```

### Server-Side Implementation Guide

The server-side script handles:

* Persistence of tree states
* Drop calculation and distribution
* Prevention systems for rare items
* XP rewards for skills

```lua
-- Example server implementation
local LumberRNG = require("lumbering_rng")

-- Handle tree chopping
RegisterServerEvent("lumbering:chopTree")
AddEventHandler("lumbering:chopTree", function(treeModel, coords)
    local source = source
    local Character = VorpCore.getUser(source).getUsedCharacter
    
    -- Generate tree ID from coordinates
    local treeId = "tree_" .. GetHashKey(coords)
    
    -- Determine tree type from model
    local treeType = GetTreeTypeFromModel(treeModel)
    
    if not treeType then
        TriggerClientEvent("vorp:TipBottom", source, "This tree cannot be harvested", 3000)
        return
    end
    
    -- Check if tree is available using gradual replenishment system
    local success, message = LumberRNG.UseResourceCharge(treeId, "tree", 1)
    
    if not success then
        TriggerClientEvent("vorp:TipBottom", source, message, 3000)
        return
    end
    
    -- Get player data
    local playerData = {
        skills = {
            lumberjack = GetPlayerSkill(source, "lumberjack") or 1,
            botanist = GetPlayerSkill(source, "botanist") or 0,
            naturalist = GetPlayerSkill(source, "naturalist") or 0,
            artisan = GetPlayerSkill(source, "artisan") or 0
        },
        equipment = {
            axe = GetPlayerAxeType(source)
        },
        trinkets = GetPlayerTrinkets(source),
        district = GetPlayerDistrict(coords)
    }
    
    -- Get drops for this tree
    local drops = LumberRNG.GetTreeDrops(treeType, playerData)
    
    -- Give items to player
    for _, drop in ipairs(drops) do
        VorpInventory.addItem(source, drop.item, drop.quantity)
        TriggerClientEvent("vorp:TipBottom", source, "You received " .. drop.quantity .. "x " .. drop.label, 3000)
        
        -- Grant XP for lumberjack skill (example)
        if drop.item:find("wood") then
            IncrementSkillXP(source, "lumberjack", drop.quantity)
        elseif drop.rarity == "rare" or drop.rarity == "epic" or drop.rarity == "legendary" then
            IncrementSkillXP(source, "naturalist", 5)
        end
    end
    
    -- Get remaining charges for client feedback
    local currentCharges, maxCharges = LumberRNG.GetResourceCharges(treeId, "tree")
    TriggerClientEvent("lumbering:updateTreeStatus", source, currentCharges, maxCharges)
end)

-- Helper function to determine tree type from model
function GetTreeTypeFromModel(model)
    local treeModels = {
        [GetHashKey("p_tree_pine_03")] = "pine",
        [GetHashKey("p_tree_oak_01")] = "oak",
        [GetHashKey("p_tree_maple_03")] = "maple",
        -- Add more mappings as needed
    }
    
    return treeModels[model]
end
```

### Example Files

Check these files for examples:

* `lumbering_example.lua`: Shows different player types and their drops
* `lumbering_rng.lua`: Core implementation using RNGEngine

### Troubleshooting

#### Common Issues

1. **Tree states not saving properly**
   * Ensure you're using correct tree IDs (consistent generation method)
   * Verify the persistence system is initialized properly
2. **Drop rates seem incorrect**
   * Check player influencer data for accuracy
   * Verify axe type is being properly detected
   * Review the RNGEngine system's selection method
3. **Performance issues with many trees**
   * Use on-demand initialization for resources rather than pre-initializing all trees
   * Consider using a grid system to only track trees in active areas
   * Optimize client-side detection code

#### Best Practices

1. **Use unique, stable tree IDs**
   * Create IDs based on coordinates or entity handles
   * Add a prefix for the tree type for easy identification
2. **Balance resource recharge rates**
   * Consider gameplay loop timing when setting recharge rates
   * Test with various player counts to ensure balance
3. **Provide visual feedback**
   * Update tree visuals based on depletion state when possible
   * Give players clear information about resource availability
