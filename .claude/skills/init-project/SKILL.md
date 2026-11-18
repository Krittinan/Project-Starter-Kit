---
name: init-project
description: ตั้งค่า Starter Kit ให้เป็นของโปรเจกต์ใหม่ — เติมค่า {{...}} ใน CLAUDE.md, เลือก/ลบ stack rule ที่ไม่ใช้, สร้าง domain rule และตรวจว่าไม่เหลือ placeholder ใช้ครั้งแรกหลังคัดลอก Starter Kit เข้าโปรเจกต์
argument-hint: "[ชื่อโปรเจกต์]"
---

# /init-project $ARGUMENTS

## 1. เก็บข้อมูล (ถามผู้ใช้ครั้งเดียวรวมทุกข้อ ข้อที่หาได้จาก repo ให้หาเองก่อน)

| ค่า | หาจาก / ถาม |
|---|---|
| `PROJECT_NAME`, คำอธิบาย 1–2 บรรทัด | `$ARGUMENTS`, `README.md`, `package.json` |
| `REQUIREMENT_FILE` | `docs/requirements/*.md` |
| `JIRA_PROJECT_NAME`, `JIRA_KEY`, `JIRA_LABEL`, `JIRA_BASE_URL` | ถามผู้ใช้ (หรือ `jira-sync` ถ้า MCP เชื่อมแล้ว) |
| `RUN_COMMAND`, `LOCAL_URL`, `CHECK_COMMAND`, `RUNTIME_CHECK` | `package.json` scripts, `compose.yaml` — ค่าเริ่มต้นของ kit: `docker compose up -d --build` · http://localhost:3000 · `pnpm lint && pnpm typecheck && pnpm test` · `docker compose up --build` ขึ้นครบทุก service (healthy) |
| ส่วนของ stack ที่ใช้ | ดูโครงสร้างจริง (`apps/web`, `apps/api`, prisma, compose) |
| domain หลัก + กฎที่ผิดไม่ได้ | ถามผู้ใช้ / requirement |

## 2. ปรับไฟล์

1. `CLAUDE.md` — แทนทุก `{{...}}`, ลบ comment บนสุด, ลบแถวในตาราง Rules ที่ไม่ใช้
2. `.claude/rules/stack/` — ลบไฟล์ของส่วนที่โปรเจกต์ไม่มี (เช่น ไม่มีหน้าบ้าน → ลบ `nextjs-mui.md`)
   stack ที่ kit ไม่มี → สร้าง rule ใหม่โดยใช้รูปแบบเดียวกัน (frontmatter `paths` + กติกาแบบตรวจได้)
3. `.claude/rules/core/architecture.md` — ปรับตาราง Stack, โครงสร้าง, Environment ให้ตรง repo จริง
4. `rules/stack/api-conventions.md` — เติมตาราง "Code เฉพาะโปรเจกต์" ถ้ารู้แล้ว
5. domain → คัดลอก `.claude/templates/domain-rule.md` ไปเป็น `.claude/rules/domain/<domain>.md` ตั้ง `paths` และเติมกฎที่รู้
6. `docs/tasks/_template.md` — แทน `<JIRA_BASE_URL>`
7. `.claude/settings.json` — ปรับ allow list ถ้าคำสั่งต่างจาก pnpm/docker compose

## 3. ตรวจ

- `grep -rn "{{" CLAUDE.md .claude/rules docs/tasks/_template.md` ต้องไม่เหลือ
- ทุก path ที่อ้างในตาราง Rules ของ `CLAUDE.md` มีไฟล์จริง
- ทุก rule ใน `rules/stack/` และ `rules/domain/` มี `paths` (ไม่งั้นจะโหลดทุกครั้ง เปลือง token)

## Output

สรุปค่าที่ตั้ง, ไฟล์ที่ลบ/สร้าง, และสิ่งที่ยังต้องให้คนเติม (เช่น กฎ domain ที่ยังไม่รู้)
