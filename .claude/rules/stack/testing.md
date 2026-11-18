---
paths:
  - "**/*.spec.ts"
  - "**/*.spec.tsx"
  - "**/*.e2e-spec.ts"
  - "**/vitest.config.ts"
  - "e2e/**"
---

# Testing

| ชั้น | เครื่องมือ | อยู่ที่ | รันด้วย |
|---|---|---|---|
| Unit (pure function, service ที่ mock repository) | Vitest | ข้างไฟล์ `*.spec.ts` | `pnpm test` |
| Component | Vitest + Testing Library | ข้างไฟล์ `*.spec.tsx` | `pnpm test` |
| API e2e (HTTP + DB จริง) | Vitest + Supertest | `apps/api/test/*.e2e-spec.ts` | `pnpm --filter api test:e2e` (ใช้ `db-test`) |
| End-to-end (เบราว์เซอร์) | Playwright | `e2e/*.spec.ts` | `pnpm test:e2e` (ระบบรันใน Docker) |

- ทุกข้อใน AC ต้องมี test อย่างน้อย 1 ตัว และชื่อ test อ้าง AC ข้อนั้น
- กฎทุกข้อใน `rules/domain/*.md` ต้องมี unit test ตรงตัว
- endpoint ใหม่ต้องมี API e2e ตรวจ status, `error.code` และรูปร่าง body
- flow หลักที่ผู้ใช้เห็นต้องมี Playwright อย่างน้อย 1 test ต่อ feature

## กติกา

- เวลา: ส่ง `Clock` ปลอมหรือ `vi.setSystemTime` — ห้ามพึ่งเวลาจริง
- API e2e: `TRUNCATE ... CASCADE` ก่อนแต่ละไฟล์ และใช้ `DATABASE_URL` ของ `db-test` เท่านั้น
- Playwright: เลือก element ด้วย role/label (`getByRole`, `getByLabel`) ไม่ใช้ CSS class ของ UI library
- ห้ามต่อ network ภายนอก, ห้าม `.only` / `.skip` ค้าง
- ค่าที่คาดหวังคำนวณมือไว้ใน test ไม่เอาผลจากโค้ดที่ test มาเป็นค่าคาดหวัง
