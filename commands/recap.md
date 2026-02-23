---
description: "สรุป progress ของ feature, bugfix หรือ quick task"
---

# Recap - สรุปสิ่งที่ทำใน Feature, Bugfix หรือ Quick Task

ตรวจสอบ progress จาก **codebase จริงเสมอ** — ห้ามใช้ SUMMARY.md เป็นแหล่งข้อมูลสถานะ implementation

## Usage

```
/nol:recap feature <number>          เช่น /nol:recap feature 1
/nol:recap feature <name>            เช่น /nol:recap feature hscode
/nol:recap bugfix <number>           เช่น /nol:recap bugfix 2
/nol:recap bugfix <name>             เช่น /nol:recap bugfix sender-data
/nol:recap quick <number>            เช่น /nol:recap quick 1
/nol:recap quick <name>              เช่น /nol:recap quick email-validation
/nol:recap feature                   (แสดงรายการทั้งหมด)
/nol:recap bugfix                    (แสดงรายการทั้งหมด)
/nol:recap quick                     (แสดงรายการทั้งหมด)
/nol:recap                           (แสดงรายการทั้งหมดทั้ง feature, bugfix และ quick)
```

## AGENTS.md Compliance

> **CRITICAL: อ่านและปฏิบัติตาม `/Users/thananchai/Projects/wsol/AGENTS.md` ก่อนทำงานทุกครั้ง**
>
> กฎสำคัญที่ต้อง follow เสมอ:
> - **Debug:** ตรวจ Docker logs ก่อนเสมอ (`docker logs --tail 300 <container>`)
> - **Development:** ทำงานผ่าน Docker Compose เท่านั้น (env: `dev` หรือ `uat`)
> - **Testing:** รัน test ใน Docker container เท่านั้น — ห้ามรันบน host โดยตรง
> - **SQL:** ใส่ `WHERE TABLE_SCHEMA = '<database_name>'` ทุกครั้งที่ query `INFORMATION_SCHEMA`

## Instructions

Parse `$ARGUMENTS` เพื่อระบุ type และ query:
- Format: `feature <query>`, `bugfix <query>`, `quick <query>`, หรือพิมพ์แค่ type เดียว/ไม่พิมพ์อะไรเลย
- `<query>` อาจเป็นตัวเลข (เช่น `1`), ชื่อเต็ม (เช่น `add-hscode-sorting`), หรือคำบางส่วน (เช่น `hscode`, `login`)

---

### Case 1: ระบุ type และ query (เช่น `/recap feature 1`, `/recap quick 2`)

1. กำหนด base path ตาม type:
   - `feature` → `.nol/feature/`
   - `bugfix` → `.nol/bugfix/`
   - `quick` → `.nol/quick/`

2. ค้นหา directory ที่ match กับ query โดย **ลำดับความสำคัญ**:
   - ถ้า query เป็นตัวเลขล้วน → หา directory ที่ขึ้นต้นด้วย `{query}-` (เช่น `1-*`)
   - ถ้า query เป็น string → หา directory ที่ชื่อ **contains** query นั้น (case-insensitive)
   - ถ้าเจอหลาย directory → แสดงรายการที่ match ทั้งหมด ให้ user เลือก
   - ถ้าไม่เจอเลย → แจ้ง user และแสดงรายการทั้งหมดที่มี

3. อ่านไฟล์ plan ทั้งหมดพร้อมกัน:
   - **feature**: `FEATURE.md`, `IMPACT.md`, `SOLUTION.md`
   - **bugfix**: `BUG.md`, `IMPACT.md`, `SOLUTION.md`
   - **quick**: `REQUIREMENT.md`, `IMPACT.md`, `SOLUTION.md`
   - **ห้ามใช้ SUMMARY.md** เป็นแหล่งข้อมูลสถานะ — ต้องตรวจ codebase เท่านั้น

4. **ตรวจสอบ Progress จาก Codebase จริง** (สำคัญ — ทำทุกครั้ง ไม่มีข้อยกเว้น):

   4.1 ดึงรายการไฟล์ที่ต้องแก้จาก `IMPACT.md` (หัวข้อ "ต้องแก้ไข" หรือ "ไฟล์ที่ต้องแก้ไข")

   4.2 สำหรับแต่ละไฟล์ใน list — อ่านไฟล์จริงจาก codebase แล้วตรวจว่า implementation ที่ระบุใน `SOLUTION.md` ถูก apply ไปแล้วหรือยัง:
   - **✅ Done** — code ที่ SOLUTION.md บอกว่าต้องเพิ่ม/แก้มีอยู่ในไฟล์จริงแล้ว
   - **❌ Not yet** — ไฟล์ยังคงเป็น code เดิม ยังไม่มีการเปลี่ยนแปลง
   - **⚠️ Partial** — มีบางส่วนถูก implement แต่ยังไม่ครบ
   - **🆕 New file** — สำหรับไฟล์ที่ต้องสร้างใหม่ ให้เช็คว่าไฟล์นั้นมีอยู่จริงไหม
   - **➖ Skip** — ไฟล์ที่ plan ระบุว่า optional หรือ "ควรแก้" (ไม่ใช่ "ต้องแก้")

   4.3 สรุปเป็น **Implementation Status**:
   - จำนวนไฟล์ที่ Done / Not yet / Partial
   - คำนวณเปอร์เซ็นต์ progress (นับเฉพาะไฟล์ required)

   4.4 **อัปเดต `SUMMARY.md`** ให้สะท้อน progress ล่าสุด:
   - ถ้ายังไม่มี SUMMARY.md → สร้างใหม่
   - ถ้ามีแล้ว → อัปเดต section "Implementation Status" ให้ตรงกับ codebase จริง

5. **Present ให้ user** ในรูปแบบนี้:

   ```
   ## 📋 Recap: [ชื่อ feature/bugfix/quick]

   **Type:** Feature / Bugfix / Quick
   **Folder:** .nol/quick/1-xxx/

   ### สรุป
   [เนื้อหาจาก FEATURE.md / BUG.md / REQUIREMENT.md — เขียนให้กระชับ ไม่ใช่แค่ copy paste]

   ### Implementation Status — [X/Y ไฟล์ · Z%]

   | ไฟล์ | สถานะ | หมายเหตุ |
   |------|--------|---------|
   | `path/to/file.js` | ✅ Done | เพิ่ม validation แล้ว |
   | `path/to/file2.js` | ❌ Not yet | ยังไม่มีการเปลี่ยนแปลง |

   ### ไฟล์ Plan
   - REQUIREMENT.md · RESEARCH.md · IMPACT.md · SOLUTION.md · TEST_MANUAL.md

   💡 **ถัดไป:** `/nol:approve [type] [number]` — เช่น `/nol:approve quick 1`
   ```

6. **Feedback Loop — ถามทันทีหลัง present และวนไม่มีที่สิ้นสุดจนกว่า user จะบอกว่าไม่มี feedback แล้ว:**

   > "มี feedback อะไรเพิ่มเติมไหมครับ?"

   **ห้าม implement ในคำสั่ง recap** ไม่ว่ากรณีใด

   ---

   **ถ้า user ให้ feedback หรือขอปรับแก้แผน:**
   1. ระบุว่า feedback นั้นกระทบไฟล์ไหน:
      - คำอธิบาย task/bug/feature → แก้ `REQUIREMENT.md`, `BUG.md` หรือ `FEATURE.md`
      - Root cause / findings → แก้ `RESEARCH.md`
      - Impact / risk → แก้ `IMPACT.md`
      - วิธี fix / approach → แก้ `SOLUTION.md`
      - Test plan → แก้ `TEST_MANUAL.md`
   2. **แก้ไขทุกไฟล์ที่มีข้อมูลที่ต้องเปลี่ยน** — ไม่ใช่แค่ไฟล์หลัก ให้ grep หา reference ที่เกี่ยวข้องในทุกไฟล์ใน directory แล้วแก้ให้ครบ (เช่น ถ้าเปลี่ยน filename ต้องแก้ทุกไฟล์ที่อ้างถึง filename นั้น ทั้ง SOLUTION.md, IMPACT.md, TEST_MANUAL.md, SUMMARY.md)
   3. **แสดง "สิ่งที่แก้เพิ่มจาก feedback"** ก่อน present recap ใหม่:
      ```
      ### ✏️ สิ่งที่แก้เพิ่มจาก Feedback
      - [ไฟล์ที่แก้]: [สิ่งที่เปลี่ยนแปลง เช่น "เพิ่ม step การ rollback ใน SOLUTION.md"]
      - [ไฟล์ที่แก้]: [สิ่งที่เปลี่ยนแปลง]
      ```
   4. Launch `summary` agent (background) เพื่อ regenerate summary จากไฟล์ที่อัปเดต → รอผล
   5. Overwrite `{FOUND_DIR}/SUMMARY.md` ด้วย summary ใหม่ (รวม Implementation Status ล่าสุด)
   6. Present recap ใหม่ทั้งหมด (ตามรูปแบบ Step 5) → **วนกลับ Step 6** ทันที

   **ถ้า user ถามคำถาม ขอความเห็น หรือถามอะไรก็ตาม:**
   - ตอบให้ครบถ้วน
   - จากนั้น **วนกลับ Step 6 ทันที** — ถามว่า "มี feedback อะไรเพิ่มเติมไหมครับ?"

   **ถ้า user บอกว่าไม่มี feedback แล้ว หรืออยากจะ implement:**
   - ห้าม implement เอง
   - แจ้งว่า: "ถ้าพร้อม implement แล้วใช้คำสั่ง `/nol:approve [type] [number]` ได้เลยครับ เช่น `/nol:approve quick 1`"
   - จบ loop

---

### Case 2: ระบุแค่ type (เช่น `/recap feature`) หรือไม่ระบุอะไร

1. ค้นหา directories ทั้งหมดใน path ที่เกี่ยวข้อง:
   - `feature` only → `.nol/feature/`
   - `bugfix` only → `.nol/bugfix/`
   - `quick` only → `.nol/quick/`
   - ไม่ระบุ → ทั้ง `.nol/feature/`, `.nol/bugfix/` และ `.nol/quick/`

2. สำหรับแต่ละ directory ให้แสดง:
   - เลขและชื่อ
   - มี SUMMARY.md หรือไม่
   - ถ้ามี → อ่าน 1-2 บรรทัดแรกของส่วนเนื้อหา (ข้าม header) มาแสดงเป็น preview
   - ถ้ามี IMPACT.md → อ่านและนับแบบเร็วว่า required files กี่ไฟล์ (ไม่ต้องเช็ค codebase ในโหมดนี้)

3. **Present รายการ** ในรูปแบบนี้:

   ```
   ## 📂 รายการทั้งหมด

   ### Features
   | # | ชื่อ | SUMMARY.md | Preview |
   |---|------|------------|---------|
   | 1 | add-hscode-sorting | ✅ | เพิ่ม sorting ให้ตาราง HS Code... |
   | 2 | user-login | ❌ | — |

   ### Bugfixes
   | # | ชื่อ | SUMMARY.md | Preview |
   |---|------|------------|---------|
   | 1 | sender-data-cleared | ✅ | ข้อมูล sender ถูกเคลียร์เมื่อ... |

   ### Quick Tasks
   | # | ชื่อ | SUMMARY.md | Preview |
   |---|------|------------|---------|
   | 1 | email-validation | ✅ | เพิ่ม validation ตรวจ email... |
   | 2 | refactor-total | ❌ | — |

   พิมพ์ `/recap quick 1` เพื่อดูรายละเอียดและ implementation status
   ```

---

## Important

- **ห้ามใช้ SUMMARY.md เป็นแหล่งข้อมูล implementation status** — ต้องตรวจ codebase โดยตรงทุกครั้งโดยไม่มีข้อยกเว้น แม้จะ recap ซ้ำหลายรอบก็ตาม
- เมื่อตรวจ progress ให้อ่านไฟล์จาก codebase และเปรียบเทียบกับ code ใน SOLUTION.md อย่างละเอียด
- ใช้ `FEATURE.md`/`BUG.md`/`REQUIREMENT.md` เป็นแหล่งสรุปเนื้อหา (ไม่ใช่ SUMMARY.md)
- สรุปให้กระชับ อ่านง่าย ไม่ต้อง copy paste ทุกบรรทัด
- ใช้ภาษาเดียวกับที่ user ใช้ใน description ของ task นั้น
- ถ้าไม่มี directory `.nol/` เลย ให้แจ้ง user ว่ายังไม่มี feature, bugfix หรือ quick task ใดถูก plan ไว้
- **อัปเดต SUMMARY.md ทุกครั้ง** หลังตรวจ codebase เพื่อให้ status เป็นปัจจุบันเสมอ

---

Arguments: $ARGUMENTS
