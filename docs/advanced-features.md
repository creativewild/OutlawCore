---
icon: wand-sparkles
---

# Advanced Features

RNGEngine offers several advanced features that provide greater control and flexibility when implementing random selection in your resources.

### History Tracking and Repeat Prevention

History tracking is a powerful feature that records previous selections and can prevent items from being selected again within a certain number of picks.

#### Enabling History Tracking

```lua
local dialogOptions = exports.alentexas_rng.CreateProbabilityList()
dialogOptions:Add("Hello there!", 1)
dialogOptions:Add("Good day to you.", 1)
dialogOptions:Add("Fine weather we're having.", 1)
dialogOptions:Add("How's business?", 1)
dialogOptions:Add("Seen any outlaws?", 1)

-- Enable history tracking with a history of 5 selections
exports.alentexas_rng.EnableHistoryTracking(dialogOptions, 5)
```

#### Preventing Repetition

```lua
-- Avoid repeating the last 3 selected items
exports.alentexas_rng.AvoidRepeats(dialogOptions, 3)

-- Now each selection will not repeat the last 3 items that were selected
for i = 1, 10 do
    local line = exports.alentexas_rng.SelectRandomItem(dialogOptions)
    print("NPC says: " .. line)
end
```

#### Prevention Methods

RNGEngine provides four different methods for preventing repeat selections, each with its own characteristics:

```lua
local RNG = exports.alentexas_rng
local Enums = RNG.Enums

-- Method 0: Off - No prevention (default)
RNG.UsePreventRepeatMethod(dialogOptions, Enums.PreventRepeatMethod.Off)

-- Method 1: Spread - Fast prevention with some bias
-- Replaces repeated items with the nearest enabled item
RNG.UsePreventRepeatMethod(dialogOptions, Enums.PreventRepeatMethod.Spread)

-- Method 2: Repick - Moderate speed, low bias
-- Re-selects until a non-repeated item is chosen
RNG.UsePreventRepeatMethod(dialogOptions, Enums.PreventRepeatMethod.Repick)

-- Method 3: Shuffle - Slowest but most accurate
-- Generates temporary shuffled pools to maintain probability distribution
RNG.UsePreventRepeatMethod(dialogOptions, Enums.PreventRepeatMethod.Shuffle, 3) -- 3 shuffle iterations
```

Choose the method that best fits your performance vs. accuracy requirements:

* Use **Off** when repetition is acceptable
* Use **Spread** for high-performance needs where some bias is acceptable
* Use **Repick** for a good balance between accuracy and performance
* Use **Shuffle** for critical probability distribution accuracy where performance is less critical

#### Accessing History

```lua
-- Get the last selected item
local lastSelection = exports.alentexas_rng.GetLastPickedItem(dialogOptions)

-- Or get the index of the last selected item
local lastIndex = exports.alentexas_rng.GetChosenItemIndex(dialogOptions)
```

#### Disabling History Tracking

```lua
-- Disable history tracking when no longer needed
exports.alentexas_rng.DisableHistoryTracking(dialogOptions)
```

### Collections for Tiered Probability

Collections allow you to organize multiple probability lists and select from them based on their own weights.

#### Creating and Using Collections

```lua
-- Create a collection for loot categories
local treasureLoot = exports.alentexas_rng.CreateCollection()

-- Create individual loot tables for different rarities
local commonLoot = exports.alentexas_rng.CreateProbabilityList()
commonLoot:Add("Tobacco", 3)
commonLoot:Add("Whiskey", 2)
commonLoot:Add("Crackers", 4)

local uncommonLoot = exports.alentexas_rng.CreateProbabilityList()
uncommonLoot:Add("Silver Earring", 2)
uncommonLoot:Add("Pocket Watch", 1)
uncommonLoot:Add("Gold Tooth", 1)

local rareLoot = exports.alentexas_rng.CreateProbabilityList()
rareLoot:Add("Gold Bar", 1)
rareLoot:Add("Emerald", 1)
rareLoot:Add("Diamond", 0.5)

-- Add lists to the collection with their own weights
exports.alentexas_rng.AddListToCollection(treasureLoot, "common", commonLoot)
treasureLoot:SetListWeight("common", 75) -- 75% chance to select from common list

exports.alentexas_rng.AddListToCollection(treasureLoot, "uncommon", uncommonLoot)
treasureLoot:SetListWeight("uncommon", 20) -- 20% chance to select from uncommon list

exports.alentexas_rng.AddListToCollection(treasureLoot, "rare", rareLoot)
treasureLoot:SetListWeight("rare", 5) -- 5% chance to select from rare list
```

#### Selecting from Collections

```lua
-- Select a random item from any list based on list weights
local item = exports.alentexas_rng.SelectRandomItemFromCollection(treasureLoot)

-- Or select from a specific list
local rareItem = exports.alentexas_rng.SelectRandomItem(treasureLoot:GetList("rare"))
```

#### Managing Collections

```lua
-- Get a specific list from the collection
local commonItems = treasureLoot:GetList("common")

-- Change a list's weight
treasureLoot:SetListWeight("rare", 10) -- Increase rare items to 10%

-- Check if a list exists in the collection
if treasureLoot:HasList("legendary") then
    -- Use the legendary list
end

-- Normalize the weights of all lists in the collection
treasureLoot:NormalizeWeights()

-- Remove a list from the collection
treasureLoot:RemoveList("common")
```

### Complex Data Structures

You can store complex data in probability items, not just simple strings or numbers.

#### Using Tables as Item Data

```lua
local weapons = exports.alentexas_rng.CreateProbabilityList()

weapons:Add({
    name = "Cattleman Revolver",
    model = "WEAPON_REVOLVER_CATTLEMAN",
    damage = 25,
    ammo = 6,
    price = 50
}, 40) -- 40% chance

weapons:Add({
    name = "Double-Action Revolver",
    model = "WEAPON_REVOLVER_DOUBLEACTION",
    damage = 30,
    ammo = 6,
    price = 90
}, 30) -- 30% chance

weapons:Add({
    name = "Schofield Revolver",
    model = "WEAPON_REVOLVER_SCHOFIELD",
    damage = 40,
    ammo = 6,
    price = 120
}, 20) -- 20% chance

weapons:Add({
    name = "Volcanic Pistol",
    model = "WEAPON_PISTOL_VOLCANIC",
    damage = 45,
    ammo = 8,
    price = 150
}, 10) -- 10% chance
```

#### Retrieving Complex Data

```lua
-- Select a random weapon
local weapon = exports.alentexas_rng.SelectRandomItem(weapons, true) -- true to include data

-- Access the weapon properties
print("Player found a " .. weapon.name)
print("Damage: " .. weapon.damage)
print("Price: $" .. weapon.price)
```

### Custom Selection Logic

You can implement custom selection logic by manipulating depletions based on external conditions.

#### Conditional Selection

```lua
local timeBasedLoot = exports.alentexas_rng.CreateProbabilityList()

timeBasedLoot:Add("Gold Nugget", 10)
timeBasedLoot:Add("Silver Watch", 20)
timeBasedLoot:Add("Tobacco", 30)
timeBasedLoot:Add("Whiskey", 40)

-- Get the current game time
local hour = GetClockHours()

-- Make certain items available only at night
local goldItem = timeBasedLoot:GetItemByIndex(1)
exports.alentexas_rng.SetDepletion(goldItem, hour > 6 and hour < 20) -- Only available at night

-- Make another item available only during the day
local silverItem = timeBasedLoot:GetItemByIndex(2)
exports.alentexas_rng.SetDepletion(silverItem, hour < 6 or hour > 20) -- Only available during day

-- Select an item with time-based availability
local item = exports.alentexas_rng.SelectRandomItem(timeBasedLoot)
```

### Specialized Systems

RNGEngine includes specialized implementation examples for common game mechanics:

#### Lootbox System

A comprehensive lootbox system with:

* Multi-tier rarity tables
* Prevention of rare item repetition
* Time-based resets
* Per-player persistence

See the Lootbox System documentation for implementation details.

#### Lumbering System

A complete woodcutting mechanic with:

* Different tree types with unique drops
* Skill/equipment influencer system
* Tree regrowth mechanics
* History tracking for rare items

See the Lumbering System documentation for implementation details.

### Server-Client Communication

For client-server architectures, RNGEngine provides events for performing random selection on the server.

#### Client-Side Request

```lua
-- On client side
RegisterNetEvent("myResource:needLoot")
AddEventHandler("myResource:needLoot", function()
    -- Define a probability list structure
    local listData = {
        items = {
            {data = "Gold Nugget", probability = 10},
            {data = "Silver Watch", probability = 20},
            {data = "Tobacco", probability = 30},
            {data = "Whiskey", probability = 40}
        },
        selectionMethod = "cumulative",
        historyTracking = true,
        historySize = 3,
        avoidRepeats = true,
        preventRepeatCount = 2
    }
    
    -- Create a unique callback event name
    local callbackId = "myResource:receiveItem_" .. GetPlayerServerId(PlayerId())
    
    -- Request random selection from server
    TriggerServerEvent("alentexas_rng:requestRandomSelection", listData, true, callbackId)
    
    -- Handle the response
    RegisterNetEvent(callbackId)
    AddEventHandler(callbackId, function(result)
        print("Server sent random item: " .. result)
        -- Use the result
    end)
end)
```

#### Server-Side Response

The server handles the request automatically through the built-in event system.

These advanced features allow you to create sophisticated RNG systems that can adapt to game mechanics, player behavior, and environmental conditions.
