


while running  minecraft_env.js 

```bash
>> node src/lib/exploration/minecraft_env.js 
(node:32529) [DEP0040] DeprecationWarning: The `punycode` module is deprecated. Please use a userland alternative instead.
(Use `node --trace-deprecation ...` to show where the warning was created)
(node:32529) ExperimentalWarning: CommonJS module /Users/ohadr/Auto-Craft-Bot/src/lib/exploration/minecraft_env.js is loading ES Module /Users/ohadr/Auto-Craft-Bot/src/lib/exploration/bot-state.js using require().
Support for loading ES Module in require() is an experimental feature and might change at any time
(node:32529) [MODULE_TYPELESS_PACKAGE_JSON] Warning: Module type of file:///Users/ohadr/Auto-Craft-Bot/src/lib/exploration/simple_astar.js is not specified and it doesn't parse as CommonJS.
Reparsing as ES module because module syntax was detected. This incurs a performance overhead.
To eliminate this warning, add "type": "module" to /Users/ohadr/Auto-Craft-Bot/package.json.
Bot has spawned.
Finished waiting 3 seconds after spawn.
Navigating to initial position: (-33, 128, 368)
Successfully reached initial position
SCAN_DEBUG: Starting environmental scan
THEORY_26_DEBUG: WALKABILITY_MISMATCH_CONFIRMED - The two walkability methods disagree!
THEORY_26_DEBUG: getCellState() uses hasSolidGroundBelow() with maxDropDown=1
THEORY_26_DEBUG: isWalkable() uses pathfinder .safe and .physical properties
THEORY_26_DEBUG: This will cause frontier component assignment failures and infinite loops
Discovered 274624 new cells. Re-evaluating components...
THINKING_DEBUG: Starting agent thinking
[Target Set] New target: (-1,128,364)
[Agent Step] Getting complete plan - Target: (-1,128,364), Robot: (-33,128,368)
[Plan Movement] Called with target: (-1,128,364), robot: (-33,128,368)
MOVEMENT_DEBUG: Starting movement to target: {"x":-1,"y":128,"z":364,"componentId":"-1,8,22_0","groupSize":1,"points":[{"x":-1,"y":128,"z":364}],"abstractCost":2,"pathCost":31.656}
MOVEMENT_DEBUG: Current bot position: {"x":-33,"y":128,"z":368}
MOVEMENT_DEBUG: Final destination: {"x":-1,"y":128,"z":364}
MOVEMENT_DEBUG: Movement success: true
MOVEMENT_DEBUG: Final bot position: {"x":-1,"y":128,"z":364}
MOVEMENT_DEBUG: Expected position: {"x":-1,"y":128,"z":364,"componentId":"-1,8,22_0","groupSize":1,"points":[{"x":-1,"y":128,"z":364}],"abstractCost":2,"pathCost":31.656}
SCAN_DEBUG: Starting environmental scan
  1. (-38,128,377): getCellState=WALKABLE, isWalkable=false, block=air
  2. (-37,128,355): getCellState=WALKABLE, isWalkable=false, block=air
  3. (-37,128,356): getCellState=WALKABLE, isWalkable=false, block=air
  4. (-37,128,357): getCellState=WALKABLE, isWalkable=false, block=air
  5. (-37,128,358): getCellState=WALKABLE, isWalkable=false, block=air
Discovered 143780 new cells. Re-evaluating components...
THINKING_DEBUG: Starting agent thinking
[Target Set] New target: (-48,128,400)
[Agent Step] Getting complete plan - Target: (-48,128,400), Robot: (-1,128,364)
[Plan Movement] Called with target: (-48,128,400), robot: (-1,128,364)
MOVEMENT_DEBUG: Starting movement to target: {"x":-48,"y":128,"z":400,"componentId":"-3,8,25_0","groupSize":1,"points":[{"x":-48,"y":128,"z":400}],"abstractCost":3,"pathCost":61.738}
MOVEMENT_DEBUG: Current bot position: {"x":-1,"y":128,"z":364}
MOVEMENT_DEBUG: Final destination: {"x":-48,"y":128,"z":400}
MOVEMENT_DEBUG: Movement success: true
MOVEMENT_DEBUG: Final bot position: {"x":-48,"y":128,"z":400}
MOVEMENT_DEBUG: Expected position: {"x":-48,"y":128,"z":400,"componentId":"-3,8,25_0","groupSize":1,"points":[{"x":-48,"y":128,"z":400}],"abstractCost":3,"pathCost":61.738}
SCAN_DEBUG: Starting environmental scan
  1. (-38,128,377): getCellState=WALKABLE, isWalkable=false, block=air
  2. (-37,128,355): getCellState=WALKABLE, isWalkable=false, block=air
  3. (-37,128,356): getCellState=WALKABLE, isWalkable=false, block=air
  4. (-37,128,357): getCellState=WALKABLE, isWalkable=false, block=air
  5. (-37,128,358): getCellState=WALKABLE, isWalkable=false, block=air
Discovered 167375 new cells. Re-evaluating components...
  Example 1: (-33,128,368) -> region(-3,8,23) boundaries[X:false, Y:true, Z:true]
  Example 2: (-65,96,336) -> region(-5,6,21) boundaries[X:false, Y:true, Z:true]
  Example 3: (-65,96,337) -> region(-5,6,21) boundaries[X:false, Y:true, Z:false]
  Example 4: (-65,96,338) -> region(-5,6,21) boundaries[X:false, Y:true, Z:false]
  Example 5: (-65,96,339) -> region(-5,6,21) boundaries[X:false, Y:true, Z:false]
THINKING_DEBUG: Starting agent thinking
THEORY_28_DEBUG: FRONTIER_ASSIGNMENT_FAILURE - Processed 26 frontier points
THEORY_28_DEBUG: FRONTIER_ASSIGNMENT_FAILURE - Successful assignments: 23
THEORY_28_DEBUG: FRONTIER_ASSIGNMENT_FAILURE - Primary lookup failures: 3
THEORY_28_DEBUG: FRONTIER_ASSIGNMENT_FAILURE - Dropped frontiers: 3
THEORY_28_DEBUG: FRONTIER_ASSIGNMENT_FAILURE - Final component-aware frontiers: 6
THINKING_DEBUG: No valid targets found. Retrying in a moment...
SCAN_DEBUG: Starting environmental scan
THINKING_DEBUG: Starting agent thinking
THEORY_28_DEBUG: FRONTIER_ASSIGNMENT_FAILURE - Processed 26 frontier points
THEORY_28_DEBUG: FRONTIER_ASSIGNMENT_FAILURE - Successful assignments: 23
THEORY_28_DEBUG: FRONTIER_ASSIGNMENT_FAILURE - Primary lookup failures: 3
THEORY_28_DEBUG: FRONTIER_ASSIGNMENT_FAILURE - Dropped frontiers: 3
THEORY_28_DEBUG: FRONTIER_ASSIGNMENT_FAILURE - Final component-aware frontiers: 6
THINKING_DEBUG: No valid targets found. Retrying in a moment...
SCAN_DEBUG: Starting environmental scan
THINKING_DEBUG: Starting agent thinking
THEORY_28_DEBUG: FRONTIER_ASSIGNMENT_FAILURE - Processed 26 frontier points
THEORY_28_DEBUG: FRONTIER_ASSIGNMENT_FAILURE - Successful assignments: 23
THEORY_28_DEBUG: FRONTIER_ASSIGNMENT_FAILURE - Primary lookup failures: 3
THEORY_28_DEBUG: FRONTIER_ASSIGNMENT_FAILURE - Dropped frontiers: 3
THEORY_28_DEBUG: FRONTIER_ASSIGNMENT_FAILURE - Final component-aware frontiers: 6
THINKING_DEBUG: No valid targets found. Retrying in a moment...
THEORY_30_DEBUG: CONFIRMED - Infinite loop detected! 3 consecutive no-target cycles
THEORY_30_DEBUG: Time since exploration start: 6.169s
SCAN_DEBUG: Starting environmental scan
  1. (-38,128,377): getCellState=WALKABLE, isWalkable=false, block=air
  2. (-37,128,355): getCellState=WALKABLE, isWalkable=false, block=air
  3. (-37,128,356): getCellState=WALKABLE, isWalkable=false, block=air
  4. (-37,128,357): getCellState=WALKABLE, isWalkable=false, block=air
  5. (-37,128,358): getCellState=WALKABLE, isWalkable=false, block=air
THINKING_DEBUG: Starting agent thinking
THEORY_28_DEBUG: FRONTIER_ASSIGNMENT_FAILURE - Processed 26 frontier points
THEORY_28_DEBUG: FRONTIER_ASSIGNMENT_FAILURE - Successful assignments: 23
THEORY_28_DEBUG: FRONTIER_ASSIGNMENT_FAILURE - Primary lookup failures: 3
THEORY_28_DEBUG: FRONTIER_ASSIGNMENT_FAILURE - Dropped frontiers: 3
THEORY_28_DEBUG: FRONTIER_ASSIGNMENT_FAILURE - Final component-aware frontiers: 6
THINKING_DEBUG: No valid targets found. Retrying in a moment...
THEORY_30_DEBUG: CONFIRMED - Infinite loop detected! 4 consecutive no-target cycles
THEORY_30_DEBUG: Time since exploration start: 9.209s
SCAN_DEBUG: Starting environmental scan
THINKING_DEBUG: Starting agent thinking
THEORY_28_DEBUG: FRONTIER_ASSIGNMENT_FAILURE - Processed 26 frontier points
THEORY_28_DEBUG: FRONTIER_ASSIGNMENT_FAILURE - Successful assignments: 23
THEORY_28_DEBUG: FRONTIER_ASSIGNMENT_FAILURE - Primary lookup failures: 3
THEORY_28_DEBUG: FRONTIER_ASSIGNMENT_FAILURE - Dropped frontiers: 3
THEORY_28_DEBUG: FRONTIER_ASSIGNMENT_FAILURE - Final component-aware frontiers: 6
THINKING_DEBUG: No valid targets found. Retrying in a moment...
THEORY_30_DEBUG: CONFIRMED - Infinite loop detected! 5 consecutive no-target cycles
THEORY_30_DEBUG: Time since exploration start: 12.294s
SCAN_DEBUG: Starting environmental scan
THINKING_DEBUG: Starting agent thinking
  Example 1: (-67,100,432) terrainState=0, walkable=true
  Example 2: (-71,101,432) terrainState=0, walkable=true
  Example 3: (-70,101,432) terrainState=0, walkable=true
THEORY_28_DEBUG: FRONTIER_ASSIGNMENT_FAILURE - Processed 26 frontier points
THEORY_28_DEBUG: FRONTIER_ASSIGNMENT_FAILURE - Successful assignments: 23
THEORY_28_DEBUG: FRONTIER_ASSIGNMENT_FAILURE - Primary lookup failures: 3
THEORY_28_DEBUG: FRONTIER_ASSIGNMENT_FAILURE - Dropped frontiers: 3
THEORY_28_DEBUG: FRONTIER_ASSIGNMENT_FAILURE - Final component-aware frontiers: 6
THINKING_DEBUG: No valid targets found. Retrying in a moment...
THEORY_30_DEBUG: CONFIRMED - Infinite loop detected! 6 consecutive no-target cycles
THEORY_30_DEBUG: Time since exploration start: 15.419s
SCAN_DEBUG: Starting environmental scan
THINKING_DEBUG: Starting agent thinking
THEORY_28_DEBUG: FRONTIER_ASSIGNMENT_FAILURE - Processed 26 frontier points
THEORY_28_DEBUG: FRONTIER_ASSIGNMENT_FAILURE - Successful assignments: 23
THEORY_28_DEBUG: FRONTIER_ASSIGNMENT_FAILURE - Primary lookup failures: 3
THEORY_28_DEBUG: FRONTIER_ASSIGNMENT_FAILURE - Dropped frontiers: 3
THEORY_28_DEBUG: FRONTIER_ASSIGNMENT_FAILURE - Final component-aware frontiers: 6
THINKING_DEBUG: No valid targets found. Retrying in a moment...
THEORY_30_DEBUG: CONFIRMED - Infinite loop detected! 7 consecutive no-target cycles
THEORY_30_DEBUG: Time since exploration start: 18.522s
SCAN_DEBUG: Starting environmental scan
  1. (-38,128,377): getCellState=WALKABLE, isWalkable=false, block=air
  2. (-37,128,355): getCellState=WALKABLE, isWalkable=false, block=air
  3. (-37,128,356): getCellState=WALKABLE, isWalkable=false, block=air
  4. (-37,128,357): getCellState=WALKABLE, isWalkable=false, block=air
  5. (-37,128,358): getCellState=WALKABLE, isWalkable=false, block=air
THINKING_DEBUG: Starting agent thinking
THEORY_28_DEBUG: FRONTIER_ASSIGNMENT_FAILURE - Processed 26 frontier points
THEORY_28_DEBUG: FRONTIER_ASSIGNMENT_FAILURE - Successful assignments: 23
THEORY_28_DEBUG: FRONTIER_ASSIGNMENT_FAILURE - Primary lookup failures: 3
THEORY_28_DEBUG: FRONTIER_ASSIGNMENT_FAILURE - Dropped frontiers: 3
THEORY_28_DEBUG: FRONTIER_ASSIGNMENT_FAILURE - Final component-aware frontiers: 6
THINKING_DEBUG: No valid targets found. Retrying in a moment...
THEORY_30_DEBUG: CONFIRMED - Infinite loop detected! 8 consecutive no-target cycles
THEORY_30_DEBUG: Time since exploration start: 21.575s
^C
>> 
<continues forver>
```