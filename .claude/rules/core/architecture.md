# Architecture — mono-repo (หน้าบ้าน / หลังบ้าน)

> ค่าเริ่มต้นของ Starter Kit ถ้าโปรเจกต์ไม่ใช้บางส่วน (เช่น ไม่มีหน้าบ้าน) ให้ลบแถว/หัวข้อนั้น และลบไฟล์ใน `rules/stack/` ที่ไม่ใช้
> เวอร์ชันทั้งหมดอยู่ในตารางนี้ที่เดียว — rule อื่นห้ามระบุเลขเวอร์ชันซ้ำ

## Stack (ล็อก major version)

| ส่วน | ใช้ | หมายเหตุ |
|---|---|---|
| Runtime | Node.js 26 LTS | `.nvmrc` + `engines` ทุก package; image `node:26-*` |
| Package / Monorepo | pnpm workspaces + Turborepo | ล็อก pnpm ด้วย `packageManager` ใน root `package.json` |
| ภาษา | TypeScript strict, ESM ทั้ง repo | |
| หน้าบ้าน `apps/web` | Next.js 16 (App Router) + React 19 + Material UI 9 + MUI X | TanStack Query, react-hook-form |
| หลังบ้าน `apps/api` | NestJS 12 (ESM) + Prisma ORM 7 + PostgreSQL 17 | ล็อก `prisma@7`, `@prisma/client@7` |
| Contract กลาง `packages/shared` | Zod 4 | schema + type + error code ใช้ร่วมทั้งสองฝั่ง |
| คุณภาพโค้ด | oxlint + Prettier | config เดียวที่ root |
| Test | Vitest, Supertest (API), Playwright (e2e) | |
| รัน | Docker Compose | dev และ prod ใช้ compose ชุดเดียวกันต่าง override |

อัป major ต้องเปิด task แยกเพื่ออัปเกรดเท่านั้น

## โครงสร้าง

```
apps/
  api/                 NestJS + Prisma            → stack/nestjs.md, stack/prisma-postgres.md
    src/
      main.ts          prefix /api, global pipe/filter, shutdown hooks
      common/          exception filter, errors, clock, decorators
      config/          ตรวจ env ด้วย Zod (fail fast)
      prisma/          PrismaModule / PrismaService (ที่เดียวที่สร้าง client)
      health/          GET /api/health (เช็ค DB ด้วย)
      modules/<feature>/   <feature>.module|controller|service|repository.ts + *.spec.ts
    prisma/            schema.prisma, migrations/, seed.ts
    test/              API e2e (Supertest + DB จริงใน Docker)
  web/                 Next.js + MUI               → stack/nextjs-mui.md
    src/
      app/             routes เท่านั้น
      features/<feature>/  components/, hooks/, api.ts
      components/      UI ที่ใช้หลาย feature
      lib/             api-client.ts, query-client.ts
      theme/           MUI theme + ThemeRegistry
packages/
  shared/              @repo/shared — Zod schemas, types, error codes, pagination
  tsconfig/            @repo/tsconfig — base / nest / next
e2e/                   Playwright (ยิงผ่าน web จริงใน Docker)
compose.yaml, compose.prod.yaml, .env.example     → stack/docker.md
docs/requirements/     requirement ของโปรเจกต์
docs/tasks/<KEY>.md    แผนและ log ของแต่ละ task
```

## ทิศทาง dependency (ห้ามฝืน)

```
apps/web ──▶ packages/shared ◀── apps/api
   └──── HTTP /api/* (JSON) ────────┘
```

- `web` กับ `api` ห้าม import กัน — คุยผ่าน HTTP + type จาก `@repo/shared` เท่านั้น
- `packages/shared` ห้าม import React, Next, Nest, Prisma หรือ module ของ Node — dependency ได้แค่ `zod`
- กฎธุรกิจอยู่หลังบ้านเท่านั้น หน้าบ้านห้ามคำนวณเอง
- หลังบ้าน: `controller` (HTTP) → `service` (กฎธุรกิจ, transaction) → `repository` (Prisma เท่านั้น)
- โค้ด pure (engine คำนวณ) ห้าม import framework หรือเรียกเวลาจริง — รับ `now` เป็น argument

## เจ้าของไฟล์ (สำหรับงานแบบแยกกันทำ)

ตารางนี้เป็นที่เดียวที่กำหนดว่า agent ไหนแก้อะไรได้ — agent ห้ามแก้ไฟล์นอกแถวของตัวเอง

| agent | แก้ได้ | ห้ามแตะ |
|---|---|---|
| `contract-dev` | `packages/shared/**`, `.claude/rules/stack/api-conventions.md` (ตาราง code) | `apps/**` |
| `backend-dev` | `apps/api/**` (รวม prisma, migration, API e2e) | `packages/shared/**`, `apps/web/**` |
| `frontend-dev` | `apps/web/**` | `packages/shared/**`, `apps/api/**` |
| `qa-tester` | `e2e/**`, `*.spec.ts(x)` / `*.e2e-spec.ts` ที่เพิ่มใหม่ | โค้ดที่ไม่ใช่ test |
| `scribe` | `docs/tasks/**` | โค้ดทั้งหมด |
| session หลัก | `package.json`, `pnpm-lock.yaml`, `compose*.yaml`, config ที่ root, `.claude/rules/domain/**` | – |

- ไฟล์ที่ใช้ร่วม (lockfile, root config) แก้โดย session หลักเท่านั้น เพื่อไม่ให้ agent ที่ทำพร้อมกันเขียนทับกัน
- agent ต้องการเปลี่ยนของคนอื่น → ไม่แก้เอง แต่รายงานใน output (`CONTRACT_CHANGE_NEEDED`, `DEPENDENCY_NEEDED`, `ROOT_CHANGE_NEEDED`)

## Request flow และ Environment

Browser → `web:3000` → Next rewrite `/api/*` → `api:4000/api/*` → Postgres `db:5432` (same-origin, ไม่ต้องเปิด CORS)

| ตัวแปร | ใช้ที่ | ค่า dev |
|---|---|---|
| `DATABASE_URL` | api | `postgresql://app:app@db:5432/app` |
| `API_PORT` | api | `4000` |
| `API_INTERNAL_URL` | web (server) | `http://api:4000` |
| `WEB_PORT` | web | `3000` |

ค่าจริงอยู่ใน `.env` (AI ห้ามอ่าน) — ตัวแปรใหม่ต้องเพิ่มใน `.env.example` และ schema ใน `apps/api/src/config` ใน task เดียวกัน

## คำสั่งมาตรฐาน (root)

| คำสั่ง | ทำอะไร |
|---|---|
| `docker compose up -d --build` | รันทั้งระบบ — web ที่ http://localhost:3000 |
| `pnpm dev` | รัน api + web บนเครื่อง (ต้องมี `docker compose up -d db`) |
| `pnpm lint` / `pnpm typecheck` / `pnpm test` | ผ่าน Turborepo ทุก package |
| `pnpm test:e2e` | Playwright กับระบบที่รันใน Docker |
| `pnpm db:migrate` / `pnpm db:seed` / `pnpm db:studio` | Prisma ของ `apps/api` |

## เพิ่ม feature ใหม่ → ใช้ `/new-feature <name>`
