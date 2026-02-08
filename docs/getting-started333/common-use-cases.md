---
hidden: true
icon: book-blank
---

# Common Use Cases

### Simple Random Selection

Basic random selection when you need a straightforward random pick from a list:

```lua
-- Simple loot table for a basic chest
local chestLoot = ProbabilityList.new()

chestLoot:AddItem({
    value = "money_clip",
    probability = 50,    -- Common item
    enabled = true
})

chestLoot:AddItem({
    value = "gold_ring",
    probability = 30,    -- Uncommon item
    enabled = true
})

chestLoot:AddItem({
    value = "platinum_watch",
    probability = 20,    -- Rare item
    enabled = true
})

-- Usage in a VORP chest event
RegisterNetEvent('vorp:openChest')
AddEventHandler('vorp:openChest', function()
    local _source = source
    local selector = CumulativeProbability.new()
    
    local success, selectedIndex = selector:SelectItem()
    if success then
        local item = chestLoot:GetItem(selectedIndex)
        exports.vorp_inventory:addItem(_source, item:GetValue(), 1)
    end
end)
```

### Weighted Probabilities

When you need different odds for different items:

```lua
-- Hunting rewards with quality tiers
local huntingRewards = ProbabilityList.new()

-- Poor quality (most common)
huntingRewards:AddItem({
    value = "poor_pelt",
    probability = 50,
    enabled = true
})

-- Good quality (less common)
huntingRewards:AddItem({
    value = "good_pelt",
    probability = 35,
    enabled = true
})

-- Perfect quality (rare)
huntingRewards:AddItem({
    value = "perfect_pelt",
    probability = 15,
    enabled = true
})

-- Usage with weapon condition influence
RegisterNetEvent('vorp:skinAnimal')
AddEventHandler('vorp:skinAnimal', function(weaponCondition)
    local _source = source
    local selector = CumulativeProbability.new()
    
    -- Better weapon condition improves chances for better quality
    local qualityInfluence = weaponCondition / 100
    huntingRewards:GetItem(3):SetInfluence(qualityInfluence) -- Influence perfect pelt chance
    
    local success, selectedIndex = selector:SelectItem()
    if success then
        local item = huntingRewards:GetItem(selectedIndex)
        exports.vorp_inventory:addItem(_source, item:GetValue(), 1)
    end
end)
```

### Basic Loot Tables

Creating and managing loot tables for different scenarios:

```lua
-- Different loot tables for different chest types
local commonChest = ProbabilityList.new()
local rareChest = ProbabilityList.new()
local legendaryChest = ProbabilityList.new()

-- Common chest items (basic loot)
commonChest:AddItem({
    value = "bread",
    probability = 40,
    enabled = true
})
commonChest:AddItem({
    value = "apple",
    probability = 40,
    enabled = true
})
commonChest:AddItem({
    value = "water",
    probability = 20,
    enabled = true
})

-- Rare chest items (better loot)
rareChest:AddItem({
    value = "gold_nugget",
    probability = 40,
    enabled = true
})
rareChest:AddItem({
    value = "jewelry",
    probability = 35,
    enabled = true
})
rareChest:AddItem({
    value = "map_piece",
    probability = 25,
    enabled = true
})

-- Usage with chest types
RegisterNetEvent('vorp:openTreasureChest')
AddEventHandler('vorp:openTreasureChest', function(chestType)
    local _source = source
    local selector = CumulativeProbability.new()
    local lootTable
    
    -- Select appropriate loot table
    if chestType == "common" then
        lootTable = commonChest
    elseif chestType == "rare" then
        lootTable = rareChest
    else
        return
    end
    
    -- Select and give items
    local success, selectedIndex = selector:SelectItem()
    if success then
        local item = lootTable:GetItem(selectedIndex)
        exports.vorp_inventory:addItem(_source, item:GetValue(), 1)
        TriggerClientEvent('vorp:ShowNotification', _source, 'You found: ' .. item:GetValue())
    end
end)
```
