# Systematic Debugging Methodology: Theory-Driven Investigation

## Core Philosophy

When debugging complex systems with multiple potential failure points, avoid the temptation to implement fixes immediately. Instead, follow a disciplined three-step cycle for each theory:

1. **Detect**: Add logging/assertions to confirm the theory is correct
2. **Fix**: Implement the solution only after confirming the issue exists  
3. **Verify**: Test again to confirm the fix resolved the issue

## The Theory-Detect-Fix-Verify Cycle

### Step 1: Theory Formation
- Start with a ranked list of potential issues
- Form specific, testable hypotheses
- Work through theories in order of likelihood or impact

### Step 2: Detection Phase
**Before implementing any fix:**
- Add targeted logging to detect if the suspected issue actually occurs
- Use assertions to crash on critical failures
- Make detection logs specific and clear about what they're testing

```
# Example detection patterns:o
print(f"THEORY_X_DEBUG: Checking if condition Y occurs...")
if suspected_bad_condition:
    print(f"THEORY_X_DEBUG: CONFIRMED - Issue Y detected!")
    assert False  # Optional: crash to stop immediately
else:
    print(f"THEORY_X_DEBUG: Issue Y not detected")
```

### Step 3: Fix Implementation
**Only after confirming the issue exists:**
- Implement the minimal fix for the confirmed problem
- Keep detection logging in place initially

### Step 4: Verification Phase
- Test the system with both detection and fix in place
- Confirm the detection logs no longer trigger
- Remove detection logging once fix is verified
- Document the result: ✅ Fixed or ❌ Not the issue

## Key Principles

### 1. One Theory at a Time
- Never test multiple theories simultaneously
- Complete the full cycle for each theory before moving to the next
- Avoid compound changes that make it unclear which fix worked

### 2. Evidence-Based Decision Making
- Don't implement fixes based on assumptions
- Require concrete evidence (logs, crashes, behavior changes) before proceeding
- Track which theories were eliminated to avoid re-testing

### 3. Incremental Understanding
- Build knowledge systematically about what IS and IS NOT causing the problem
- Use detection logs to understand the actual system behavior vs expected behavior
- Document findings to create a clear investigation trail

## Detection Strategies

### Assertion-Based Detection
```
# Binary pass/fail for critical conditions
if critical_condition_should_never_happen:
    assert False, "Theory X confirmed: critical condition occurred"
```

### Comparative Detection  
```
# Compare expected vs actual values
expected = calculate_expected_value()
actual = get_actual_value()
if expected != actual:
    print(f"THEORY_DEBUG: Expected {expected}, got {actual}")
```

### Flow Tracking Detection
```
# Track execution paths through complex logic
print(f"FLOW_DEBUG: Entering component X with input Y")
result = process_component_x(input_y)
print(f"FLOW_DEBUG: Component X returned {result}")
```

### State Inspection Detection
```
# Capture system state at critical decision points
print(f"STATE_DEBUG: Component state - {relevant_state_subset}")
print(f"STATE_DEBUG: Available options - {available_options}")
```

## Benefits of This Methodology

### Eliminates Wasted Effort
- Avoids implementing fixes for non-existent problems
- Prevents multiple simultaneous changes that obscure results
- Reduces debugging time by focusing on actual issues

### Builds Systematic Understanding
- Creates a clear record of what was tested and eliminated
- Provides concrete evidence for root cause analysis
- Enables knowledge transfer to team members

### Improves Fix Quality
- Ensures fixes target the actual problem, not symptoms
- Provides immediate verification that fixes work
- Reduces regression risk by understanding the real issue

## Common Anti-Patterns to Avoid

### Shotgun Debugging
- Implementing multiple potential fixes simultaneously
- Making changes based on guesswork rather than evidence
- Skipping the detection phase and going straight to fixes

### Assumption-Based Fixes
- Believing you know the issue without confirming it exists
- Implementing complex solutions for simple problems
- Ignoring evidence that contradicts initial theories

### Investigation Pollution
- Leaving debug code in place between different theories
- Testing new theories with previous debug code still active
- Mixing detection logic from multiple theories

## Documentation Template

For each theory investigated:

```
### Theory N: [Brief Description]
**Hypothesis**: [What you think is wrong]
**Detection Method**: [How you'll confirm it]
**Test Results**: [What the detection logs showed]
**Status**: ❌ Not the issue / ✅ Fixed / 🔄 Partially confirmed
**Notes**: [Key findings or insights]
```

## Conclusion

This methodology transforms debugging from a chaotic trial-and-error process into a systematic investigation. By requiring evidence before implementing fixes, you ensure that effort is focused on real problems rather than imagined ones. The detection-first approach also builds deep understanding of system behavior, leading to more robust and targeted solutions.