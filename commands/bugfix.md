# Bugfix - Analyze & Plan a Bug Fix

Debug ทันทีใน foreground ใช้ systematic-debugging skill — ไม่ใช้ background agents

## Usage

```
/bugfix [description of the bug]
```

**ตัวอย่าง:**
```
/bugfix ปุ่ม reset ไม่ล้างรูป
/bugfix login error on mobile
/bugfix ตรงนี้ยังไม่ผ่าน
/bugfix feature 1 ปุ่ม submit ไม่ทำงานใน step 3
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

ใช้ `$ARGUMENTS` ทั้งหมดเป็น bug description — **ทำงานใน foreground ทั้งหมด ไม่ใช้ background sub-agents**

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
6. Report ให้ user รู้ว่ากำลังจะ debug ทันที: `"เริ่ม debug: {bug description}"`

---

### NW-2: Apply Systematic Debugging Skill

**Invoke the `systematic-debugging` skill** โดยทำงานใน foreground:

```
Use Skill tool: systematic-debugging
```

ระหว่าง investigation ให้ปฏิบัติตาม systematic-debugging methodology ครบทุก phase:

**Phase 1 — Root Cause Investigation:**
- อ่าน Docker logs ก่อนเสมอ (per AGENTS.md): `docker logs --tail 300 <container>`
- ตรวจ error messages, stack traces
- Reproduce ให้ได้ก่อน
- ตรวจ git history ล่าสุด
- Trace data flow จาก entry point ถึง failure point

**Phase 2 — Pattern Analysis:**
- หา working examples ใน codebase เดียวกัน
- Compare working vs broken

**Phase 3 — Hypothesis:**
- ตั้ง hypothesis ที่ชัดเจน: "Root cause คือ X เพราะ Y"
- Test minimally

**ทุกครั้งที่เจอข้อมูลสำคัญ** → append ลงใน `{BUGFIX_DIR}/RESEARCH.md` ทันที:
```markdown
# Bug Research: {bug description}

## Root Cause Hypothesis
(update เมื่อมี hypothesis ที่ชัดเจน)

## Evidence
- [file:line] {สิ่งที่เจอ}

## Data Flow Trace
{trace จาก entry point → failure}

## Pattern Comparison
{working vs broken}

## Key Files Involved
| File | Role |
|------|------|
| ... | ... |

## Open Questions
(ถ้ายังมีสิ่งที่ไม่แน่ใจ)
```

---

### NW-3: Live Reproduction ด้วย Chrome DevTools

ก่อนอื่นให้ **ตรวจสอบว่า bug เป็น frontend หรือ backend** จาก bug description และ RESEARCH.md:

- **Frontend** = ปัญหาเกี่ยวกับ UI, rendering, button, form, animation, state ที่แสดงผล
- **Backend** = ปัญหาเกี่ยวกับ API response, database, logic ฝั่ง server, validation

#### ถ้าเป็น Frontend Bug:

1. ใช้ `mcp__chrome-devtools__list_pages` เพื่อดูว่ามี browser เปิดอยู่ไหม
2. ถ้ามี → `mcp__chrome-devtools__select_page` เลือก page ที่เกี่ยวข้อง
3. Navigate ไปยังหน้าที่มีปัญหา (`mcp__chrome-devtools__navigate_page`)
4. พยายาม **reproduce bug จริงๆ** บน browser:
   - ใช้ `mcp__chrome-devtools__take_snapshot` ดู state ปัจจุบันของ UI
   - ใช้ `mcp__chrome-devtools__click`, `mcp__chrome-devtools__fill` เพื่อทำ action ที่ trigger bug
   - ตรวจ console errors: `mcp__chrome-devtools__list_console_messages` (filter `types: ["error", "warn"]`)
   - ตรวจ network requests ที่เกี่ยวข้อง: `mcp__chrome-devtools__list_network_requests`
5. `mcp__chrome-devtools__take_screenshot` บันทึก visual evidence
6. **บันทึก findings** ลงใน `{BUGFIX_DIR}/RESEARCH.md` ในส่วน `## Live Reproduction` (append ต่อท้าย)

#### ถ้าเป็น Backend Bug:

1. ใช้ `mcp__chrome-devtools__list_pages` เพื่อดูว่ามี browser เปิดอยู่ไหม
2. ถ้ามี → `mcp__chrome-devtools__select_page` เลือก page ที่เกี่ยวข้อง
3. **Login ผ่าน browser** (ถ้ายังไม่ได้ login):
   - Navigate ไปหน้า login ของ app
   - กรอก credentials และ submit
   - รอจน login สำเร็จ
4. **หา network request ที่เกี่ยวกับ bug**:
   - Navigate ไปหน้าหรือ action ที่ trigger bug
   - ใช้ `mcp__chrome-devtools__list_network_requests` (filter `resourceTypes: ["fetch", "xhr"]`)
   - ใช้ `mcp__chrome-devtools__get_network_request` ดู request/response body ของ request ที่เกี่ยวข้อง
5. **สร้าง curl command** จาก request นั้น (รวม headers, cookies, body)
6. **รัน curl จริงๆ** ผ่าน Bash tool เพื่อดู response ตรงๆ
7. **บันทึก findings** ลงใน `{BUGFIX_DIR}/RESEARCH.md` ในส่วน `## Live Reproduction` (append ต่อท้าย)

#### ถ้าไม่สามารถ reproduce ได้ (browser ไม่เปิด, app ไม่ running, etc.):

- ข้ามขั้นตอนนี้ไปเงียบๆ

---

### NW-4: เขียน SOLUTION.md โดยตรง

หลังจาก root cause ชัดเจนแล้ว เขียน `{BUGFIX_DIR}/SOLUTION.md` ด้วย Write tool:
```markdown
# Fix Solution: {bug description}

## Root Cause (Confirmed)
{root cause ที่พิสูจน์แล้ว}

## Implementation Steps
1. {step แรก — file:line ที่ต้องแก้}
2. {step ต่อไป}
...

## Files to Change
| File | Change |
|------|--------|
| ... | ... |

## Test Plan
- [ ] {criteria 1}
- [ ] {criteria 2}
```

---

### NW-5: เขียน SUMMARY.md และ Present

เขียน `{BUGFIX_DIR}/SUMMARY.md` สรุปสั้นๆ แล้ว present ให้ user:
- Root cause ที่เจอ
- Fix approach
- Files ที่จะแก้
- Links ไปทุกไฟล์ (BUG.md, RESEARCH.md, SOLUTION.md)

---

### NW-6: Feedback Loop

> "debug เสร็จแล้ว — root cause คือ [X] อยากให้ implement ทันทีหรือปรับ approach ก่อน?"

**ถ้า user อนุมัติ** → implement ตาม SOLUTION.md ทันที (Phase 4 ของ systematic-debugging)

**ถ้า user มี feedback** → อัปเดต RESEARCH.md / SOLUTION.md แล้ว repeat NW-6

---

## Execution Flow

```
NW-1 (self):  Create BUGFIX_DIR + Write BUG.md
       ↓
NW-2 (self):  Apply systematic-debugging skill in foreground
               ├─ Phase 1: Root cause investigation (Docker logs → trace → evidence)
               ├─ Phase 2: Pattern analysis
               └─ Phase 3: Hypothesis → test minimally
               → Write RESEARCH.md as findings are discovered (append continuously)
       ↓
NW-3 (self):  Live Reproduction via Chrome DevTools
               → append findings to RESEARCH.md
       ↓
NW-4 (self):  Write SOLUTION.md directly (after root cause confirmed)
       ↓
NW-5 (self):  Write SUMMARY.md → Present to user
       ↓
NW-6:  Feedback loop
        ├─ Approved → Implement immediately (Phase 4 of systematic-debugging)
        └─ Changes  → Update RESEARCH.md / SOLUTION.md → Repeat NW-6
```

**Total agents: 0** — ทำทุกอย่างใน foreground ไม่มี background sub-agents

---

## Important

- **DO NOT implement** the fix until user explicitly approves.
- อ่าน Docker logs ก่อนเสมอ per AGENTS.md — ห้ามแก้โค้ดก่อนตรวจ logs
- Write in the same language the user used in the description.

---

Arguments: $ARGUMENTS
