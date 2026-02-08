---
description: Randomness forged by the goddess of fortune and the essence of chaos.
icon: dice-d20
cover: .gitbook/assets/banner.png
coverY: 0
---

# TYCHE - RNG ENGINE

Welcome to Tyche RNG Engine!&#x20;

A comprehensive weighted random selection and advanced depletion management system designed specifically for RedM/VORP servers.

The system provides extensive control over item distribution through multiple selection methodologies including gradual recharge depletion, collection-based organization, and history-based probability modifications.

It features both simple and advanced depletion tracking capabilities, with support for persistent states and dynamic adjustments based on various in-game conditions such as time-based recharging, player interactions, and server events.

RNGEngine is a powerful framework for implementing randomized gameplay mechanics in your RedM server. Whether you're creating loot tables, fishing systems, dialog options, or any other feature that requires weighted randomization, RNGEngine provides the tools you need.

Below you can find a list of some of the features and applications of this system!

***

### Key Features

* **Weighted Random Selection**: Assign different probabilities to items
* **Multiple Selection Methods**: Choose from Cumulative, Linear, or Random algorithms
* **Advanced Depletion System**:
  * Simple binary depletion (available/depleted)
  * Advanced gradual recharge system with multiple units that recharge over time
  * Perfect for mining nodes, fishing spots, treasure chests, and other resources
* **History Tracking**: Prevent repetition in selections
* **Collections**: Organize multiple probability lists with their own weights
* **Persistence**: Save and load RNG states to/from database with configurable reset timers
* **Specialized Systems**: Ready-to-use implementations for common gameplay mechanics:
  * **Lootbox System**: Multi-tiered loot distribution with rarity tiers
  * **Lumbering System**: Complete woodcutting mechanics with influencers and tree types

### Core Features

#### Probability Management

* **Dual Selection System**: Supporting both weight-based calculations and normalized probability distributions
* **Multiple Selection Methods**: Cumulative, Linear, and Random algorithms optimized for different use cases
* **Dynamic Weight Adjustment**: Real-time probability modifications with collection-based organization
* **Built-in Protection**: Safeguards against probability overflow and calculation errors

#### Advanced Depletion System

* **Dual Depletion Modes**: Simple binary depletion and advanced gradual recharge systems
* **Automatic Recharging**: Time-based unit regeneration with configurable intervals
* **Mixed Systems**: Combine simple and advanced depletion in the same probability list
* **Quality Scaling**: Use remaining units to determine drop quality and rarity
* **Flexible Configuration**: Different items can have different recharge rates and maximum units

#### Collection Management

* **Hierarchical Organization**: Organize multiple probability lists with individual weights
* **Nested Selection**: Select from collections of lists for complex loot distribution
* **Weight Normalization**: Automatic weight balancing for precise percentage control
* **Dynamic List Management**: Add, remove, and modify lists at runtime

#### Zone Management System

* **PolyZone Integration**: Area-specific loot tables with circle, box, polygon, and combo zone support
* **Dynamic Zone Creation**: Create and modify zones at runtime with priority handling
* **Zone-specific Systems**: Individual charge and regeneration settings per zone
* **Overlapping Zone Support**: Priority-based selection when player is in multiple zones
* **Influence Modifiers**: Zone-specific probability adjustments for different items
* **Event-driven Architecture**: Automatic player tracking with enter/exit events

#### History Tracking

* **Repeat Prevention**: Configurable history tracking to avoid repetitive selections
* **Multiple Prevention Methods**: Choose from Off, Spread, Repick, or Shuffle algorithms
* **Customizable History Size**: Control how many previous selections to remember
* **Performance Optimized**: Efficient memory management with object pooling

#### Persistence System

* **Database Integration**: Automatic saving and loading of RNG states
* **Time-based Resets**: Configurable reset timers (hourly, daily, weekly, custom)
* **Resource Isolation**: Per-resource data management with unique identifiers
* **State Recovery**: Automatic restoration after server restarts

### Technical Implementation

The system utilizes a modular architecture allowing for easy expansion and modification. It includes:

* **State Management**: Persistent data handling with automatic cleanup
* **Object Pooling**: Memory-efficient history entry management
* **Optimized Algorithms**: Fast probability calculations with minimal overhead
* **Built-in Safeguards**: Protection against common exploitation vectors and edge cases
* **Flexible API**: Comprehensive function library for all use cases

### Practical Applications

Ideal for implementing:

* **Complex Loot Systems**: Multi-tiered treasure chests with rarity-based distribution
* **Resource Gathering**: Mining nodes, fishing spots, and harvesting points with gradual depletion
* **Time-based Events**: Special drop rates during specific periods with automatic resets
* **Progressive Systems**: Quality degradation based on resource usage
* **Dialog Systems**: NPC conversations with history tracking to prevent repetition
* **Treasure Hunting**: Location-based loot with persistence and recharge mechanics
* **Crafting Systems**: Material gathering with realistic depletion and regeneration
* **Quest Rewards**: Dynamic reward distribution with prevention of exploitation
* **Zone-based Events**: Area-specific loot bonuses and special event mechanics
* **Territory Control**: Different loot tables for different controlled areas
* **Mining/Gathering Zones**: Location-specific resource yields with influence modifiers

### Performance Considerations

The system is optimized for:

* **Minimal Server Impact**: Efficient algorithms with low CPU usage
* **Scalable Architecture**: Handles large player bases and extensive item lists
* **Memory Efficiency**: Object pooling and smart garbage collection
* **Database Optimization**: Efficient persistence with minimal database calls
* **Network Efficiency**: Reduced client-server communication overhead

***

### Integration with Other Resources

RNGEngine is designed to integrate seamlessly with other VORP resources. Check the examples for guidance on:

* Integrating with VORP Inventory
* Working with VORP Character skills and attributes
* Using with VORP Crafting systems
* Implementation in minigames and activities
