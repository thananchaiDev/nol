# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## What This Repo Is

**nol** is a Claude Code plugin that provides a structured feature development workflow via slash commands and sub-agents. It is installed as a plugin into other projects and adds commands like `/nol:feature`, `/nol:quick`, `/nol:bugfix`, `/nol:recap`, `/nol:approve`, `/nol:backlog`, `/nol:done`. It also has an internal `learn` agent for recording lessons learned.

## When Adding New Features to nol

เมื่อเพิ่ม command ใหม่, agent ใหม่, หรือเปลี่ยน pipeline — ต้องอัปเดตไฟล์เหล่านี้ให้ครบ:

1. **`CLAUDE.md`** (ไฟล์นี้) — อัปเดต File Structure, Pipeline descriptions, Output Location
2. **`README.md`** — อัปเดต Commands table, Pipelines diagrams, Output Structure, File Structure
3. **Version** (3 ไฟล์ต้องตรงกันเสมอ) — bump version ทุกครั้งที่มีการเปลี่ยนแปลง

## Versioning

Version is stored in **three files** — all must be updated together:
- `.claude-plugin/plugin.json` → `"version"` field
- `.claude-plugin/marketplace.json` → `"version"` field inside the `plugins[0]` object
- `README.md` → `Current version: **x.x.x**` line in the `## Version` section

## File Structure

```
commands/     — slash commands invoked by the user (/nol:<name>)
agents/       — sub-agents launched by commands (internal)
  learn.md        — records nol pipeline mistakes as lesson learned
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
5–6 agents: reads mistakes → Context → Research + Solution + Impact + Test-Manual (4 parallel) → writes SUMMARY.md → Learn (background, if references previous work) → calls `recap`

### `/nol:bugfix` (mostly foreground, 1–2 background agents)
reads mistakes + detects reference → Self: BUG.md → systematic-debugging skill → RESEARCH.md → ROOTCAUSE.md → SOLUTION.md → Test-Manual (background) → TEST_MANUAL.md → SUMMARY.md → Learn (background, if references previous work) → NW-7 feedback loop

### `/nol:recap`
Reads plan files + checks actual codebase to verify implementation status → feedback loop → recommends `approve`

### `/nol:approve`
Reads SOLUTION.md + TEST_MANUAL.md → implements → runs tests until all pass → updates SUMMARY.md → recommends `backlog`

### `/nol:backlog`
Scans `.nol/feature/`, `.nol/bugfix/`, `.nol/quick/` in the target project for pending items. Cross-references git log for 🔄 Planning items to detect possibly-committed work.

### `/nol:done`
Marks a feature/bugfix/quick item as Implemented when it was implemented outside the nol pipeline. Creates/updates SUMMARY.md with `## Implementation Report` so backlog tracks it correctly. Cross-references git log to find related commits.

## Output Location (in target project)

All planning output is written to the **target project's** `.nol/` directory:
- `.nol/feature/{n}-{name}/` — FEATURE.md, RESEARCH.md, IMPACT.md, SOLUTION.md, TEST_MANUAL.md, SUMMARY.md
- `.nol/bugfix/{n}-{name}/` — BUG.md, RESEARCH.md, ROOTCAUSE.md, SOLUTION.md, TEST_MANUAL.md, SUMMARY.md
- `.nol/quick/{n}-{name}/` — REQUIREMENT.md, RESEARCH.md, IMPACT.md, SOLUTION.md, TEST_MANUAL.md, SUMMARY.md
- `.nol/mistake/{YYYY-MM-DD}.md` — lesson learned เมื่อ nol pipeline พลาด (เขียนโดย `learn` agent)

## Implementation Status Detection (used by recap + backlog)

- **✅ Implemented** — has SUMMARY.md with `## Implementation Report` section
- **📋 Ready** — has SUMMARY.md + SOLUTION.md but no Implementation Report
- **🔄 Planning** — has FEATURE.md/BUG.md/REQUIREMENT.md but no SUMMARY.md (use `/nol:done` if already implemented outside pipeline)
- **🐛 Open Bugs** — Implemented but `## Known Bugs` section has `❌ Open` rows
