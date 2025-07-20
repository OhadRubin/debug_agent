### Theory 9: Component Clearing Race Condition

**Hypothesis**: The component lookup table is being cleared or overwritten after components are created but before they can be used, creating a race condition where components exist in the graph but not in the lookup table.

**Supporting Evidence from Bug Report**:
- Components appear to be detected initially (component graph is created)
- All subsequent lookups fail, suggesting the lookup table becomes empty
- The repeated scanning cycles might be triggering component clearing
- `clearRegionComponents` sets entries to -1, which `getCompId3D` treats as failure

**Suspected Root Cause**:
The `clearRegionComponents` method or repeated `updateComponents` calls are clearing the component lookup table entries faster than they can be used, or there's an async timing issue where the table is cleared between component creation and frontier detection.

**Detection Method**:
Add debug logging to track:
1. When `clearRegionComponents` is called and what entries it clears
2. The state of the lookup table before and after each `updateComponents` call
3. Whether lookup table entries are being overwritten with -1 values
4. Timing of component clearing vs component lookup operations

**Specific Locations to Instrument**:
- `clearRegionComponents` to log what entries are being cleared
- `updateComponents` to log lookup table state before/after updates
- `getCompId3D` to detect if entries were cleared (value = -1)
- Component update timing in minecraft_env.js scan cycle

**Expected Evidence if Theory is Correct**:
- Lookup table entries will show -1 values where components should be
- `clearRegionComponents` will be clearing entries that were just created
- Timing mismatch between component creation and usage
- Lookup table becomes progressively emptier despite new walkable cells being found