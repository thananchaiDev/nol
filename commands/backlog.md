---
description: "แสดง backlog รายการ feature/bugfix/quick ที่รอ implement"
---

# Backlog - แสดงรายการงานที่ plan แล้วแต่ยังไม่ได้ implement

สแกน `.nol/feature/`, `.nol/bugfix/` และ `.nol/quick/` เพื่อหา item ที่มี plan แต่ยังรอ implement

## Usage

```
/nol:backlog              แสดง backlog ทั้งหมด (feature + bugfix + quick)
/nol:backlog feature      แสดงเฉพาะ feature backlog
/nol:backlog bugfix       แสดงเฉพาะ bugfix backlog
/nol:backlog quick        แสดงเฉพาะ quick task backlog
```

## Instructions

Parse `$ARGUMENTS` เพื่อกำหนด scope:
- ไม่มี argument หรือ `all` → สแกนทั้ง feature, bugfix และ quick
- `feature` → สแกนเฉพาะ `.nol/feature/`
- `bugfix` → สแกนเฉพาะ `.nol/bugfix/`
- `quick` → สแกนเฉพาะ `.nol/quick/`

### Step 0: อ่าน Mistake Files

ก่อนเริ่มทำงาน:

1. ใช้ Glob tool หา `.nol/mistake/*.md`
2. ถ้ามี → Read ทุกไฟล์ และนำมาเป็น context เพื่อ avoid ข้อผิดพลาดซ้ำ
3. ถ้าไม่มี → ข้ามไปเงียบๆ

---

### Step 1: สแกน directories

ตรวจ directory ตาม scope:
- Feature: `.nol/feature/*/`
- Bugfix: `.nol/bugfix/*/`
- Quick: `.nol/quick/*/`

ถ้าไม่มี `.nol/` เลย → แจ้ง user ว่า "ยังไม่มี feature, bugfix หรือ quick task ใดถูก plan ไว้" แล้วหยุด

---

### Step 2: จำแนกสถานะแต่ละ item

สำหรับแต่ละ directory ให้ตรวจสถานะตามลำดับนี้:

**✅ Implemented (เสร็จแล้ว — ข้ามไป ไม่แสดงใน backlog)**
- มี `SUMMARY.md` **และ** SUMMARY.md มี section `## Implementation Report`
- (สำหรับ feature: ตรวจด้วยว่าไม่มี `❌ Open` ใน section `## Known Bugs`)

**🐛 Has Open Bugs (implement แล้ว แต่ยังมี bug ค้างอยู่)**
- มี `SUMMARY.md` + มี `## Implementation Report`
- แต่ใน section `## Known Bugs` ยังมี row ที่ status = `❌ Open`
- แสดงรายการ bug ที่ยังเปิดอยู่ในแต่ละ row

**📋 Ready to Implement (plan ครบแล้ว รอ approve)**
- มี `SUMMARY.md` แต่ **ไม่มี** section `## Implementation Report`
- มี `SOLUTION.md` อยู่ด้วย

**🔄 Planning in Progress (plan ยังไม่สมบูรณ์)**
- ไม่มี `SUMMARY.md` แต่มีอย่างน้อย `FEATURE.md`, `BUG.md` หรือ `REQUIREMENT.md`

**⚠️ Incomplete (ข้อมูลไม่พอ)**
- directory ว่างเปล่า หรือไม่มีไฟล์ที่คาดหวัง

---

### Step 3: อ่าน summary preview

สำหรับ item ที่มีสถานะ **📋 Ready** หรือ **🔄 Planning**:
- ถ้ามี `SUMMARY.md` → อ่าน 2-3 บรรทัดแรกของส่วนเนื้อหา (ข้าม header) มาใช้เป็น preview
- ถ้าไม่มี `SUMMARY.md`:
  - feature → อ่าน `FEATURE.md` description 1-2 บรรทัดแรก
  - bugfix → อ่าน `BUG.md` description 1-2 บรรทัดแรก
  - quick → อ่าน `REQUIREMENT.md` description 1-2 บรรทัดแรก

---

### Step 4: Present Backlog

แสดงผลลัพธ์ในรูปแบบตารางรวม:

```
## 📌 Backlog

| Type | # | ชื่อ | สถานะ | Preview |
|------|---|------|-------|---------|
| Feature | 2 | edit-item-button | 📋 Ready | เพิ่มปุ่มแก้ไข inline... |
| Feature | 3 | user-auth | 🔄 Planning | ระบบ login สำหรับ admin... |
| Feature | 1 | add-feature | 🐛 Open Bugs (1) | มี bug ค้างอยู่ใน... |
| Bugfix | 1 | sender-data-cleared | 📋 Ready | ข้อมูล sender ถูกเคลียร์เมื่อ... |
| Quick | 1 | email-validation | 📋 Ready | เพิ่ม validation ตรวจ email... |
| Quick | 2 | refactor-total | 🔄 Planning | refactor function calculateTotal... |

#### 🐛 Open Bugs ใน Features (ถ้ามี)
| Feature | Bug # | คำอธิบาย | Folder |
|---------|-------|----------|--------|
| 1-add-feature | 1 | ปุ่ม submit ไม่ทำงาน | .nol/feature/1-add-feature/bugfix/1-submit-broken/ |

---
📋 Ready to implement: {total} รายการ
🔄 Planning in progress: {total} รายการ
🐛 Open bugs in features: {total} รายการ

ใช้ `/nol:approve feature 2` หรือ `/nol:approve bugfix 1` หรือ `/nol:approve quick 1` เพื่อ implement
ใช้ `/nol:approve feature 1 bugfix 1` เพื่อ implement bugfix ภายใน feature
ใช้ `/nol:recap quick 1` เพื่อดูรายละเอียดก่อน approve
```

- เรียง Features ก่อน แล้วตามด้วย Bugfixes แล้วตามด้วย Quick Tasks
- ถ้าไม่มี Open Bugs ใน Features ให้ข้ามส่วน `🐛 Open Bugs ใน Features`

ถ้าไม่มี backlog เลย (ทุก item เสร็จแล้ว) ให้แสดง:
```
✅ ไม่มี backlog — ทุก plan ได้รับการ implement แล้ว
```

---

### Step 5: Prompt ให้ user ดำเนินการต่อ

หลัง present backlog แล้ว ถามว่า:

> "อยากดูรายละเอียดของ item ไหน หรือ approve ให้ implement เลยไหมครับ?"

**ถ้า user ระบุ item ที่ต้องการดู** → อ่าน SUMMARY.md ของ item นั้นแล้วสรุปเหมือน `/recap`
**ถ้า user ต้องการ approve** → แจ้งให้ใช้คำสั่ง `/nol:approve [type] [number]`
**ถ้า user ไม่ต้องการอะไรเพิ่ม** → จบ

---

## Important

- **แสดงเฉพาะ item ที่ยังไม่เสร็จ** — item ที่มี `## Implementation Report` ใน SUMMARY.md ให้ข้ามไปเงียบๆ
- ตรวจไฟล์จริงด้วย Glob และ Read — ห้ามเดาสถานะ
- ใช้ภาษาไทยในการแสดงผล (ยกเว้นชื่อไฟล์และ path)

---

Arguments: $ARGUMENTS
