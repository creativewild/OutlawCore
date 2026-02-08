---
hidden: true
icon: globe-pointer
---

# Publish your docs

## RNG System Documentation

### Quick Start Guide

* Basic Setup
  * Installation and Requirements
  * First Probability List
  * Basic Item Selection
* Understanding Probabilities
  * Weight vs Probability
  * Influence Systems
  * Selection Methods

### Core Components

#### Interfaces

* ICleanable
* IProbabilityItem
* IProbabilityList
* ISelectionMethod
* IProbabilityInfluenceProvider
* IProbabilityItemColorProvider
* IProbabilityItemInfoProvider
* ISeedProvider
* Editor Actions Interfaces
  * IPLCollectionEditorActions
  * IProbabilityListEditorActions

#### Selection Methods

* Base Implementation
  * SelectionMethodBase
  * SelectionTools
* Available Methods
  * CumulativeProbability
  * CumulativeProbabilityBurst
  * LinearSearchMethod
  * LinearSearchMethodBurst
  * RandomSelectionMethod

#### Core Systems

* Probability Management
  * ProbabilityItem
  * ProbabilityList
  * PLCollection
  * ProbabilityTools
* History & Tracking
  * HistoryEntry
  * PickHistory
* Random Number Generation
  * Random
  * DefaultSeedProvider
* Lab & Testing
  * LabLogger
  * LabPool
  * TestResults

#### Utilities

* JsonUtils
* CoreUtils
* SelectJobUtility

#### Enums

* CompareMethod
* ItemProviderType
* LogMessageType
* PreventRepeatMethod

### Advanced Usage

* Drop Rate Systems
* Seed Management
* Selection Features
* Lab Testing System

### Implementation Examples

* Loot Systems
* Crafting Systems
* World Events
* Custom Providers
