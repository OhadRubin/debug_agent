# Bug Investigation Agent Instructions

Each agent will investigate a specific theory about the "No valid targets found" infinite loop bug in the exploration system that occurs when the bot gets stuck after reaching its second target.

## Shared Steps for All Agents:

### Step 1: Read Your Assigned Theory
Read your assigned theory file in /Users/ohadr/Auto-Craft-Bot/bug_report_theories/theory_{theory_name}_{theory_num:02d}/hypothesis.md

### Step 2: Review Core Context 
Read the following files for context:
- /Users/ohadr/Auto-Craft-Bot/bug_report.md (the actual bug report)
- /Users/ohadr/Auto-Craft-Bot/src/lib/exploration/ARCHITECTURE.md (system overview) 
- /Users/ohadr/Auto-Craft-Bot/debugging_methodology.md (debugging approach)

### Step 3: Analyze Code Implementation
Read and analyze the relevant code sections for your theory:
- /Users/ohadr/Auto-Craft-Bot/src/lib/exploration/bot-state.js (focus on component management)
- /Users/ohadr/Auto-Craft-Bot/src/lib/exploration/bot-actions.js (focus on frontier detection)
- /Users/ohadr/Auto-Craft-Bot/src/lib/exploration/minecraft_env.js (focus on live integration)

**CRITICAL: This is READ-ONLY analysis. Do not modify any business logic code.**

### Step 4: Develop Detection Strategy
Based on the debugging methodology, design specific debug prints that would confirm or disprove your theory. Your debug strategy should:

1. **Follow Detection-First Approach**: Design logging to detect the issue before implementing fixes
2. **Be Non-Spammy**: If your detection would run frequently, design summarized logging or counters
3. **Provide Clear Evidence**: Design logs that will definitively prove or disprove your theory
4. **Be Specific to Your Theory**: Focus on the exact failure mode your theory predicts

### Step 5: Write Investigation Report
Write your detailed report in /Users/ohadr/Auto-Craft-Bot/bug_report_theories/theory_{theory_name}_{theory_num:02d}/report.md

Your report must include:

#### Theory Analysis Section:
- **Theory Summary**: Brief restatement of your assigned theory
- **Code Analysis**: What you found in the code that supports/contradicts the theory
- **Likelihood Assessment**: How likely this theory is to be the root cause (High/Medium/Low)

#### Detection Strategy Section:
- **Detection Method**: Specific debug logging approach you recommend
- **Key Locations**: Exact files and functions where debug code should be added
- **Expected Evidence**: What the logs will show if your theory is correct
- **Expected Evidence**: What the logs will show if your theory is wrong

#### Debug Implementation Section:
- **Specific Debug Code**: Actual logging statements to add (follow THEORY_X_DEBUG format)
- **Performance Considerations**: How to avoid spam while getting useful data
- **Success Criteria**: How to recognize when your theory is confirmed/disproven

#### Recommended Testing Steps:
- **Detection Phase**: Step-by-step process to run the system with your debug code
- **Verification Method**: How to interpret the debug output
- **Next Steps**: What to investigate further if your theory is confirmed

## Current Bug Context:
**Error**: Bot gets stuck in infinite "No valid targets found" loop after successfully reaching two initial targets.
**Key Debug Clues**: 
- Bot successfully reaches (-1,128,364) and (-48,128,400)
- After second target, infinite loop: SCAN_DEBUG → THINKING_DEBUG → "No valid targets found" → repeat
- Debug output shows: `getCellState=WALKABLE, isWalkable=false, block=air` inconsistency
- Frontier detection appears to be failing to produce valid targets

## Agent Assignments:
- Agent 26: theory_walkability_method_inconsistency_26 
- Agent 27: theory_pathfinder_block_property_validation_27
- Agent 28: theory_frontier_component_assignment_failure_28
- Agent 29: theory_ground_detection_logic_discrepancy_29 
- Agent 30: theory_infinite_loop_no_targets_30

## Remember:
- Focus only on your assigned theory
- Do not run files or modify code - this is analysis only
- Design detection-first debugging following the methodology
- Provide specific, actionable debug logging recommendations