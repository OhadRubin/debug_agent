# Theory 14: Async Component Update Race Condition

## Hypothesis
Multiple rapid sensor scans and component updates are happening asynchronously, causing race conditions that corrupt the componentLookupTable or prevent proper component creation.

## Supporting Evidence from Bug Report
- Components created successfully on first scan (1 component found)
- Subsequent scans find 0 components despite walkable cells
- componentLookupTable entries being cleared/corrupted between scans
- Rapid SCANNING→THINKING→SCANNING loop might cause overlapping updates

## Key Observations
1. Component updates use deep copy of componentLookupTable
2. Multiple async updateComponents() calls could interfere
3. clearRegionComponents() might be clearing data that another update is using
4. Component graph and maze updates might not be atomic
5. Fast scan-move-scan cycle could cause timing issues

## Detection Strategy
Add detailed logging to track:
1. Timing of updateComponents() calls (start/end timestamps)
2. Overlapping component updates (multiple concurrent calls)
3. componentLookupTable state before/after each update
4. Component graph node counts during rapid updates

## Test Cases
1. Force sequential component updates and compare results
2. Add locks/barriers to prevent concurrent component updates
3. Track component lookup table integrity during rapid scans
4. Test if slow down between scans prevents the issue

## Expected Fix
If confirmed, fix could involve:
- Adding mutex/locks around component update operations
- Making component updates fully synchronous
- Queuing component updates instead of allowing concurrency
- Ensuring atomic updates of both graph and lookup table