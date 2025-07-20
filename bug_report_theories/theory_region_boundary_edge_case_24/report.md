# Theory 24: Region Boundary Edge Case - Investigation Report

## Theory Analysis Section

### Theory Summary
The hypothesis predicts that frontier position (-36,128,378) falls on a region boundary where component assignment logic fails due to incorrect region coordinate calculations or boundary edge cases in the component maze structure.

### Code Analysis
After analyzing the codebase, I found several key insights:

**Region Coordinate Calculation** (bot-state.js:248-255):
```javascript
getRegionCoords(position) {
    return {
        regionX: Math.floor(position.x / this.regionSize),
        regionY: Math.floor(position.y / this.regionSize), 
        regionZ: Math.floor(position.z / this.regionSize)
    };
}
```

**Critical Finding - Position (-36,128,378) Region Mapping**:
- regionX = Math.floor(-36 / 16) = Math.floor(-2.25) = **-3**
- regionY = Math.floor(128 / 16) = Math.floor(8.0) = **8** 
- regionZ = Math.floor(378 / 16) = Math.floor(23.625) = **23**

**Boundary Analysis**:
- **x = -36**: This is exactly -2.25 * 16, placing it on the boundary between regions -3 and -2
- **y = 128**: This is exactly 8.0 * 16, placing it on the boundary between regions 7 and 8  
- **z = 378**: This is 23.625 * 16, NOT on a region boundary

**The position (-36,128,378) is on TWO region boundaries simultaneously (x and y)**.

**getCompId3D Function Analysis** (bot-state.js:527-584):
The function that fails reports "No Z entry", but the lookup sequence is:
1. Check `componentLookupTable[worldPosition.x]` - this should exist for x=-36
2. Check `componentLookupTable[x][worldPosition.y]` - this should exist for y=128
3. **FAILS** at `componentLookupTable[x][y][worldPosition.z]` - this fails for z=378

**Component Update Process**:
Looking at `clearRegionComponents` (bot-state.js:412-439), when regions are cleared they iterate through world coordinates in the region. For region (-3,8,23):
- worldStartX = -3 * 16 = **-48**
- worldStartY = 8 * 16 = **128** 
- worldStartZ = 23 * 16 = **368**

The region spans from (-48,128,368) to (-33,143,383). Position (-36,128,378) should be within this region.

**Potential Issue**: The position (-36,128,378) being on dual boundaries could cause it to be processed by one region but looked up as belonging to another region, or cause edge cases in the component update logic.

### Likelihood Assessment
**HIGH** - The position being on dual region boundaries (x and y) creates a high probability for edge case failures in region-based component management.

## Detection Strategy Section

### Detection Method
Add detailed logging to track region coordinate mapping, boundary detection, and component maze structure around the failing position (-36,128,378).

### Key Locations
1. **bot-state.js:248-255** - `getRegionCoords()` function
2. **bot-state.js:527-584** - `getCompId3D()` function  
3. **bot-state.js:412-439** - `clearRegionComponents()` function
4. **bot-state.js:376-404** - `updateComponentMaze()` function

### Expected Evidence
**If theory is correct:**
- Position (-36,128,378) will show boundary conditions (coordinates divisible by 16)
- Region mapping will show dual boundary status
- Component maze structure will show gaps or inconsistencies around boundary positions
- Different phases (update vs lookup) may assign the position to different regions

**If theory is wrong:**
- Position (-36,128,378) will show normal region assignment
- Component maze structure will be consistent across regions
- No boundary-related issues will be detected

## Debug Implementation Section

### Specific Debug Code

```javascript
// Add to getRegionCoords function (bot-state.js:248)
getRegionCoords(position) {
    const regionCoords = {
        regionX: Math.floor(position.x / this.regionSize),
        regionY: Math.floor(position.y / this.regionSize),
        regionZ: Math.floor(position.z / this.regionSize)
    };
    
    // THEORY_24_DEBUG: Track boundary conditions
    const isBoundaryX = (position.x % this.regionSize) === 0;
    const isBoundaryY = (position.y % this.regionSize) === 0; 
    const isBoundaryZ = (position.z % this.regionSize) === 0;
    
    if (position.x === -36 && position.y === 128 && position.z === 378) {
        console.log(`THEORY_24_DEBUG: TARGET_POSITION - (${position.x},${position.y},${position.z}) maps to region (${regionCoords.regionX},${regionCoords.regionY},${regionCoords.regionZ})`);
        console.log(`THEORY_24_DEBUG: BOUNDARY_STATUS - X:${isBoundaryX}, Y:${isBoundaryY}, Z:${isBoundaryZ}`);
        console.log(`THEORY_24_DEBUG: REGION_START - (${regionCoords.regionX * this.regionSize},${regionCoords.regionY * this.regionSize},${regionCoords.regionZ * this.regionSize})`);
    }
    
    return regionCoords;
}

// Add to getCompId3D function (bot-state.js:527)
function getCompId3D(worldPosition, componentLookupTable, regionSize) {
    if (worldPosition.x === -36 && worldPosition.y === 128 && worldPosition.z === 378) {
        console.log(`THEORY_24_DEBUG: LOOKUP_ATTEMPT - Position (${worldPosition.x},${worldPosition.y},${worldPosition.z})`);
        console.log(`THEORY_24_DEBUG: MAZE_STRUCTURE - X exists: ${!!componentLookupTable[worldPosition.x]}`);
        if (componentLookupTable[worldPosition.x]) {
            console.log(`THEORY_24_DEBUG: MAZE_STRUCTURE - Y exists: ${!!componentLookupTable[worldPosition.x][worldPosition.y]}`);
            if (componentLookupTable[worldPosition.x][worldPosition.y]) {
                console.log(`THEORY_24_DEBUG: MAZE_STRUCTURE - Z exists: ${componentLookupTable[worldPosition.x][worldPosition.y][worldPosition.z] !== undefined}`);
                console.log(`THEORY_24_DEBUG: MAZE_STRUCTURE - Available Z values: [${Object.keys(componentLookupTable[worldPosition.x][worldPosition.y]).join(', ')}]`);
            }
        }
    }
    
    // ... rest of existing function
}

// Add to updateComponentMaze function (bot-state.js:376)
updateComponentMaze(componentLookupTable, components) {
    let theory24_targetFound = false;
    
    components.forEach((component, localComponentId) => {
        component.forEach(cell => {
            if (cell.x === -36 && cell.y === 128 && cell.z === 378) {
                theory24_targetFound = true;
                console.log(`THEORY_24_DEBUG: TARGET_MAZE_UPDATE - Position (${cell.x},${cell.y},${cell.z}) assigned component ${localComponentId}`);
            }
            // ... existing maze update logic
        });
    });
    
    if (!theory24_targetFound) {
        console.log(`THEORY_24_DEBUG: TARGET_NOT_FOUND - Position (-36,128,378) not updated in this component batch`);
    }
}

// Add boundary region tracking
// Add to clearRegionComponents function (bot-state.js:412)
clearRegionComponents(componentLookupTable, regionStart, regionSize) {
    const { worldStartX, worldStartY, worldStartZ } = regionStart;
    
    console.log(`THEORY_24_DEBUG: REGION_CLEAR - Clearing region starting at (${worldStartX},${worldStartY},${worldStartZ})`);
    
    if (worldStartX <= -36 && worldStartX + regionSize > -36 &&
        worldStartY <= 128 && worldStartY + regionSize > 128 &&
        worldStartZ <= 378 && worldStartZ + regionSize > 378) {
        console.log(`THEORY_24_DEBUG: TARGET_REGION_CLEAR - Region contains target position (-36,128,378)`);
    }
    
    // ... existing clear logic
}
```

### Performance Considerations
The debug logging is position-specific (only triggers for (-36,128,378)) to avoid spam while providing detailed boundary analysis for the exact failing case.

### Success Criteria
**Theory confirmed if:**
- Position (-36,128,378) shows boundary conditions in multiple dimensions
- Region mapping inconsistencies appear between update and lookup phases
- Component maze gaps occur around boundary positions

**Theory disproven if:**
- Position (-36,128,378) shows normal region assignment without boundary issues
- Component maze structure is consistent across all phases

## Recommended Testing Steps

### Detection Phase
1. Add debug logging to the four key functions listed above
2. Run `node src/lib/exploration/minecraft_env.js`
3. Monitor output for THEORY_24_DEBUG messages during the failure

### Verification Method
1. **Boundary Detection**: Look for boundary status showing X:true, Y:true for position (-36,128,378)
2. **Region Mapping**: Verify the position maps to region (-3,8,23) consistently
3. **Maze Structure**: Check if the position exists in the component maze after updates
4. **Clear Operations**: Verify the correct region is cleared and updated

### Next Steps
If theory is confirmed:
1. Investigate region boundary handling logic for dual-boundary positions
2. Consider special handling for positions exactly on region boundaries
3. Review component update vs lookup region calculation consistency
4. Implement boundary-aware component assignment logic

If theory is disproven:
1. The region boundary is not the root cause
2. Focus investigation on other theories (component update timing, maze corruption, etc.)
3. The issue likely lies in component creation or maze update synchronization rather than coordinate calculations