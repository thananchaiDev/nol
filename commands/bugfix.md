---
description: "วิเคราะห์ debug และวางแผนแก้ bug"
---

# Bugfix - Investigate & Plan a Bug Fix

Investigate bug จากของจริง (logs, browser, network) ใน foreground → ส่งต่อให้ background agents วิเคราะห์ codebase, rootcause, solution, และ test plan

## Usage

```
/nol:bugfix [description of the bug]
```

**ตัวอย่าง:**
```
/nol:bugfix ปุ่ม reset ไม่ล้างรูป
/nol:bugfix login error on mobile
/nol:bugfix ตรงนี้ยังไม่ผ่าน
/nol:bugfix feature 1 ปุ่ม submit ไม่ทำงานใน step 3
```

## AGENTS.md Compliance

> **CRITICAL: อ่านและปฏิบัติตาม `/Users/thananchai/Projects/wsol/AGENTS.md` ก่อนทำงานทุกครั้ง**
>
> กฎสำคัญที่ต้อง follow เสมอ:
> - **Debug:** ตรวจ Docker logs ก่อนเสมอ (`docker logs --tail 300 <container>`) — ก่อนแก้โค้ดใดๆ
> - **Development:** ทำงานผ่าน Docker Compose เท่านั้น (env: `dev` หรือ `uat`)
> - **Testing:** รัน test ใน Docker container เท่านั้น — ห้ามรันบน host โดยตรง; ใช้ `.mocharc.json`
> - **SQL:** ใส่ `WHERE TABLE_SCHEMA = '<database_name>'` ทุกครั้งที่ query `INFORMATION_SCHEMA`

## Instructions

ใช้ `$ARGUMENTS` ทั้งหมดเป็น bug description

---

### NW-0: อ่าน Mistake Files, ตรวจ Reference และ Knowledge

**A. อ่าน Mistake Files:**

1. ใช้ Glob tool หา `.nol/mistake/*.md`
2. ถ้ามี → Read ทุกไฟล์ และนำมาเป็น context เพื่อ avoid ข้อผิดพลาดซ้ำ
3. ถ้าไม่มี → ข้ามไปเงียบๆ

**B. ตรวจ Reference ถึงงานเดิม:**

ตรวจ `$ARGUMENTS` ว่ามีการอ้างถึงงานเดิมหรือไม่ โดยหาคีย์เวิร์ด เช่น `feature 1`, `feature add-hscode`, `quick 2`, `bugfix 3`:

- ถ้ามี reference → ตั้งค่า `REFERENCED_LABEL` = type + identifier เช่น `"feature 1"`
  - ค้นหา directory ที่ match ใน `.nol/{type}/` (เช่น `.nol/feature/1-*/`)
  - ตั้งค่า `REFERENCE_SUMMARY_PATH` = path ของ SUMMARY.md ที่เจอ (ถ้ามี)
- ถ้าไม่มี reference → ตั้งค่า `REFERENCED_LABEL` = `""` (ว่าง)

**C. อ่าน Project Knowledge:**

1. ใช้ Glob tool หา `.nol/knowledge.md`
2. ถ้ามี → Read ไฟล์ และนำมาเป็น context พื้นฐานของ project
3. ถ้าไม่มี → Launch **knowledge agent** กับ `run_in_background: true` (ไม่รอผล):
   - `PROJECT_ROOT` = `.`
   - `KNOWLEDGE_FILE` = `.nol/knowledge.md`

---

### NW-1: สร้าง Directory และ BUG.md

1. นับ bugfix ที่มีอยู่แล้วใน `.nol/bugfix/` เพื่อหาเลขถัดไป
   - ถ้าไม่มี directory เลย → เริ่มที่ `1`
   - ถ้ามีแล้ว → ใช้ highest number + 1
2. สร้างชื่อ kebab-case จาก bug description (2–4 words)
3. ตั้งค่า `BUGFIX_DIR` = `.nol/bugfix/{number}-{bug-name}/`
4. สร้าง directory: `mkdir -p {BUGFIX_DIR}`
5. เขียน `{BUGFIX_DIR}/BUG.md` ด้วย Write tool:
   ```markdown
   # Bug Report: {bug description}

   ## Bug Description
   {ข้อมูลจาก bug description ที่ user ให้มา}

   ## Steps to Reproduce
   (จะถูก update ระหว่าง investigation)

   ## Expected vs Actual Behavior
   (จะถูก update ระหว่าง investigation)

   ## Affected Component/Area
   (จะถูก update ระหว่าง investigation)

   ## Fix Goal
   (จะถูก update ระหว่าง investigation)
   ```
6. Report ให้ user รู้: `"เริ่ม investigate: {bug description}"`

---

### NW-2: Investigate — Live Evidence เท่านั้น

> **สำคัญ:** phase นี้เน้นดูของจริง (logs, browser, network) เท่านั้น — **ห้ามอ่าน source code files** ในขั้นตอนนี้ ปล่อยให้ research agent ทำ

เก็บ live evidence ให้ครบ แล้วบันทึกลงใน `{BUGFIX_DIR}/RESEARCH.md`:

```markdown
# Bug Research: {bug description}

## Live Evidence

### Error Messages / Logs
(Docker logs, stack traces, error output)

### Reproduction Steps
(สิ่งที่ทำแล้ว bug เกิด)

### Expected vs Actual
(ผลที่ควรได้ vs ผลที่ได้จริง)

### Network Requests
(API calls ที่เกี่ยวข้อง — method, path, request body, response, status code)

### Console Errors
(browser console errors/warnings)

### Screenshots
(path ถ้ามี)

### Key Signals
(จุดที่น่าสงสัยที่สุดจาก evidence ข้างต้น)
```

#### A. Docker Logs (ถ้า backend bug หรือไม่แน่ใจ):

1. รัน `docker logs --tail 300 <container>` ผ่าน Bash
2. บันทึก error messages, stack traces, หรือ log ที่เกี่ยวกับ bug
3. ถ้าไม่มี Docker หรือ app ไม่ running → ข้ามไปเงียบๆ

#### B. Chrome DevTools Live Reproduction:

1. ใช้ `mcp__chrome-devtools__list_pages` ดูว่ามี browser เปิดอยู่ไหม
2. ถ้าไม่มี → ข้ามขั้นตอนนี้ไปเงียบๆ
3. ถ้ามี → `mcp__chrome-devtools__select_page` เลือก page ที่เกี่ยวข้อง

**ถ้า bug เกี่ยวกับ Frontend (UI, rendering, form, state):**
- Navigate ไปหน้าที่มีปัญหา
- `mcp__chrome-devtools__take_snapshot` ดู state ปัจจุบัน
- ทำ action ที่ trigger bug (`click`, `fill`)
- ตรวจ console: `mcp__chrome-devtools__list_console_messages` (filter `types: ["error", "warn"]`)
- ตรวจ network: `mcp__chrome-devtools__list_network_requests` (filter `resourceTypes: ["fetch", "xhr"]`)
- `mcp__chrome-devtools__take_screenshot` บันทึก visual evidence

**ถ้า bug เกี่ยวกับ Backend (API, database, server logic):**
- Login ผ่าน browser ถ้ายังไม่ได้ login
- Navigate/trigger action ที่ reproduce bug
- `mcp__chrome-devtools__list_network_requests` หา failing request
- `mcp__chrome-devtools__get_network_request` ดู request body + response body
- สร้าง curl command จาก request นั้น แล้วรันผ่าน Bash ดู raw response

#### C. Update BUG.md:

หลังจาก investigate เสร็จ ให้ update `{BUGFIX_DIR}/BUG.md` ด้วย:
- Steps to Reproduce (จากสิ่งที่ทำได้จริง)
- Expected vs Actual Behavior (จากที่เห็นจริง)
- Affected Component/Area (ประมาณจาก evidence — frontend/backend/API path)
- Fix Goal (จาก expected behavior)

#### D. Write REPRODUCE.md:

หลังจาก investigate เสร็จ ให้เขียน `{BUGFIX_DIR}/REPRODUCE.md` ด้วย Write tool:

```markdown
# Reproduction Steps: {bug description}

## Environment
- URL / Page: {หน้าที่เกิด bug}
- User role / State: {ถ้าต้อง login หรือมีเงื่อนไขก่อน}

## Prerequisites
{สิ่งที่ต้องทำก่อน เช่น login, มีข้อมูลใน DB, etc.}

## Steps to Reproduce
1. {step 1}
2. {step 2}
3. ...

## Expected Result
{ผลที่ควรได้}

## Actual Result
{ผลที่ได้จริง — พร้อม error message หรือ screenshot path ถ้ามี}

## Additional Context
{Docker logs snippet, request/response body, หรือ evidence อื่นๆ ที่เกี่ยวข้อง}
```

---

### NW-3: Research Agent (Background)

Launch **bug-research agent** กับ `run_in_background: true`:

| Agent | subagent_type | Prompt Variables |
|-------|--------------|------------------|
| Bug Researcher | `bug-research` | `BUG_DESCRIPTION` = bug description, `BUG_MD_PATH` = `{BUGFIX_DIR}/BUG.md`, `LIVE_EVIDENCE_PATH` = `{BUGFIX_DIR}/RESEARCH.md`, `OUTPUT_PATH` = `{BUGFIX_DIR}/RESEARCH.md` |

ใช้ `TaskOutput` รอจนเสร็จก่อนไป NW-4

---

### NW-4: Rootcause Agent (Background)

Launch **bug-rootcause agent** กับ `run_in_background: true`:

| Agent | subagent_type | Prompt Variables |
|-------|--------------|------------------|
| Root Cause Analyst | `bug-rootcause` | `BUG_DESCRIPTION` = bug description, `BUG_MD_PATH` = `{BUGFIX_DIR}/BUG.md`, `RESEARCH_MD_PATH` = `{BUGFIX_DIR}/RESEARCH.md`, `OUTPUT_PATH` = `{BUGFIX_DIR}/ROOTCAUSE.md` |

ใช้ `TaskOutput` รอจนเสร็จก่อนไป NW-5

---

### NW-5: Solution Agent (Background)

Launch **bug-solution agent** กับ `run_in_background: true`:

| Agent | subagent_type | Prompt Variables |
|-------|--------------|------------------|
| Fix Designer | `bug-solution` | `BUG_DESCRIPTION` = bug description, `RESEARCH_MD_PATH` = `{BUGFIX_DIR}/RESEARCH.md`, `ROOTCAUSE_MD_PATH` = `{BUGFIX_DIR}/ROOTCAUSE.md`, `OUTPUT_PATH` = `{BUGFIX_DIR}/SOLUTION.md` |

ใช้ `TaskOutput` รอจนเสร็จก่อนไป NW-6

---

### NW-6: Test Manual Agent (Background)

Launch **test-manual agent** กับ `run_in_background: true`:

| Agent | subagent_type | Prompt Variables |
|-------|--------------|------------------|
| Test Planner | `test-manual` | `FEATURE_DESCRIPTION` = bug description, `IMPACT_MD_PATH` = `{BUGFIX_DIR}/ROOTCAUSE.md`, `SOLUTION_MD_PATH` = `{BUGFIX_DIR}/SOLUTION.md`, `OUTPUT_PATH` = `{BUGFIX_DIR}/TEST_MANUAL.md` |

ใช้ `TaskOutput` รอจนเสร็จก่อนไป NW-7

---

### NW-7: เขียน SUMMARY.md และ Present

เขียน `{BUGFIX_DIR}/SUMMARY.md` สรุปสั้นๆ แล้ว present ให้ user:
- Root cause ที่พบ
- Fix approach
- Files ที่จะแก้
- Links ไปทุกไฟล์ (BUG.md, RESEARCH.md, ROOTCAUSE.md, SOLUTION.md, TEST_MANUAL.md)

---

### NW-7.5: Launch Learn Agent

Launch **1 agent** กับ `run_in_background: true`:

| Agent | subagent_type | Prompt Variables |
|-------|--------------|------------------|
| Learn | `learn` | `TASK_DESCRIPTION` = bug description, `CURRENT_DIR` = `{BUGFIX_DIR}`, `REFERENCE_LABEL` = `{REFERENCED_LABEL}` หรือ `"bugfix {number}"` ถ้า REFERENCED_LABEL ว่าง, `REFERENCE_SUMMARY_PATH` = `{REFERENCE_SUMMARY_PATH}` หรือ `"{BUGFIX_DIR}/SUMMARY.md"` ถ้า REFERENCED_LABEL ว่าง, `MISTAKE_DIR` = `.nol/mistake/` |

ไม่ต้องรอผล — ดำเนินการต่อไป NW-8 ได้เลย

---

### NW-8: Feedback Loop

> "debug เสร็จแล้ว — root cause คือ [X] อยากให้ implement ทันทีหรือปรับ approach ก่อน?"
>
> 💡 **ถัดไป:** implement ทันทีได้เลย หรือใช้ `/nol:approve bugfix {number}` ในภายหลัง

**ถ้า user อนุมัติ** → implement ตาม SOLUTION.md ทันที แล้วแจ้ง:
> 💡 **ถัดไป:** `/nol:backlog` — ดู task ที่ยังรอ implement

**ถ้า user มี feedback** → อัปเดต ROOTCAUSE.md / RESEARCH.md / SOLUTION.md แล้ว repeat NW-8

---

## Execution Flow

```
NW-0 (self):   Read .nol/mistake/*.md + ตรวจ Reference ใน $ARGUMENTS
               + Read .nol/knowledge.md (or launch knowledge bg, no wait)
       ↓
NW-1 (self):   Create BUGFIX_DIR + Write BUG.md
       ↓
NW-2 (self):   Investigate — live evidence เท่านั้น (ห้ามอ่าน source code)
               ├─ Docker logs
               ├─ Chrome DevTools: reproduce, console errors, network requests
               ├─ Write RESEARCH.md (live evidence) + Update BUG.md
               └─ Write REPRODUCE.md (step-by-step reproduction guide)
       ↓
NW-3 (background, wait):  bug-research agent → ขุด codebase → RESEARCH.md (full)
       ↓
NW-4 (background, wait):  bug-rootcause agent → ROOTCAUSE.md
       ↓
NW-5 (background, wait):  bug-solution agent → SOLUTION.md
       ↓
NW-6 (background, wait):  test-manual agent → TEST_MANUAL.md
       ↓
NW-7 (self):   Write SUMMARY.md → Present to user
       ↓
NW-7.5 (background, no wait):  learn agent → .nol/mistake/YYYY-MM-DD.md
       ↓
NW-8:  Feedback loop
        ├─ Approved → Implement immediately
        └─ Changes  → Update files → Repeat NW-8
```

**Total agents: 4** — bug-research (NW-3) + bug-rootcause (NW-4) + bug-solution (NW-5) + test-manual (NW-6) — all background sequential + learn (NW-7.5, background no-wait)

---

## Important

- **DO NOT read source code files** during NW-2 Investigate — that's the research agent's job.
- **DO NOT implement** the fix until user explicitly approves.
- อ่าน Docker logs ก่อนเสมอ per AGENTS.md — ก่อนแก้โค้ดใดๆ
- Write in the same language the user used in the description.

---

Arguments: $ARGUMENTS
