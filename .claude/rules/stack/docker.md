---
paths:
  - "compose*.yaml"
  - "**/Dockerfile"
  - "**/.dockerignore"
  - ".env.example"
  - "docker/**"
---

# Docker

## Services (`compose.yaml`)

| service | image / build | พอร์ต host | หมายเหตุ |
|---|---|---|---|
| `db` | `postgres:<major>-alpine` | 5432 | volume `db-data`, healthcheck `pg_isready` |
| `migrate` | build `apps/api` target `migrate` | – | `prisma migrate deploy` แล้วจบ; `depends_on: db (service_healthy)` |
| `api` | build `apps/api` | 4000 | `depends_on: migrate (service_completed_successfully)`, healthcheck `GET /api/health` |
| `web` | build `apps/web` | 3000 | `depends_on: api (service_healthy)`, `API_INTERNAL_URL=http://api:4000` |
| `db-test` | `postgres:<major>-alpine` (profile `test`) | 5433 | ข้อมูลบน `tmpfs` ใช้กับ API e2e |
| `pgadmin` | `dpage/pgadmin4:<major>` (profile `tools`) | 5050 | dev เท่านั้น |

- dev: `compose.yaml` ใช้ target `dev`, mount source เพื่อ hot reload, `node_modules` อยู่ใน named volume
- prod: `compose.prod.yaml` override เป็น target `runner`, ไม่ mount source, `restart: unless-stopped`
- ทุก service อ่านค่าผ่าน `env_file`; ค่าเริ่มต้น dev อยู่ใน `.env.example`

## Dockerfile (multi-stage, เหมือนกันทั้ง api และ web)

1. `base` — `node:<major>-bookworm-slim` + pnpm ตาม `packageManager`
2. `pruner` — `turbo prune <app> --docker`
3. `deps` — copy `out/json` + lockfile แล้ว `pnpm install --frozen-lockfile`
4. `build` — copy `out/full`, `prisma generate` (api), `turbo build --filter=<app>`
5. `dev` — `pnpm --filter <app> dev`
6. `migrate` (api) — `prisma migrate deploy`
7. `runner` — เฉพาะ output ที่รันจริง (api: `dist` + prod deps; web: `.next/standalone` + `.next/static` + `public`), `USER node`, `NODE_ENV=production`

## ข้อห้าม

- ห้ามใส่ secret ใน Dockerfile, `ARG`, image หรือ compose — ใช้ `env_file` / secrets ตอนรัน
- ห้าม `latest` tag; major ของ image ต้องตรงกับตาราง Stack ใน `architecture.md`
- `.dockerignore` ตัด `node_modules`, `.next`, `dist`, `.env*` (ยกเว้น `.env.example`), `.git`, `coverage`
- runner รันด้วย user ที่ไม่ใช่ root และมี healthcheck
- ห้าม `docker compose down -v` หรือลบ volume โดยไม่ถามคน
- เปลี่ยนพอร์ตหรือเพิ่ม service ต้องแก้ตารางนี้และตาราง Environment ใน `architecture.md` ใน task เดียวกัน
