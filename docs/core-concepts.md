---
icon: square-code
---

# Core Concepts

### Understanding RNGEngine

RNGEngine is built around several key concepts that form the foundation of its random selection system. Understanding these concepts will help you implement effective RNG mechanics in your resources.

### Probability Lists

A **Probability List** is the main container for items you want to select randomly. It maintains:

* A collection of items with their associated probabilities
* The total probability of all items
* Settings for selection methods and history tracking

```lua
-- Creating a probability list
local lootTable = exports.alentexas_rng.CreateProbabilityList()
```

### Probability Items

A **Probability Item** represents a single entry in a probability list. Each item has:

* Data (any value or table)
* A probability weight (relative to other items)
* A depletion flag (to temporarily remove it from selection)

```lua
-- Adding an item to a probability list
lootTable:Add("Gold Nugget", 10) -- Simple string with 10% weight

-- Or with a complex data structure
lootTable:Add({
    name = "Gold Nugget",
    value = 50,
    model = "p_goldnugget01x"
}, 10)
```

### Probability Weights

**Probability Weights** determine the relative chance of an item being selected. The weight can be any positive number:

* Higher weights increase the chance of selection
* The actual percentage chance depends on the total weight of all items

```lua
-- Items with their weights
lootTable:Add("Common Item", 70)  -- 70% chance if only these three items exist
lootTable:Add("Uncommon Item", 25) -- 25% chance if only these three items exist
lootTable:Add("Rare Item", 5)     -- 5% chance if only these three items exist
```

If you add an item with weight 10 to a list with total weight 90, the new item has a 10% chance (10/100) of being selected.

### Depletion

**Depletion** allows you to temporarily mark items as unavailable for selection without removing them from the list. RNGEngine supports two types of depletion systems:

#### Simple Depletion

Simple depletion uses a binary state (available/depleted):

* Depleted items remain in the list but are skipped during selection
* Useful for one-time rewards or resources that need time to replenish
* Can be reset individually or all at once

```lua
-- Mark an item as depleted
local goldItem = lootTable:GetItemByIndex(1)
exports.alentexas_rng.SetDepletion(goldItem, true)

-- Reset all depletions
exports.alentexas_rng.ResetAllDepletions(lootTable)
```

#### Advanced Depletion (Gradual Recharge)

Advanced depletion allows items to have multiple "units" that recharge over time:

* Items can be used multiple times before becoming depleted
* Units automatically recharge based on configurable time intervals
* Perfect for resources like mining nodes, fishing spots, or treasure chests
* Can be mixed with simple depletion in the same list

```lua
-- Enable advanced depletion for all items in a list
-- 3 max units, 10 minutes per unit recharge, auto-recharge enabled
exports.alentexas_rng.EnableAdvancedDepletion(lootTable, 3, 10, true)

-- Or enable for a specific item only
exports.alentexas_rng.EnableItemAdvancedDepletion(lootTable, 0, 5, 15, true)

-- Use units from an item
local success, message = exports.alentexas_rng.UseItemUnits(lootTable, 0, 1)

-- Check current status
local units, maxUnits, timeToNext, isDepleted = exports.alentexas_rng.GetItemUnitsInfo(lootTable, 0)
print(string.format("Item has %d/%d units, recharge in: %.1f minutes", 
    units, maxUnits, timeToNext/60))
```

**Advanced Depletion Features**

* **Automatic Recharge**: Units recharge automatically based on elapsed time
* **Mixed Systems**: Combine simple and advanced depletion in the same list
* **Quality Scaling**: Use remaining units to determine drop quality or rarity
* **Flexible Configuration**: Different items can have different recharge rates
* **Administrative Control**: Reset, disable, or modify recharge settings

```lua
-- Example: Quality based on remaining units
local units, maxUnits = exports.alentexas_rng.GetItemUnitsInfo(fishingSpot, 0)
local qualityMultiplier = units / maxUnits
local fishQuality = qualityMultiplier > 0.8 and "Excellent" or 
                   qualityMultiplier > 0.5 and "Good" or "Poor"
```

### Collections

A **Collection** allows you to organize multiple probability lists under a single manager:

* Each list in the collection has its own weight
* You can select from specific lists or let the system choose a list based on weights

```lua
-- Create a collection
local allLoot = exports.alentexas_rng.CreateCollection()

-- Add lists with weights
exports.alentexas_rng.AddListToCollection(allLoot, "common", commonLootTable)
allLoot:SetListWeight("common", 80) -- 80% chance of selecting from this list

exports.alentexas_rng.AddListToCollection(allLoot, "rare", rareLootTable)
allLoot:SetListWeight("rare", 20) -- 20% chance of selecting from this list
```

### History Tracking

**History Tracking** records previous selections to prevent repetition:

* Configurable history size
* Option to avoid repeating recent selections
* Helps create more varied and natural-feeling randomization

```lua
-- Enable history tracking with a history size of 5
exports.alentexas_rng.EnableHistoryTracking(lootTable, 5)

-- Avoid repeating the last 3 selections
exports.alentexas_rng.AvoidRepeats(lootTable, 3)
```

Understanding these core concepts provides the foundation for using the more advanced features of RNGEngine in your resources.
