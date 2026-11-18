---
paths:
  - "apps/api/src/**"
  - "apps/api/test/**"
---

# Backend (NestJS)

## ไฟล์ของแต่ละ feature (`src/modules/<feature>/`)

| ไฟล์ | หน้าที่ | ห้าม |
|---|---|---|
| `<feature>.module.ts` | ประกอบ provider, export เฉพาะ service ที่ module อื่นต้องใช้ | export repository |
| `<feature>.controller.ts` | route + validate + เรียก service + คืนผล | business logic, Prisma |
| `<feature>.service.ts` | กฎธุรกิจ, transaction, โยน error ที่มี code | สร้าง HTTP response เอง |
| `<feature>.repository.ts` | query Prisma, แปลง row → object ของ domain | กฎธุรกิจ |
| `<feature>.service.spec.ts` | unit test (mock repository) | ต่อ DB จริง |

- module ใช้ของกันผ่าน export ของ Nest module เท่านั้น ห้าม import repository ของ module อื่น

## Validation

- ใช้ Standard Schema: `@Body({ schema })`, `@Query({ schema })`, `@Param({ schema })` กับ Zod schema จาก `@repo/shared`
- ลงทะเบียน `StandardSchemaValidationPipe` แบบ global ใน `main.ts`; ไม่ใช้ class-validator / class-transformer
- ห้ามประกาศ DTO ซ้ำกับ schema ใน shared

## Error

- service โยน `DomainError(code, message, details?)` จาก `src/common/errors.ts` — code ต้องอยู่ใน `ErrorCode` ของ `@repo/shared`
- `AllExceptionsFilter` (global) แปลงทุกอย่างเป็นรูปแบบใน `api-conventions.md`:
  validation → 422 `VALIDATION_ERROR` · JSON พัง → 400 `INVALID_JSON` · body > 1 MB → 413 · route ไม่มี → 404 ·
  `DomainError` → status ตาม code · Prisma `P2002` → 409 · อื่น ๆ → 500 `INTERNAL_ERROR` + log
- ห้ามส่ง stack trace หรือข้อความจาก Prisma ออกไปหา client

## อื่น ๆ

- `main.ts`: `setGlobalPrefix('api')`, `enableShutdownHooks()`, ฟังที่ `0.0.0.0:${API_PORT}`
- env อ่านผ่าน `ConfigService` ที่ validate ด้วย Zod (`src/config/env.schema.ts`) เท่านั้น ห้าม `process.env` กระจาย
- เวลา: inject `Clock` (`src/common/clock.ts`) แทน `new Date()` ใน service
- list endpoint ใช้ `PaginationQuerySchema` จาก shared
- endpoint ใหม่ต้องมี e2e ใน `apps/api/test/<feature>.e2e-spec.ts`
