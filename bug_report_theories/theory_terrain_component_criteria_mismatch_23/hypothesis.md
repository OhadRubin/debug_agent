# Theory 23: Terrain State vs Component Assignment Criteria Mismatch

## Hypothesis
Cells are marked as WALKABLE by terrain logic but don't meet the stricter criteria for component assignment, creating gaps where terrain knowledge exists but component maze entries don't.

## Evidence from Bug Report
- Position (-36,128,378) has terrain state 0 (WALKABLE) 
- Same position has maze value undefined
- `THEORY_7_DEBUG: 16 walkability mismatches out of 1067 checks (1.5% mismatch rate)`

## Predicted Root Cause
Different walkability detection methods:
1. `getCellState()` in MineflayerWorldProvider marks cells as WALKABLE
2. `isWalkable()` used by component analysis has different criteria
3. The 1.5% mismatch rate indicates these methods disagree

## Detection Method
Add logging to compare:
1. What getCellState() returns for failed frontier positions
2. What isWalkable() returns for the same positions
3. Whether mismatched positions correlate with component lookup failures

## Expected Confirmation
If this theory is correct, position (-36,128,378) should be marked WALKABLE by getCellState() but return false/undefined from isWalkable() during component analysis.