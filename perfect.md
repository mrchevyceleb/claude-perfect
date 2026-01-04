---
description: Automated test-fix-verify loop with Playwright scoring (score 100 to pass)
argument-hint: [task prompt] --[max-loops]
allowed-tools: Read, Write, Edit, Bash(*), Grep, Glob
---

# /perfect - Automated Test-Fix-Verify Loop

You are executing the `/perfect` command. This command implements an automated loop that:
1. **Optionally enters planning mode** (user's choice)
2. Executes a task/prompt
3. Tests with Playwright MCP
4. Scores the result (0-100)
5. **AUTOMATICALLY LOOPS** to fix issues until score=100 or max loops reached

## ⚠️ CRITICAL: AUTO-LOOP BEHAVIOR

**YOU MUST KEEP LOOPING AUTOMATICALLY** until one of these conditions is met:
- ✅ Score reaches **100/100** → Commit and report success
- ❌ Max loops reached (and score < 100) → Report failure without commit

**DO NOT STOP AND ASK THE USER** after each loop. The entire point of `/perfect` is autonomous iteration.
If score < 100 and loops remain, **immediately fix issues and re-test**. No user confirmation needed.

---

## STEP 0: Planning Mode Question (ALWAYS ASK FIRST)

**Before doing anything else**, ask the user:

Use the `AskUserQuestion` tool with this question:
- **Question**: "Would you like me to plan the implementation first before starting?"
- **Option 1**: "Yes - Plan first" (description: "Analyze codebase and create implementation plan, then auto-proceed to test-fix loop")
- **Option 2**: "No - Start immediately" (description: "Skip planning and proceed directly to implementation and testing")

### IF USER SELECTS "YES - PLAN FIRST":

**IMPORTANT: Do NOT use `EnterPlanMode` tool - it requires user approval which breaks the auto-flow.**

Instead, perform **inline planning**:

1. **Analyze the Codebase**: Use Glob, Grep, and Read tools to:
   - Find all relevant files related to the task
   - Understand existing patterns and architecture
   - Identify dependencies and imports

2. **Create Implementation Plan**: Output a structured plan:
   ```
   ═══════════════════════════════════════════════════════════════════
                        IMPLEMENTATION PLAN
   ═══════════════════════════════════════════════════════════════════

   TASK: [the task from arguments]

   FILES TO MODIFY:
   - [file1] - [what changes]
   - [file2] - [what changes]

   FILES TO CREATE (if any):
   - [file] - [purpose]

   IMPLEMENTATION STEPS:
   1. [Step 1]
   2. [Step 2]
   3. [Step 3]

   EDGE CASES TO HANDLE:
   - [edge case 1]
   - [edge case 2]

   TESTING CONSIDERATIONS:
   - [what to verify]

   ═══════════════════════════════════════════════════════════════════
   ```

3. **Auto-Proceed**: Immediately continue to STEP 1 (no user approval needed)

### IF USER SELECTS "NO - START IMMEDIATELY":

- Skip directly to **STEP 1** below

---

## STEP 1: Parse Arguments

Parse the arguments to extract the task prompt and max loops.

**Arguments received:** $ARGUMENTS

**Parsing rules:**
- Look for the pattern `--N` at the END of the arguments (where N is a number)
- Everything BEFORE the last `--N` is the **prompt/task**
- The number N is **maxLoops** (0 = unlimited loops until score=100)
- If no `--N` found, default maxLoops to 3

**Examples:**
- `fix the login button --3` → prompt="fix the login button", maxLoops=3
- `add dark mode toggle --0` → prompt="add dark mode toggle", maxLoops=unlimited
- `fix bug` → prompt="fix bug", maxLoops=3 (default)

**Initialize tracking variables:**
```
currentLoop = 1
loopHistory = []
finalScore = 0
activePort = null
```

---

## STEP 2: Execute the Task

Now implement the requested changes based on the extracted prompt.

**Requirements:**
1. Read all relevant files FIRST to understand the codebase
2. Make the necessary code changes
3. Be thorough and complete - implement the full feature
4. Follow existing code patterns and style

**DO NOT proceed to testing until the implementation is complete.**

---

## STEP 3: Start Dev Server

Check and start the development server on port 3008 (or next available).

**Port detection (Windows):**
```bash
netstat -ano | findstr :3008
```

**Logic:**
1. Start with port 3008
2. If port is in use, try 3009, 3010, etc.
3. Once you find an available port, start the server

**Framework detection (check package.json):**
- Next.js: `npm run dev -- -p [PORT]`
- Vite: `npm run dev -- --port [PORT]`
- Create React App: `PORT=[PORT] npm start` or `set PORT=[PORT] && npm start` (Windows)
- Generic: `npm run dev -- --port [PORT]`

**Start server in background** and store the active port.

**Wait for server to be ready** - check that the server responds before proceeding.

---

## STEP 4: Test with Playwright MCP

**CRITICAL: Playwright MCP must be enabled for this step.**

If Playwright MCP tools are not available:
- Stop immediately
- Inform the user: "Playwright MCP is not enabled. Run `/mcp` and enable playwright to use /perfect"
- Do not continue

**Testing Protocol:**

### 4.1 Navigate
Use Playwright MCP to navigate to `http://localhost:[PORT]`

### 4.2 Initial Screenshot
Take a screenshot to capture the current state.

### 4.3 Feature Testing
Based on the task prompt, test the specific functionality:
- Navigate to the relevant page/section
- Interact with the implemented feature
- Verify the expected behavior occurs

### 4.4 Console Error Check
Check the browser console for JavaScript errors:
- Count all errors
- Document each error message and source

### 4.5 Network Error Check
Monitor network requests for failures:
- Look for 4xx and 5xx responses
- Document failed requests (URL, status code)

### 4.6 Visual Verification
Take screenshots of the affected UI areas:
- Check that elements render correctly
- Verify layout and styling
- Check for visual regressions

### 4.7 Interaction Testing
Test any interactive elements:
- Buttons click correctly
- Forms submit properly
- Navigation works
- State updates as expected

**Document ALL findings:**
```
consoleErrors: [list of errors]
networkErrors: [list of failed requests]
uiIssues: [list of visual problems]
functionalityIssues: [what doesn't work]
uxIssues: [interaction problems]
```

---

## STEP 5: Calculate Score

**Scoring Rubric (100 points total):**

| Category | Max Points | Scoring Criteria |
|----------|------------|------------------|
| Functionality | 40 | Feature works exactly as specified end-to-end |
| Console Errors | 20 | Start at 20, subtract 5 per error (min 0) |
| Network Errors | 15 | Start at 15, subtract 5 per error (min 0) |
| UI Correctness | 15 | All visual elements render correctly |
| UX/Interactions | 10 | All interactions work smoothly |

**Scoring Logic:**

```
functionalityScore = 40 if feature works completely, 0-39 based on partial functionality
consoleScore = max(0, 20 - (consoleErrorCount * 5))
networkScore = max(0, 15 - (networkErrorCount * 5))
uiScore = 15 if UI is perfect, deduct for each visual issue
uxScore = 10 if interactions are smooth, deduct for each UX issue

totalScore = functionalityScore + consoleScore + networkScore + uiScore + uxScore
```

**Calculate and store the score.**

---

## STEP 6: Loop Decision (AUTOMATIC - NO USER INPUT NEEDED)

**Record this loop:**
```
loopHistory.push({
  loop: currentLoop,
  score: totalScore,
  issues: [list of all issues found],
  fixes: [list of fixes attempted this loop]
})
```

**Decision Tree (EXECUTE IMMEDIATELY - DO NOT ASK USER):**

### IF score = 100:
✅ The implementation is PERFECT. Proceed to **STEP 7** (Report + Commit).

### ELSE IF maxLoops = 0 (unlimited) OR currentLoop < maxLoops:
⚠️ **AUTOMATICALLY continue fixing without asking the user!**

1. Document all issues found in this loop
2. **IMMEDIATELY fix the identified issues:**
   - Address console errors (fix the JavaScript)
   - Fix network errors (correct API calls, URLs)
   - Fix UI issues (correct styles, layout)
   - Fix functionality issues (complete the implementation)
   - Fix UX issues (improve interactions)
3. Increment currentLoop
4. **AUTOMATICALLY return to STEP 4** (re-test with Playwright)

**DO NOT output a report or ask the user what to do. Just fix and re-test.**

### ELSE (maxLoops reached AND score < 100):
❌ Maximum loops reached without achieving perfect score.
Proceed to **STEP 7** (Report only, NO commit).

---

## STEP 7: Generate Report

Output this report:

```
═══════════════════════════════════════════════════════════════════════════════
                         /PERFECT EXECUTION REPORT
═══════════════════════════════════════════════════════════════════════════════

TASK: [original prompt from arguments]

FINAL SCORE: [totalScore]/100
LOOPS EXECUTED: [currentLoop] of [maxLoops or "unlimited"]
STATUS: [PERFECT if score=100, else INCOMPLETE]

───────────────────────────────────────────────────────────────────────────────
                              SCORE BREAKDOWN
───────────────────────────────────────────────────────────────────────────────

Functionality:    [X]/40   [brief details]
Console Errors:   [X]/20   [N] error(s) found
Network Errors:   [X]/15   [N] error(s) found
UI Correctness:   [X]/15   [brief details]
UX/Interactions:  [X]/10   [brief details]

───────────────────────────────────────────────────────────────────────────────
                              LOOP HISTORY
───────────────────────────────────────────────────────────────────────────────

Loop 1: [score]/100 - [summary of issues found and fixes attempted]
Loop 2: [score]/100 - [summary of issues found and fixes attempted]
[... for each loop ...]

───────────────────────────────────────────────────────────────────────────────

[IF STATUS = PERFECT]
                               GIT STATUS
───────────────────────────────────────────────────────────────────────────────

Committed and pushed to main.
Commit hash: [hash]
Files changed:
  - [file1]
  - [file2]
  - [...]

[IF STATUS = INCOMPLETE]
                            REMAINING ISSUES
───────────────────────────────────────────────────────────────────────────────

The following issues could not be resolved within [maxLoops] loops:

Console Errors:
  - [error 1]
  - [error 2]

Network Errors:
  - [error 1]

UI Issues:
  - [issue 1]

Functionality Issues:
  - [issue 1]

UX Issues:
  - [issue 1]

NOT COMMITTED - Fix remaining issues and run /perfect again.

═══════════════════════════════════════════════════════════════════════════════
```

---

## STEP 8: Commit (Only if score = 100)

**IF finalScore = 100:**

1. Stage all changed files:
   ```bash
   git add -A
   ```

2. Create commit with descriptive message:
   ```bash
   git commit -m "feat: [brief description based on task prompt]

   /perfect score: 100/100
   Loops: [currentLoop]

   Generated with Claude Code"
   ```

3. Push to main:
   ```bash
   git push origin main
   ```

4. Report the commit hash in the final report.

**IF finalScore < 100:**

Do NOT commit. The report should clearly state "NOT COMMITTED" and list remaining issues.

---

## IMPORTANT RULES

1. **Never guess** - If the task is unclear, implement what was literally requested
2. **Playwright MCP required** - Stop and inform user if not available
3. **Thorough testing** - Test ALL aspects of the implementation
4. **Meaningful progress** - Each loop should fix at least one issue
5. **Accurate scoring** - Be honest about issues, don't inflate scores
6. **Clean commits** - Only commit perfect (100/100) implementations
7. **Complete reports** - Always generate the full report at the end
