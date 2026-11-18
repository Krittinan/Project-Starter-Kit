---
paths:
  - "apps/web/**"
---

# Frontend (Next.js App Router + Material UI)

## โครงสร้าง

- `src/app/` มีแค่ route, layout, `loading.tsx`, `error.tsx` — ไม่มี logic; หน้าเรียก component จาก `src/features/<feature>/`
- `src/features/<feature>/`: `components/`, `hooks/` (TanStack Query เช่น `useItems`, `useCreateItem`), `api.ts`
- `src/components/`: UI ที่ใช้หลาย feature เท่านั้น (DataTable, ConfirmDialog, FormErrors ...)
- เรียก backend ผ่าน `src/lib/api-client.ts` ที่เดียว: parse ด้วย schema จาก `@repo/shared` หรือโยน `ApiError { status, code, message, details }`

## Server / Client

- ค่าเริ่มต้นเป็น Server Component; `'use client'` เฉพาะ component ที่มี state, event หรือ hook
- หน้าที่โต้ตอบ (ตาราง, ฟอร์ม) เป็น client component + TanStack Query; หลัง mutation invalidate query ที่เกี่ยวข้อง
- เบราว์เซอร์เรียก `/api/*` (rewrite ใน `next.config.ts` ไป `API_INTERNAL_URL`) — ห้าม hard-code host และห้ามเก็บค่าลับใน `NEXT_PUBLIC_*`
- `next.config.ts` ตั้ง `output: 'standalone'`

## Material UI

- `app/layout.tsx` ห่อด้วย `AppRouterCacheProvider` (`@mui/material-nextjs`) + `ThemeProvider` + `CssBaseline` ใน `src/theme/ThemeRegistry.tsx`
- สี ตัวอักษร ระยะ มาจาก theme (`src/theme/theme.ts`) เท่านั้น ห้าม hard-code สี; `sx` ใช้กับ layout เฉพาะจุด
- ฟอนต์ผ่าน `next/font` ตั้งใน theme typography (ภาษาไทย: เช่น IBM Plex Sans Thai)
- ตารางใช้ `DataGrid` (`@mui/x-data-grid`), วันเวลาใช้ `@mui/x-date-pickers`
- `Link` ของ Next ส่งเข้า prop `component` ของ MUI ผ่าน wrapper ที่เป็น client component

## ฟอร์มและ error

- react-hook-form + `zodResolver` กับ schema เดียวกับที่ API ใช้
- error จาก API: `details[].field` → `setError` ช่องที่ตรงกัน; ที่ไม่มีช่องรองรับแสดงใน `Alert` บนฟอร์ม
- ปุ่มที่ทำลาย/ปิดข้อมูลต้องผ่าน `ConfirmDialog`

## ความถูกต้องและการเข้าถึง

- ห้ามคำนวณกฎธุรกิจ (ราคา ส่วนลด สิทธิ์) ใน UI — เรียก API
- `<html lang>` ตรงภาษา UI; ทุก input มี label; ปุ่ม icon มี `aria-label`
- ห้าม `dangerouslySetInnerHTML` กับข้อมูลจาก API
