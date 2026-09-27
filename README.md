# DataHub GH — Data Bundle Reselling Platform

A Next.js + TypeScript platform for bulk data bundle ordering and reselling in Ghana,
built around your user stories: landing/auth, dashboard, data packages (single/bulk
text/Excel with validation), agent store, order & transaction history, profile, and
a full admin dashboard.

## Stack
- **Next.js 14 (App Router) + TypeScript** — frontend & API routes
- **PostgreSQL + Prisma** — database
- **NextAuth** — authentication with admin-approval gating
- **Tailwind CSS** — styling
- **Paystack** — card/MoMo payments (manual bank transfer also supported)

## 1. Install dependencies
```bash
npm install
```

## 2. Set up your database
Create a free PostgreSQL database on [Supabase](https://supabase.com) or
[Neon](https://neon.tech), then copy `.env.example` to `.env` and fill in:

```bash
cp .env.example .env
```

Fill in `DATABASE_URL`, `NEXTAUTH_SECRET` (generate with `openssl rand -base64 32`),
and your Paystack keys from https://dashboard.paystack.com/#/settings/developer.

## 3. Push the database schema
```bash
npm run db:generate
npm run db:push
```

## 4. Seed network prefixes, packages, and admin account
```bash
npm run db:seed
```
This creates:
- MTN, Vodafone, AirtelTigo prefix mappings (for the prefix-validation feature)
- Sample data packages for each network
- An admin account (email/password shown in the terminal — **change the password
  immediately after first login**)

## 5. Run the app
```bash
npm run dev
```
Visit http://localhost:3000

## How the pieces map to your user stories

| User Story | Where it lives |
|---|---|
| Landing page, login/signup, password toggle, pending approval | `src/app/(auth)/login`, `src/app/(auth)/signup` |
| Dashboard: wallet, daily orders, transaction history, weekly/monthly filter | `src/app/dashboard` |
| Data Packages: single/bulk text/Excel, Validate button (auto-format, prefix check, duplicate/pending check, amount breakdown) | `src/app/packages`, `src/lib/validation/numberValidator.ts` |
| Agent Store: custom pricing, shareable link, sales tracking | `src/app/agent-store` |
| Order History: 6 statuses, filters | `src/app/history/orders` |
| Transaction History: top-up/purchase breakdown | `src/app/history/transactions` |
| Profile + wallet top-up (Paystack + manual transfer) | `src/app/profile`, `src/components/WalletTopup.tsx` |
| Admin dashboard: approve/reject sign-ups | `src/app/admin` |

## Important notes before going live

1. **Network prefixes** (`prisma/seed.ts`) reflect Ghana's numbering plan as of
   this build — MTN networks add new prefixes occasionally, so double-check
   against the NCA's current allocation before launch.
2. **Manual bank transfer top-ups** are recorded as pending and need an admin
   confirmation endpoint (`/api/admin/topups/confirm`) — a stub for this pattern
   exists in the wallet top-up route as a comment; wire it to your admin UI when
   ready.
3. **Paystack verification** happens server-side (`/api/wallet/topup`) — never
   trust a client-supplied amount, which this code already accounts for.
4. **Real data package fulfillment** (actually sending the bundle to the number)
   requires integrating your MTN/Telco bulk-data API — the order pipeline here
   handles wallet deduction and order tracking, and marks orders `PENDING`
   pending that integration.
5. Change the seeded admin password immediately.

## Project structure
```
src/
  app/            → pages & API routes (App Router)
  components/     → shared UI (Sidebar, AppShell, WalletTopup)
  lib/
    auth.ts       → NextAuth config
    db.ts         → Prisma client
    validation/   → the core number-validation engine
prisma/
  schema.prisma   → database schema
  seed.ts         → seed data
```
