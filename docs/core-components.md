---
hidden: true
icon: unity
---

# Core Components

## Interfaces

#### IProbabilityItem

The core interface for probability items. Defines the basic structure and functionality of items in the system.

```lua
-- Basic implementation
local myItem = ProbabilityItem.new({
    value = "gold_bar",
    probability = 25,
    enabled = true,
    isDepletable = false
})

-- Core methods
myItem:GetValue()        -- Returns "gold_bar"
myItem:GetProbability()  -- Returns 25
myItem:IsEnabled()       -- Returns true
myItem:IsDepletable()    -- Returns false

-- Influence system
myItem:SetInfluence(0.5)    -- Reduces probability by 50%
myItem:GetInfluence()       -- Returns current influence value
myItem:ClearInfluence()     -- Resets influence to default
```

#### IProbabilityList

Interface for managing collections of probability items.

```lua
-- Create a new list
local lootTable = ProbabilityList.new()

-- Basic list management
lootTable:AddItem({
    value = "gold_bar",
    probability = 25,
    enabled = true
})

lootTable:RemoveItem(1)              -- Remove by index
lootTable:GetItem(1)                 -- Get item by index
lootTable:GetItemCount()             -- Get total items
lootTable:Clear()                    -- Remove all items

-- List configuration
lootTable:SetPreventRepeat(PreventRepeatMethod.Repick)  -- Prevent duplicates
lootTable:GetPreventRepeat()                            -- Get current method

-- Selection settings
lootTable:SetPickCountRange(1, 3)    -- Set min/max items to pick
lootTable:GetPickCountMin()          -- Get minimum picks
lootTable:GetPickCountMax()          -- Get maximum picks
```

#### ISelectionMethod

Interface for implementing different selection algorithms.

```lua
-- Available selection methods
local cumulative = CumulativeProbability.new()
local linear = LinearSearchMethod.new()
local random = RandomSelectionMethod.new()

-- Basic usage
local success, selectedIndex = cumulative:SelectItem()
local success, selectedIndices = linear:SelectItems(3)  -- Select multiple items

-- Method-specific settings
cumulative:SetSeed(12345)            -- Set specific seed
linear:SetPreventRepeat(true)        -- Prevent duplicates
random:EnableWeightedSelection(true) -- Enable weighted selection
```

#### IProbabilityInfluenceProvider

Interface for implementing systems that modify probabilities.

```lua
-- Example influence provider for player skills
local skillInfluence = {
    GetInfluence = function(self, playerLevel)
        -- Increase chances based on player level
        return 1 + (playerLevel * 0.1)  -- 10% bonus per level
    end
}

-- Usage in a mining system
RegisterNetEvent('vorp:miningReward')
AddEventHandler('vorp:miningReward', function()
    local _source = source
    local playerLevel = 5  -- Get this from your skill system
    
    -- Apply influence to rare items based on skill
    local influence = skillInfluence:GetInfluence(playerLevel)
    lootTable:GetItem("rare_gem"):SetInfluence(influence)
    
    -- Select item with modified probabilities
    local success, selectedIndex = selector:SelectItem()
    if success then
        local item = lootTable:GetItem(selectedIndex)
        exports.vorp_inventory:addItem(_source, item:GetValue(), 1)
    end
end)
```

***

## Enums

#### PreventRepeatMethod

Controls how the system handles repeated selections.

```lua
-- Available prevention methods
local PreventRepeatMethod = {
    Off = "Off",         -- No prevention, allows repeats
    Spread = "Spread",   -- Picks nearest non-repeat
    Repick = "Repick",   -- Rerolls on repeat
    Shuffle = "Shuffle"  -- Shuffles recent picks
}

-- Usage examples
local lootTable = ProbabilityList.new()

-- No repeat prevention (default)
lootTable:SetPreventRepeat(PreventRepeatMethod.Off)

-- Prevent repeats by rerolling
lootTable:SetPreventRepeat(PreventRepeatMethod.Repick)

-- Example: Treasure chest that never gives same item twice
RegisterNetEvent('vorp:openTreasureChest')
AddEventHandler('vorp:openTreasureChest', function()
    local _source = source
    local selector = CumulativeProbability.new()
    
    -- Set up loot table with repeat prevention
    lootTable:SetPreventRepeat(PreventRepeatMethod.Repick)
    
    -- Select multiple unique items
    local success, items = selector:SelectItems(3)  -- Will pick 3 different items
    
    if success then
        for _, itemIndex in ipairs(items) do
            local item = lootTable:GetItem(itemIndex)
            exports.vorp_inventory:addItem(_source, item:GetValue(), 1)
        end
    end
end)
```

#### CompareMethod

Defines comparison operations for probability calculations.

```lua
local CompareMethod = {
    GreaterThan = "GreaterThan",
    GreaterOrEqual = "GreaterOrEqual",
    EqualTo = "EqualTo",
    LessThan = "LessThan",
    LessOrEqual = "LessOrEqual"
}

-- Example: Quality-based loot system
local function checkQualityThreshold(quality, threshold, compareMethod)
    if compareMethod == CompareMethod.GreaterThan then
        return quality > threshold
    elseif compareMethod == CompareMethod.GreaterOrEqual then
        return quality >= threshold
    -- ... other comparisons
    end
end

-- Usage in hunting rewards
RegisterNetEvent('vorp:huntingReward')
AddEventHandler('vorp:huntingReward', function(killQuality)
    local _source = source
    
    -- Check quality threshold for better rewards
    if checkQualityThreshold(killQuality, 0.8, CompareMethod.GreaterOrEqual) then
        -- Use high quality loot table
        lootTable = highQualityRewards
    else
        -- Use standard loot table
        lootTable = standardRewards
    end
    
    -- Select and give reward
    local success, selectedIndex = selector:SelectItem()
    if success then
        local item = lootTable:GetItem(selectedIndex)
        exports.vorp_inventory:addItem(_source, item:GetValue(), 1)
    end
end)
```

#### LogMessageType

Defines types of log messages for system monitoring.

```lua
local LogMessageType = {
    None = 0,
    Info = 1,
    Hint = 2,
    Warning = 4,
    Debug = 8,
    All = 15
}

-- Example: Logging system usage
RegisterNetEvent('vorp:lootChest')
AddEventHandler('vorp:lootChest', function()
    local _source = source
    
    -- Log attempt
    RLogger.Log("Player attempting to loot chest", LogMessageType.Info)
    
    local success, selectedIndex = selector:SelectItem()
    if success then
        local item = lootTable:GetItem(selectedIndex)
        if item then
            exports.vorp_inventory:addItem(_source, item:GetValue(), 1)
            RLogger.Log("Item awarded: " .. item:GetValue(), LogMessageType.Info)
        else
            RLogger.Log("Failed to get item", LogMessageType.Warning)
        end
    else
        RLogger.Log("Selection failed", LogMessageType.Warning)
    end
end)
```
