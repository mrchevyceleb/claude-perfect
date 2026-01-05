# /perfect - Automated Test-Fix-Verify Loop for Claude Code

A powerful slash command for [Claude Code](https://claude.ai/claude-code) that implements an **automated test-fix-verify loop** using Playwright. It executes your task, tests the implementation, scores the result (0-100), and automatically loops to fix issues until achieving a perfect score.

**No more manual testing cycles. No more "it works on my machine." Just perfect code.**

![Version](https://img.shields.io/badge/Version-2.0-blue)
![Score: 100/100](https://img.shields.io/badge/Target_Score-100%2F100-brightgreen)
![Claude Code](https://img.shields.io/badge/Claude_Code-Slash_Command-blue)
![Playwright](https://img.shields.io/badge/Powered_by-Playwright-45ba63)

---

## What's New in v2.0

- **Pre-flight Checks** - Build + TypeScript must pass before testing
- **Explicit Scoring Rubric** - Detailed point values for all categories
- **Score Interpretation Guide** - Know what 70/100 vs 90/100 means
- **Responsive Testing** - Desktop (1280px) AND Mobile (375px) mandatory
- **Detailed Deduction Tables** - Exact points for each issue type

---

## Features

- **Optional Planning Mode** - Design your implementation approach before coding
- **Pre-Flight Checks** - Build and TypeScript validation before testing
- **Automated Loop** - Keeps fixing and re-testing until perfect (or max loops)
- **100-Point Scoring** - Detailed rubric covering functionality, errors, UI, and UX
- **Dual Viewport Testing** - Desktop (1280px) and Mobile (375px)
- **Playwright Testing** - Visual verification with real browser automation
- **Detailed Reports** - Know exactly what passed, failed, and why
- **Auto-Deploy** - Commits and pushes only when score hits 100

---

## Installation

### 1. Copy the command file

```bash
mkdir -p ~/.claude/commands
cp perfect.md ~/.claude/commands/perfect.md
```

### 2. Enable Playwright MCP

Add to your `~/.claude/mcp_settings.json`:

```json
{
  "mcpServers": {
    "playwright": {
      "command": "npx",
      "args": ["@playwright/mcp@latest"]
    }
  }
}
```

### 3. Verify installation

```bash
/mcp
# Ensure playwright is enabled
```

---

## Usage

### Basic Syntax

```
/perfect [your task description] --[max-loops]
```

### Arguments

| Part | Description | Default |
|------|-------------|---------|
| `[task]` | What you want implemented | Required |
| `--N` | Maximum loops (0 = unlimited) | 3 |

### Examples

```bash
# Fix a button with default 3 max loops
/perfect fix the login button - it should be blue and redirect to /dashboard

# Add a feature with unlimited loops until perfect
/perfect add dark mode toggle to the settings page --0

# Quick styling fix with 5 attempts
/perfect center the hero text on mobile --5
```

---

## How It Works

```
+----------------------------------------------------------+
| STEP 0: Planning Mode Question                            |
| "Would you like me to plan the implementation first?"     |
+----------------------------------------------------------+
                           |
+----------------------------------------------------------+
| STEP 1: Parse Arguments                                   |
| Extract task prompt and max loops                         |
+----------------------------------------------------------+
                           |
+----------------------------------------------------------+
| STEP 2: Execute Task                                      |
| Read codebase, implement changes                          |
+----------------------------------------------------------+
                           |
+----------------------------------------------------------+
| STEP 3: Pre-Flight Checks (v2.0 NEW!)                     |
| - npm run build (must pass)                               |
| - npx tsc --noEmit (TypeScript check)                     |
| GATE: Must pass before testing                            |
+----------------------------------------------------------+
                           |
+----------------------------------------------------------+
| STEP 4: Start Dev Server                                  |
| Auto-detect framework, find available port                |
+----------------------------------------------------------+
                           |
+----------------------------------------------------------+
| STEP 5: Test with Playwright                              |
| - Desktop viewport (1280x720)                             |
| - Mobile viewport (375x667)                               |
| - Console error monitoring                                |
| - Network error monitoring                                |
| - Feature-specific tests                                  |
+----------------------------------------------------------+
                           |
+----------------------------------------------------------+
| STEP 6: Calculate Score (0-100)                           |
| Functionality: 40 | Console: 20 | Network: 15             |
| UI: 15 | UX: 10                                           |
+----------------------------------------------------------+
                           |
+----------------------------------------------------------+
| STEP 7: Decision                                          |
| - Score = 100   -> Commit and push                        |
| - Score < 100   -> Fix issues, loop back to Step 3        |
| - Max loops hit -> Report issues, no commit               |
+----------------------------------------------------------+
```

---

## Scoring Rubric (v2.0)

| Category | Points | Criteria |
|----------|--------|----------|
| **Functionality** | 40 | Feature works as specified |
| **Console Errors** | 20 | -5 per error (max 4 errors = 0) |
| **Network Errors** | 15 | -5 per error (max 3 errors = 0) |
| **UI Correctness** | 15 | Visual elements render correctly |
| **UX/Interactions** | 10 | Interactions work smoothly |
| **TOTAL** | **100** | |

### Functionality Score (40 points)

| Points | Criteria |
|--------|----------|
| 40 | Feature works 100% - all requirements met |
| 35 | Works but minor edge case fails |
| 30 | Core works, 1-2 minor issues |
| 25 | Mostly works but has gaps |
| 20 | Basic functionality, incomplete |
| 15 | Partial - some parts work |
| 10 | Minimal implementation |
| 5 | Fundamentally broken |
| 0 | Not implemented |

### UI Deductions (from 15 points)

| Issue | Deduction |
|-------|-----------|
| Missing element | -5 |
| Major layout break | -4 |
| Wrong colors/fonts | -3 |
| Alignment issues | -2 |
| Overflow/clipping | -2 |
| Minor styling issue | -1 |
| Mobile-specific break | -3 |

### UX Deductions (from 10 points)

| Issue | Deduction |
|-------|-----------|
| Broken interaction | -4 |
| Wrong behavior | -3 |
| Missing feedback | -2 |
| Sluggish/janky | -2 |
| Confusing flow | -2 |
| Missing hover/focus | -1 |
| Touch targets <44px | -2 |

---

## Score Interpretation

| Score | Status | Meaning |
|-------|--------|---------|
| **100** | PERFECT | Ready to ship. Auto-commits. |
| **90-99** | EXCELLENT | Very close. 1-2 small fixes. |
| **80-89** | GOOD | Functional but has issues. |
| **70-79** | FAIR | Usable but rough. |
| **60-69** | PARTIAL | Significant gaps. |
| **50-59** | INCOMPLETE | Major issues. |
| **<50** | BROKEN | Needs rework. |

---

## Pre-Flight Checks (v2.0)

Before Playwright testing begins:

### Build Check
```bash
npm run build
```
- Build must pass before testing
- Build errors = score 0

### TypeScript Check
```bash
npx tsc --noEmit
```
- TS errors in changed files must be fixed
- Errors in unchanged files noted but don't block

---

## Responsive Testing

**Both viewports are mandatory:**

| Viewport | Size | Tests |
|----------|------|-------|
| Desktop | 1280x720 | Full feature tests |
| Mobile | 375x667 | Full tests + responsive checks |

Mobile-specific checks:
- No horizontal scroll
- Touch targets >= 44px
- Text readable without zoom
- Layout adapts correctly

---

## Report Format

```
===============================================================================
                         /PERFECT EXECUTION REPORT
===============================================================================

TASK: Add dark mode toggle to settings page

FINAL SCORE: 100/100 - PERFECT
LOOPS EXECUTED: 2 of unlimited
STATUS: PERFECT

-------------------------------------------------------------------------------
                              SCORE BREAKDOWN
-------------------------------------------------------------------------------

Functionality:    40/40   Toggle works, persists, applies theme
Console Errors:   20/20   0 error(s) found
Network Errors:   15/15   0 error(s) found
UI Correctness:   15/15   Desktop: 0 issues, Mobile: 0 issues
UX/Interactions:  10/10   Smooth transitions, instant feedback

-------------------------------------------------------------------------------
                              LOOP HISTORY
-------------------------------------------------------------------------------

Loop 1: 75/100 - Fixed: Console error, missing dark styles
Loop 2: 100/100 - All issues resolved

-------------------------------------------------------------------------------
                               GIT STATUS
-------------------------------------------------------------------------------

Committed and pushed to main.
Commit hash: a1b2c3d
Files changed: 3

===============================================================================
```

---

## Troubleshooting

| Issue | Solution |
|-------|----------|
| "Playwright MCP not available" | Run `/mcp` and enable playwright |
| Build fails | Fix build errors before testing |
| TypeScript errors | Fix TS errors in changed files |
| Dev server won't start | Check package.json scripts |
| Score stuck < 100 | Review report, fix issues, re-run |

---

## /perfect vs /bigtest

| Aspect | /perfect | /bigtest |
|--------|----------|----------|
| **Scope** | Single task | Entire app |
| **Purpose** | Implement + fix loop | Full audit |
| **Loops** | Yes, until 100 | No, one pass |
| **Auto-commit** | Yes at 100 | No |
| **Scoring** | 0-100 | 0-100 |

**Use together:** `/bigtest` to audit, `/perfect` to fix each issue!

---

## License

MIT License - feel free to use, modify, and distribute.

---

**Made with Claude Code by [Matt Johnston](https://mattjohnston.io)**

*Star this repo if you find it useful!*
