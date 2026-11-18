---
name: work-task
description: ทำ Jira task ตั้งแต่หยิบงาน วางแผน เขียนโค้ด ตรวจ review จนอัปเดต Jira โดยกระจายงานให้ agent (jira-sync, planner, contract-dev, backend-dev, frontend-dev แบบขนาน, verifier, qa-tester, reviewer, scribe) ใช้เมื่อผู้ใช้สั่งให้ทำ task, ทำงานถัดไป หรือทำต่อจากคนอื่น
argument-hint: "<KEY> | next | all"
---

# /work-task $ARGUMENTS

ทำตาม flow ใน `CLAUDE.md` ทีละขั้น ห้ามข้ามขั้น ค่า Jira (key, label) อยู่ใน `CLAUDE.md`
หลักการ: session หลักเขียนโค้ดและตัดสินใจ — งานอ่าน/รัน/ซิงก์ที่ให้ผลยาวส่งให้ agent และรับกลับแค่สรุป

## 0. เลือก task

- `$ARGUMENTS` เป็น key → ใช้ key นั้น
- `next` หรือว่าง → `jira-sync next`
- `all` → ทำแบบ `next` วนจนไม่มีงานพร้อม แล้วสรุปรวม
- `NONE READY` → รายงานว่ารออะไรอยู่ แล้วหยุด

## 1. เริ่มงาน

1. ถ้ามี `docs/tasks/<KEY>.md` → อ่าน Log ก่อน แล้วทำต่อจากจุดที่ค้าง (ข้ามขั้นที่ทำแล้ว)
2. `jira-sync start <KEY>` → ได้ AC แบบย่อ

## 2. วางแผน

- เรียก `planner` ส่ง key + AC แบบย่อ (ไม่ต้อง copy rules ไปใน prompt)
- ผลขึ้นต้น `BLOCKED:` → `jira-sync comment <KEY> <คำถาม>`, Log "รอคำตอบ" แล้วหยุด
- อ่านแผนเฉพาะ Files / Steps / Test plan

## 3. เขียนโค้ด

ดูหัวข้อ Work packages ในแผน

### 3A. `single` — session หลักทำเอง

- ต้องการรู้ว่าอะไรอยู่ไหนหรือมีของให้ใช้ซ้ำไหม → `scout` (อย่าเปิดไล่หลายไฟล์เอง)
- ทำ Steps ทีละขั้น; logic ใหม่เขียน test ก่อนหรือพร้อมกัน
- จบแต่ละ step → `verifier quick <package>` แล้ว `scribe log`
- ต้องออกนอกแผน → แก้แผนก่อนและจดเหตุผลใน Log

### 3B. แบ่ง package — แยกให้ agent ทำ (subagent เรียก subagent ไม่ได้ session หลักจึงเป็นผู้ประสานเสมอ)

```
root (session หลัก) → contract-dev → ┬ backend-dev  ┐ → qa-tester
                                     └ frontend-dev ┘   (ถ้ามี package qa)
```

1. **root** — session หลักเพิ่ม dependency / แก้ root config ตาม package `root` แล้ว `pnpm install` ครั้งเดียว
2. **contract** — เรียก `contract-dev` (ส่ง key + path task file) รอจน `STATUS: DONE`
3. **backend + frontend พร้อมกัน** — เรียก `backend-dev` และ `frontend-dev` ใน message เดียว (Agent สองตัวขนานกัน)
   prompt ของแต่ละตัวมีแค่: key, path task file, ชื่อ package, และ `EXPORTS` / `ERROR CODES` จาก contract-dev ทั้งก้อน
   ไม่ต้อง copy AC, rules หรือโค้ดไปใน prompt — agent อ่านจาก task file และ rule เอง
4. **รวมผล** — ส่ง output ของแต่ละ agent ให้ `scribe record` (เรียกพร้อมกันได้) แล้วจัดการ `NEEDS`:
   - `CONTRACT_CHANGE_NEEDED` → รวมทุกคำขอ เรียก `contract-dev` ครั้งเดียว แล้วเรียกเฉพาะ agent ที่ขอให้ทำต่อ
   - `DEPENDENCY_NEEDED` / `ROOT_CHANGE_NEEDED` → session หลักทำเอง แล้วเรียก agent ที่ขอให้ทำต่อ
   - `STATUS: BLOCKED` ที่แก้ไม่ได้ → `jira-sync comment` แล้วหยุด
5. **qa** — มี package qa → `qa-tester` หลัง backend และ frontend `DONE` ทั้งคู่

ห้ามเรียก agent สองตัวที่เจ้าของไฟล์ทับกันพร้อมกัน และห้ามรัน `verifier runtime` ระหว่างที่ dev agent ยังทำงาน

## 4. ตรวจ

1. `verifier full` แล้ว `verifier runtime` — FAIL → แก้ตาม FAILURES แล้วรันเฉพาะชุดที่พังซ้ำ
   FAILURES ในโฟลเดอร์ของ agent ใด → ส่งกลับให้ agent นั้นแก้ (3B) หรือแก้เอง (3A)
2. มี endpoint/flow ใหม่และยังไม่ได้เรียกในข้อ 3B → `qa-tester` (BUGS → ส่งให้เจ้าของไฟล์แก้ แล้ว `verifier` ซ้ำ)
3. `reviewer` ส่ง key + AC + ผล verifier ล่าสุด
   - `CHANGES_REQUESTED` → แยก MUST FIX ตามเจ้าของไฟล์ แล้วส่งให้แต่ละ agent แก้พร้อมกัน (หรือแก้เอง) แล้วเรียกใหม่
     (สูงสุด 3 รอบ ถ้ายังไม่ผ่าน → `jira-sync comment` แล้วหยุด)
   - `PASS` → ไปต่อ

## 5. ปิดงาน

1. `scribe close <KEY>` ส่งผล verifier, reviewer และวิธีตรวจด้วยมือ (รวม `MANUAL CHECK` ของ frontend-dev)
2. `jira-sync done <KEY>` พร้อมข้อความ:
   ```
   ✅ เสร็จแล้ว (AI agent)
   - สิ่งที่ทำ: ...
   - ไฟล์: ...
   - Test: lint/typecheck ผ่าน, test N/N, e2e N/N, runtime healthy
   - Review: PASS
   - ตรวจด้วยมือ: ...
   - รายละเอียด: docs/tasks/<KEY>.md
   ```
3. โหมด `all` → กลับไปข้อ 0 (เริ่ม task ใหม่ให้อ่านจาก task file ไม่ต้องพึ่งบริบทของ task ก่อน)

## ส่งต่องานกลางทาง

ต้องหยุดก่อนเสร็จ → `scribe log` ว่าทำถึง step / package ไหน อะไรค้าง ขั้นต่อไปคืออะไร
แล้ว `jira-sync comment <KEY> "หยุดที่ step X — ดู Log ใน docs/tasks/<KEY>.md"` โดยไม่ย้ายสถานะ
