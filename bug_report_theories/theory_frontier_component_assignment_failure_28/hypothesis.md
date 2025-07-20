# Theory 28: Frontier Component Assignment Failure

## Hypothesis
The frontier detection system successfully finds frontier points, but these points fail to get assigned to components during the `detectComponentAwareFrontiers()` process, causing all frontiers to be dropped and leading to the "No valid targets found" infinite loop.

**The bug occurs because**:
- `detectFrontiers()` successfully identifies frontier points adjacent to unknown areas
- `detectComponentAwareFrontiers()` attempts to assign component IDs to these frontier points
- The component assignment fails because `componentProvider.getComponentId(discretePosition)` returns null
- This happens due to walkability mismatches between frontier detection and component lookup
- Frontiers without valid component IDs are dropped (see lines 590-597 in `bot-actions.js`)
- When all frontiers are dropped, no valid targets remain for exploration

## Evidence from Code
In `bot-actions.js` lines 538-596, the frontier assignment logic:
```javascript
let associatedComponentId = componentProvider.getComponentId(discretePosition);

if (!associatedComponentId) {
    // ... extensive debug logging for failures ...
    theory4_primaryLookupFails++;
    // Frontiers without components get dropped
    theory4_droppedFrontiers++;
}
```

The code already tracks "dropped frontiers" suggesting this is a known issue.

## Root Cause Analysis
- Frontier points are detected based on `getCellState()` walkability 
- But component assignment uses the component maze which is built using `isWalkable()`
- Due to walkability method inconsistency (Theory 26), frontier points exist in positions that don't have component data
- The component lookup fails, causing frontiers to be discarded
- This is the mechanism by which the walkability mismatch translates to exploration failure

## Detection Method
Add logging to track:
1. How many frontiers are initially detected by `detectFrontiers()`
2. How many fail component assignment in `detectComponentAwareFrontiers()`
3. The specific positions where component lookup fails
4. Correlation between failed component lookups and walkability mismatches

## Expected Outcome
If this theory is correct, we should see:
1. `detectFrontiers()` finding valid frontier points
2. High rate of component assignment failures in `detectComponentAwareFrontiers()`
3. Progressive reduction in available frontiers until none remain
4. Component lookup failures correlating with walkability method mismatches
5. The final frontier count reaching zero, triggering the infinite retry loop

## Priority
**HIGH** - This is the direct mechanism by which the underlying walkability issues (Theories 26-27) manifest as exploration failure.