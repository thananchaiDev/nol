---
description: "อนุมัติ plan และ implement feature, bugfix หรือ quick task"
---

# Approve - อนุมัติ Plan และ Implement

อ่าน SOLUTION.md จาก feature/bugfix/quick folder แล้ว implement ตาม plan จากนั้นเทสตาม TEST_MANUAL.md จนผ่านทั้งหมด แล้วอัปเดต SUMMARY.md

## Usage

```
/nol:approve feature <number|name>                    อนุมัติ feature implementation
/nol:approve bugfix <number|name>                     อนุมัติ standalone bugfix
/nol:approve quick <number|name>                      อนุมัติ quick task implementation
/nol:approve feature <number|name> bugfix <number|name>   อนุมัติ bugfix ภายใน feature
```

**ตัวอย่าง:**
```
/nol:approve feature 1
/nol:approve bugfix 2
/nol:approve quick 1
/nol:approve quick email-validation
/nol:approve feature 1 bugfix 1
/nol:approve feature add-hscode bugfix submit-broken
```

## AGENTS.md Compliance

> **CRITICAL: อ่านและปฏิบัติตาม `/Users/thananchai/Projects/wsol/AGENTS.md` ก่อนทำงานทุกครั้ง**
>
> กฎสำคัญที่ต้อง follow เสมอ:
> - **Debug:** ตรวจ Docker logs ก่อนเสมอ (`docker logs --tail 300 <container>`) — ก่อน debug หรือแก้โค้ดใดๆ
> - **Development:** ทำงานผ่าน Docker Compose เท่านั้น (env: `dev` หรือ `uat`)
> - **Testing:** รัน test ใน Docker container เท่านั้น — ห้ามรันบน host โดยตรง; ใช้ config จาก `.mocharc.json`
> - **SQL:** ใส่ `WHERE TABLE_SCHEMA = '<database_name>'` ทุกครั้งที่ query `INFORMATION_SCHEMA`

## Instructions

Parse `$ARGUMENTS` เพื่อตรวจว่าเป็น mode ไหน:

### Mode Detection

- **Feature mode**: `feature <query>` — approve feature implementation
- **Bugfix mode**: `bugfix <query>` — approve standalone bugfix
- **Quick mode**: `quick <query>` — approve quick task implementation
- **Feature Bugfix mode**: `feature <query> bugfix <query>` — approve bugfix ภายใน feature

---

### Step 1: Locate Target Directory

#### กรณี Feature mode / Bugfix mode / Quick mode

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
4. ตั้งค่า `FEATURE_DIR` = ไม่มี (ไม่ใช่ feature bugfix mode)

#### กรณี Feature Bugfix mode

1. แยก feature query และ bugfix query ออกจาก arguments
2. หา feature directory:
   - ค้นหาใน `.nol/feature/` เหมือนข้างบน
   - ตั้งค่า `FEATURE_DIR` = directory ที่เจอ เช่น `.nol/feature/1-add-hscode-sorting/`
3. หา bugfix directory ภายใน feature:
   - ค้นหาใน `{FEATURE_DIR}/bugfix/` โดยใช้ bugfix query
   - ถ้าไม่เจอ → แจ้ง user และแสดงรายการ bugfix ทั้งหมดใน feature นั้น แล้วหยุด
4. ตั้งค่า `TARGET_DIR` = `{FEATURE_DIR}/bugfix/{matched-bugfix-dir}/`

---

### Step 2: Read Plan Files

อ่านไฟล์เหล่านี้จาก `{TARGET_DIR}` ก่อน implement:

- `SOLUTION.md` — implementation steps (บังคับ ถ้าไม่มีให้หยุดและแจ้ง user)
- `TEST_MANUAL.md` — test cases และ acceptance criteria (บังคับ ถ้าไม่มีให้แจ้ง user แต่ยังดำเนินการต่อได้)
- `SUMMARY.md` — อ่านเพื่อเตรียม update ภายหลัง

แจ้ง user ว่ากำลังจะ implement plan จาก `{TARGET_DIR}/SOLUTION.md`

---

### Step 3: Implementation Loop

**ติดตามความคืบหน้าด้วย TodoWrite ตลอดเวลา**

ทำตาม steps ใน `SOLUTION.md` ทีละ step:

1. อ่าน step ปัจจุบัน — เข้าใจให้ชัดก่อนแก้ไข
2. อ่านไฟล์ที่เกี่ยวข้องจริง (ไม่เดา)
3. Implement การเปลี่ยนแปลง
4. Mark todo เป็น complete
5. ทำ step ถัดไปจนครบทุก step ใน SOLUTION.md

**บันทึกระหว่าง implement:**
- ไฟล์ที่แก้ไข (path + สิ่งที่เปลี่ยน)
- ปัญหาที่เจอและวิธีแก้
- สิ่งที่ต้องตัดสินใจ (decisions made)

---

### Step 4: Verification Loop

หลัง implement ครบแล้ว ให้เทสตาม `TEST_MANUAL.md` **ทุก test case**

สำหรับแต่ละ test case:

1. อ่าน test case และ acceptance criteria
2. รัน test ตาม AGENTS.md (Docker containers, mocha, ฯลฯ)
3. บันทึกผลลัพธ์: ผ่าน ✅ หรือไม่ผ่าน ❌

**ถ้า test ไม่ผ่าน → วน loop จนกว่าจะผ่านทั้งหมด:**

```
ไม่ผ่าน → วิเคราะห์สาเหตุ → แก้ไขโค้ด → รัน test ใหม่ → ทำซ้ำ
```

- ห้ามออกจาก loop จนกว่า **ทุก test case ผ่าน**
- ถ้าแก้แล้วยังไม่ผ่านหลายรอบ → แจ้ง user พร้อม error detail และรอ input

**ถ้าไม่มี `TEST_MANUAL.md`:**
- รัน test suite หลักของ project แทน (ตาม AGENTS.md)
- ถ้าไม่มี test → แจ้ง user และให้ manual verify ก่อน approve

---

### Step 5: Update SUMMARY.md

เมื่อทุก test ผ่านแล้ว ให้ **append section ใหม่** ต่อท้าย `{TARGET_DIR}/SUMMARY.md`:

```markdown
---

## Implementation Report

**Date:** {วันที่ implement}
**Status:** ✅ Completed

### Files Changed
| File | Change |
|------|--------|
| path/to/file.js | เพิ่ม function X, แก้ logic Y |
| path/to/file2.js | อัปเดต schema Z |

### Decisions Made
- [ถ้ามี] decision ที่ตัดสินใจระหว่าง implement เช่น เลือก approach A แทน B เพราะ...

### Issues Encountered
- [ถ้ามี] ปัญหาที่เจอระหว่าง implement และวิธีที่แก้

### Test Results
| Test Case | Result |
|-----------|--------|
| [ชื่อ test] | ✅ Pass |
| [ชื่อ test] | ✅ Pass |

**All {n} tests passed.**
```

ถ้าไม่มีปัญหาหรือ decision พิเศษ ให้ระบุว่า "ไม่มี" แทนที่จะเว้นว่าง

---

### Step 5.5: อัปเดต Feature SUMMARY.md (Feature Bugfix mode เท่านั้น)

**ทำขั้นตอนนี้เฉพาะเมื่อ `FEATURE_DIR` ถูกตั้งค่าไว้** (คือเป็น Feature Bugfix mode)

อัปเดต `{FEATURE_DIR}/SUMMARY.md` เพื่อ mark bug นี้ว่า fixed:

1. อ่าน `{FEATURE_DIR}/SUMMARY.md`
2. หา row ใน section `## Known Bugs` ที่ตรงกับ bugfix directory นี้
3. เปลี่ยน `❌ Open` → `✅ Fixed` ใน column Status ของ row นั้น

ตัวอย่าง: เปลี่ยน
```
| 1 | ปุ่ม submit ไม่ทำงาน | ❌ Open | bugfix/1-submit-broken/ |
```
เป็น
```
| 1 | ปุ่ม submit ไม่ทำงาน | ✅ Fixed | bugfix/1-submit-broken/ |
```

ถ้าไม่มี section `## Known Bugs` หรือหา row ไม่เจอ → ข้ามไปเงียบๆ

---

### Step 6: Done

แจ้ง user สรุปผล:

```
✅ Approved & Implemented: [ชื่อ feature/bugfix/quick]

- Implemented: {n} steps ตาม SOLUTION.md
- Tests: {n}/{n} passed
- SUMMARY.md: อัปเดตแล้ว

ดูรายละเอียดได้ที่ {TARGET_DIR}/SUMMARY.md
```

---

## Important

- **ห้ามข้าม test** — ทุก test case ใน TEST_MANUAL.md ต้องผ่านจริงก่อนออก loop
- ถ้าไม่มี `SOLUTION.md` ให้หยุดทันที ไม่ implement เดา
- ปฏิบัติตาม AGENTS.md อย่างเคร่งครัด (Docker logs ก่อน debug, รัน test ใน container, ฯลฯ)
- บันทึกทุกการเปลี่ยนแปลงลง SUMMARY.md — ไม่ว่าจะเล็กหรือใหญ่

---

Arguments: $ARGUMENTS
