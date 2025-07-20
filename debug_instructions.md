Do NOT create a subagent unless I tell you to. Create a todo list of the steps below.

Step 1:
Read src/lib/exploration/ARCHITECTURE.md

Step 2:
Tell me what possible bugs may exist

Step 3:
Read /Users/ohadr/Auto-Craft-Bot/bug_report.md

Step 4:
Compare the bug report and your guess above

Step 5:
5.1 - Read bot-state.js, bot-actions.js and simulation_env.js (**CRITICAL: for the duration of this session you should NOT modify the business logic unless explicitly told to do so - this is READ-ONLY analysis**)
5.1.5 - Think about it. And note how previous code prints summaries of logs of previous theories, this will be important later.
5.2 - Read minecraft_env.js


**Important: Read the entire content of the files, not just range of lines.**

Step 5.5:
Read this file /Users/ohadr/Auto-Craft-Bot/debug_instructions.md **again** to remind yourself of the instructions.

Step 6:
First, check the existing theory folders in /Users/ohadr/Auto-Craft-Bot/bug_report_theories/ to identify the highest theory number.
Then read ./debugging_methodology.md and brainstorm 5 theories, start enumerating them from the next available theory number. Write each theory in a new file in /Users/ohadr/Auto-Craft-Bot/bug_report_theories/theory_{theory_name}_{theory_num:02d}/hypothesis.md
For example, if the highest theory number is 5, the next theory number should be 6.

Step 7:
Deploy 5 agents in parallel. Each agent will do steps 1, 3, 5 and 6 and each will investigate the bug following the theory you suggested.
You should create a new file that would contain their shared numbered instructions (same as I did here), and refer them to their assigned theory file.
(it's possible that such file already exists in /Users/ohadr/Auto-Craft-Bot/bug_report_theories/, in that case, modify it to focus on the current bug)
They should suggest debug prints that would confirm/disprove the theory.
They should write their report in /Users/ohadr/Auto-Craft-Bot/bug_report_theories/theory_{theory_name}_{theory_num:02d}/report.md
They should not run files or modify the code.
Make sure you name them according to their assigned theory numbers.
For example, if the theories are 6-10, name agents 6-10.

Step 8:
Read the reports the subagents wrote, notice that /Users/ohadr/Auto-Craft-Bot/bug_report_theories/ already contains other reports from a previous session, they are not relevant to the current bug.

Step 9:
Implement the debug logs suggested by the subagents. **IN A NON-SPAMMY WAY**. Remember 5.1.5, this is important!
Make sure that the log messages are not spammy using `timeout 5 node src/lib/exploration/minecraft_env.js`
If they are spammy, modify them to print a summary of the results instead.
For example: if the agent suggests to print something that checks for some condition A but it runs 20 times a second, one can modify it to count how many times condition A was true and print a summary of the results every 10 seconds.

Step 10:
Let me know when you are done.
