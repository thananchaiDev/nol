# Feature - Plan & Research a New Feature

Create a structured feature folder using specialized sub-agents with dual-draft research and confirmation for higher quality results.

## Usage

```
/feature [description of the feature]
```

## AGENTS.md Compliance

> **CRITICAL: อ่านและปฏิบัติตาม `/Users/thananchai/Projects/wsol/AGENTS.md` ก่อนทำงานทุกครั้ง**
>
> กฎสำคัญที่ต้อง follow เสมอ:
> - **Debug:** ตรวจ Docker logs ก่อนเสมอ (`docker logs --tail 300 <container>`)
> - **Development:** ทำงานผ่าน Docker Compose เท่านั้น (env: `dev` หรือ `uat`)
> - **Testing:** รัน test ใน Docker container เท่านั้น — ห้ามรันบน host โดยตรง
> - **SQL:** ใส่ `WHERE TABLE_SCHEMA = '<database_name>'` ทุกครั้งที่ query `INFORMATION_SCHEMA`
>
> Sub-agents ทุกตัวที่ถูก launch ต้องได้รับ instruction ให้อ่าน AGENTS.md ด้วย

## Instructions

Parse `$ARGUMENTS` as the feature description.

### Step 1: Determine Feature Number & Name

1. Check existing directories under `.nol/feature/` to find the next available number.
   - If no directories exist, start at `1`.
   - Otherwise, take the highest existing number + 1.
2. Generate a short kebab-case name from the description (e.g., `add-hscode-sorting`, `user-authentication`). Keep it concise (2-4 words max).
3. Set `FEATURE_DIR` = `.nol/feature/{number}-{feature-name}/`
4. Create directories immediately: `mkdir -p {FEATURE_DIR}/drafts`

### Silent Execution Rule

**CRITICAL: ห้าม output อะไรให้ user ระหว่าง Step 2–6 ทั้งสิ้น**
- ทำงานทุก phase อย่างเงียบๆ จนกว่า SUMMARY.md จะถูกเขียนเสร็จสมบูรณ์
- ห้ามรายงานสถานะ ห้ามบอกว่า "กำลังรัน agent X" ห้ามสรุปผลระหว่างกลาง
- **output ครั้งแรกสุดกับ user ต้องเป็น Step 7 เท่านั้น** หลังจาก SUMMARY.md เขียนเสร็จแล้ว

### How Sub-Agents Work

**CRITICAL — follow this exactly for every agent launch:**

1. Set `subagent_type` to the agent name (e.g., `feature-research`, `feature-confirm`, etc.)
2. Pass a `prompt` that provides all variable values the agent needs — the agent's instructions are loaded automatically from its definition file
3. The sub-agent CANNOT see the conversation context — the prompt you pass provides the variable values it needs

### Step 2: Phase 1a — Context + Research (3 Agents, parallel)

Launch **3 agents in a single message**, all with `run_in_background: true`.
รอ task IDs ทั้ง 3 ก่อน — **ไม่ต้อง output อะไรกับ user**

| Agent | subagent_type | Prompt Variables |
|-------|--------------|------------------|
| A1: Context Gatherer | `feature-context` | `FEATURE_DESCRIPTION`, `OUTPUT_PATH` = `{FEATURE_DIR}/FEATURE.md` |
| A2: Researcher #1 | `feature-research` | `FEATURE_DESCRIPTION`, `FEATURE_MD_PATH` (tell it to search codebase independently — no FEATURE.md exists yet), `OUTPUT_PATH` = `{FEATURE_DIR}/drafts/RESEARCH-1.md` |
| A3: Researcher #2 | `feature-research` | `FEATURE_DESCRIPTION`, `FEATURE_MD_PATH` (tell it to search codebase independently — no FEATURE.md exists yet), `OUTPUT_PATH` = `{FEATURE_DIR}/drafts/RESEARCH-2.md` |

ใช้ `TaskOutput` รอจนครบทั้ง 3 — **เงียบๆ ไม่ต้อง report ผลระหว่างรอ**

### Step 3: Phase 1b — Confirm Research (1 Agent)

Launch **1 confirm agent** with `run_in_background: true`:

| Agent | subagent_type | Prompt Variables |
|-------|--------------|------------------|
| Confirm-Research | `feature-confirm` | `DRAFT_1_PATH` = `{FEATURE_DIR}/drafts/RESEARCH-1.md`, `DRAFT_2_PATH` = `{FEATURE_DIR}/drafts/RESEARCH-2.md`, `OUTPUT_PATH` = `{FEATURE_DIR}/RESEARCH.md`, `AGENT_ROLE` = "Researcher" |

ใช้ `TaskOutput` รอจนเสร็จ — **เงียบๆ ไม่ต้อง report ผลระหว่างรอ**

### Step 4: Phase 2 — Impact + Solution (2 Agents)

Launch **2 agents in a single message**, both with `run_in_background: true`:

| Agent | subagent_type | Prompt Variables |
|-------|--------------|------------------|
| B: Impact Analyzer | `feature-impact` | `FEATURE_DESCRIPTION`, `RESEARCH_MD_PATH` = `{FEATURE_DIR}/RESEARCH.md`, `OUTPUT_PATH` = `{FEATURE_DIR}/IMPACT.md` |
| C: Solution Architect | `feature-solution` | `FEATURE_DESCRIPTION`, `RESEARCH_MD_PATH` = `{FEATURE_DIR}/RESEARCH.md`, `IMPACT_MD_PATH` (tell it to base primarily on RESEARCH.md since Impact may still be writing), `OUTPUT_PATH` = `{FEATURE_DIR}/SOLUTION.md` |

ใช้ `TaskOutput` รอจนครบทั้ง 2 — **เงียบๆ ไม่ต้อง report ผลระหว่างรอ**

### Step 5: Phase 3 — Verification Plan (1 Agent)

Launch **1 agent** with `run_in_background: true`:

| Agent | subagent_type | Prompt Variables |
|-------|--------------|------------------|
| D: Verification Planner | `feature-verify` | `FEATURE_DESCRIPTION`, `IMPACT_MD_PATH` = `{FEATURE_DIR}/IMPACT.md`, `SOLUTION_MD_PATH` = `{FEATURE_DIR}/SOLUTION.md`, `OUTPUT_PATH` = `{FEATURE_DIR}/VERIFY.md` |

ใช้ `TaskOutput` รอจนเสร็จ — **เงียบๆ ไม่ต้อง report ผลระหว่างรอ**

### Step 6: Phase 4 — Summary Synthesis (1 Agent)

Launch **1 agent** with `run_in_background: true`:

| Agent | subagent_type | Prompt Variables |
|-------|--------------|------------------|
| E: Summary Synthesizer | `feature-summary` | `FEATURE_DIR` = `{FEATURE_DIR}` |

This agent reads all 5 final files and returns a summary (does NOT write a file).

ใช้ `TaskOutput` รอจนเสร็จ — **นี่คือ background agent ตัวสุดท้าย รอให้เสร็จ 100% ก่อน**

หลังจาก Agent E return แล้ว ให้ **write SUMMARY.md ก่อน แล้วค่อย present** — ห้าม output กับ user จนกว่าจะ write เสร็จ:

เขียน `{FEATURE_DIR}/SUMMARY.md` ด้วย Write tool โดยมีเนื้อหา:
- Header: ชื่อ feature + วันที่
- Full summary text จาก Agent E
- Table of contents link ไปทุกไฟล์:
  - `FEATURE.md` — Feature description & requirements
  - `RESEARCH.md` — Codebase research findings
  - `IMPACT.md` — Impact & risk analysis
  - `SOLUTION.md` — Implementation plan & steps
  - `VERIFY.md` — Verification & test plan

### Step 7: Present Summary

**เมื่อ SUMMARY.md เขียนเสร็จแล้วเท่านั้น** จึงค่อย present ให้ user ครั้งแรก
Include:
- The feature folder path: `{FEATURE_DIR}`
- Path to the summary file: `{FEATURE_DIR}/SUMMARY.md`
- Links to all 5 final files for reference

### Step 8: Wait for User Feedback

After presenting the summary, **explicitly ask the user** one of the following:

> "แผนนี้พร้อมแล้ว — อยากให้ปรับแก้อะไร หรืออนุมัติให้ implement เลยไหมครับ?"

Wait for the user's response, then:

**If the user approves (e.g., "implement", "go ahead", "อนุมัติ", "ทำได้เลย")**:
- Proceed to implement the feature according to `{FEATURE_DIR}/SOLUTION.md`
- Follow the implementation steps exactly as documented
- Mark each step as complete as you progress

**If the user provides feedback or requests changes**:
1. Identify which files need updating based on the feedback:
   - Scope/requirement changes → update `{FEATURE_DIR}/FEATURE.md`
   - Research findings → update `{FEATURE_DIR}/RESEARCH.md`
   - Impact analysis → update `{FEATURE_DIR}/IMPACT.md`
   - Solution/approach changes → update `{FEATURE_DIR}/SOLUTION.md`
   - Test plan changes → update `{FEATURE_DIR}/VERIFY.md`
2. Apply all changes to the relevant files
3. Re-run the summary agent (Step 6) to regenerate the summary with updated content
4. Overwrite `{FEATURE_DIR}/SUMMARY.md` with the new summary
5. Present the updated summary to the user and **repeat Step 8**

Continue this feedback loop until the user explicitly approves implementation.

## Agent Execution Flow

```
Phase 1a (3 background):  A1 (Context → FEATURE.md)  |  A2 + A3 (Research)
         ↓ wait all 3
Phase 1b (1 background):  Confirm-Research → RESEARCH.md
         ↓ wait
Phase 2  (2 background):  B (Impact)  |  C (Solution)
         ↓ wait both
Phase 3  (1 background):  D (VERIFY.md)
         ↓ wait
Phase 4  (1 background):  E (Summary → return)
         ↓ wait
                           Present to user
         ↓
Step 8:  Wait for feedback
         ├─ Approved → Implement
         └─ Changes  → Update files → Re-summarize → Repeat Step 8
```

**Total agents: 8** (3 + 1 + 2 + 1 + 1) + optional re-summary on revision

## Important

- **DO NOT implement** until user explicitly approves.
- **CRITICAL**: Set `subagent_type` to the agent name (e.g., `feature-research`). The agent's instructions are loaded automatically. Your prompt only needs to provide the variable values.
- Write in the same language the user used in the description.
- If any agent fails, report the error and continue with remaining agents.
- When updating plan files based on feedback, edit the files directly — do not re-run the full pipeline unless a major re-research is needed.

---

Arguments: $ARGUMENTS
