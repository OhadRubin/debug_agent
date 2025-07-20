### Theory 8: Region Boundary Calculation Error

**Hypothesis**: The region-based component system is calculating incorrect region coordinates or boundaries, causing components to be stored in wrong regions of the lookup table, making them unfindable during `getCompId3D` lookups.

**Supporting Evidence from Bug Report**:
- `getCompId3D` failures show "No X entry at (-35,131,378)" - the coordinates may be looking in wrong region
- Component detection seems to work (components are created) but lookups fail
- Region-based system uses `Math.floor(position.x / regionSize)` which could have off-by-one errors

**Suspected Root Cause**:
The `getRegionCoords()` method in `BotComponentManager` or `getCompId3D()` function is calculating different region coordinates for the same world position, causing components to be stored in one region but looked up in another.

**Detection Method**:
Add debug logging to track:
1. Region coordinates calculated during component creation vs lookup
2. Whether components are being stored in the expected region coordinates 
3. Direct verification of region coordinate calculation consistency
4. Comparison of region calculation in `updateComponents` vs `getCompId3D`

**Specific Locations to Instrument**:
- `getRegionCoords()` method in both component creation and lookup paths
- `updateComponentMaze` to log where components are actually stored
- `getCompId3D` to log calculated region coordinates vs actual lookup table structure
- Direct comparison of region calculation results for the same coordinates

**Expected Evidence if Theory is Correct**:
- Different region coordinates calculated for the same world position
- Components stored in regions different from where they're looked up
- Mismatch between `componentLookupTable` structure and `getCompId3D` calculations
- Off-by-one errors in region boundary calculations