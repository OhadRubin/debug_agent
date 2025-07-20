# Exploration Algorithms Architecture

## Overview

This directory contains an autonomous exploration system designed to help a bot efficiently explore unknown 3D environments. The system uses hierarchical pathfinding, component-based navigation, and intelligent frontier detection to systematically map out spaces like Minecraft worlds.

## Core Components

### 1. `bot-state.js` - State Management & Observation (SARS: STATE)

**Main Classes:**
- **`BotKnowledgeManager`** - The bot's memory and world model
- **`ComponentGraph`** - Graph of connected walkable regions  
- **`TerrainKnowledgeMap`** - 3D grid of explored cells
- **`BotComponentManager`** - Handles component detection and updates
- **World Data Providers** - Abstract interfaces for world knowledge access

**Key Responsibilities:**
- **State Management**: Maintains ENV_STATE and AGENT_STATE data structures
- **Memory Management**: Maintains internal knowledge of discovered areas and exploration history
- **World Knowledge**: Tracks terrain states (WALKABLE, WALL, UNKNOWN)
- **Component Management**: Manages connected region detection and connectivity
- **State Updates**: Handles sensor reading integration and state synchronization

### 2. `bot-actions.js` - Action Generation & Planning (SARS: ACTION)

**Main Classes:**
- **`BotExplorer`** - The decision-making intelligence and SARS orchestrator
- **`BotPathfindingManager`** - Hierarchical A* pathfinding
- **`BotFrontierManager`** - Frontier detection and target selection

**Key Responsibilities:**
- **Decision Making**: Analyzes current state and decides where to explore next
- **Strategic Planning**: Finds frontiers, evaluates targets, handles opportunistic switching
- **Movement Planning**: Calculates intended movements using hierarchical pathfinding
- **Target Management**: Tracks current target, previous targets, and failed attempts
- **SARS Coordination**: Implements the `step(env_state, agent_state) → action` pattern

### 3. `simulation_env.js` - The Test Environment

**Main Classes:**
- **`SimulationHarness`** - Orchestrates the entire simulation
- **`WorldManager`** - Manages the simulated world
- **`SensorSimulator`** - Simulates bot sensors (vision, etc.)
- **`CoverageCalculator`** - Measures exploration progress

**Key Responsibilities:**
- **Environment Simulation**: Provides controlled world for testing exploration algorithms
- **Action Execution**: Validates and executes agent's intended movements
- **Sensor Simulation**: Generates realistic sensor readings at robot's current position
- **Movement Validation**: Determines actual robot position based on world constraints
- **State Management**: Tracks robot position, direction, and environmental observations
- **Progress Monitoring**: Measures exploration coverage and performance metrics

### 4. `simulation_visualization.html` - The Visualization

**Responsibilities:**
- Provides a web-based 3D visualization of the exploration process
- Shows the bot's current position, planned path, and discovered areas
- Displays frontiers, components, and exploration progress in real-time
- Allows testing and debugging of exploration algorithms

### 5. `minecraft_env.js` - The Live Minecraft Integration

**Main Classes:**
- **`MineflayerWorldProvider`** - Adapter that bridges abstract exploration logic to live Minecraft world
- **`ExplorerController`** - Main controller that orchestrates real-time exploration

**Key Responsibilities:**
- **Live World Adaptation**: Translates Mineflayer bot API into the abstract world interface
- **Real-time Execution**: Implements the SARS loop for live Minecraft exploration
- **Movement Coordination**: Uses mineflayer-pathfinder for actual bot movement
- **State Management**: Coordinates between exploration logic states (SCANNING, THINKING, MOVING)
- **Error Handling**: Manages failed targets and pathfinding errors in live environment

## Current Architecture Flow

### 1. Initialization
```
SimulationHarness.initialize()
├── Creates WorldManager (defines the world to explore)
├── Creates BotKnowledgeManager (bot's memory starts empty)
├── Creates BotExplorer (decision-making logic)
└── Creates SensorSimulator (simulates bot's "eyes")
```

### 2. SARS-Based Exploration Loop
The system follows a clean **State-Action-Reward-State** pattern:

```
For each iteration:
1. ENV STATE: Environment provides current observations
   └── currentPosition, currentDirection, coverage
   └── Based on previous sensor readings and movement results

2. AGENT STATE: Agent's internal knowledge and memory
   └── knowledgeMap, componentGraph, currentTarget
   └── previousTargets, failedTargets, explorationPath

3. AGENT STEP: Agent makes complete decision
   └── BotExplorer.step(env_state, agent_state)
   └── ├── decideNextAction() → picks target, handles opportunistic switching
   └── └── planMovement() → calculates intended movement
   └── Returns: { action, movementPlan, shouldContinue }

4. ENV STEP: Environment validates and executes
   └── Validates intended movement against world state
   └── Determines actualPosition (may differ from intended)
   └── Executes movement and sensor scanning
   └── Updates robot position and provides new observations

5. AGENT UPDATE: Agent updates internal state
   └── Handles target reached logic
   └── Updates previousTargets if needed
```

### 3. Key Data Structures

**World Knowledge:**
- `knowledgeMap` - 3D grid of cells (UNKNOWN, WALKABLE, WALL)
- `componentGraph` - Graph of connected walkable regions
- `componentColoredMaze` - Visual representation of components

**Exploration State:**
- `robotPosition` - Current bot location
- `currentTarget` - Where the bot is trying to go
- `previousTargets` - Places already visited
- `frontiers` - Boundaries between known/unknown areas

## Hierarchical Pathfinding System

### The Problem
Direct A* pathfinding in large 3D spaces is computationally expensive and can be slow.

### The Solution
**Two-Level Pathfinding:**

1. **Component Level** - Find path between large connected regions
   - Each "component" represents a connected walkable area
   - A* search finds optimal sequence of components to traverse
   - Fast because it operates on a high-level graph

2. **Detailed Level** - Find exact path within each component
   - A* search finds precise block-by-block path
   - Only runs within individual components, so search space is smaller

### Benefits
- **Speed**: Much faster than monolithic A* on large spaces
- **Scalability**: Performance doesn't degrade as world size increases
- **Reliability**: Can navigate complex multi-level structures

## Agent Decision-Making Architecture

### BotExplorer.step() - The Complete Agent Decision Process

**What it does:**
```javascript
async step(env_state, agent_state) {
    // 1. Agent makes complete decision (including opportunistic switching)
    const action = await this.decideNextAction(agent_state);
    
    // 2. Agent plans complete path to target
    const movementPlan = await this.planMovement(explorationState);
    
    // 3. Returns action and complete plan to environment
    return { 
        action, 
        plan: {
            target: action.target,
            fullPath: movementPlan.path,      // Complete path to target
            nextStep: movementPlan.nextStep,  // Just the next position
            nextDirection: movementPlan.nextDirection,
            cost: movementPlan.cost
        },
        shouldContinue 
    };
}
```

### BotExplorer.planMovement() - Movement Planning Only

**What it does:**
```javascript
async planMovement(state) {
    // 1. Get hierarchical path from robot to target
    const pathResult = this.pathfindingManager.findPath(robotPosition, target, ...);
    
    // 2. Return ONLY the intended next step (NOT success!)
    const nextStep = pathResult.path[1];
    return {
        intendedPosition: nextStep,
        intendedDirection: calculateDirection(robotPosition, nextStep),
        path: pathResult.path,
        cost: pathResult.cost
    };
}
```

**Key Architecture Principles:**
- **Agent plans, Environment executes**: Agent returns intentions, environment determines reality
- **No self-success claims**: Agent doesn't decide if movement succeeded
- **Complete decision-making**: All agent logic (including opportunistic switching) happens in `step()`
- **Clean separation**: Agent handles strategy, environment handles execution and validation

## SARS Pattern Benefits

### Why SARS (State-Action-Reward-State)?
Even though this isn't a learning system, the SARS pattern provides excellent conceptual clarity:

**State Separation:**
- **env_state**: What the environment looks like (position, coverage, observations)
- **agent_state**: What the agent knows/remembers (knowledge map, targets, history)

**Clear Responsibilities:**
- **Agent**: Decides what to do based on both states
- **Environment**: Executes actions and reports what actually happened
- **No mixed responsibilities**: Agent doesn't execute, environment doesn't decide

**Benefits:**
- **Easier to reason about**: Clear boundaries between agent and environment
- **Easier to test**: Can mock either agent or environment independently  
- **Easier to debug**: Clear flow of information and responsibility
- **Future-proof**: Could add actual learning/rewards later without architectural changes

## Frontier Detection System

**What are Frontiers?**
- Cells that are adjacent to both WALKABLE and UNKNOWN areas
- Represent the "edge" of explored territory
- Prime candidates for exploration targets

**How it Works:**
1. Scan through all known WALKABLE cells
2. Check if any adjacent cell is UNKNOWN
3. If yes, mark as frontier
4. Rank frontiers by distance, isolation, and strategic value

**Frontier Selection:**
- Prefers frontiers that are:
  - Reachable (pathfinding succeeds)
  - Not too far away (efficiency)
  - Not previously failed (avoid dead ends)
  - In under-explored regions (coverage optimization)

## Component-Based Navigation

**Why Components?**
- Large spaces often have distinct connected regions
- Example: Ground floor vs. second floor of a building
- Navigation between regions requires different strategies

**How it Works:**
1. **Component Detection** - Flood fill to find connected walkable regions
2. **Component Graph** - Build graph showing how components connect
3. **Component Pathfinding** - A* search on component graph
4. **Detailed Pathfinding** - A* within each component on the path


🔍 Found 7 .js, .jsx files (git-tracked)

📁 src/lib/exploration/bot-actions.js
----------------------------------------
🏛️ class BotPathfindingManager (16-329)
  🔧 def getMovementDirections (18-23)
  🔧 def switch (37-42)
  🔧 def findAbstractPath (53-58)
  🔧 def aStar (67-72)
  🔧 def componentGraph.getNode (71-76)
  🔧 def findPathInComponent (81-86)
  🔧 def getNeighbors: (105-110)
  🔧 def validComponentCells.has (117-122)
  🔧 def availableNeighbors.push (118-123)
  🔧 def findNearestValidCell (135-140)
  🔧 def buildDetailedPath (150-155)
  🔧 def shouldSkipFirstNode (217-222)
  🔧 def isPositionEqual (223-228)
  🔧 def findHierarchicalPath (229-234)
  🔧 def nearbyPositions.push (256-261)
  🔧 def Error (267-272)
  🔧 def Error (271-276)
  🔧 def Error (279-284)
  🔧 def Error (293-298)
  🔧 def findPath (304-309)
🏛️ class BotFrontierManager (331-736)
  🔧 def detectFrontiers (336-341)
  🔧 def console.log (343-348)
  🔧 def console.log (344-349)
  🔧 def console.log (350-355)
  🔧 def console.log (351-356)
  🔧 def frontierClusters.map (353-358)
  🔧 def findFrontierPoints (361-366)
  🔧 def discoveredFrontierPoints.push (392-397)
  🔧 def getNeighbors (417-422)
  🔧 def validNeighbors.push (432-437)
  🔧 def groupFrontierPoints (440-445)
  🔧 def getPointKey (484-489)
  🔧 def `${point.x.toFixed (486-491)
  🔧 def calculateDistance (489-494)
  🔧 def detectComponentAwareFrontiers (498-503)
  🔧 def console.log (503-508)
  🔧 def console.log (507-512)
  🔧 def console.log (553-558)
  🔧 def console.log (566-571)
  🔧 def console.log (567-572)
  🔧 def console.log (569-574)
  🔧 def console.log (571-576)
  🔧 def componentAwareFrontiers.push (583-588)
  🔧 def console.log (595-600)
  🔧 def console.log (596-601)
  🔧 def console.log (597-602)
  🔧 def console.log (598-603)
  🔧 def console.log (599-604)
  🔧 def console.log (600-605)
  🔧 def console.log (601-606)
  🔧 def findClosestComponent (607-612)
  🔧 def calculateNavigationTarget (625-630)
  🔧 def isComponentReachable (629-634)
  🔧 def selectOptimalFrontier (657-662)
🏛️ class BotExplorer (738-1038)
  🔧 def decideNextAction (756-761)
  🔧 def console.log (761-766)
  🔧 def console.log (762-767)
  🔧 def console.log (763-768)
  🔧 def console.log (766-771)
  🔧 def console.log (772-777)
  🔧 def console.log (786-791)
  🔧 def console.log (787-792)
  🔧 def console.log (805-810)
  🔧 def console.log (806-811)
  🔧 def console.log (816-821)
  🔧 def console.log (818-823)
  🔧 def console.log (820-825)
  🔧 def console.log (841-846)
  🔧 def isTargetReached (863-868)
  🔧 def checkForBetterTarget (871-876)
  🔧 def step (901-906)
  🔧 def console.log (921-926)
  🔧 def planMovement (938-943)
  🔧 def console.log (942-947)
  🔧 def Error (956-961)
  🔧 def this.explorationStats.uniquePositions.add (979-984)
  🔧 def calculateMovementDirection (990-995)
  🔧 def calculateRotationPath (1018-1023)

📁 src/lib/exploration/bot-logic.js
----------------------------------------
⚙️ def calculateHeuristicDistance (13-31)
⚙️ def createDeepCopyOf3DArray (37-49)
⚙️ def getCompId3D (438-473)
⚙️ def printGetCompId3DStats (476-484)
⚙️ def getCompId3D (438-473)
⚙️ def printGetCompId3DStats (476-484)
🏛️ class PrivilegedWorldProvider (73-93)
  🔧 def isWalkable (79-84)
  🔧 def getCellState (86-91)
  🔧 def getBounds (90-95)
🏛️ class BotKnowledgeProvider (95-130)
  🔧 def isWalkable (102-107)
  🔧 def getCellState (107-112)
  🔧 def getBounds (117-122)
  🔧 def getKnownWalkablePositions (122-127)
  🔧 def walkablePositions.push (126-131)
🏛️ class BotComponentManager (132-430)
  🔧 def updateComponents (139-144)
  🔧 def this.clearRegionComponents (156-161)
  🔧 def getRegionsToUpdate (172-177)
  🔧 def newCells.forEach (176-181)
  🔧 def regionsToUpdate.add (180-185)
  🔧 def getRegionCoords (187-192)
  🔧 def getRegionStart (196-201)
  🔧 def findComponentsInRegion (205-210)
  🔧 def createVisitedArray (239-244)
  🔧 def components[currentComponentId].push (268-273)
  🔧 def getConnectivityPattern (278-283)
  🔧 def createComponentNodes (293-298)
  🔧 def components.forEach (297-302)
  🔧 def updateComponentMaze (315-320)
  🔧 def components.forEach (317-322)
  🔧 def component.forEach (321-326)
  🔧 def removeOldComponents (330-335)
  🔧 def clearRegionComponents (336-341)
  🔧 def resetGraphEdges (354-359)
  🔧 def detectComponentEdges (358-363)
  🔧 def areDifferentRegions (394-399)
  🔧 def getComponentIdFromMaze (404-409)
  🔧 def addBidirectionalEdge (419-424)
🏛️ class DiscoveredComponentProvider (486-503)
  🔧 def getGraph (491-496)
  🔧 def getMaze (495-500)
  🔧 def getComponentId (499-504)
🏛️ class TerrainKnowledgeMap (505-569)
  🔧 def updateFromSensorReadings (514-519)
  🔧 def newlyDiscoveredCells.push (532-537)
  🔧 def newlyDiscoveredCells.push (537-542)
  🔧 def getDiscoveryStats (550-555)
  🔧 def getCompatibilityData (565-570)
🏛️ class ComponentGraph (571-661)
  🔧 def addNode (576-581)
  🔧 def this.nodes.set (577-582)
  🔧 def removeNode (588-593)
  🔧 def getNode (599-604)
  🔧 def hasNode (603-608)
  🔧 def addEdge (607-612)
  🔧 def fromNode.transitions.push (623-628)
  🔧 def toNode.transitions.push (624-629)
  🔧 def removeNodesWithPrefix (628-633)
  🔧 def resetAllEdges (641-646)
  🔧 def getAllNodes (648-653)
  🔧 def addNodes (655-660)
🏛️ class BotKnowledgeManager (663-772)
  🔧 def this.terrainKnowledge.updateFromSensorReadings (671-676)
  🔧 def updateFromSensorReadings (702-707)
  🔧 def updateRobotPosition (706-711)
  🔧 def this.explorationPath.push (709-714)
  🔧 def updateRobotDirection (713-718)
  🔧 def updateTargetTracking (717-722)
  🔧 def addToPreviousTargets (727-732)
  🔧 def getWorldDataProvider (740-745)
  🔧 def getComponentProvider (745-750)
  🔧 def getExplorationState (750-755)
  🔧 def getDiscoveryStats (768-773)
🏛️ class BotPathfindingManager (774-1087)
  🔧 def getMovementDirections (776-781)
  🔧 def switch (795-800)
  🔧 def findAbstractPath (811-816)
  🔧 def aStar (825-830)
  🔧 def componentGraph.getNode (829-834)
  🔧 def findPathInComponent (839-844)
  🔧 def getNeighbors: (863-868)
  🔧 def validComponentCells.has (875-880)
  🔧 def availableNeighbors.push (876-881)
  🔧 def findNearestValidCell (893-898)
  🔧 def buildDetailedPath (908-913)
  🔧 def shouldSkipFirstNode (975-980)
  🔧 def isPositionEqual (981-986)
  🔧 def findHierarchicalPath (987-992)
  🔧 def nearbyPositions.push (1014-1019)
  🔧 def Error (1025-1030)
  🔧 def Error (1029-1034)
  🔧 def Error (1037-1042)
  🔧 def Error (1051-1056)
  🔧 def findPath (1062-1067)
🏛️ class BotFrontierManager (1089-1494)
  🔧 def detectFrontiers (1094-1099)
  🔧 def console.log (1101-1106)
  🔧 def console.log (1102-1107)
  🔧 def console.log (1108-1113)
  🔧 def console.log (1109-1114)
  🔧 def frontierClusters.map (1111-1116)
  🔧 def findFrontierPoints (1119-1124)
  🔧 def discoveredFrontierPoints.push (1150-1155)
  🔧 def getNeighbors (1175-1180)
  🔧 def validNeighbors.push (1190-1195)
  🔧 def groupFrontierPoints (1198-1203)
  🔧 def getPointKey (1242-1247)
  🔧 def `${point.x.toFixed (1244-1249)
  🔧 def calculateDistance (1247-1252)
  🔧 def detectComponentAwareFrontiers (1256-1261)
  🔧 def console.log (1261-1266)
  🔧 def console.log (1265-1270)
  🔧 def console.log (1311-1316)
  🔧 def console.log (1324-1329)
  🔧 def console.log (1325-1330)
  🔧 def console.log (1327-1332)
  🔧 def console.log (1329-1334)
  🔧 def componentAwareFrontiers.push (1341-1346)
  🔧 def console.log (1353-1358)
  🔧 def console.log (1354-1359)
  🔧 def console.log (1355-1360)
  🔧 def console.log (1356-1361)
  🔧 def console.log (1357-1362)
  🔧 def console.log (1358-1363)
  🔧 def console.log (1359-1364)
  🔧 def findClosestComponent (1365-1370)
  🔧 def calculateNavigationTarget (1383-1388)
  🔧 def isComponentReachable (1387-1392)
  🔧 def selectOptimalFrontier (1415-1420)
🏛️ class BotExplorer (1496-1796)
  🔧 def decideNextAction (1514-1519)
  🔧 def console.log (1519-1524)
  🔧 def console.log (1520-1525)
  🔧 def console.log (1521-1526)
  🔧 def console.log (1524-1529)
  🔧 def console.log (1530-1535)
  🔧 def console.log (1544-1549)
  🔧 def console.log (1545-1550)
  🔧 def console.log (1563-1568)
  🔧 def console.log (1564-1569)
  🔧 def console.log (1574-1579)
  🔧 def console.log (1576-1581)
  🔧 def console.log (1578-1583)
  🔧 def console.log (1599-1604)
  🔧 def isTargetReached (1621-1626)
  🔧 def checkForBetterTarget (1629-1634)
  🔧 def step (1659-1664)
  🔧 def console.log (1679-1684)
  🔧 def planMovement (1696-1701)
  🔧 def console.log (1700-1705)
  🔧 def Error (1714-1719)
  🔧 def this.explorationStats.uniquePositions.add (1737-1742)
  🔧 def calculateMovementDirection (1748-1753)
  🔧 def calculateRotationPath (1776-1781)
🏛️ class PrivilegedWorldProvider (73-93)
  🔧 def isWalkable (79-84)
  🔧 def getCellState (86-91)
  🔧 def getBounds (90-95)
🏛️ class BotKnowledgeProvider (95-130)
  🔧 def isWalkable (102-107)
  🔧 def getCellState (107-112)
  🔧 def getBounds (117-122)
  🔧 def getKnownWalkablePositions (122-127)
  🔧 def walkablePositions.push (126-131)
🏛️ class DiscoveredComponentProvider (486-503)
  🔧 def getGraph (491-496)
  🔧 def getMaze (495-500)
  🔧 def getComponentId (499-504)
🏛️ class BotKnowledgeManager (663-772)
  🔧 def this.terrainKnowledge.updateFromSensorReadings (671-676)
  🔧 def updateFromSensorReadings (702-707)
  🔧 def updateRobotPosition (706-711)
  🔧 def this.explorationPath.push (709-714)
  🔧 def updateRobotDirection (713-718)
  🔧 def updateTargetTracking (717-722)
  🔧 def addToPreviousTargets (727-732)
  🔧 def getWorldDataProvider (740-745)
  🔧 def getComponentProvider (745-750)
  🔧 def getExplorationState (750-755)
  🔧 def getDiscoveryStats (768-773)
🏛️ class BotExplorer (1496-1796)
  🔧 def decideNextAction (1514-1519)
  🔧 def console.log (1519-1524)
  🔧 def console.log (1520-1525)
  🔧 def console.log (1521-1526)
  🔧 def console.log (1524-1529)
  🔧 def console.log (1530-1535)
  🔧 def console.log (1544-1549)
  🔧 def console.log (1545-1550)
  🔧 def console.log (1563-1568)
  🔧 def console.log (1564-1569)
  🔧 def console.log (1574-1579)
  🔧 def console.log (1576-1581)
  🔧 def console.log (1578-1583)
  🔧 def console.log (1599-1604)
  🔧 def isTargetReached (1621-1626)
  🔧 def checkForBetterTarget (1629-1634)
  🔧 def step (1659-1664)
  🔧 def console.log (1679-1684)
  🔧 def planMovement (1696-1701)
  🔧 def console.log (1700-1705)
  🔧 def Error (1714-1719)
  🔧 def this.explorationStats.uniquePositions.add (1737-1742)
  🔧 def calculateMovementDirection (1748-1753)
  🔧 def calculateRotationPath (1776-1781)

📁 src/lib/exploration/bot-state.js
----------------------------------------
⚙️ def calculateHeuristicDistance (13-31)
⚙️ def createDeepCopyOf3DArray (37-49)
⚙️ def getCompId3D (438-473)
⚙️ def printGetCompId3DStats (476-484)
🏛️ class PrivilegedWorldProvider (73-93)
  🔧 def isWalkable (79-84)
  🔧 def getCellState (86-91)
  🔧 def getBounds (90-95)
🏛️ class BotKnowledgeProvider (95-130)
  🔧 def isWalkable (102-107)
  🔧 def getCellState (107-112)
  🔧 def getBounds (117-122)
  🔧 def getKnownWalkablePositions (122-127)
  🔧 def walkablePositions.push (126-131)
🏛️ class BotComponentManager (132-430)
  🔧 def updateComponents (139-144)
  🔧 def this.clearRegionComponents (156-161)
  🔧 def getRegionsToUpdate (172-177)
  🔧 def newCells.forEach (176-181)
  🔧 def regionsToUpdate.add (180-185)
  🔧 def getRegionCoords (187-192)
  🔧 def getRegionStart (196-201)
  🔧 def findComponentsInRegion (205-210)
  🔧 def createVisitedArray (239-244)
  🔧 def components[currentComponentId].push (268-273)
  🔧 def getConnectivityPattern (278-283)
  🔧 def createComponentNodes (293-298)
  🔧 def components.forEach (297-302)
  🔧 def updateComponentMaze (315-320)
  🔧 def components.forEach (317-322)
  🔧 def component.forEach (321-326)
  🔧 def removeOldComponents (330-335)
  🔧 def clearRegionComponents (336-341)
  🔧 def resetGraphEdges (354-359)
  🔧 def detectComponentEdges (358-363)
  🔧 def areDifferentRegions (394-399)
  🔧 def getComponentIdFromMaze (404-409)
  🔧 def addBidirectionalEdge (419-424)
🏛️ class DiscoveredComponentProvider (486-503)
  🔧 def getGraph (491-496)
  🔧 def getMaze (495-500)
  🔧 def getComponentId (499-504)
🏛️ class TerrainKnowledgeMap (505-569)
  🔧 def updateFromSensorReadings (514-519)
  🔧 def newlyDiscoveredCells.push (532-537)
  🔧 def newlyDiscoveredCells.push (537-542)
  🔧 def getDiscoveryStats (550-555)
  🔧 def getCompatibilityData (565-570)
🏛️ class ComponentGraph (571-661)
  🔧 def addNode (576-581)
  🔧 def this.nodes.set (577-582)
  🔧 def removeNode (588-593)
  🔧 def getNode (599-604)
  🔧 def hasNode (603-608)
  🔧 def addEdge (607-612)
  🔧 def fromNode.transitions.push (623-628)
  🔧 def toNode.transitions.push (624-629)
  🔧 def removeNodesWithPrefix (628-633)
  🔧 def resetAllEdges (641-646)
  🔧 def getAllNodes (648-653)
  🔧 def addNodes (655-660)
🏛️ class BotKnowledgeManager (663-772)
  🔧 def this.terrainKnowledge.updateFromSensorReadings (671-676)
  🔧 def updateFromSensorReadings (702-707)
  🔧 def updateRobotPosition (706-711)
  🔧 def this.explorationPath.push (709-714)
  🔧 def updateRobotDirection (713-718)
  🔧 def updateTargetTracking (717-722)
  🔧 def addToPreviousTargets (727-732)
  🔧 def getWorldDataProvider (740-745)
  🔧 def getComponentProvider (745-750)
  🔧 def getExplorationState (750-755)
  🔧 def getDiscoveryStats (768-773)

📁 src/lib/exploration/directional_sensor.js
----------------------------------------
🏛️ class DirectionalSensorSimulator (2-89)
  🔧 def scan (12-17)
  🔧 def this.isWithinDirectionalCone (34-39)
  🔧 def getDirectionVector (49-54)
  🔧 def isWithinDirectionalCone (64-69)

📁 src/lib/exploration/minecraft_env.js
----------------------------------------
🏛️ class MineflayerWorldProvider (35-121)
  🔧 def constructor (36-41)
  🔧 def isWalkable (55-60)
  🔧 def getCellState (84-89)
  🔧 def console.log (101-106)
  🔧 def console.log (102-107)
  🔧 def console.log (109-114)
  🔧 def clearCache (118-123)
🏛️ class ExplorerController (131-383)
  🔧 def constructor (132-137)
  🔧 def start (146-151)
  🔧 def stop (164-169)
  🔧 def loop (173-178)
  🔧 def console.log (180-185)
  🔧 def console.log (181-186)
  🔧 def console.log (182-187)
  🔧 def switch (185-190)
  🔧 def console.log (192-197)
  🔧 def console.log (217-222)
  🔧 def console.log (218-223)
  🔧 def console.log (219-224)
  🔧 def console.log (225-230)
  🔧 def console.log (228-233)
  🔧 def console.log (229-234)
  🔧 def console.log (233-238)
  🔧 def console.log (235-240)
  🔧 def console.log (242-247)
  🔧 def this.bot.chat (250-255)
  🔧 def this.bot.chat (251-256)
  🔧 def console.log (262-267)
  🔧 def console.log (263-268)
  🔧 def console.log (264-269)
  🔧 def console.log (269-274)
  🔧 def console.log (272-277)
  🔧 def console.log (273-278)
  🔧 def console.log (274-279)
  🔧 def console.log (287-292)
  🔧 def console.log (289-294)
  🔧 def catch (295-300)
  🔧 def scan (307-312)
  🔧 def readings.push (332-337)
  🔧 def console.log (339-344)
  🔧 def console.log (340-345)
  🔧 def console.log (344-349)
  🔧 def console.log (352-357)
  🔧 def moveToTarget (369-374)
  🔧 def catch (378-383)
  🔧 def console.warn (379-384)

📁 src/lib/exploration/simple_astar.js
----------------------------------------
⚙️ def getKey (1-3)
⚙️ def aStar (5-49)
⚙️ def reconstructPath (51-61)

📁 src/lib/exploration/simulation_env.js
----------------------------------------
⚙️ def loadRefactoredComponents (460-474)
⚙️ def createExplorationSystem (478-526)
⚙️ def runLegacyExploration (528-552)
⚙️ def createExplorationSystem (478-526)
⚙️ def runLegacyExploration (528-552)
⚙️ def loadRefactoredComponents (460-474)
⚙️ def createExplorationSystem (478-526)
⚙️ def runLegacyExploration (528-552)
⚙️ def createExplorationSystem (478-526)
⚙️ def runLegacyExploration (528-552)
🏛️ class WorldManager (17-53)
  🔧 def createWorld3D (29-34)
  🔧 def defaultIsWalkable (35-40)
  🔧 def getBounds (39-44)
  🔧 def createPrivilegedProvider (43-48)
  🔧 def isWithinBounds (47-52)
🏛️ class SensorReading (55-62)
🏛️ class SensorSimulator (64-125)
  🔧 def scan (74-79)
  🔧 def isWithinBounds (111-116)
  🔧 def hasLineOfSight (121-126)
🏛️ class CoverageCalculator (127-197)
  🔧 def calculate (135-140)
  🔧 def calculateTotalWalkableCells (142-147)
  🔧 def calculateKnownWalkableCells (166-171)
  🔧 def areBoundsEqual (186-191)
🏛️ class SimulationHarness (199-454)
  🔧 def initialize (210-215)
  🔧 def runSimulation (234-239)
  🔧 def console.log (264-269)
  🔧 def onProgress (267-272)
  🔧 def catch (268-273)
  🔧 def console.log (280-285)
  🔧 def console.log (351-356)
  🔧 def onProgress (359-364)
  🔧 def performSensorScanning (379-384)
  🔧 def executeMovement (402-407)
  🔧 def rotateAndSense (417-422)
  🔧 def getSimulationState (448-453)

