---
description: "วิเคราะห์และวางแผน task ขนาดเล็ก-กลาง ด้วย parallel multi-agent pipeline ที่เร็วกว่า feature"
---

# Quick - Plan a Task Fast

สร้าง quick folder ด้วย 5 parallel sub-agents: 1 agent เขียน REQUIREMENT.md ก่อน จากนั้น spawn อีก 4 agent พร้อมกันทันที

## Usage

```
/nol:quick [description of the task]
```

**ตัวอย่าง:**
```
/nol:quick เพิ่ม validation ตรวจ email ก่อน submit
/nol:quick refactor function calculateTotal ให้ clean ขึ้น
/nol:quick แก้ bug ปุ่ม save ไม่ทำงานใน form นี้
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

Parse `$ARGUMENTS` as the task description.

### Step 1: Determine Number & Name

1. ตรวจ directories ที่มีอยู่ใน `.nol/quick/` เพื่อหาเลขถัดไป
   - ถ้าไม่มี directory เลย → เริ่มที่ `1`
   - ถ้ามีแล้ว → ใช้ highest number + 1
2. สร้างชื่อ kebab-case จาก description (2–4 words max)
3. ตั้งค่า `QUICK_DIR` = `.nol/quick/{number}-{task-name}/`
4. สร้าง directory ทันที: `mkdir -p {QUICK_DIR}`

### Silent Execution Rule

**CRITICAL: ห้าม output อะไรให้ user ระหว่าง Step 2–4 ทั้งสิ้น**
- ทำงานทุก phase อย่างเงียบๆ จนกว่า SUMMARY.md จะถูกเขียนเสร็จสมบูรณ์
- ห้ามรายงานสถานะ ห้ามบอกว่า "กำลังรัน agent X" ห้ามสรุปผลระหว่างกลาง
- **output ครั้งแรกสุดกับ user ต้องเป็น Step 5 (recap) เท่านั้น** หลังจาก SUMMARY.md เขียนเสร็จแล้ว

### How Sub-Agents Work

**CRITICAL — follow this exactly for every agent launch:**

1. Set `subagent_type` to the agent name (e.g., `context`, `research`, etc.)
2. Pass a `prompt` that provides all variable values the agent needs — the agent's instructions are loaded automatically from its definition file
3. The sub-agent CANNOT see the conversation context — the prompt you pass provides the variable values it needs

### Step 2: Phase 1 — Requirement (1 Agent, background)

Launch **1 agent** with `run_in_background: true`:

| Agent | subagent_type | Prompt Variables |
|-------|--------------|------------------|
| A: Requirement Writer | `context` | `FEATURE_DESCRIPTION` = task description, `OUTPUT_PATH` = `{QUICK_DIR}/REQUIREMENT.md` |

ใช้ `TaskOutput` รอจนเสร็จ — **เงียบๆ ไม่ต้อง report ผลระหว่างรอ**

> หมายเหตุ: ใช้ `context` agent เพราะมีหน้าที่ structure ความต้องการเป็น document — output จะเป็นไฟล์ชื่อ REQUIREMENT.md (ตาม OUTPUT_PATH ที่ส่งให้)

### Step 3: Phase 2 — Analysis (4 Agents, parallel background)

Launch **4 agents ในข้อความเดียว** ทั้งหมด `run_in_background: true`:

| Agent | subagent_type | Prompt Variables |
|-------|--------------|------------------|
| B1: Researcher | `research` | `FEATURE_DESCRIPTION` = task description, `FEATURE_MD_PATH` = `{QUICK_DIR}/REQUIREMENT.md` (อ่านไฟล์นี้เป็น context ก่อน), `OUTPUT_PATH` = `{QUICK_DIR}/RESEARCH.md` |
| B2: Solution Architect | `solution` | `FEATURE_DESCRIPTION` = task description, `RESEARCH_MD_PATH` = `{QUICK_DIR}/REQUIREMENT.md` (research ยังไม่มี — ใช้ REQUIREMENT.md แทน), `IMPACT_MD_PATH` = `{QUICK_DIR}/REQUIREMENT.md` (impact ยังไม่มี — ใช้ REQUIREMENT.md แทน), `OUTPUT_PATH` = `{QUICK_DIR}/SOLUTION.md` |
| B3: Impact Analyzer | `impact` | `FEATURE_DESCRIPTION` = task description, `RESEARCH_MD_PATH` = `{QUICK_DIR}/REQUIREMENT.md` (research ยังไม่มี — ใช้ REQUIREMENT.md แทน), `OUTPUT_PATH` = `{QUICK_DIR}/IMPACT.md` |
| B4: Test Planner | `test-manual` | `FEATURE_DESCRIPTION` = task description, `IMPACT_MD_PATH` = `{QUICK_DIR}/REQUIREMENT.md` (impact ยังไม่มี — ใช้ REQUIREMENT.md แทน), `SOLUTION_MD_PATH` = `{QUICK_DIR}/REQUIREMENT.md` (solution ยังไม่มี — ใช้ REQUIREMENT.md แทน), `OUTPUT_PATH` = `{QUICK_DIR}/TEST_MANUAL.md` |

ใช้ `TaskOutput` รอจนครบทั้ง 4 — **เงียบๆ ไม่ต้อง report ผลระหว่างรอ**

### Step 4: Write SUMMARY.md

**ห้าม output กับ user จนกว่าจะ write เสร็จ**

เขียน `{QUICK_DIR}/SUMMARY.md` ด้วย Write tool โดยมีเนื้อหา:
- Header: ชื่อ task + วันที่
- Table of contents link ไปทุกไฟล์:
  - `REQUIREMENT.md` — Task requirement & description
  - `RESEARCH.md` — Codebase research findings
  - `IMPACT.md` — Impact & risk analysis
  - `SOLUTION.md` — Implementation plan & steps
  - `TEST_MANUAL.md` — Verification & test plan

### Step 5: Recap

**เมื่อ SUMMARY.md เขียนเสร็จแล้วเท่านั้น** จึงค่อย present ให้ user ครั้งแรก โดยเรียก `/nol:recap quick {number}` เพื่อแสดงผลและเปิด feedback loop

## Agent Execution Flow

```
Phase 1 (1 background):   A (context → REQUIREMENT.md)
         ↓ wait
Phase 2 (4 background):   B1 (Research)  |  B2 (Solution)  |  B3 (Impact)  |  B4 (Test)
         ↓ wait all 4
Step 4:  Write SUMMARY.md (header + links)
         ↓
Step 5:  /nol:recap quick {number}  →  feedback loop (handled by recap)
```

**Total agents: 5** (1 + 4)

## Quick vs Feature

| ด้าน | /quick | /feature |
|------|--------|---------|
| เวลา | เร็วกว่า (~5 phases → 2 phases) | ช้ากว่า (sequential + confirm) |
| คุณภาพ | ดี (agents ทำงานอิสระ) | ดีมาก (research confirmed + sequential) |
| เหมาะกับ | task ขนาดเล็ก-กลาง, bug fix, refactor | feature ใหม่ซับซ้อน, architectural change |

## Important

- **DO NOT implement** until user explicitly approves.
- **CRITICAL**: Set `subagent_type` to the agent name (e.g., `context`). The agent's instructions are loaded automatically. Your prompt only needs to provide the variable values.
- Write in the same language the user used in the description.
- If any agent fails, report the error and continue with remaining agents.
- When updating plan files based on feedback, edit the files directly — do not re-run the full pipeline unless a major re-research is needed.

---

Arguments: $ARGUMENTS
