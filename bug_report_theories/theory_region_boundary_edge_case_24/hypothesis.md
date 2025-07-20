# Theory 24: Region Boundary Edge Case

## Hypothesis
The frontier position (-36,128,378) falls on a region boundary where component assignment logic fails due to incorrect region coordinate calculations or boundary edge cases.

## Evidence from Bug Report
- Specific position (-36,128,378) consistently fails
- `getCompId3D` function fails with "No Z entry" error
- Component maze structure uses region-based indexing

## Predicted Root Cause
Position (-36,128,378) maps to region coordinates that:
1. Fall on exact boundary between regions
2. Have rounding/floor calculation errors
3. Get processed by one region but looked up as another region
4. Create sparse maze structure gaps

## Detection Method
Add logging to track:
1. What region coordinates (-36,128,378) maps to
2. Whether position is on region boundary (x/y/z mod 16 == 0)
3. Which regions process this position vs which region it's looked up in
4. Component maze structure around this position

## Expected Confirmation
If this theory is correct, position (-36,128,378) should either be on a region boundary or have region coordinate mapping discrepancies between storage and lookup.