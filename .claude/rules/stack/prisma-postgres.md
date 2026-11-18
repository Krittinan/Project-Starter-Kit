---
paths:
  - "apps/api/prisma/**"
  - "apps/api/prisma.config.ts"
  - "apps/api/src/prisma/**"
  - "apps/api/src/**/*.repository.ts"
---

# Database (Prisma + PostgreSQL)

## ตั้งค่า

- `prisma.config.ts` กำหนด schema path, migrations path, seed และ `datasource.url = env("DATABASE_URL")`
- generator `provider = "prisma-client"`, `output = "../src/generated/prisma"` — โฟลเดอร์ generated ไม่แก้ด้วยมือและไม่ส่งงาน
- สร้าง client ด้วย driver adapter `@prisma/adapter-pg` ใน `PrismaService` ที่เดียว
- ติดตั้งด้วยเวอร์ชันที่ล็อกใน `architecture.md` เสมอ (`add prisma` เฉย ๆ อาจได้ major ใหม่ที่ API ต่างกัน)

## Schema

- model PascalCase เอกพจน์, field camelCase; ตาราง/คอลัมน์ snake_case ด้วย `@@map` / `@map`
- id: `String @id @default(uuid())`
- ทุก model มี `createdAt @default(now()) @map("created_at")` และ `updatedAt @updatedAt @map("updated_at")`
- เงินเป็น `Int` หน่วยย่อยสุด (ห้าม Float); ถ้าต้องการทศนิยมจริงให้ใช้ `Decimal` และเขียนเหตุผลใน task file
- ลบ = soft delete ด้วย `active Boolean @default(true)` (ยกเว้น domain rule บอกเป็นอย่างอื่น)
- field ที่ค้น/กรองบ่อยต้องมี `@@index`; ความไม่ซ้ำทาง business ใช้ `@unique`
- unique แบบมีเงื่อนไข: เช็คใน service + partial unique index ใน migration SQL
- enum ของ domain ต้องตรงกับค่าใน `@repo/shared`

## Migration

- สร้างด้วย `pnpm db:migrate --name <อธิบายสั้น>` เท่านั้น ห้ามแก้ migration ที่ apply แล้ว
- migration ที่ลบคอลัมน์/ตาราง หรือเปลี่ยน type ที่ข้อมูลอาจหาย ต้องเขียนเหตุผลใน task file และให้คนอนุมัติก่อน
- ถ้า `migrate dev` ขอ reset database ให้หยุดและถามคน ห้ามตอบตกลงเอง
- prod ใช้ `prisma migrate deploy` ผ่าน service `migrate` ใน compose เท่านั้น

## Repository

- รับ `Prisma.TransactionClient` แบบ optional เพื่อให้ service ครอบหลาย repository ใน `$transaction` เดียว
- select เฉพาะ field ที่ใช้ ห้ามคืน row ดิบออกนอก repository
- seed รันซ้ำได้ (upsert) และใช้ข้อมูลตัวอย่าง ไม่ใช่ข้อมูลจริงของลูกค้า
