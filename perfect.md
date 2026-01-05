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

## CRITICAL: AUTO-LOOP BEHAVIOR

**YOU MUST KEEP LOOPING AUTOMATICALLY** until one of these conditions is met:
- Score reaches **100/100** -> Commit and report success
- Max loops reached (and score < 100) -> Report failure without commit

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

2. **Create Implementation Plan**: Output a structured plan showing:
   - TASK: [the task from arguments]
   - FILES TO MODIFY: [list with what changes]
   - FILES TO CREATE: [if any]
   - IMPLEMENTATION STEPS: [numbered list]
   - EDGE CASES TO HANDLE: [list]
   - TESTING CONSIDERATIONS: [what to verify]

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
- `fix the login button --3` -> prompt="fix the login button", maxLoops=3
- `add dark mode toggle --0` -> prompt="add dark mode toggle", maxLoops=unlimited
- `fix bug` -> prompt="fix bug", maxLoops=3 (default)

**Initialize tracking variables:**
- currentLoop = 1
- loopHistory = []
- finalScore = 0
- activePort = null

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

## STEP 3: Pre-Flight Checks (GATE - Must Pass Before Testing)

**Before starting Playwright tests, verify these prerequisites pass:**

### 3.1 Build Check
```bash
npm run build
```

**If build fails:**
- Document all build errors
- Fix build errors FIRST before proceeding
- Do NOT proceed to Playwright testing with a broken build
- Build errors are blocking - score is automatically 0 until build passes

### 3.2 TypeScript Check (if applicable)
```bash
npx tsc --noEmit
```

**If TypeScript errors exist in changed files:**
- Document TS errors
- Fix TypeScript errors before proceeding
- TS errors in unchanged files can be noted but do not block

**Gate Result:**
- Build passes + No TS errors in changed files -> Proceed to STEP 4
- Build fails OR TS errors in changed files -> Fix and re-check (counts as a loop)

---

## STEP 4: Start Dev Server

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
- Create React App: `set PORT=[PORT] && npm start` (Windows)
- Generic: `npm run dev -- --port [PORT]`

**Start server in background** and store the active port.

**Wait for server to be ready** - check that the server responds before proceeding.

---

## STEP 5: Test with Playwright MCP

**CRITICAL: Playwright MCP must be enabled for this step.**

If Playwright MCP tools are not available:
- Stop immediately
- Inform the user: "Playwright MCP is not enabled. Run `/mcp` and enable playwright to use /perfect"
- Do not continue

---

### MANDATORY TEST CHECKLIST (Run These For EVERY Task)

Before task-specific testing, run these baseline checks:

**Baseline Tests (ALWAYS run):**
1. Page loads without crash
2. No JavaScript errors in console on initial load
3. No 404/500 network errors on initial load
4. Target element/feature is visible on page
5. Page is interactive (not frozen/hung)

**Responsive Tests (ALWAYS run both viewports):**
6. Desktop viewport (1280x720) - screenshot + verify layout
7. Mobile viewport (375x667) - screenshot + verify layout

---

### Testing Protocol:

### 5.1 Navigate
Use Playwright MCP to navigate to `http://localhost:[PORT]`

### 5.2 Desktop Viewport Testing (1280x720)
1. Set viewport to 1280x720
2. Take a screenshot to capture the current state
3. Run baseline tests 1-5
4. Proceed to feature-specific testing

### 5.3 Feature Testing
Based on the task prompt, test the specific functionality:
- Navigate to the relevant page/section
- Interact with the implemented feature
- Verify the expected behavior occurs

**Test types by feature category:**

| Feature Type | Tests to Run |
|--------------|--------------|
| **Form** | Submit with valid data, submit with invalid data, check validation messages, verify success/error states |
| **Button** | Click fires correct action, loading state appears, success state appears |
| **Navigation** | Route changes correctly, correct page loads, back button works |
| **Data Display** | Correct data renders, loading state works, empty state works, error state works |
| **Modal/Dialog** | Opens correctly, closes correctly, escape key works, click outside closes |
| **List/Table** | Items render, pagination works, sorting works (if applicable), filtering works (if applicable) |

### 5.4 Console Error Check
Check the browser console for JavaScript errors:
- Count all errors
- Document each error message and source

**What counts as an error:**
- JavaScript runtime errors (TypeError, ReferenceError, SyntaxError, etc.)
- Unhandled promise rejections
- React/framework errors
- Failed assertions

**What does NOT count:**
- Console.log/info/debug statements
- Deprecation warnings (unless causing issues)
- Third-party library warnings outside your control

### 5.5 Network Error Check
Monitor network requests for failures:
- Look for 4xx and 5xx responses
- Document failed requests (URL, status code)

**What counts as an error:**
- 4xx errors (400, 401, 403, 404, etc.)
- 5xx errors (500, 502, 503, etc.)
- Failed/aborted requests
- CORS errors
- Timeout errors

**What does NOT count:**
- Intentional 404s (checking if resource exists)
- Expected auth redirects (401 -> login flow)
- Cancelled requests due to navigation

### 5.6 Mobile Viewport Testing (375x667)
1. Set viewport to 375x667
2. Take a screenshot
3. Verify layout adapts correctly:
   - No horizontal scroll
   - Text is readable (not too small)
   - Buttons are tappable (min 44x44px touch targets)
   - No overlapping elements
4. Re-run feature tests if interaction differs on mobile

### 5.7 Visual Verification
Compare screenshots against expected behavior:
- Check that elements render correctly
- Verify layout and styling
- Check for visual regressions
- Verify responsive behavior between viewports

### 5.8 Interaction Testing
Test any interactive elements:
- Buttons click correctly
- Forms submit properly
- Navigation works
- State updates as expected
- Hover states work (desktop)
- Touch targets adequate (mobile)

**Document ALL findings:**
- buildPassed: true/false
- tsErrors: [list of TypeScript errors in changed files]
- consoleErrors: [list of errors]
- networkErrors: [list of failed requests]
- uiIssues: [list of visual problems - desktop]
- uiIssuesMobile: [list of mobile-specific issues]
- functionalityIssues: [what does not work]
- uxIssues: [interaction problems]

---

## STEP 6: Calculate Score

**Scoring Rubric (100 points total):**

| Category | Max Points | Scoring Criteria |
|----------|------------|------------------|
| Functionality | 40 | Feature works exactly as specified end-to-end |
| Console Errors | 20 | Start at 20, subtract 5 per error (min 0) |
| Network Errors | 15 | Start at 15, subtract 5 per error (min 0) |
| UI Correctness | 15 | All visual elements render correctly (desktop + mobile) |
| UX/Interactions | 10 | All interactions work smoothly |

---

### 6.1 Functionality Score (40 points)

Award points based on implementation completeness:

| Points | Criteria |
|--------|----------|
| **40** | Feature works 100% as specified - all requirements met, all edge cases handled |
| **35** | Feature works but minor edge case fails (e.g., empty state not handled) |
| **30** | Core functionality works, 1-2 minor issues that do not block usage |
| **25** | Feature mostly works but has noticeable gaps in implementation |
| **20** | Basic functionality present but incomplete - missing key behaviors |
| **15** | Partial implementation - some parts work, others do not |
| **10** | Minimal implementation visible - mostly broken |
| **5** | Attempted but fundamentally broken |
| **0** | Not implemented or completely non-functional |

**Scoring approach:** Start at 40, deduct based on severity:
- Critical bug (blocks primary use case): -15 to -20
- Major bug (significant functionality broken): -10
- Minor bug (edge case or non-blocking): -5
- Cosmetic/polish issue: -2

---

### 6.2 Console Errors Score (20 points)

```
consoleScore = max(0, 20 - (consoleErrorCount * 5))
```

| Errors | Points | Notes |
|--------|--------|-------|
| 0 | 20 | Perfect - no JS errors |
| 1 | 15 | One error |
| 2 | 10 | Two errors |
| 3 | 5 | Three errors |
| 4+ | 0 | Four or more errors |

---

### 6.3 Network Errors Score (15 points)

```
networkScore = max(0, 15 - (networkErrorCount * 5))
```

| Errors | Points | Notes |
|--------|--------|-------|
| 0 | 15 | Perfect - all requests successful |
| 1 | 10 | One failed request |
| 2 | 5 | Two failed requests |
| 3+ | 0 | Three or more failed requests |

---

### 6.4 UI Correctness Score (15 points)

Start at 15, deduct for each visual issue:

| Issue Type | Deduction |
|------------|-----------|
| Missing element (button, text, image not rendered) | -5 |
| Major layout break (overlapping, off-screen, collapsed) | -4 |
| Wrong colors/fonts (does not match design/spec) | -3 |
| Alignment issues (misaligned, uneven spacing) | -2 |
| Overflow/clipping (text cut off, scrollbars appearing incorrectly) | -2 |
| Minor styling inconsistency (slight spacing, border differences) | -1 |
| Z-index issues (wrong stacking order) | -2 |
| **Mobile-specific layout break** | -3 per issue |

**Note:** Issues must be tested at BOTH desktop (1280px) and mobile (375px). Mobile-specific breaks count separately.

---

### 6.5 UX/Interactions Score (10 points)

Start at 10, deduct for each interaction issue:

| Issue Type | Deduction |
|------------|-----------|
| Broken interaction (click does nothing, form does not submit) | -4 |
| Wrong behavior (action does something unexpected) | -3 |
| Missing feedback (no loading state, no success/error message) | -2 |
| Sluggish/janky interaction (noticeable lag, stuttering) | -2 |
| Confusing flow (unclear what to do next) | -2 |
| Missing hover/focus states | -1 |
| Poor keyboard navigation | -1 |
| Touch targets too small on mobile (<44px) | -2 |

---

### 6.6 Calculate Total Score

```
totalScore = functionalityScore + consoleScore + networkScore + uiScore + uxScore
```

**Score must be an integer from 0-100.**

---

### 6.7 Score Interpretation Guide

| Score Range | Status | Meaning |
|-------------|--------|---------|
| **100** | PERFECT | Ready to ship. Auto-commits and pushes. |
| **90-99** | EXCELLENT | Very close. Minor polish needed. Usually 1-2 small fixes away. |
| **80-89** | GOOD | Functional but has issues. Core works, needs attention on details. |
| **70-79** | FAIR | Usable but rough. Multiple issues need addressing. |
| **60-69** | PARTIAL | Significant gaps. May need architectural changes. |
| **50-59** | INCOMPLETE | Major functionality missing or broken. |
| **Below 50** | BROKEN | Fundamental issues. Likely needs rework. |

**Calculate and store the score.**

---

## STEP 7: Loop Decision (AUTOMATIC - NO USER INPUT NEEDED)

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
The implementation is PERFECT. Proceed to **STEP 8** (Report + Commit).

### ELSE IF maxLoops = 0 (unlimited) OR currentLoop < maxLoops:
**AUTOMATICALLY continue fixing without asking the user!**

1. Document all issues found in this loop
2. **IMMEDIATELY fix the identified issues:**
   - Address console errors (fix the JavaScript)
   - Fix network errors (correct API calls, URLs)
   - Fix UI issues (correct styles, layout)
   - Fix mobile-specific issues (responsive CSS)
   - Fix functionality issues (complete the implementation)
   - Fix UX issues (improve interactions)
3. Increment currentLoop
4. **AUTOMATICALLY return to STEP 3** (pre-flight checks, then re-test)

**DO NOT output a report or ask the user what to do. Just fix and re-test.**

### ELSE (maxLoops reached AND score < 100):
Maximum loops reached without achieving perfect score.
Proceed to **STEP 8** (Report only, NO commit).

---

## STEP 8: Generate Report

Output this report:

```
===============================================================================
                         /PERFECT EXECUTION REPORT
===============================================================================

TASK: [original prompt from arguments]

FINAL SCORE: [totalScore]/100 [status based on score interpretation]
LOOPS EXECUTED: [currentLoop] of [maxLoops or "unlimited"]
STATUS: [PERFECT if score=100, else INCOMPLETE]

-------------------------------------------------------------------------------
                              SCORE BREAKDOWN
-------------------------------------------------------------------------------

Functionality:    [X]/40   [brief details]
Console Errors:   [X]/20   [N] error(s) found
Network Errors:   [X]/15   [N] error(s) found
UI Correctness:   [X]/15   [brief details] (Desktop: X issues, Mobile: X issues)
UX/Interactions:  [X]/10   [brief details]

-------------------------------------------------------------------------------
                              LOOP HISTORY
-------------------------------------------------------------------------------

Loop 1: [score]/100 - [summary of issues found and fixes attempted]
Loop 2: [score]/100 - [summary of issues found and fixes attempted]
[... for each loop ...]

-------------------------------------------------------------------------------

[IF STATUS = PERFECT]
                               GIT STATUS
-------------------------------------------------------------------------------

Committed and pushed to main.
Commit hash: [hash]
Files changed:
  - [file1]
  - [file2]
  - [...]

[IF STATUS = INCOMPLETE]
                            REMAINING ISSUES
-------------------------------------------------------------------------------

The following issues could not be resolved within [maxLoops] loops:

Console Errors:
  - [error 1]
  - [error 2]

Network Errors:
  - [error 1]

UI Issues (Desktop):
  - [issue 1]

UI Issues (Mobile):
  - [issue 1]

Functionality Issues:
  - [issue 1]

UX Issues:
  - [issue 1]

NOT COMMITTED - Fix remaining issues and run /perfect again.

===============================================================================
```

---

## STEP 9: Commit (Only if score = 100)

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
2. **Build must pass** - Do not proceed to Playwright testing if build fails
3. **Playwright MCP required** - Stop and inform user if not available
4. **Test both viewports** - ALWAYS test desktop (1280px) AND mobile (375px)
5. **Thorough testing** - Test ALL aspects of the implementation
6. **Meaningful progress** - Each loop should fix at least one issue
7. **Accurate scoring** - Be honest about issues, do not inflate scores
8. **Clean commits** - Only commit perfect (100/100) implementations
9. **Complete reports** - Always generate the full report at the end
