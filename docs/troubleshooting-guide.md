---
icon: wrench
---

# Troubleshooting Guide

This guide addresses common issues that developers might encounter when implementing the RNGEngine system in their resources.

### Common Issues and Solutions

#### Selection Issues

**Issue: Always Getting the Same Item**

**Symptoms:**

* Random selection consistently returns the same item
* No variation in results despite probabilities

**Possible Causes and Solutions:**

1. **All other items are depleted**
   * Check if you've accidentally marked all other items as depleted
   * Use `ResetAllDepletions(list)` to clear depletion flags
2. **History tracking with too few items**
   * If you're avoiding repeats but have too few items, the system may be forced to select the same item
   * Either reduce the number of items to avoid or add more items to the list
3. **Extremely unbalanced probabilities**
   * If one item has an extremely high probability (e.g., 999) compared to others (e.g., 1), it will almost always be selected
   * Consider normalizing probabilities or adjusting the weights to be more balanced

**Issue: Selection Seems Biased**

**Symptoms:**

* Some items appear more frequently than their probabilities suggest
* First items in the list seem to be selected more often

**Possible Causes and Solutions:**

1. **Cumulative method bias**
   * The default Cumulative method can have a slight bias toward items at the beginning of the list
   * Try switching to the Linear method: `UseLinearMethod(list)`
2.  **Small sample size**

    * Random selection needs many trials to demonstrate the true probabilities
    * Increase your test size or use the test function to verify distribution:

    ```lua
    -- Run 1000 tests to verify distribution
    local results = {}
    for i = 1, 1000 do
        local item = RNG.SelectRandomItem(list)
        results[item] = (results[item] or 0) + 1
    end
    -- Check the distribution
    for item, count in pairs(results) do
        print(item, count, count/10 .. "%")
    end
    ```

#### Persistence Issues

**Issue: Items Not Saving Properly**

**Symptoms:**

* Depletion states reset unexpectedly
* Changes don't persist between server restarts

**Possible Causes and Solutions:**

1. **Persistence not initialized**
   * Ensure you've called `Persistence.Initialize()` before using other persistence functions
   * Example: `Persistence.Initialize("my_resource", 24)`
2. **Incorrect resource name**
   * The resource name must exactly match your resource folder name
   * Case-sensitive and should not include any trailing spaces
3. **Inconsistent identifiers**
   * Make sure you're using the same identifier when saving and loading
   * Use prefixes to avoid conflicts with other resources, e.g., "myResource\_chest1"
4. **Server-side only**
   * Persistence functions must be called server-side only
   * Use client-server events if you need to trigger persistence from the client

**Issue: Items Reset Too Soon or Not at All**

**Symptoms:**

* Depleted items become available earlier than expected
* Items stay depleted forever

**Possible Causes and Solutions:**

1. **Incorrect reset time**
   * Reset time is specified in hours, make sure you're using the correct value
   * Example for 24-hour reset: `Persistence.SaveList("chest1", list, 24)`
2. **Time zone issues**
   * The system uses UTC time internally, so server restarts across time zones won't affect it
   * If you need specific local time behavior, adjust your reset times accordingly
3. **Manual server wipes**
   * If the database is manually wiped or the table deleted, all persistence will be lost
   * Use `Persistence.SetDebugMode(true)` to monitor persistence operations

#### Performance Issues

**Issue: Slow Selection in Large Lists**

**Symptoms:**

* Noticeable delay when selecting from lists with many items
* Server lag when many selections happen simultaneously

**Possible Causes and Solutions:**

1. **Inefficient selection method**
   * For very large lists, the Random method can be slow
   * Use the Cumulative method for best performance: `UseCumulativeMethod(list)`
2. **Too many history items**
   * Tracking history for large lists can impact performance
   * Reduce history size or disable if not needed: `DisableHistoryTracking(list)`
3. **Complex item data**
   * Very large or complex data structures in items may slow down serialization
   * Keep item data as simple as possible or use references to external data

**Issue: Memory Usage Concerns**

**Symptoms:**

* Server memory usage increases over time
* Resource seems to leak memory

**Possible Causes and Solutions:**

1. **Too many unused lists**
   * Create lists only when needed and clear them when done
   * Use `list:Clear()` to remove all items from a list
2. **Persisting unnecessary data**
   * Only persist what's needed, avoid saving very large data structures
   * Consider using `SaveData()` with minimal information and reconstructing full objects when needed

### Implementation Issues

#### Issue: Items with Zero Probability

**Symptoms:**

* Items with zero probability are still being selected
* Or conversely, items with probability are never selected

**Possible Causes and Solutions:**

1. **Zero vs. depletion**
   * An item with zero probability is still eligible for selection in some methods
   * To completely remove an item from selection, use depletion: `SetDepletion(item, true)`
2. **Probability calculation errors**
   * Double-check your probability calculations, especially when dynamically setting weights
   * Use `list:GetTotalProbability()` to verify the total

#### Issue: Collections Not Working as Expected

**Symptoms:**

* Selection from collections always chooses the same list
* List weights don't seem to be respected

**Possible Causes and Solutions:**

1. **Missing list weight setting**
   * After adding a list to a collection, you need to set its weight
   * Example: `collection:SetListWeight("common", 80)`
2. **Zero weight lists**
   * Lists with zero weight will never be selected
   * Ensure all lists have positive weights
3. **Forgetting to normalize**
   * If you want weights to represent exact percentages, normalize the collection
   * Use `NormalizeWeights(collection)` to make weights sum to 1.0

#### Advanced Depletion Issues

**Issue: Units Not Recharging**

**Symptoms:**

* Items remain depleted even after the recharge time has passed
* Units don't automatically increase over time

**Possible Causes and Solutions:**

1. **Auto-recharge disabled**
   * Ensure auto-recharge is enabled when setting up advanced depletion
   * Example: `EnableAdvancedDepletion(list, 5, 10, true)` (last parameter is auto-recharge)
2. **Manual processing needed**
   * If auto-recharge is disabled, you need to manually process recharges
   * Call `ProcessRecharges(list)` periodically to update units
3. **Server time issues**
   * Recharge calculations depend on server time
   * Server restarts or time changes may affect recharge timing

**Issue: UseItemUnits Always Fails**

**Symptoms:**

* `UseItemUnits()` always returns false
* Items appear to have units but can't be used

**Possible Causes and Solutions:**

1. **Wrong item index**
   * Ensure you're using the correct 0-based index
   * Use `GetItemUnitsInfo()` to verify the item exists and has units
2. **Simple depletion mixed with advanced**
   * If an item uses simple depletion, it won't have units to use
   * Check if the item has advanced depletion enabled
3. **Already depleted**
   * Items with 0 units can't be used further
   * Check current units before attempting to use: `GetItemUnitsInfo(list, index)`

**Issue: Recharge Times Not Working as Expected**

**Symptoms:**

* Items recharge too fast or too slow
* Recharge time calculations seem incorrect

**Possible Causes and Solutions:**

1. **Time unit confusion**
   * Recharge time is specified in minutes, not seconds or hours
   * For 1-hour recharge: use 60, not 1 or 3600
2. **Multiple units recharging**
   * Each unit recharges independently based on the time interval
   * If an item has been depleted for 30 minutes with 10-minute recharge, it will gain 3 units
3. **Fractional recharge handling**
   * Partial recharge times are tracked internally
   * An item might be 80% toward its next recharge but won't show the unit until 100%

### Best Practices

#### Organizing RNG Systems

1. **Use meaningful identifiers**
   * For persistence, use structured identifiers like `"region_valentine_chest_3"`
   * For collections, use clear list names like `"common"`, `"rare"`, `"epic"`
2. **Reset at appropriate intervals**
   * Consider gameplay loops when setting reset times
   * Common resources might reset faster (3-6 hours)
   * Rare resources might reset slower (24-72 hours)
3.  **Handle edge cases**

    * Always check if a list is empty before selection
    * Have fallback items if all items become depleted

    ```lua
    local item = nil
    if list:ItemCount() > 0 then
        item = RNG.SelectRandomItem(list)
    else
        item = "Fallback Item"
    end
    ```

#### Debugging and Testing

1. **Verify probability distribution**
   * Run multiple test trials to ensure your probabilities work as expected
   * Create a test function to run hundreds or thousands of selections
2. **Enable debug mode during development**
   * Use `Persistence.SetDebugMode(true)` to see detailed logs
   * Check the server console for persistence operations
3. **Test across server restarts**
   * Ensure your persistence works correctly after server restarts
   * Test with different reset times to verify the timing logic

### Getting Help

If you continue to experience issues after trying these solutions:

1. Double-check the API Reference for correct function usage
2. Review the Examples for implementation patterns
3. Contact us through the official VORP Discord channel with specific details about your issue
