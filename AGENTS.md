# AGENTS.md - Anveyra POS

This file is the operating guide for AI coding agents and human contributors working in this repository.

## Product Identity

Anveyra POS is a SaaS POS platform for Indonesian retail UMKM: warung, toko kelontong, toko sembako, minimarket lokal, and simple retail shops.

Primary positioning:

> Anveyra POS - kasir modern untuk toko Indonesia. Tetap jalan saat offline.

This is not an F&B POS in the MVP. Do not add table, dine-in, split bill, KDS, modifier, restaurant tax/service charge, or kitchen workflows unless the PRD is explicitly changed.

## Source Of Truth

Use these files in this order:

1. `.taskmaster/docs/prd.md` - canonical PRD.
2. `.taskmaster/tasks/tasks.json` - canonical MVP task graph.
3. `docs/architecture/anveyra-pos-saas-mvp-design.md` - strategy and architecture blueprint.
4. `README.md` - project entry point.

If these documents disagree, stop and update the documents before implementing behavior.

## Required Stack

- Backend/API: Laravel.
- Database: PostgreSQL.
- Queue/cache/rate limit: Redis.
- Mobile POS: Flutter Android-first, Drift + SQLite for offline local storage.
- Customer dashboard: Next.js.
- Internal admin: Next.js.

Do not replace the stack without explicit approval.

## Repository Direction

The product is being rebuilt from scratch as a SaaS platform. The previous Flutter POS project is a reference and component source only. Do not copy its local-only schema or flows directly into this repo without adapting them to tenant, outlet, device, shift, sync, RBAC, subscription, and audit requirements.

Expected structure:

```text
apps/
  mobile-pos/
  customer-dashboard/
  internal-admin/
services/
  api/
packages/
  contracts/
  design-tokens/
docs/
  architecture/
  api/
  database/
```

## Product Rules

- Bahasa Indonesia first for cashier-facing mobile UI.
- Use natural Indonesian copy: `Buka Shift`, `Uang Diterima`, `Kembalian`, `Ada transaksi belum terkirim`.
- Format money as IDR without cents, for example `Rp 25.000`.
- MVP payment methods are cash, QRIS manual/static, manual bank transfer, and optional pay-later.
- QRIS manual is cashier-confirmed, not automatically verified. Show clear warnings.
- Mobile must work comfortably on both Android phones and tablets.
- Tablet landscape uses split catalog/cart layout.
- Phone portrait uses single-column catalog with cart as bottom sheet or floating cart.
- Offline states must reassure users that local data is safe.

## Technical Rules

- Backend is the source of truth.
- Mobile stores local operational data and syncs durable events.
- Every customer-facing backend query must be tenant-scoped.
- `tenant_id` must come from authenticated context, not request body or query string.
- Server-side RBAC is mandatory; do not rely on UI hiding.
- Transactions are append-only after completion except for void status.
- Product/price snapshots are mandatory on transaction items.
- Stock changes must use `StockMovement`; do not silently overwrite stock.
- Sync must be idempotent and duplicate-safe.
- Failed sync must never delete local data automatically.
- Printer failure must never cancel a completed transaction.
- Sensitive actions must write `AuditLog`.

## Sync Rules

Sync is the highest-risk system area. Treat it as core infrastructure, not a feature afterthought.

Required concepts:

- `local_transaction_id` as UUID.
- Local invoice number as display only.
- Sync queue with `pending`, `syncing`, `synced`, `sync_failed`, and `dependency_missing`.
- Per-item batch acknowledgement.
- Explicit dependencies for parent-child events.
- Exponential backoff and manual retry.
- Device sync monitor via `SyncLog`.

## Security Rules

- Passwords use bcrypt or argon2.
- PIN hashes are never stored in plaintext.
- Device tokens are stored securely on mobile.
- Login, PIN verify, sync, and export endpoints must be rate-limited.
- Cross-tenant access tests are mandatory.
- Internal support is read-mostly and all access is audited.

## Testing Expectations

Before claiming work is complete, run the relevant verification:

- Backend: Laravel tests for touched domains.
- Mobile: Flutter analyze and tests for touched features.
- Web: Next.js lint/tests/build for touched apps.
- Docs/task graph: JSON parse and artifact scan.

High-priority test areas:

- Tenant isolation.
- Device activation idempotency.
- Offline transaction creation.
- Duplicate sync.
- Partial batch sync.
- Void approval and reversal.
- Shift expected cash.
- Subscription grace/suspended behavior.
- Mobile phone/tablet layout.

## Implementation Order

Follow `.taskmaster/tasks/tasks.json` unless the PRD changes.

Preferred first sequence:

1. Monorepo foundation.
2. ERD and API contracts.
3. Laravel core, tenant isolation, auth, RBAC.
4. Device activation and master sync.
5. Product and stock movement backend.
6. Transaction, shift, and sync engine.
7. Flutter mobile shell and local database.
8. Mobile cashier flow.
9. Dashboard and internal admin.
10. QA/UAT.

## Design Constraints

The PNG files in `C:\laragon\www\pos\template` are mobile POS visual references only. Do not use them as a web dashboard requirement. Remove F&B-specific elements from any adapted mobile layout.

Mobile UI should feel modern, calm, fast, and Indonesian retail-friendly. Avoid decorative complexity that slows cashier workflows.

## Change Discipline

- Keep changes scoped to the active task.
- Update PRD/task graph/architecture docs when changing product direction.
- Add or update tests when changing behavior.
- Do not introduce new paid external services without approval.
- Do not add out-of-scope MVP features because they are interesting.
