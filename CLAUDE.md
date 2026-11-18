<!--
  TEMPLATE — รัน /init-project เพื่อเติมค่าให้อัตโนมัติ หรือแทนทุก {{...}} ด้วยมือแล้วลบ comment นี้
  ไฟล์นี้ถูกโหลดทุก request: ใส่เฉพาะสิ่งที่ต้องรู้ทุก task รายละเอียดอยู่ใน .claude/rules/
-->

# {{PROJECT_NAME}} — คู่มือสำหรับ Claude และทีม

{{หนึ่งถึงสองบรรทัด: ระบบนี้คืออะไร ใครใช้ ทำอะไรได้}}
Requirement: `docs/requirements/{{REQUIREMENT_FILE}}.md`

## Stack และคำสั่ง

stack, โครงสร้าง, layer และคำสั่งทั้งหมดอยู่ใน `.claude/rules/core/architecture.md`

- รันทั้งระบบ: `{{RUN_COMMAND}}` → {{LOCAL_URL}}
- ตรวจก่อนส่งงาน: `{{CHECK_COMMAND}}`

## งานมาจากไหน

- Jira project **{{JIRA_PROJECT_NAME}}** · key `{{JIRA_KEY}}` · label `{{JIRA_LABEL}}` · {{JIRA_BASE_URL}}
- Acceptance Criteria (AC) ใน Jira คือ scope ของงาน ห้ามทำเกินหรือขาด ถ้า AC ไม่ชัดให้ถามใน Jira comment แล้วหยุด
- แผนและบันทึกของแต่ละ task อยู่ที่ `docs/tasks/<KEY>.md` — **อ่านไฟล์นี้ก่อนทำอะไรกับ task นั้นเสมอ**
- เริ่มงานด้วย `/work-task <KEY>` · `/work-task next` · `/work-task all`

## Flow การทำงาน

1. เลือก task ที่ To Do และ "is blocked by" เป็น Done ครบ (มีใบ In Progress ค้าง → ทำใบนั้นก่อน)
2. Jira → In Progress + comment ว่าเริ่ม
3. `planner` เขียนแผนใน `docs/tasks/<KEY>.md`
4. เขียนโค้ดตาม `.claude/rules/` — logic ใหม่ต้องมี test (งานหลายชั้น: `contract-dev` → `backend-dev` ∥ `frontend-dev`)
5. `verifier` รัน `{{CHECK_COMMAND}}` และ {{RUNTIME_CHECK}} จนผ่าน
6. `reviewer` ตรวจเทียบ AC + rules แก้จน PASS
7. `scribe` อัปเดต task file → `jira-sync` comment สรุปและย้ายเป็น Done

## Definition of Done

- ทำครบทุกข้อใน AC และมี test ครอบทุกข้อที่ test ได้ (ไม่มี skip/only)
- `{{CHECK_COMMAND}}` ผ่าน 100% และ {{RUNTIME_CHECK}}
- ไม่ละเมิด rules ใน `.claude/rules/`
- `docs/tasks/<KEY>.md` มีแผน, ไฟล์ที่แก้, ผล test และวิธีตรวจด้วยมือ; Jira มี comment สรุป

## Agents — ใช้แทนการทำเองใน session หลัก

| งาน | agent | model |
|---|---|---|
| ค้นโค้ด/หาว่าอะไรอยู่ไหน (มากกว่า 2–3 ไฟล์) | `scout` | haiku |
| อ่าน AC, ย้ายสถานะ, comment Jira | `jira-sync` | haiku |
| รัน lint / typecheck / test / docker แล้วสรุปเฉพาะที่พัง | `verifier` | haiku |
| บันทึก Log / Files / ผลลัพธ์ลง task file | `scribe` | haiku |
| เขียนแผน + แบ่ง work package ตามเจ้าของไฟล์ | `planner` | sonnet |
| contract กลาง `packages/shared` (รันก่อน) | `contract-dev` | sonnet |
| หลังบ้าน `apps/api` (ขนานกับ frontend-dev) | `backend-dev` | sonnet |
| หน้าบ้าน `apps/web` (ขนานกับ backend-dev) | `frontend-dev` | sonnet |
| เติม test จาก AC + e2e + smoke test | `qa-tester` | sonnet |
| ตรวจงานก่อนปิด (อ่านอย่างเดียว) | `reviewer` | sonnet |

แยกกันทำโดยไม่ชนกัน: แต่ละ agent แก้ได้เฉพาะไฟล์ในตาราง "เจ้าของไฟล์" ของ `core/architecture.md` · ไฟล์ร่วม (lockfile, root config) เป็นของ session หลัก · ต้องการแก้ของคนอื่นให้รายงานใน `NEEDS` แทนการแก้เอง

ประหยัด token: อ่าน task file ก่อน (มันคือความจำของงาน) · อ่านไฟล์เฉพาะช่วงที่ต้องใช้ · ไม่อ่านไฟล์ซ้ำ · ไม่ให้ log ยาวเข้า session หลัก (ส่งให้ `verifier`) · ส่ง agent เฉพาะ key + path ไม่ต้อง copy เนื้อหาไฟล์ไปใน prompt

## Rules

rule ที่มี `paths` โหลดเฉพาะเมื่อแตะไฟล์ที่ตรง — **ก่อนสร้างไฟล์ใหม่ในโฟลเดอร์ใด ให้อ่านไฟล์ที่มีอยู่ในโฟลเดอร์นั้นก่อน**

| ไฟล์ | โหลดเมื่อ |
|---|---|
| `core/architecture.md`, `core/coding-style.md` | ทุกครั้ง |
| `stack/typescript.md` | `*.ts`, `*.tsx` |
| `stack/nestjs.md` | `apps/api/src/**`, `apps/api/test/**` |
| `stack/prisma-postgres.md` | `apps/api/prisma/**`, `*.repository.ts` |
| `stack/nextjs-mui.md` | `apps/web/**` |
| `stack/api-conventions.md` | controller, `common/`, `api-client.ts`, `packages/shared/**` |
| `stack/docker.md` | `compose*.yaml`, `Dockerfile`, `.env.example` |
| `stack/testing.md` | `*.spec.ts(x)`, `*.e2e-spec.ts`, `e2e/**` |
| `domain/{{DOMAIN}}.md` | {{path ของ domain}} — **กฎ{{DOMAIN_TH}}อยู่ที่นี่ที่เดียว** |

## ข้อห้าม

- ห้ามอ่านหรือเขียน `.env` / `.env.local` (แก้ได้เฉพาะ `.env.example`) และห้ามใส่ token/secret ในโค้ด เอกสาร หรือ image
- ห้าม `prisma migrate reset`, `docker compose down -v` หรือลบ volume ข้อมูล โดยไม่ถามคน
- ห้ามเปลี่ยนกฎใน `domain/*.md` ในโค้ดโดยไม่แก้ไฟล์ rule นั้นใน task เดียวกัน
- {{ข้อห้ามเฉพาะโปรเจกต์ — ไม่มีให้ลบบรรทัดนี้}}
