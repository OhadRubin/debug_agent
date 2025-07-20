# Theory 12: Component Region Size Mismatch

## Hypothesis
The component region size (DEF_REG_SIZE = 16) doesn't align properly with the scanning range (scanRange = 16) or world coordinate calculations, causing components to be created in the wrong regions and making them unfindable during frontier-to-component lookup.

## Supporting Evidence from Bug Report
- Bug report shows: "THEORY_10_DEBUG: CONFIRMED - Missing X entries in lookup table (has 1 X keys, missing -47)"
- All frontier lookups fail because coordinates don't exist in componentLookupTable
- Component update creates 0 components despite finding walkable cells
- Scan center is at (-31,128,362) but missing frontier is at (-47,128,348)

## Key Observations
1. Scan range of 16 creates a 33x33x33 cube (center ± 16)
2. Region size of 16 means regions are 16x16x16 blocks
3. Region coordinates calculated as `Math.floor(position / regionSize)`
4. Potential misalignment between scan boundaries and region boundaries
5. Components might be created in adjacent regions and not findable

## Detection Strategy
Add detailed logging to track:
1. Scan boundaries vs region boundaries for each scan
2. Which regions are being updated vs which regions contain scanned cells
3. Component creation coordinates vs lookup coordinates
4. Region coordinate calculations for both scan center and frontier positions

## Test Cases
1. Compare scan boundaries (-31±16) with region boundaries
2. Verify frontier coordinates (-47,128,348) should be in which region
3. Check if components are created in correct regions for their cell coordinates
4. Test if componentLookupTable entries match the regions being scanned

## Expected Fix
If confirmed, fix could involve:
- Adjusting region size to better align with scan patterns
- Expanding region update logic to include boundary regions
- Fixing region coordinate calculation logic
- Ensuring all scanned regions are properly updated for components