# <KEY> — <summary>

Jira: <JIRA_BASE_URL>/browse/<KEY> · Epic: <EPIC> · รอ: <blockers>
สถานะ: Planning | In Progress | Review | Done

## AC

- [ ] ...

## Design

- ...

## Files

| ไฟล์ | สร้าง/แก้ | เหตุผล |
|---|---|---|

## Steps

1. ...

## Work packages

`single` หรือแบ่งตามตารางเจ้าของไฟล์ (`core/architecture.md`) — ไฟล์หนึ่งอยู่ได้ package เดียว

| package | agent | ไฟล์ | AC | รอ |
|---|---|---|---|---|
| root | session หลัก | dependency / root config | – | – |
| contract | contract-dev | `packages/shared/src/...` | | root |
| backend | backend-dev | `apps/api/...` | | contract |
| frontend | frontend-dev | `apps/web/...` | | contract |
| qa | qa-tester | `e2e/...` | | backend, frontend |

## Test plan

| AC | test |
|---|---|

## Risks / คำถาม

- ...

## ผลลัพธ์

- lint / typecheck / test:
- Runtime (Docker):
- Review:
- ตรวจด้วยมือ:

## Log

- <YYYY-MM-DD HH:mm> <ใคร> — <ทำอะไร / ค้างอะไร / ขั้นต่อไป>
