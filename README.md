# nol

A Claude Code plugin that adds a structured feature development workflow to any project via slash commands and sub-agents.

## What It Does

nol gives you a set of `/nol:*` commands that turn a feature description or bug report into a full planning document — then let you approve and implement it when ready. All planning output is written to `.nol/` inside your target project.

## Commands

| Command | Description |
|---------|-------------|
| `/nol:feature` | Plan a new feature using a 7-agent pipeline (slow, high quality) |
| `/nol:quick` | Plan a small-to-medium task using a 3-phase pipeline: Research first, then Solution + Impact + Test in parallel (fast, good quality) |
| `/nol:bugfix` | Debug and plan a bug fix in foreground (no background agents) |
| `/nol:recap` | Review planning progress and verify implementation status against the real codebase |
| `/nol:approve` | Implement a plan from SOLUTION.md, run tests, and update SUMMARY.md |
| `/nol:backlog` | Show all pending features, bugfixes, and quick tasks |

## Pipelines

### `/nol:feature` — 7 agents

```
Phase 1a (3×, parallel):  Context  |  Research #1  |  Research #2
          ↓
Phase 1b (1×):            Confirm-Research → RESEARCH.md
          ↓
Phase 2  (2×, parallel):  Impact  |  Solution
          ↓
Phase 3  (1×):            Test-Manual → TEST_MANUAL.md
          ↓
          Write SUMMARY.md → /nol:recap (feedback loop)
```

### `/nol:quick` — 5–6 agents

```
Step 0:                   Read .nol/mistake/* + detect reference
          ↓
Phase 1 (1×):             Context → REQUIREMENT.md
          ↓
Phase 2 (1×):             Research → RESEARCH.md
          ↓  ← wait (Solution needs real codebase findings)
Phase 3 (3×, parallel):   Solution  |  Impact  |  Test-Manual
          ↓
          Write SUMMARY.md
          ↓
Step 4.5 (1×, bg, if ref): learn agent → .nol/mistake/YYYY-MM-DD.md
          ↓
          /nol:recap (feedback loop)
```

### `/nol:bugfix` — 1–2 background agents

```
NW-0:                     Read .nol/mistake/* + detect reference
          ↓
Foreground:               investigate → RESEARCH.md → ROOTCAUSE.md → SOLUTION.md
          ↓
NW-5.5 (1×, background):  test-manual agent → TEST_MANUAL.md
          ↓
          Write SUMMARY.md
          ↓
NW-6.5 (1×, bg, if ref):  learn agent → .nol/mistake/YYYY-MM-DD.md
          ↓
          feedback loop
```

## Output Structure

All files are written to the **target project's** `.nol/` directory:

```
.nol/
├── feature/{n}-{name}/
│   ├── FEATURE.md        Feature description & requirements
│   ├── RESEARCH.md       Codebase research findings
│   ├── IMPACT.md         Impact & risk analysis
│   ├── SOLUTION.md       Implementation plan & steps
│   ├── TEST_MANUAL.md    Verification & test plan
│   └── SUMMARY.md        Overview + implementation report
│
├── bugfix/{n}-{name}/
│   ├── BUG.md
│   ├── RESEARCH.md
│   ├── ROOTCAUSE.md
│   ├── SOLUTION.md
│   ├── TEST_MANUAL.md
│   └── SUMMARY.md
│
├── quick/{n}-{name}/
│   ├── REQUIREMENT.md
│   ├── RESEARCH.md
│   ├── IMPACT.md
│   ├── SOLUTION.md
│   ├── TEST_MANUAL.md
│   └── SUMMARY.md
│
└── mistake/
    └── YYYY-MM-DD.md     Lesson learned เมื่อ nol pipeline พลาด
```

## Implementation Status Detection

`/nol:recap` and `/nol:backlog` detect status by inspecting actual files:

| Status | Condition |
|--------|-----------|
| ✅ Implemented | SUMMARY.md has `## Implementation Report` section |
| 📋 Ready | Has SUMMARY.md + SOLUTION.md but no Implementation Report |
| 🔄 Planning | Has FEATURE.md / BUG.md / REQUIREMENT.md but no SUMMARY.md |
| 🐛 Open Bugs | Implemented but `## Known Bugs` section has `❌ Open` rows |

## Typical Workflow

```bash
# 1. Plan
/nol:feature add user authentication

# 2. Review + adjust
/nol:recap feature 1

# 3. Implement
/nol:approve feature 1

# 4. Check remaining work
/nol:backlog
```

## Installation

```bash
# 1. Add the marketplace
claude plugin marketplace add https://github.com/thananchaiDev/nol

# 2. Install the plugin
claude plugin install nol
```

## Version

Current version: **1.1.4** — defined in both `.claude-plugin/plugin.json` and `.claude-plugin/marketplace.json`.

## File Structure

```
commands/               Slash commands (/nol:<name>)
agents/                 Sub-agents launched internally by commands
  learn.md              Records nol pipeline mistakes as lesson learned
.claude-plugin/
  plugin.json           Plugin metadata + version
  marketplace.json      Marketplace listing + version
```
