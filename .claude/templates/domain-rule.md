---
# คัดลอกไฟล์นี้ไปเป็น .claude/rules/domain/<domain>.md แล้วตั้ง paths ให้ตรงโค้ดของ domain
# (template อยู่นอก rules/ เพื่อไม่ให้ถูกโหลดเปลือง token)
paths:
  - "apps/api/src/modules/<domain>/**"
  - "packages/shared/src/<domain>*"
  - "apps/web/src/features/<domain>/**"
---

# <Domain> (แหล่งความจริงเดียวของกฎ<ชื่อไทย>)

เปลี่ยนกฎในโค้ดต้องแก้ไฟล์นี้ใน task เดียวกัน และทุกกฎต้องมี unit test ใน `<path ของ spec>`

## คำศัพท์

| คำ | ความหมาย |
|---|---|

## โครงสร้างข้อมูล

| field | กฎ |
|---|---|

## กฎธุรกิจ (ลำดับการคำนวณ / เงื่อนไข)

1. ...

## ตัวอย่างที่ต้องได้ผลตรงนี้ (ใช้เป็น test case)

| input | ผลที่ต้องได้ | เพราะ |
|---|---|---|
