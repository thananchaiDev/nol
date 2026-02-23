# nol

A Claude Code plugin that adds a structured feature development workflow to any project via slash commands and sub-agents.

## What It Does

nol gives you a set of `/nol:*` commands that turn a feature description or bug report into a full planning document — then let you approve and implement it when ready. All planning output is written to `.nol/` inside your target project.

## Commands

| Command | Description |
|---------|-------------|
| `/nol:feature` | Plan a new feature using a 7-agent pipeline (slow, high quality) |
| `/nol:quick` | Plan a small-to-medium task using a 3-phase pipeline: Research first, then Solution + Impact + Test in parallel (fast, good quality) |
| `/nol:bugfix` | Investigate bug from live evidence (foreground), then research + rootcause + solution + test plan via background agents |
| `/nol:recap` | Review planning progress and verify implementation status against the real codebase |
| `/nol:approve` | Implement a plan from SOLUTION.md, run tests, and update SUMMARY.md |
| `/nol:backlog` | Show all pending features, bugfixes, and quick tasks |
| `/nol:done` | Mark an item as implemented when it was committed outside the nol pipeline |

## Pipelines

### `/nol:feature` — 7 agents

```
Step 0:                   Read .nol/mistake/* + read .nol/knowledge.md (or launch knowledge bg)
          ↓
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

### `/nol:quick` — 6 agents

```
Step 0:                   Read .nol/mistake/* + detect reference
                          + read .nol/knowledge.md (or launch knowledge bg)
          ↓
Phase 1 (1×):             Context → REQUIREMENT.md
          ↓
Phase 2 (1×):             Research → RESEARCH.md
          ↓  ← wait (Solution needs real codebase findings)
Phase 3 (3×, parallel):   Solution  |  Impact  |  Test-Manual
          ↓
          Write SUMMARY.md
          ↓
Step 4.5 (1×, bg, always): learn agent → .nol/mistake/YYYY-MM-DD.md
          ↓
          /nol:recap (feedback loop)
```

### `/nol:bugfix` — 4 background agents (sequential chain)

```
NW-0 (self):              Read .nol/mistake/* + detect reference
                          + read .nol/knowledge.md (or launch knowledge bg)
          ↓
NW-1 (self):              Create BUG.md
          ↓
NW-2 (self):              Investigate — live evidence only (logs, DevTools, no code reading)
                          → Write RESEARCH.md (live evidence) + REPRODUCE.md
          ↓
NW-3 (background, wait):  bug-research agent → RESEARCH.md (full, with code findings)
          ↓
NW-4 (background, wait):  bug-rootcause agent → ROOTCAUSE.md
          ↓
NW-5 (background, wait):  bug-solution agent → SOLUTION.md
          ↓
NW-6 (background, wait):  test-manual agent → TEST_MANUAL.md
          ↓
          Write SUMMARY.md
          ↓
NW-7.5 (background, no wait): learn agent → .nol/mistake/YYYY-MM-DD.md
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
│   ├── REPRODUCE.md
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
├── mistake/
│   └── YYYY-MM-DD.md     Lesson learned เมื่อ nol pipeline พลาด
│
└── knowledge.md          Project context (type, stack, purpose) — สร้างครั้งแรก ใช้ซ้ำ
```

## Implementation Status Detection

`/nol:recap` and `/nol:backlog` detect status by inspecting actual files:

| Status | Condition |
|--------|-----------|
| ✅ Implemented | SUMMARY.md has `## Implementation Report` section |
| 📋 Ready | Has SUMMARY.md + SOLUTION.md but no Implementation Report |
| 🔄 Planning | Has FEATURE.md / BUG.md / REQUIREMENT.md but no SUMMARY.md |
| 🐛 Open Bugs | Implemented but `## Known Bugs` section has `❌ Open` rows |

> If an item was committed manually (outside the pipeline) and shows as 🔄 Planning, use `/nol:done` to mark it as implemented.

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

# If you committed a fix directly (outside pipeline) and it shows as 🔄 Planning:
/nol:done bugfix 3
```

## Installation

```bash
# 1. Add the marketplace
claude plugin marketplace add https://github.com/thananchaiDev/nol

# 2. Install the plugin
claude plugin install nol
```

## Version

Current version: **1.1.10** — defined in both `.claude-plugin/plugin.json` and `.claude-plugin/marketplace.json`.

## File Structure

```
commands/               Slash commands (/nol:<name>)
agents/                 Sub-agents launched internally by commands
  knowledge.md          Analyzes the project and writes .nol/knowledge.md
  learn.md              Records nol pipeline mistakes as lesson learned
  bug-research.md       Digs codebase for bugs using live evidence as guide
  bug-rootcause.md      Confirms root cause from research findings
  bug-solution.md       Designs minimal fix from confirmed root cause
.claude-plugin/
  plugin.json           Plugin metadata + version
  marketplace.json      Marketplace listing + version
```
