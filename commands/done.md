---
description: "mark feature/bugfix/quick ว่า implement แล้ว เมื่อถูก implement นอก nol pipeline"
---

# Done - Mark as Implemented

ใช้เมื่อ feature/bugfix/quick ถูก implement โดยตรง (นอก nol pipeline) แต่ยังไม่มี `## Implementation Report` ใน SUMMARY.md ทำให้ `/nol:backlog` track สถานะผิด

## Usage

```
/nol:done bugfix <number|name>    mark bugfix ว่า implement แล้ว
/nol:done feature <number|name>   mark feature ว่า implement แล้ว
/nol:done quick <number|name>     mark quick task ว่า implement แล้ว
```

**ตัวอย่าง:**
```
/nol:done bugfix 3
/nol:done feature 1
/nol:done quick email-validation
```

## Instructions

Parse `$ARGUMENTS` เพื่อแยก type (`feature` / `bugfix` / `quick`) และ query (ตัวเลขหรือชื่อ)

---

### Step 1: Locate Target Directory

1. กำหนด base path ตาม type:
   - `feature` → `.nol/feature/`
   - `bugfix` → `.nol/bugfix/`
   - `quick` → `.nol/quick/`

2. ค้นหา directory ที่ match กับ query:
   - ถ้า query เป็นตัวเลขล้วน → หา directory ที่ขึ้นต้นด้วย `{query}-`
   - ถ้า query เป็น string → หา directory ที่ชื่อ **contains** query นั้น (case-insensitive)
   - ถ้าเจอหลาย directory → แสดงรายการ ให้ user เลือกก่อน แล้วหยุดรอ
   - ถ้าไม่เจอเลย → แจ้ง user และแสดงรายการทั้งหมดที่มี แล้วหยุด

3. ตั้งค่า `TARGET_DIR` = directory ที่เจอ

---

### Step 2: ตรวจสถานะปัจจุบัน

อ่าน `{TARGET_DIR}/SUMMARY.md` (ถ้ามี) แล้วตรวจว่ามี section `## Implementation Report` แล้วหรือยัง:

- **ถ้ามีแล้ว** → แจ้ง user ว่า item นี้ถูก mark เป็น done แล้ว แล้วหยุด
- **ถ้ายังไม่มี** → ดำเนินการต่อ

---

### Step 3: อ่านข้อมูลสำหรับสร้าง report

อ่านไฟล์เหล่านี้จาก `{TARGET_DIR}` เพื่อนำมาสรุปใน Implementation Report:

- ไฟล์หลักตาม type:
  - `bugfix` → `BUG.md`
  - `feature` → `FEATURE.md`
  - `quick` → `REQUIREMENT.md`
- `SOLUTION.md` (ถ้ามี) — อ่านเพื่อดู implementation plan ที่วางไว้
- `SUMMARY.md` (ถ้ามี) — อ่านเพื่อ append ต่อท้าย

---

### Step 4: Cross-reference กับ git log

1. รัน `git log --oneline -30` เพื่อดู recent commits
2. ดึง keywords จากชื่อ directory (เช่น `3-chart-wording-wrong` → keywords: `chart`, `wording`)
   - ตัดคำสั้นที่ไม่มีความหมาย (เช่น ตัวเลข, `-`, `quick`, `bugfix`, `feature`) ออก
3. หา commits ที่ message มี keyword ที่ตรงกัน (case-insensitive)
4. บันทึก commits ที่ match เพื่อใส่ใน report

---

### Step 5: สร้างหรืออัปเดต SUMMARY.md

#### กรณี SUMMARY.md ยังไม่มี

สร้างไฟล์ `{TARGET_DIR}/SUMMARY.md` ใหม่ โดยดึง title/description จากไฟล์หลัก:

```markdown
# Summary: {ชื่อ directory}

> {description 1-2 บรรทัดแรกจากไฟล์หลัก (BUG.md / FEATURE.md / REQUIREMENT.md)}

---

## Implementation Report
...
```

#### กรณี SUMMARY.md มีอยู่แล้ว (แต่ไม่มี Implementation Report)

Append section ต่อท้าย:

```markdown

---

## Implementation Report
...
```

#### เนื้อหา Implementation Report (ทั้ง 2 กรณี)

```markdown
## Implementation Report

**Date:** {วันที่ปัจจุบัน}
**Status:** ✅ Completed (marked via `/nol:done`)
**Note:** Implemented outside nol pipeline — no automated test results recorded

### Related Commits
| Commit | Message |
|--------|---------|
| {hash} | {message} |
```

_(ถ้าไม่เจอ commit ที่ match → ระบุ "ไม่พบ commit ที่ตรงกับชื่อ item นี้ — ตรวจสอบด้วย `git log` ด้วยตนเอง")_

```markdown
### Files Changed
_(ไม่ทราบ — implement นอก pipeline; ตรวจสอบจาก git diff หรือ commit ข้างต้น)_

### Test Results
_(ไม่ทราบ — implement นอก pipeline; ควร verify ด้วยตนเองก่อน deploy)_
```

---

### Step 6: Done

แจ้ง user สรุปผล:

```
✅ Marked as Done: {ชื่อ directory}

- SUMMARY.md: อัปเดตแล้วที่ {TARGET_DIR}/SUMMARY.md
- item นี้จะไม่ปรากฏใน /nol:backlog อีกต่อไป
{ถ้าเจอ related commits → "Related commits: {list}"}
{ถ้าไม่เจอ → "⚠️ ไม่พบ commit ที่ match — ตรวจสอบ git log ด้วยตนเอง"}

💡 ถัดไป: `/nol:backlog` — ดู task ที่ยังรออยู่
```

---

## Important

- ถ้า `## Implementation Report` มีอยู่แล้ว → ห้าม overwrite ให้แจ้ง user และหยุด
- แก้ไขเฉพาะ `SUMMARY.md` เท่านั้น — ห้ามแตะไฟล์อื่นใน `{TARGET_DIR}`
- ห้าม implement code — command นี้ทำแค่ mark สถานะเท่านั้น
- ถ้าไม่มีไฟล์หลัก (BUG.md / FEATURE.md / REQUIREMENT.md) → แจ้ง user และหยุด (directory อาจไม่ใช่ nol item)

---

Arguments: $ARGUMENTS
