# 🎯 /perfect - Automated Test-Fix-Verify Loop for Claude Code

A powerful slash command for [Claude Code](https://claude.ai/claude-code) that implements an **automated test-fix-verify loop** using Playwright. It executes your task, tests the implementation, scores the result (0-100), and automatically loops to fix issues until achieving a perfect score.

**No more manual testing cycles. No more "it works on my machine." Just perfect code.**

![Score: 100/100](https://img.shields.io/badge/Target_Score-100%2F100-brightgreen)
![Claude Code](https://img.shields.io/badge/Claude_Code-Slash_Command-blue)
![Playwright](https://img.shields.io/badge/Powered_by-Playwright-45ba63)

---

## ✨ Features

- **🧠 Optional Planning Mode** - Design your implementation approach before coding
- **🔄 Automated Loop** - Keeps fixing and re-testing until perfect (or max loops)
- **📊 Scoring System** - 100-point rubric covering functionality, errors, UI, and UX
- **🎭 Playwright Testing** - Visual verification with real browser automation
- **📝 Detailed Reports** - Know exactly what passed, failed, and why
- **🚀 Auto-Deploy** - Commits and pushes only when score hits 100

---

## 📦 Installation

### 1. Copy the command file

```bash
# Create the commands directory if it doesn't exist
mkdir -p ~/.claude/commands

# Copy the command file
cp perfect.md ~/.claude/commands/perfect.md
```

### 2. Enable Playwright MCP

Add to your `~/.claude/mcp_settings.json`:

```json
{
  "mcpServers": {
    "playwright": {
      "command": "npx",
      "args": ["@anthropic-ai/claude-code-mcp-adapter", "npx", "@anthropic-ai/mcp-server-playwright@latest"]
    }
  }
}
```

### 3. Verify installation

```bash
# In Claude Code, run:
/mcp
# Ensure playwright is enabled
```

---

## 🚀 Usage

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

# Complex feature with planning mode
/perfect implement user profile page with avatar upload --0
```

---

## 🔄 How It Works

```
┌─────────────────────────────────────────────────────────────┐
│  STEP 0: Planning Mode Question                             │
│  ─────────────────────────────────────────────────────────  │
│  "Would you like me to plan the implementation first?"      │
│                                                             │
│  • Yes → Inline analysis + plan, then auto-proceed          │
│  • No  → Skip directly to implementation                    │
└─────────────────────────────────────────────────────────────┘
                              ↓
┌─────────────────────────────────────────────────────────────┐
│  STEP 1: Parse Arguments                                    │
│  ─────────────────────────────────────────────────────────  │
│  Extract task prompt and max loops from your command        │
└─────────────────────────────────────────────────────────────┘
                              ↓
┌─────────────────────────────────────────────────────────────┐
│  STEP 2: Execute Task                                       │
│  ─────────────────────────────────────────────────────────  │
│  Read codebase, implement changes, follow patterns          │
└─────────────────────────────────────────────────────────────┘
                              ↓
┌─────────────────────────────────────────────────────────────┐
│  STEP 3: Start Dev Server                                   │
│  ─────────────────────────────────────────────────────────  │
│  Auto-detect framework, find available port (3008+)         │
└─────────────────────────────────────────────────────────────┘
                              ↓
┌─────────────────────────────────────────────────────────────┐
│  STEP 4: Test with Playwright                               │
│  ─────────────────────────────────────────────────────────  │
│  • Navigate to app                                          │
│  • Test implemented feature                                 │
│  • Check console for JS errors                              │
│  • Monitor network for failed requests                      │
│  • Verify UI renders correctly                              │
│  • Test all interactions                                    │
└─────────────────────────────────────────────────────────────┘
                              ↓
┌─────────────────────────────────────────────────────────────┐
│  STEP 5: Calculate Score (0-100)                            │
│  ─────────────────────────────────────────────────────────  │
│  Functionality: 40 | Console: 20 | Network: 15              │
│  UI: 15 | UX: 10                                            │
└─────────────────────────────────────────────────────────────┘
                              ↓
┌─────────────────────────────────────────────────────────────┐
│  STEP 6: Decision                                           │
│  ─────────────────────────────────────────────────────────  │
│  • Score = 100    → Generate report, commit, push ✅        │
│  • Score < 100    → Fix issues, return to Step 4 🔄         │
│  • Max loops hit  → Generate report, list issues ❌         │
└─────────────────────────────────────────────────────────────┘
```

---

## 📊 Scoring Rubric

| Category | Points | What's Checked |
|----------|--------|----------------|
| **Functionality** | 40 | Does the feature work exactly as specified? |
| **Console Errors** | 20 | No JavaScript errors (−5 per error) |
| **Network Errors** | 15 | No failed API calls or 404s (−5 per error) |
| **UI Correctness** | 15 | Visual elements render properly |
| **UX/Interactions** | 10 | Smooth, responsive interactions |
| **TOTAL** | **100** | |

### Score Calculation

```javascript
functionalityScore = feature_works ? 40 : partial_score(0-39)
consoleScore       = Math.max(0, 20 - (consoleErrors * 5))
networkScore       = Math.max(0, 15 - (networkErrors * 5))
uiScore            = 15 - visualIssueDeductions
uxScore            = 10 - interactionIssueDeductions

totalScore = functionalityScore + consoleScore + networkScore + uiScore + uxScore
```

---

## 🧠 Planning Mode

When you run `/perfect`, you'll first be asked:

> "Would you like me to plan the implementation first before starting?"

### Option 1: "Yes - Plan first"

Performs **inline planning** (no permission prompts):

1. Analyzes codebase using Glob, Grep, and Read tools
2. Identifies relevant files and existing patterns
3. Outputs a structured implementation plan:
   - Files to modify/create
   - Step-by-step approach
   - Edge cases to handle
   - Testing considerations
4. **Auto-proceeds** to implementation (no waiting)

**Note:** Does NOT use the `EnterPlanMode` tool (which requires user approval). Instead performs the same thorough analysis inline to maintain the fully automated flow.

**Best for:** Complex features, unfamiliar codebases, architectural decisions

### Option 2: "No - Start immediately"

- Skips directly to implementation
- Faster for simple fixes

**Best for:** Quick bug fixes, styling changes, obvious implementations

---

## 📝 Report Format

After completion, you'll receive a detailed report:

```
═══════════════════════════════════════════════════════════════════════════════
                         /PERFECT EXECUTION REPORT
═══════════════════════════════════════════════════════════════════════════════

TASK: Add dark mode toggle to settings page

FINAL SCORE: 100/100
LOOPS EXECUTED: 2 of unlimited
STATUS: PERFECT ✅

───────────────────────────────────────────────────────────────────────────────
                              SCORE BREAKDOWN
───────────────────────────────────────────────────────────────────────────────

Functionality:    40/40   Toggle works, persists preference, applies theme
Console Errors:   20/20   0 error(s) found
Network Errors:   15/15   0 error(s) found
UI Correctness:   15/15   Theme applies correctly to all components
UX/Interactions:  10/10   Smooth transition, instant feedback

───────────────────────────────────────────────────────────────────────────────
                              LOOP HISTORY
───────────────────────────────────────────────────────────────────────────────

Loop 1: 75/100 - Fixed: Console error in ThemeContext, missing dark styles
Loop 2: 100/100 - All issues resolved

───────────────────────────────────────────────────────────────────────────────
                               GIT STATUS
───────────────────────────────────────────────────────────────────────────────

Committed and pushed to main.
Commit hash: a1b2c3d
Files changed:
  - src/contexts/ThemeContext.tsx
  - src/components/Settings/DarkModeToggle.tsx
  - src/styles/themes.css

═══════════════════════════════════════════════════════════════════════════════
```

---

## ⚙️ Configuration

### Supported Frameworks

The command auto-detects your framework and starts the dev server accordingly:

| Framework | Command Used |
|-----------|-------------|
| Next.js | `npm run dev -- -p [PORT]` |
| Vite | `npm run dev -- --port [PORT]` |
| Create React App | `set PORT=[PORT] && npm start` |
| Generic | `npm run dev -- --port [PORT]` |

### Port Selection

- Starts at port **3008**
- If occupied, tries 3009, 3010, etc.
- Uses first available port

---

## 🔧 Troubleshooting

| Issue | Solution |
|-------|----------|
| "Playwright MCP not available" | Run `/mcp` and enable playwright |
| Dev server won't start | Check `package.json` for correct scripts |
| Port conflicts | Command auto-finds available port |
| Git push fails | Ensure push access to main branch |
| Score stuck below 100 | Review remaining issues, fix manually, re-run |
| Planning mode not working | Planning is now inline - no special tool needed |

---

## 🎯 Tips for Best Results

### 1. Be Specific

```bash
# ✅ Good
/perfect fix the login button - should be blue (#0066CC),
         have hover state (#0052A3), and redirect to /dashboard on click

# ❌ Bad
/perfect fix the button
```

### 2. Use Unlimited Loops for Complex Features

```bash
/perfect implement complete user authentication flow --0
```

### 3. Use Limited Loops for Quick Fixes

```bash
/perfect fix typo in header --2
```

### 4. Let Planning Mode Help on Complex Tasks

When prompted, choose "Yes - Plan first" for:
- New feature implementations
- Unfamiliar codebases
- Multi-file changes
- Architectural decisions

---

## 📁 File Structure

```
~/.claude/
└── commands/
    └── perfect.md    # The command file
```

---

## 🤝 Contributing

Found a bug? Have an improvement idea?

1. Fork this repo
2. Create your feature branch (`git checkout -b feature/amazing-feature`)
3. Commit your changes (`git commit -m 'Add amazing feature'`)
4. Push to the branch (`git push origin feature/amazing-feature`)
5. Open a Pull Request

---

## 📄 License

MIT License - feel free to use, modify, and distribute.

---

## 🙏 Acknowledgments

- Built for [Claude Code](https://claude.ai/claude-code) by Anthropic
- Powered by [Playwright](https://playwright.dev/) for browser automation
- Inspired by the need for automated quality assurance in AI-assisted development

---

**Made with 🤖 by [Matt Johnston](https://mattjohnston.io)**

*Star ⭐ this repo if you find it useful!*
