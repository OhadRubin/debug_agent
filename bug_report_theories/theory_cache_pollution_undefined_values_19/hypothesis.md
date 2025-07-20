# Theory 19: Cache Pollution by Undefined Values  

## Hypothesis
The `MineflayerWorldProvider` walkability cache is being polluted with `undefined` values, which are then returned on subsequent cache hits, perpetuating the undefined return problem and making it persist across multiple scan cycles.

## Root Cause Analysis
In `minecraft_env.js` lines 122-123:
```javascript
this.walkabilityCache.set(key, isSafe)
return isSafe
```

When `isSafe` is `undefined` due to the boolean expression bug, it gets cached. Later cache hits return this `undefined` value directly, meaning even if the underlying block properties were fixed, the cache would continue returning `undefined`.

## Detection Method
- Track cache hit rates and what values are returned
- Count how many cached values are `undefined` vs proper booleans
- Monitor cache population during first scan vs cache hits in subsequent scans
- Verify that cache clearing resolves some of the undefined returns temporarily

## Expected Evidence
- High cache hit rate returning `undefined` values
- Cache populated with thousands of `undefined` entries after first scan
- New calculations still returning `undefined` even after cache clear
- Cache growing with more `undefined` entries than boolean entries

## Impact Assessment
This creates a persistent problem where:
1. First scan populates cache with `undefined` values
2. Subsequent scans return cached `undefined` values immediately
3. Cache clearing only provides temporary relief until cache is repopulated
4. The bug becomes "sticky" across multiple exploration cycles

From bug report evidence:
- Line 127: "1067/1067 cache hits returned undefined"
- Lines 112-113: Cache cleared and immediately repopulated with undefined values

## Fix Strategy
Multiple approaches:
1. **Fix root cause** (Theory 16): Prevent undefined values from being calculated
2. **Cache validation**: Don't cache undefined values:
   ```javascript
   if (isSafe !== undefined) {
       this.walkabilityCache.set(key, isSafe)
   }
   ```
3. **Cache filtering**: Filter out undefined values when retrieving from cache
4. **Periodic cache invalidation**: Clear cache periodically to prevent pollution buildup

## Related Evidence
From bug report:
- `THEORY_11_DEBUG: CACHE_SUMMARY - 1067/1067 cache hits returned undefined`
- `THEORY_11_DEBUG: FIRST_UNDEFINED_CACHE - -25,127,350`