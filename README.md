# POC Starter Kit — ชุด `.claude/` กลางสำหรับทุกโปรเจกต์

ชุดตั้งต้นของ Claude Code สำหรับทีม: flow AI SDLC (Jira → แผน → โค้ด → test → review → Done),
rules ของ stack หลัก (Next.js + MUI · NestJS + Prisma + PostgreSQL · Docker) และ agent ที่แบ่งงานเพื่อลด token

## ติดตั้งในโปรเจกต์

```bash
cp -R ~/Desktop/Playgrounds/POC_Starter_Kit/{.claude,.mcp.json,CLAUDE.md,docs} <path-โปรเจกต์>/
```

แล้วเปิด `claude` ในโปรเจกต์ และสั่ง:

```
/init-project <ชื่อโปรเจกต์>
```

ตรวจ: `/agents` ต้องเห็น 10 ตัว · `/mcp` ต้องเห็น atlassian (ครั้งแรกต้อง authorize)

## มีอะไรบ้าง

```
CLAUDE.md                 template คู่มือหลัก (โหลดทุก request — เขียนให้สั้น)
.mcp.json                 Atlassian MCP v2
docs/tasks/_template.md   แม่แบบแผน + log ของ task
.claude/
  settings.json           permission: อนุญาตคำสั่งตรวจ, ห้ามอ่าน .env, ห้ามคำสั่งลบข้อมูล
  rules/
    core/                 architecture, coding-style  → โหลดทุกครั้ง
    stack/                typescript, nestjs, prisma-postgres, nextjs-mui, api-conventions, docker, testing
                          → โหลดเฉพาะเมื่อแตะไฟล์ที่ตรง paths
    domain/               กฎธุรกิจของโปรเจกต์ (สร้างจาก templates/domain-rule.md)
  agents/                 scout, verifier, jira-sync, scribe (haiku)
                          planner, contract-dev, backend-dev, frontend-dev, qa-tester, reviewer (sonnet)
  skills/                 /work-task, /init-project, /new-feature
  templates/              domain-rule.md (อยู่นอก rules/ จึงไม่ถูกโหลด)
```

## Agents — ใครทำอะไร และทำไมช่วยประหยัด

| agent | model | งาน | สิ่งที่ไม่ต้องเข้า session หลัก |
|---|---|---|---|
| `scout` | haiku | ค้นโค้ด ตอบเป็น file:line + ของที่ใช้ซ้ำได้ | เนื้อหาไฟล์ที่เปิดไล่หา |
| `verifier` | haiku | รัน lint/typecheck/test/docker สรุปเฉพาะที่พัง | log ยาวของ test และ build |
| `jira-sync` | haiku | หา task, อ่าน AC แบบย่อ, ย้ายสถานะ, comment | JSON ของ issue |
| `scribe` | haiku | บันทึก Log / Files / ผลลัพธ์ลง task file | การเปิดและแก้ task file |
| `planner` | sonnet | เขียนแผน + แบ่ง work package ตามเจ้าของไฟล์ | การอ่าน rules/โค้ดเพื่อวางแผน |
| `contract-dev` | sonnet | schema / type / error code ใน `packages/shared` | – (ผลของมันคือ input ของ dev ทุกตัว) |
| `backend-dev` | sonnet | `apps/api` ครบทุกชั้น + unit + API e2e | โค้ดและ test หลังบ้าน |
| `frontend-dev` | sonnet | `apps/web` feature, hooks, components, page | โค้ดและ test หน้าบ้าน |
| `qa-tester` | sonnet | เติม test จาก AC + e2e + smoke test | การเขียน/รัน test ทีละเคส |
| `reviewer` | sonnet | ตรวจเทียบ AC + rules (อ่านอย่างเดียว) | การอ่านไฟล์ทั้งหมดที่แก้ |

session หลักเหลือแค่ "ประสานงานและตัดสินใจ" — งานที่ผลยาวแต่ใช้แค่ข้อสรุปถูกส่งให้ agent ที่ใช้ model ถูกกว่า

## แยกกันทำงาน (งานหลายชั้น)

```
planner ─ แบ่ง Work packages ใน task file
   │
root (session หลัก: dependency, root config, domain rule)
   │
contract-dev ─ EXPORTS ─┬─ backend-dev  ─┐   ← ทำพร้อมกัน
                        └─ frontend-dev ─┘
                                 │
                 qa-tester → verifier → reviewer → scribe / jira-sync
```

ทำไมไม่ซ้ำและไม่ชนกัน

- **Contract ก่อน** — type/schema เกิดที่ `contract-dev` ครั้งเดียว สองฝั่งใช้ของเดียวกัน frontend จึงไม่ต้องรอ backend
- **เจ้าของไฟล์ชัด** — ตาราง "เจ้าของไฟล์" ใน `core/architecture.md` (ที่เดียว) บอกว่าใครแก้อะไรได้ ไฟล์หนึ่งอยู่ได้ work package เดียว
- **ไฟล์ร่วมไม่ให้ agent แตะ** — lockfile / root config เป็นของ session หลัก agent ที่ต้องการให้รายงาน `DEPENDENCY_NEEDED` / `CONTRACT_CHANGE_NEEDED` แทน แล้ว session หลักรวมคำขอทำครั้งเดียว
- **prompt สั้น** — ส่งแค่ key + path task file + EXPORTS ไม่ copy AC/rules/โค้ด เพราะ agent อ่านเองจาก task file
- **งานเล็กไม่แยก** — planner เขียน `single` เมื่อแตะ ≤ 3 ไฟล์หรือฝั่งเดียว เพราะค่าเปิด agent แพงกว่างาน

## หลักลดงานซ้ำและลด token ที่ kit นี้ใช้

1. **โหลดเท่าที่ใช้** — `CLAUDE.md` + `core/` สั้นที่สุด; ที่เหลือมี `paths` จึงโหลดเฉพาะเมื่อแตะไฟล์ตรงกัน
2. **แหล่งความจริงเดียว** — เวอร์ชันอยู่ใน `architecture.md` ที่เดียว, กฎธุรกิจอยู่ใน `domain/` ที่เดียว, agent อ้าง rule แทนการ copy เนื้อหา
3. **Output แบบฟอร์มตายตัว** — ทุก agent ตอบเป็นรูปแบบสั้นที่กำหนด (VERDICT / FAILURES / WHERE) ไม่ส่งของดิบกลับ
4. **Task file คือความจำ** — `docs/tasks/<KEY>.md` เก็บแผนและ Log ทำให้ session ใหม่หรือคนใหม่ทำต่อได้โดยไม่ต้องสำรวจใหม่
5. **ทำซ้ำได้ด้วย skill** — `/work-task` (flow ทั้งหมด), `/new-feature` (โครง feature ทุกชั้นจากต้นแบบที่มีอยู่)
6. **ไม่รันซ้ำ** — reviewer รับผล verifier ไปใช้, verifier รันเฉพาะชุดที่พังซ้ำ

## ปรับสำหรับ stack อื่น

- ไม่ใช้ส่วนไหน → ลบไฟล์ใน `rules/stack/` และแถวในตาราง Rules ของ `CLAUDE.md`
- stack ใหม่ (เช่น Go, Python, React Native) → เพิ่มไฟล์ใน `rules/stack/` ตามรูปแบบเดิม: frontmatter `paths` + กติกาแบบตรวจได้ แล้วปรับชุดคำสั่งใน `agents/verifier.md` และ allow list ใน `settings.json`
- แก้ kit กลาง (ไม่ใช่ในโปรเจกต์) เมื่อเป็นมาตรฐานที่ทุกโปรเจกต์ควรได้ แล้วค่อย copy ลงโปรเจกต์อื่น

## ปรับ model ของ agent

แก้ `model:` ใน frontmatter ของ `.claude/agents/<name>.md` (`haiku` | `sonnet` | `opus` | `inherit`)
งานที่พลาดแล้วแพง (เช่น reviewer ของระบบการเงิน) เปลี่ยนเป็น `opus` หรือ `inherit` ได้
