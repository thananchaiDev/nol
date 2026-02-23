# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## What This Repo Is

**nol** is a Claude Code plugin that provides a structured feature development workflow via slash commands and sub-agents. It is installed as a plugin into other projects and adds commands like `/nol:feature`, `/nol:quick`, `/nol:bugfix`, `/nol:recap`, `/nol:approve`, and `/nol:backlog`.

## Versioning

Version is stored in **two files** — both must be updated together:
- `.claude-plugin/plugin.json` → `"version"` field
- `.claude-plugin/marketplace.json` → `"version"` field inside the `plugins[0]` object

## File Structure

```
commands/     — slash commands invoked by the user (/nol:<name>)
agents/       — sub-agents launched by commands (internal)
.claude-plugin/
  plugin.json       — plugin metadata + version
  marketplace.json  — marketplace listing + version
```

## File Formats

**Commands** (`commands/*.md`) — frontmatter with `description`, then markdown instructions. End with `Arguments: $ARGUMENTS`.

**Agents** (`agents/*.md`) — frontmatter with `name`, `description`, `model`, then markdown instructions. Agents receive variables via the prompt from the parent command — they cannot see conversation context.

## Command → Agent Pipeline

### `/nol:feature` (slow, high quality)
7 agents: Context + 2× Research (parallel) → Confirm-Research → Impact + Solution (parallel) → Test-Manual → writes SUMMARY.md → calls `recap`

### `/nol:quick` (fast, good quality)
5 agents: Context → Research + Solution + Impact + Test-Manual (4 parallel) → writes SUMMARY.md → calls `recap`

### `/nol:bugfix` (foreground only, no sub-agents)
Self: BUG.md → systematic-debugging skill → RESEARCH.md → ROOTCAUSE.md → SOLUTION.md → SUMMARY.md → NW-7 feedback loop

### `/nol:recap`
Reads plan files + checks actual codebase to verify implementation status → feedback loop → recommends `approve`

### `/nol:approve`
Reads SOLUTION.md + TEST_MANUAL.md → implements → runs tests until all pass → updates SUMMARY.md → recommends `backlog`

### `/nol:backlog`
Scans `.nol/feature/`, `.nol/bugfix/`, `.nol/quick/` in the target project for pending items

## Output Location (in target project)

All planning output is written to the **target project's** `.nol/` directory:
- `.nol/feature/{n}-{name}/` — FEATURE.md, RESEARCH.md, IMPACT.md, SOLUTION.md, TEST_MANUAL.md, SUMMARY.md
- `.nol/bugfix/{n}-{name}/` — BUG.md, RESEARCH.md, ROOTCAUSE.md, SOLUTION.md, SUMMARY.md
- `.nol/quick/{n}-{name}/` — REQUIREMENT.md, RESEARCH.md, IMPACT.md, SOLUTION.md, TEST_MANUAL.md, SUMMARY.md

## Implementation Status Detection (used by recap + backlog)

- **✅ Implemented** — has SUMMARY.md with `## Implementation Report` section
- **📋 Ready** — has SUMMARY.md + SOLUTION.md but no Implementation Report
- **🔄 Planning** — has FEATURE.md/BUG.md/REQUIREMENT.md but no SUMMARY.md
- **🐛 Open Bugs** — Implemented but `## Known Bugs` section has `❌ Open` rows
