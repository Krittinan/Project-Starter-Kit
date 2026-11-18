---
paths:
  - "**/*.ts"
  - "**/*.tsx"
  - "**/tsconfig*.json"
---

# TypeScript

- `strict: true` ทุก package, ESM (`"type": "module"`), extend config จาก `@repo/tsconfig`
- ห้าม `any` (ใช้ `unknown` แล้ว narrow); ห้าม non-null assertion `!` ยกเว้นมี comment บอกเหตุผล
- type ของข้อมูลที่ข้ามหน้าบ้าน/หลังบ้านมาจาก `z.infer` ใน `@repo/shared` เท่านั้น ห้ามประกาศซ้ำ
- ตัวแปร/ฟังก์ชัน camelCase · class/type/component PascalCase · ค่าคงที่ระดับ module UPPER_SNAKE_CASE
- ห้าม `console.log` ในโค้ดที่ส่งงาน (backend ใช้ `Logger` ของ Nest)
- `pnpm lint` (oxlint) และ `pnpm format:check` (Prettier) ต้องผ่าน
