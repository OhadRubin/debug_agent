# Theory 22: Component Update Race Condition

## Hypothesis
Frontier detection runs on terrain knowledge before component analysis completes, creating a race condition where frontiers are found based on terrain state but component assignments haven't been created yet.

## Evidence from Bug Report
- Terrain state shows WALKABLE (0) but maze value is undefined
- `THEORY_4_DEBUG: Primary lookup failures: 1` suggests timing mismatch
- Component update happens after sensor reading update

## Predicted Root Cause
The sequence in minecraft_env.js scan() method:
1. Update terrain knowledge from sensor readings
2. Update component analysis from new cells
But frontier detection happens after step 1 and before step 2 completes.

## Detection Method
Add timestamps to track:
1. When terrain knowledge gets updated
2. When component analysis starts/completes  
3. When frontier detection runs
4. Whether component lookup happens during component analysis

## Expected Confirmation
If this theory is correct, we should see frontier detection accessing component maze while component analysis is still updating it, or immediately after terrain update but before component analysis.