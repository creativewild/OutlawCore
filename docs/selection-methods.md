---
icon: list-radio
---

# Selection Methods

RNGEngine offers three different selection methods, each with its own characteristics and use cases. Understanding the differences between these methods will help you choose the most appropriate one for your specific needs.

### Overview of Selection Methods

| Method     | Description                                               | Best for                                        |
| ---------- | --------------------------------------------------------- | ----------------------------------------------- |
| Cumulative | Aggregates probabilities, then selects a random point     | General purpose, most balanced                  |
| Linear     | Iterates through items until a random threshold is passed | Simple, intuitive, slight bias to earlier items |
| Random     | Pure random selection, ignoring weights                   | Equal probability for all items                 |

### Cumulative Method (Default)

The **Cumulative Method** is the default and generally recommended approach for weighted random selection.

#### How It Works

1. Calculates the cumulative sum of all item probabilities
2. Generates a random number between 0 and the total probability sum
3. Selects the first item whose cumulative probability exceeds the random number

#### Example

```lua
local lootTable = exports.alentexas_rng.CreateProbabilityList()
lootTable:Add("Common", 70)
lootTable:Add("Uncommon", 25)
lootTable:Add("Rare", 5)

-- Set or confirm cumulative method (default)
exports.alentexas_rng.UseCumulativeMethod(lootTable)

-- Select a random item
local item = exports.alentexas_rng.SelectRandomItem(lootTable)
```

#### Characteristics

* Most mathematically accurate for weighted probabilities
* Guarantees each item has exactly its designated probability of selection
* Computationally efficient for large lists
* Best for most general-purpose RNG needs

### Linear Method

The **Linear Method** uses a simpler approach to item selection that can be more intuitive to understand.

#### How It Works

1. Generates a random number between 0 and the total probability sum
2. Iterates through each item in the list
3. Subtracts each item's probability from the random number
4. Selects the item that brings the random number below zero

#### Example

```lua
local dialogOptions = exports.alentexas_rng.CreateProbabilityList()
dialogOptions:Add("Hello there!", 1)
dialogOptions:Add("Good day to you.", 1)
dialogOptions:Add("Fine weather we're having.", 1)

-- Use linear method
exports.alentexas_rng.UseLinearMethod(dialogOptions)

-- Select a random greeting
local greeting = exports.alentexas_rng.SelectRandomItem(dialogOptions)
```

#### Characteristics

* Slightly less computationally efficient than Cumulative
* Good for educational purposes or when you want to easily explain the selection process
* Has a very slight bias toward earlier items in the list
* Works well for lists where probabilities are equal or similar

### Random Method

The **Random Method** ignores weights entirely and gives each item an equal chance of selection.

#### How It Works

1. Counts the total number of non-depleted items in the list
2. Selects a random index between 1 and the total count
3. Returns the item at that index

#### Example

```lua
local cardDeck = exports.alentexas_rng.CreateProbabilityList()
-- Add cards with equal weights (weights are ignored in Random mode)
cardDeck:Add("Ace of Spades", 1)
cardDeck:Add("King of Hearts", 1)
cardDeck:Add("Queen of Diamonds", 1)
cardDeck:Add("Jack of Clubs", 1)

-- Use random method for equal probability
exports.alentexas_rng.UseRandomMethod(cardDeck)

-- Draw a card
local card = exports.alentexas_rng.SelectRandomItem(cardDeck)
```

#### Characteristics

* Ignores all probability weights
* Each item has exactly the same chance of being selected
* Useful for fair selection when all items should be equally likely
* Simplest and most computationally efficient method

### Switching Methods

You can change the selection method for a list at any time:

```lua
-- Switch to Cumulative method
exports.alentexas_rng.UseCumulativeMethod(myList)

-- Switch to Linear method
exports.alentexas_rng.UseLinearMethod(myList)

-- Switch to Random method
exports.alentexas_rng.UseRandomMethod(myList)
```

### Implementation Considerations

* **Item Order**: In the Linear method, item order can slightly affect probabilities. Consider this when organizing your list.
* **Depleted Items**: All methods automatically skip depleted items during selection.
* **Performance**: For very large lists, the Cumulative method is generally most efficient.
* **Fairness**: The Random method is the fairest when equal probability is desired, regardless of specified weights.

### When to Use Each Method

* **Cumulative Method**: Default choice for most random selection needs, especially when accurate probabilities are important.
* **Linear Method**: When you need a simple, easy-to-understand selection algorithm or when iterating through the list is needed for other purposes.
* **Random Method**: When you want all items to have an equal chance, regardless of their defined weights.
