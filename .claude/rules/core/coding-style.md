# Coding style (ทุกภาษา ทุกโปรเจกต์)

- identifier และ comment ภาษาอังกฤษ; ข้อความที่ผู้ใช้เห็น เอกสาร และ Jira ภาษาไทยได้; comment อธิบาย "ทำไม" ไม่ใช่ "ทำอะไร"
- ชื่อไฟล์ kebab-case (ยกเว้น convention ของ framework เช่น React component เป็น PascalCase)
- ฟังก์ชันสั้น (~40 บรรทัด) ทำอย่างเดียว ห้าม mutate argument
- ไม่ copy logic ซ้ำ — ก่อนเขียนฟังก์ชันใหม่ให้ค้นว่ามีอยู่แล้วหรือไม่ (ใช้ `scout`)
- ไม่ทิ้ง debug log / โค้ดที่ comment ไว้ / TODO ที่ไม่มี task key
- ห้ามปิด lint rule ด้วย comment โดยไม่เขียนเหตุผล
- เงินเก็บเป็นจำนวนเต็มหน่วยย่อยสุด (เช่น สตางค์) แปลงเฉพาะตอนแสดงผล; เวลาเก็บเป็น UTC

## Dependency

- เพิ่ม package ที่ package ที่ใช้จริงเท่านั้น (`pnpm --filter <pkg> add ...`) ไม่ใส่ที่ root ยกเว้น tooling
- package ที่ไม่อยู่ในตาราง Stack ของ `architecture.md` ต้องเขียนเหตุผลใน task file และให้ reviewer ตรวจ
- ห้ามอัป major เองใน task ที่ไม่ได้มีไว้เพื่ออัปเกรด
