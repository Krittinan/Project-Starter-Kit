---
paths:
  - "apps/api/src/**/*.controller.ts"
  - "apps/api/src/common/**"
  - "apps/api/test/**"
  - "apps/web/src/lib/api-client.ts"
  - "packages/shared/**"
---

# API conventions

## URL และ method

- ทุก endpoint อยู่ใต้ `/api`, resource เป็นพหูพจน์ kebab-case: `/api/<resources>`
- `GET /api/<res>` list · `POST /api/<res>` สร้าง · `GET|PATCH /api/<res>/:id` · `DELETE /api/<res>/:id`
- `DELETE` = soft delete (`active: false`) ตอบ 200 พร้อม resource
- action ที่ไม่ใช่ CRUD: `POST /api/<res>/:id/<action>`

## Request / Response

- JSON เท่านั้น ขนาด body ไม่เกิน 1 MB; field camelCase; field ที่ไม่รู้จักถูกตัดทิ้ง (ไม่ใช้ `.strict()`)
- schema ของทุก request/response อยู่ใน `@repo/shared` ใช้ทั้ง validate ฝั่ง API และฟอร์มฝั่ง web
- resource เดียว: ตอบ object ตรง ๆ (สร้าง 201, อื่น ๆ 200)
- list: `{ "items": [...], "total": <หลังกรอง>, "page": 1, "pageSize": 20 }` · `page` เริ่ม 1 · `pageSize` ค่าเริ่ม 20 สูงสุด 100
- ทุก resource มี `id` (UUID), `createdAt`, `updatedAt` (ISO 8601 UTC)
- เงินเป็นจำนวนเต็มหน่วยย่อยสุด

## Error

```json
{ "error": { "code": "VALIDATION_ERROR", "message": "ข้อมูลไม่ถูกต้อง", "details": [{ "field": "items[0].qty", "message": "..." }] } }
```

| HTTP | code | ใช้เมื่อ |
|---|---|---|
| 400 | `INVALID_JSON` | body ไม่ใช่ JSON object ที่ถูกต้อง |
| 401 | `UNAUTHORIZED` | ไม่ได้ login (ถ้าโปรเจกต์มี auth) |
| 403 | `FORBIDDEN` | ไม่มีสิทธิ์ (ถ้าโปรเจกต์มี auth) |
| 404 | `NOT_FOUND` | ไม่มี path, method หรือ resource |
| 409 | `<ENTITY>_TAKEN`, `<RULE>_CONFLICT` | ขัดกับข้อมูลที่มีอยู่ |
| 413 | `PAYLOAD_TOO_LARGE` | body เกิน 1 MB |
| 422 | `VALIDATION_ERROR`, `<ENTITY>_UNAVAILABLE` | ข้อมูลผิดกฎ (`details` ราย field) |
| 500 | `INTERNAL_ERROR` | ไม่ได้ตั้งใจ (ห้ามส่ง stack trace) |

### Code เฉพาะโปรเจกต์

| HTTP | code | ใช้เมื่อ |
|---|---|---|
| | | |

- code ทั้งหมดอยู่ใน `ErrorCode` ของ `@repo/shared` — code ใหม่ต้องเพิ่มทั้งที่นั่นและตารางนี้ใน task เดียวกัน
