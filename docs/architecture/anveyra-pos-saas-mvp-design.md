# Anveyra POS SaaS MVP Design

Date: 2026-06-16
Status: Draft for user review
Source PRD: `C:\laragon\www\pos\PRD Final - SaaS POS UMKM Indonesia (MVP).md`

## 1. Product Direction

Anveyra POS is a SaaS POS platform for Indonesian retail UMKM: warung, toko kelontong, minimarket lokal, sembako shops, and simple small wholesalers. The product should feel modern and trustworthy, but still practical for non-technical users.

Positioning:

> Anveyra POS - kasir modern untuk toko Indonesia. Tetap jalan saat offline.

The MVP focuses on retail, not F&B. It must avoid restaurant concepts such as tables, dine-in, split bill, kitchen display, modifiers, or service charge flows.

Primary user outcomes:

- Cashiers can sell quickly on Android phone or tablet.
- Owners can see sales and stock from anywhere after sync.
- Shops can keep selling when internet is unstable.
- Shift cash reconciliation separates cash, QRIS manual, transfer, and pay-later.
- Stock changes are traceable through movements, not silent overwrites.

## 2. Build Strategy

Build a new SaaS platform instead of heavily rewriting the existing Flutter app in place.

Decision:

- Create a new platform/monorepo for the SaaS product.
- Use the existing Flutter POS repository as a reference and component source.
- Port useful parts selectively after the backend domain and sync contracts are clear.

Reasoning:

- The existing app is a strong offline local POS, but the SaaS MVP needs tenant, outlet, device, shift, sync queue, RBAC, subscription, and audit concepts from the foundation.
- Retrofitting those concepts into the current schema and flows would create high migration risk.
- A new app can model sync and multi-tenant rules correctly from day one while still reusing proven pieces.

Reusable from the existing codebase:

- Clean Architecture structure.
- Drift/SQLite local persistence patterns.
- Cart and checkout business behavior where compatible.
- Barcode scanner flow.
- Receipt/PDF generation ideas.
- Product image handling ideas.
- Existing test coverage patterns.
- Selected UI widgets after visual redesign.

Do not directly reuse:

- Current local-only database schema as the SaaS schema.
- Current app startup/onboarding as the SaaS activation flow.
- Current settings model as the final SaaS settings model.
- Current sale/history/report model without tenant, outlet, device, shift, and sync fields.

## 3. Platform Architecture

Recommended repository shape:

```text
anveyra-pos/
  apps/
    mobile-pos/          Flutter Android POS, offline-first
    customer-dashboard/  Next.js owner/admin dashboard
    internal-admin/      Next.js startup admin/support/billing
  services/
    api/                 Laravel API
  packages/
    contracts/           OpenAPI/schema/type contract artifacts if needed
    design-tokens/       Shared brand tokens if needed
  docs/
    prd/
    architecture/
    api/
    database/
```

Core runtime stack:

- Backend/API: Laravel.
- Database: PostgreSQL.
- Queue/cache/rate limit: Redis.
- Mobile POS: Flutter, Android-first, local SQLite with Drift.
- Web customer dashboard: Next.js.
- Web internal admin: Next.js.
- API style: REST JSON under `/api/v1`.

Backend is the source of truth. Mobile stores local operational state and syncs durable events to the server.

## 4. Core Domain Model

The SaaS domain must be designed before UI implementation.

Required entities:

- Tenant
- Outlet
- User
- Role
- Permission
- Device
- Shift
- Product
- Category
- StockMovement
- Transaction
- TransactionItem
- Payment
- Receipt
- CashMovement
- VoidApproval
- AuditLog
- SubscriptionPlan
- TenantSubscription
- ImportJob
- SyncLog

Rules:

- Every customer-facing entity is tenant-scoped.
- `tenant_id` is derived from auth context, never trusted from request body/query.
- Outlet is present from MVP even though the UI starts with one outlet.
- Device identity is mandatory for mobile sync, invoice sequence, and support monitoring.
- Transaction item data must use product/price snapshots.
- Stock is movement-based. Product stock can exist as a cache, but not as the audit source.

## 5. Offline Sync Design

Sync is the highest-risk part of the product and must be implemented before broad feature expansion.

Mobile local database stores:

- Master data: product, category, users/PIN hash, outlet settings, QRIS image reference, bank account settings.
- Active and historical shifts.
- Transactions, transaction items, payments.
- Cash movements.
- Local stock movements.
- Sync queue.
- Sync metadata: cursor, last sync time, last known server time.

Required sync behavior:

- First device activation requires internet.
- After activation, mobile can sell offline.
- Each transaction has a stable `local_transaction_id` UUID.
- Local invoice number is display-only, not the unique system key.
- Sync queue uses per-item status: `pending`, `syncing`, `synced`, `sync_failed`, `dependency_missing`.
- Batch sync returns per-item acknowledgement.
- Retry uses exponential backoff and manual retry.
- Parent-child dependencies are explicit: transaction before items/payments/void events.
- Server APIs are idempotent by tenant, outlet, device or device install, and local id.
- Failed sync never deletes local data automatically.

Critical user-facing states:

- Online/offline indicator.
- Pending sync count.
- Sync failed detail and retry.
- Warning when pending sync exceeds 20 transactions or 6 hours.
- Warning before logout/revoke when there are unsynced items.

## 6. Mobile POS UX Direction

The PNG templates in `C:\laragon\www\pos\template` define the mobile visual direction only. They should not define web dashboard UI.

Visual style:

- Clean white surfaces.
- Navy/blue primary brand.
- Green success/accent for completed payments, online state, and safe cash outcomes.
- Large touch targets.
- Calm operational copy.
- Strong money hierarchy.
- Clear offline and pending sync badges.

The current template has F&B concepts that must be removed:

- `Dine In`
- `Tambah Pelanggan`
- `Pisah Bill`
- Food/drink category assumptions

Replace with retail POS concepts:

- Scan barcode.
- Search by product name, SKU, or barcode.
- Product categories for retail goods.
- Stock low/out-of-stock warnings.
- Current shift and cashier.
- Pending sync.
- Cash, QRIS manual, transfer, pay-later.

Adaptive layout:

- Tablet landscape: split-screen cashier layout. Catalog on the left, cart and payment summary on the right.
- Phone portrait: single-column catalog. Cart opens as a bottom sheet or floating cart button.
- Tablet portrait / large phones: hybrid layout with persistent summary when there is enough width.
- Same business logic across layouts. Only presentation changes.

Priority mobile screens:

1. Device activation.
2. Initial sync.
3. User selection and PIN login.
4. Open shift with opening cash.
5. POS catalog and cart.
6. Barcode scanner.
7. Cart review.
8. Payment method selection.
9. Cash payment.
10. QRIS manual confirmation.
11. Transfer manual confirmation.
12. Transaction success and print/share receipt.
13. Print error state.
14. Transaction history.
15. Transaction detail.
16. Void approval.
17. Cash in/out.
18. Close shift.
19. Sync center.
20. Minimal device/printer/settings area.

## 7. Indonesia-Friendly Product Rules

Language must be natural Indonesian, not translated enterprise jargon.

Preferred copy examples:

- `Hubungkan Perangkat`
- `Masuk Kasir`
- `Buka Shift`
- `Kas Awal`
- `Mulai Transaksi`
- `Pilih Pembayaran`
- `Uang Diterima`
- `Kembalian`
- `Pembayaran Diterima`
- `Transaksi Berhasil`
- `Tutup Shift`
- `Ada transaksi belum terkirim`
- `Internet putus, transaksi tetap bisa berjalan`
- `Data aman dan akan dikirim saat online`

Local defaults:

- Currency: IDR, no decimal cents.
- Payment methods: cash, QRIS manual/static, manual bank transfer, pay-later.
- Hardware: low-cost Android phone/tablet and Bluetooth thermal printer 58mm/80mm.
- Subscription billing: manual transfer in MVP.
- QRIS manual must show a confirmation warning because payment is not automatically validated.

## 8. Web Product Scope

Web dashboard and internal admin should have their own admin-focused UI later. Do not force the mobile PNG style directly onto the web apps.

Customer dashboard MVP:

- Sales dashboard.
- Transaction report and export.
- Product/category management.
- Product import.
- Stock movement and stock adjustment.
- User/role management.
- Outlet settings, receipt settings, logo, QRIS image, bank accounts.
- Device activation code management.
- Subscription status view.

Internal admin MVP:

- Internal login.
- Tenant list/detail.
- Subscription management.
- Support troubleshooting view.
- Device sync monitor.
- Audit log.

Internal admin should remain lean until the product has more tenants.

## 9. Testing And Acceptance Strategy

Testing must focus first on correctness of money, stock, sync, and tenant isolation.

Required test groups:

- Cross-tenant access denial.
- Device activation idempotency.
- PIN login and lockout.
- Open/close shift.
- Cash movement expected cash calculation.
- Offline transaction creation.
- Duplicate sync idempotency.
- Partial batch sync ack.
- Void approval and void sync dependency.
- Stock movement on sale and void reversal.
- Local invoice sequence stability.
- Subscription expired/grace/suspended behavior.
- Dashboard shows only synced data.
- Printer failure does not cancel transaction.

## 10. First Implementation Track

Recommended first implementation order:

1. Create SaaS platform repository structure.
2. Write ERD and API contract for tenant, outlet, user, role, device, product, shift, transaction, payment, stock movement, sync, subscription, and audit.
3. Build Laravel auth, tenant isolation, RBAC, and device activation.
4. Build master data endpoints and initial sync contract.
5. Build Flutter SaaS app shell with activation, initial sync, PIN login, shift open, and a basic POS home skeleton.
6. Build local Drift schema for SaaS mobile sync.
7. Build transaction creation offline and sync queue.
8. Build transaction sync endpoint with idempotency and per-item ack.
9. Build customer dashboard only after product, device, transaction, and report APIs are stable.
10. Build internal admin minimum after tenant/subscription/support APIs exist.

## 11. Open Decisions

These decisions should be finalized before implementation planning:

- Final monorepo tooling and package manager.
- Laravel auth choice: Sanctum or Passport.
- Whether PostgreSQL RLS is used in MVP or reserved as defense-in-depth later.
- Exact brand color tokens for Anveyra POS.
- Whether mobile printer support is implemented in MVP phase 1 or after transaction sync is stable.
- Whether pay-later is included in MVP phase 1 or treated as Should Have.

## 12. Non-Goals For MVP

- F&B mode.
- Dynamic QRIS/payment gateway settlement.
- Full multi-outlet UI.
- Accounting/journal automation.
- Loyalty/member points.
- Marketplace integrations.
- AI insights.
- Payroll.
- Advanced tax/service charge logic.
