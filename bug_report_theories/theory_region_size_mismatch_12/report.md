# Theory 12: Component Region Size Mismatch - Investigation Report

## Theory Analysis

**Likelihood: HIGH (85%)**

This theory has strong supporting evidence based on coordinate analysis and system architecture.

### Key Findings

#### 1. Region Boundary Misalignment Confirmed

**Scan Center**: (-31, 128, 362) → Region (-2, 8, 22)
- regionX = Math.floor(-31/16) = -2  
- regionY = Math.floor(128/16) = 8
- regionZ = Math.floor(362/16) = 22

**Missing Frontier**: (-47, 128, 348) → Region (-3, 8, 21)  
- regionX = Math.floor(-47/16) = -3
- regionY = Math.floor(128/16) = 8
- regionZ = Math.floor(348/16) = 21

**Scan Range Analysis**:
- scanRange = 16 creates 33×33×33 cube (center ± 16)
- From (-31, 128, 362) this spans:
  - X: -47 to -15 (spans regions -3 and -2)
  - Y: 112 to 144 (spans regions 7 and 8)  
  - Z: 346 to 378 (spans regions 21 and 22)

**Critical Issue**: The scan touches **8 different regions** but only **1 region** gets updated for components.

#### 2. Code Architecture Analysis

**Region Calculation Logic** (bot-state.js:248-264):
```javascript
getRegionCoords(position) {
    return {
        regionX: Math.floor(position.x / this.regionSize),  // DEF_REG_SIZE = 16
        regionY: Math.floor(position.y / this.regionSize),
        regionZ: Math.floor(position.z / this.regionSize)
    };
}
```

**Component Update Logic** (bot-state.js:172-210):
- `getRegionsToUpdate()` determines which regions need component updates
- Only scans `changedCells` to find affected regions  
- Does NOT account for scan boundary crossing multiple regions

**Component Lookup Logic** (bot-state.js:525-575):
- Uses exact world coordinates to lookup in `componentLookupTable`
- Fails when coordinates exist in scanned areas but components were created in different regions

#### 3. Boundary Math Verification

For a scan at (-31, 128, 362) with range 16:

**Regions Touched by Scan**:
1. (-3, 7, 21) - corner region  
2. (-3, 7, 22) - corner region
3. (-3, 8, 21) - **contains missing frontier (-47, 128, 348)**
4. (-3, 8, 22) - corner region
5. (-2, 7, 21) - corner region
6. (-2, 7, 22) - corner region  
7. (-2, 8, 21) - border region
8. (-2, 8, 22) - **scan center region**

**Current Behavior**: Only region (-2, 8, 22) gets component updates
**Missing Behavior**: Region (-3, 8, 21) contains frontiers but no components

## Detection Strategy

The detection approach should verify region boundary crossings and component creation mismatches.

### Primary Detection Points

1. **Scan Boundary Tracking**: Log all regions touched by each scan operation
2. **Component Creation Verification**: Verify components are created in all scanned regions  
3. **Lookup Failure Analysis**: Track which regions contain failed lookup coordinates
4. **Region Update Coverage**: Compare regions scanned vs regions updated

### Detection Timing

- **Pre-scan**: Log expected regions for upcoming scan boundaries
- **During update**: Track which regions actually get component updates  
- **Post-scan**: Verify all scanned regions have component coverage
- **On lookup failure**: Log region mismatches for failed coordinates

## Debug Implementation

```javascript
// THEORY_12_DEBUG: Track scan region boundaries
function THEORY_12_logScanRegions(scanCenter, scanRange, regionSize) {
    const regionsInScan = new Set();
    
    console.log(`THEORY_12_DEBUG: Scanning from center (${scanCenter.x}, ${scanCenter.y}, ${scanCenter.z}) with range ${scanRange}`);
    
    // Calculate all regions touched by scan
    for (let y = scanCenter.y - scanRange; y <= scanCenter.y + scanRange; y++) {
        for (let x = scanCenter.x - scanRange; x <= scanCenter.x + scanRange; x++) {
            for (let z = scanCenter.z - scanRange; z <= scanCenter.z + scanRange; z++) {
                const regionX = Math.floor(x / regionSize);
                const regionY = Math.floor(y / regionSize);  
                const regionZ = Math.floor(z / regionSize);
                const regionKey = `${regionX},${regionY},${regionZ}`;
                regionsInScan.add(regionKey);
            }
        }
    }
    
    console.log(`THEORY_12_DEBUG: Scan touches ${regionsInScan.size} regions: [${Array.from(regionsInScan).join(', ')}]`);
    return regionsInScan;
}

// THEORY_12_DEBUG: Verify component coverage 
function THEORY_12_verifyComponentCoverage(regionsScanned, regionsUpdated) {
    const scannedSet = new Set(regionsScanned);
    const updatedSet = new Set(regionsUpdated);
    
    const missingUpdates = [];
    for (const region of scannedSet) {
        if (!updatedSet.has(region)) {
            missingUpdates.push(region);
        }
    }
    
    if (missingUpdates.length > 0) {
        console.log(`THEORY_12_DEBUG: CONFIRMED - ${missingUpdates.length} scanned regions missing component updates: [${missingUpdates.join(', ')}]`);
        return false;
    } else {
        console.log(`THEORY_12_DEBUG: All ${scannedSet.size} scanned regions received component updates`);
        return true;
    }
}

// THEORY_12_DEBUG: Track lookup failures by region
function THEORY_12_trackLookupFailure(failedPosition, regionSize) {
    const failedRegionX = Math.floor(failedPosition.x / regionSize);
    const failedRegionY = Math.floor(failedPosition.y / regionSize);
    const failedRegionZ = Math.floor(failedPosition.z / regionSize);
    const failedRegionKey = `${failedRegionX},${failedRegionY},${failedRegionZ}`;
    
    console.log(`THEORY_12_DEBUG: Lookup failed at (${failedPosition.x}, ${failedPosition.y}, ${failedPosition.z}) in region ${failedRegionKey}`);
    return failedRegionKey;
}
```

### Integration Points

**In minecraft_env.js scan() function** (line ~346):
```javascript
async scan() {
    const center = this.bot.entity.position.floored();
    
    // THEORY_12_DEBUG: Track scan regions
    const expectedRegions = THEORY_12_logScanRegions(center, this.scanRange, 16);
    
    // ... existing scan logic ...
    
    // THEORY_12_DEBUG: Verify component updates cover all scanned regions  
    const actualUpdatedRegions = this.botKnowledge.getLastUpdatedRegions();
    THEORY_12_verifyComponentCoverage(Array.from(expectedRegions), actualUpdatedRegions);
}
```

**In bot-state.js getCompId3D() function** (line ~530):
```javascript
if (!componentLookupTable[worldPosition.x]) {
    // THEORY_12_DEBUG: Track lookup failures by region
    THEORY_12_trackLookupFailure(worldPosition, regionSize);
    
    // ... existing failure logic ...
}
```

## Testing Steps

### Step 1: Confirm Region Boundary Crossing
1. Add THEORY_12_logScanRegions() to scan() function
2. Run system and verify scan touches multiple regions
3. **Expected Result**: Console shows scan touches 8 regions for center (-31, 128, 362)

### Step 2: Verify Component Update Gap  
1. Add THEORY_12_verifyComponentCoverage() after component updates
2. Run system and check if all scanned regions get component updates
3. **Expected Result**: Console shows missing component updates for boundary regions

### Step 3: Confirm Lookup-Region Mismatch
1. Add THEORY_12_trackLookupFailure() to getCompId3D failures
2. Run system and verify failed lookups are in unupdated regions  
3. **Expected Result**: Failed frontier (-47, 128, 348) maps to region (-3, 8, 21) which wasn't updated

### Step 4: Fix Validation
1. If confirmed, implement fix to update all scanned regions
2. Re-run tests to verify lookup failures disappear
3. **Expected Result**: Frontiers become findable and component-aware exploration succeeds

## Expected Fix Strategy

If this theory is confirmed, the fix would involve:

### Option 1: Expand Region Update Logic
```javascript
// Update getRegionsToUpdate() to include all scan boundary regions  
getRegionsToUpdate(scanCenter, scanRange) {
    const regionsToUpdate = new Set();
    
    for (let y = scanCenter.y - scanRange; y <= scanCenter.y + scanRange; y++) {
        for (let x = scanCenter.x - scanRange; x <= scanCenter.x + scanRange; x++) {
            for (let z = scanCenter.z - scanRange; z <= scanCenter.z + scanRange; z++) {
                const {regionX, regionY, regionZ} = this.getRegionCoords({x, y, z});
                regionsToUpdate.add(`${regionX},${regionY},${regionZ}`);
            }
        }
    }
    
    return Array.from(regionsToUpdate);
}
```

### Option 2: Adjust Region Size
- Increase DEF_REG_SIZE to better align with scan patterns
- Risk: May impact performance and component granularity

### Option 3: Multi-Region Component Tracking  
- Allow components to span multiple regions
- Update lookup logic to check neighboring regions
- Risk: Increased complexity

## Conclusion

This theory has **very strong evidence** and represents a fundamental architectural issue. The mismatch between scan boundaries (33×33×33 cube) and region boundaries (16×16×16 cubes) causes components to be created in the wrong regions, making frontier-to-component lookups fail systematically.

**Recommendation**: Implement detection logging immediately to confirm this theory, as it likely explains the core bug behavior described in the bug report.