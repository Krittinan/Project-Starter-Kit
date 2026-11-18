---
name: new-feature
description: สร้าง feature ใหม่ครบทุกชั้น (shared schema → Prisma/Nest → web → e2e) โดยแยกให้ contract-dev แล้ว backend-dev กับ frontend-dev ทำขนานกัน ตาม pattern ของ feature ที่มีอยู่ ใช้เมื่อ task ต้องเพิ่ม resource/หน้าใหม่
argument-hint: "<feature-name> [--api-only | --web-only]"
---

# /new-feature $ARGUMENTS

ชื่อ feature เป็น kebab-case พหูพจน์ (เช่น `order-items`) — ใช้ใน path, route และ resource URL
ต้องมี task file ที่มี Design (endpoint, schema) แล้ว — ถ้ายังไม่มีให้เรียก `planner` ก่อน

## 0. หาต้นแบบ (ห้ามข้าม)

`scout`: "หา feature ที่ครบทุกชั้นที่สุดใน `packages/shared/src/`, `apps/api/src/modules/`, `apps/web/src/features/` แล้วคืนรายการไฟล์ของมัน"
ส่งรายการไฟล์ต้นแบบนี้ให้ทุก dev agent — copy โครงและ pattern ไม่คิดใหม่ ถ้ายังไม่มี feature เลย ใช้ rule ใน `rules/stack/` เป็นแม่แบบ

## 1. แยกงาน

| ลำดับ | agent | สร้าง | rule |
|---|---|---|---|
| 1 | `contract-dev` | `packages/shared/src/<feature>.ts` schema create/update/response/list query + export, error code ใหม่ | `stack/api-conventions.md` |
| 2 (ขนาน) | `backend-dev` | Prisma model + migration, `apps/api/src/modules/<feature>/` ครบ 4 ไฟล์ + spec, ลงทะเบียนใน `app.module.ts`, `apps/api/test/<feature>.e2e-spec.ts` | `stack/nestjs.md`, `stack/prisma-postgres.md` |
| 2 (ขนาน) | `frontend-dev` | `apps/web/src/features/<feature>/` `api.ts`, `hooks/`, `components/` + `app/.../<feature>/page.tsx` + เมนู | `stack/nextjs-mui.md` |
| 3 | `qa-tester` | `e2e/<feature>.spec.ts` อย่างน้อย 1 flow | `stack/testing.md` |

- `--api-only` ทำ 1 → backend-dev · `--web-only` ทำ frontend-dev อย่างเดียว (ต้องมี schema ใน shared แล้ว)
- ลำดับ 2 เรียกสองตัวใน message เดียว แล้วจัดการ `NEEDS` ตามข้อ 3B ของ `/work-task`
- ใส่เฉพาะ field/endpoint ที่ AC ต้องการ ไม่เติม CRUD ที่ไม่ได้ขอ
- กฎธุรกิจที่ซับซ้อนแยกเป็น pure function + มี domain rule ใน `rules/domain/` (session หลักเขียน rule ก่อนแยกงาน)

## 2. ตรวจ

`verifier full` → FAILURES ส่งกลับให้ agent เจ้าของไฟล์แก้ → `scribe record` ผลของแต่ละ agent ลง task file
