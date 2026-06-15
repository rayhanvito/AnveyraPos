# PRD - Anveyra POS SaaS MVP

Version: 1.0
Date: 2026-06-16
Status: MVP build baseline
Owner: Anveyra POS

## 1. Executive Summary

Anveyra POS is a cloud SaaS and offline-first point of sale platform for Indonesian retail UMKM. It is designed for warung, toko kelontong, toko sembako, minimarket lokal, simple small wholesalers, and general small retail stores that want to move from manual notes, calculators, and spreadsheets into a reliable digital cash register.

The product has three applications:

- Mobile POS: Flutter Android app for cashiers and supervisors. It must work on both phones and tablets, keep selling offline, and sync when internet is available.
- Customer Dashboard: Next.js web app for owners and admins to manage products, stock, users, outlets, reports, QRIS settings, and subscription status.
- Internal Admin: Next.js web app for the Anveyra team to manage tenants, subscriptions, support troubleshooting, device sync monitoring, and audit logs.

The backend is a Laravel API with PostgreSQL and Redis. It is the system of record for tenants, users, products, stock movements, synced transactions, reports, subscription status, and audit logs.

The MVP must be Indonesia-friendly: Bahasa Indonesia first, IDR formatting without cents, QRIS static/manual payments, manual bank transfer, cash reconciliation per shift, cheap Android devices, Bluetooth thermal printer support, and copy that feels calm and practical for shop staff.

## 2. Product Positioning

Primary positioning:

> Anveyra POS - kasir modern untuk toko Indonesia. Tetap jalan saat offline.

Secondary positioning:

> Aplikasi kasir simpel untuk toko, warung, dan UMKM retail.

Anveyra POS competes with products such as Majoo, Moka, Olsera, Kasir Pintar, and POST, but the MVP does not try to match every feature. It wins by being simple, affordable, reliable when the internet is unstable, and focused on the daily needs of Indonesian small retail stores.

## 3. Target Users

### 3.1 Owner

The owner buys and pays for the product. They want to know daily sales, prevent cash leakage, see best-selling items, check low stock, and manage the store without waiting for manual recaps.

### 3.2 Cashier

The cashier uses the mobile POS daily. They need a fast and forgiving interface: PIN login, big buttons, barcode scan, quick cash payment, clear change amount, printable receipt, and offline mode that does not panic them.

### 3.3 Supervisor

The supervisor approves voids, monitors shifts, checks cash differences, and helps with operational exceptions.

### 3.4 Admin

The admin manages products, categories, stock adjustments, imports, users, PIN resets, outlet settings, receipt settings, QRIS images, and bank account information.

### 3.5 Internal Support

Internal support helps tenants with activation, sync issues, subscription status, and troubleshooting. Support is read-mostly and all access must be audited.

## 4. Goals

### 4.1 Business Goals

- Launch a sellable MVP for Indonesian retail UMKM.
- Validate with 5 to 15 beta merchants.
- Support subscription billing manually at first.
- Build a foundation that can later support multi-outlet, payment gateway integration, loyalty, and accounting.

### 4.2 User Goals

- Cashiers complete a normal transaction in under 15 seconds.
- Owners see synced sales reports from anywhere.
- Stores keep selling when internet is offline or unstable.
- Shift close shows expected cash, actual cash, and cash difference clearly.
- Stock changes are traceable through stock movements.

### 4.3 Technical Goals

- Multi-tenant isolation on every customer-facing data path.
- Offline-first mobile data model using SQLite and Drift.
- Idempotent sync with no duplicate transactions.
- Server-side RBAC enforcement, not just UI hiding.
- Audit logs for sensitive actions.
- API versioning under `/api/v1`.

## 5. MVP Scope

### 5.1 In Scope

Mobile POS:

- Device activation with outlet activation code.
- Initial master sync.
- Offline-capable PIN login.
- Open shift with opening cash.
- Cash in and cash out during shift.
- Product catalog with categories, search, SKU, barcode, and stock indicators.
- Barcode scanner using camera.
- Cart, quantity edit, item delete, simple discount, and transaction note.
- Cash payment with received amount and change.
- QRIS static/manual payment with confirmation warning.
- Manual bank transfer payment with reference or reason.
- Optional pay-later status if implementation capacity allows.
- Transaction success screen.
- Bluetooth thermal receipt printing, or at minimum receipt reprint/share-ready structure if printer support is deferred.
- Transaction history and detail with sync and void status.
- Void flow with approver PIN and mandatory reason.
- Close shift with reconciliation.
- Sync center with pending count, failed queue, and retry.
- Adaptive phone and tablet layouts.

Customer Dashboard:

- Owner/admin login.
- Sales dashboard.
- Transaction report and detail.
- Transaction export to CSV; Excel can be added if feasible.
- Product and category management.
- Product import with validation summary.
- Stock movement history.
- Stock in and adjustment.
- User and role management.
- PIN reset.
- Outlet, receipt, logo, QRIS image, bank account, and printer settings.
- Device management and activation code generation.
- Subscription status view.

Internal Admin:

- Internal login.
- Tenant list and tenant detail.
- Tenant create/edit/suspend.
- Manual subscription management.
- Support view for tenant summary and device sync status.
- Device sync monitor.
- Audit log search/filter.

Cross-cutting:

- Multi-tenant API.
- RBAC.
- Audit log.
- Sync log.
- Subscription status and grace behavior.
- Basic observability for sync failures.

### 5.2 Out of Scope

The MVP excludes:

- F&B mode: table, dine-in, split bill, KDS, modifiers, menu variants.
- Dynamic QRIS and automatic payment gateway settlement.
- Full multi-outlet operational UI.
- Accounting and journal automation.
- Loyalty/member points.
- Supplier and purchase order management.
- Marketplace integration.
- Payroll.
- AI insights.
- Advanced tax and service charge logic.
- Partial refund and complex return workflows.

## 6. Product Principles

1. Indonesia-first: Bahasa Indonesia, rupiah, QRIS, transfer, thermal printer, and familiar shop workflows.
2. Offline-safe: if the cashier has already created a transaction locally, the app must preserve it until sync succeeds.
3. Money clarity: total, received amount, change, payment method, and cash difference must be visually obvious.
4. Stock movement first: every stock change is an auditable movement.
5. Server-enforced security: permissions and tenant isolation are enforced in backend logic.
6. Simple before complete: remove features that slow down cashier flow or delay MVP validation.

## 7. Platform Architecture

Recommended repository structure:

```text
AnveyraPos/
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

Runtime stack:

- Backend: Laravel.
- Database: PostgreSQL.
- Queue/cache/rate limit: Redis.
- Mobile: Flutter, Android first, Drift and SQLite.
- Customer dashboard: Next.js.
- Internal admin: Next.js.

Backend is the source of truth. Mobile stores operational data locally and sends durable events to backend through idempotent sync.

## 8. Core Data Model

Every customer-facing table must be tenant-scoped unless explicitly internal.

### 8.1 Tenant

Fields: `id`, `name`, `business_type`, `owner_user_id`, `status`, `created_at`, `updated_at`.

Rules:

- One UMKM business equals one tenant.
- Tenant status controls access and subscription behavior.
- All customer data queries derive `tenant_id` from auth context.

### 8.2 Outlet

Fields: `id`, `tenant_id`, `outlet_code`, `name`, `address`, `phone`, `qris_image_url`, `bank_accounts`, `receipt_footer`, `logo_url`, `status`.

Rules:

- MVP UI focuses on one outlet.
- Data model must support more than one outlet later.
- `outlet_code` is used in local invoice number display.

### 8.3 User, Role, Permission

User fields: `id`, `tenant_id`, `name`, `email`, `password_hash`, `pin_hash`, `role_id`, `status`, `last_login_at`.

Rules:

- Web login uses password.
- Mobile cashier login uses PIN.
- PIN hash is synced locally so cashier can login offline.
- RBAC is permission-key based and enforced by the API.

### 8.4 Device

Fields: `id`, `tenant_id`, `outlet_id`, `device_id`, `device_install_id`, `name`, `activation_status`, `last_sync_at`, `last_seen_at`, `pending_count`, `last_pending_count`.

Rules:

- First activation requires internet.
- One device is tied to one outlet at a time.
- `device_install_id` changes after reinstall or clear data.
- Device is part of idempotency and support monitoring.

### 8.5 Shift

Fields: `id`, `tenant_id`, `outlet_id`, `device_id`, `user_id`, `local_shift_id`, `opening_cash`, `closing_actual_cash`, `expected_cash_provisional`, `expected_cash_final`, `cash_diff`, `recap_status`, `status`, `opened_at`, `closed_at`, `sync_status`.

Rules:

- Cashier cannot transact before opening a shift.
- One active shift per user/device.
- Offline shift close is allowed with `recap_status=provisional`.
- Final recap is calculated after all related sync items are synced.

### 8.6 Product and Category

Product fields: `id`, `tenant_id`, `category_id`, `name`, `sku`, `barcode`, `sell_price`, `cost_price`, `stock`, `base_unit`, `min_stock`, `is_active`, `image_url`.

Category fields: `id`, `tenant_id`, `name`, `is_active`, `sort_order`.

Rules:

- SKU and barcode are unique per tenant when present.
- Product stock is a cache; StockMovement is the audit source.
- Product and price changes only affect mobile after master sync.

### 8.7 StockMovement

Fields: `id`, `tenant_id`, `outlet_id`, `product_id`, `type`, `qty_delta`, `ref_type`, `ref_id`, `note`, `is_negative_flag`, `created_at`.

Types: `sale`, `void_reversal`, `in`, `adjustment`, `import`.

Rules:

- No stock overwrite without movement.
- Offline sales are accepted even if stock becomes negative.
- Negative stock is flagged, not rejected.

### 8.8 Transaction

Fields: `id`, `tenant_id`, `outlet_id`, `device_id`, `shift_id`, `cashier_user_id`, `local_transaction_id`, `local_invoice_no`, `server_ref_no`, `subtotal`, `discount_total`, `grand_total`, `status`, `sync_status`, `note`, `created_at_local`, `synced_at`.

Rules:

- `local_transaction_id` is UUID and is the idempotency identity.
- Local invoice number is display only.
- Transaction is append-only after completion except void status.

### 8.9 TransactionItem

Fields: `id`, `transaction_id`, `product_id`, `product_name`, `sku`, `unit_price`, `cost_price`, `qty`, `discount`, `line_total`.

Rules:

- Item stores product snapshot at transaction time.
- Historical receipts never depend on current product master.

### 8.10 Payment

Fields: `id`, `transaction_id`, `method`, `amount`, `received_amount`, `change_amount`, `reference`, `status`, `metadata`.

Methods: `cash`, `qris_manual`, `transfer_manual`, `pay_later`.

Rules:

- Cash contributes to expected physical cash.
- QRIS and transfer are separated in reports and shift recap.
- QRIS manual status means cashier-confirmed, not automatically verified.
- Transfer manual requires reference or reason.

### 8.11 CashMovement

Fields: `id`, `tenant_id`, `outlet_id`, `shift_id`, `type`, `amount`, `note`, `local_id`, `sync_status`, `created_at`.

Rules:

- Note is mandatory.
- Cash out above configurable threshold requires supervisor/owner approval.

### 8.12 VoidApproval

Fields: `id`, `tenant_id`, `transaction_id`, `approved_by_user_id`, `reason`, `approved_at`, `local_void_id`, `sync_status`.

Rules:

- Cashier cannot approve void.
- Void requires approver PIN and reason.
- Void creates stock reversal movement.

### 8.13 SubscriptionPlan and TenantSubscription

Plan fields: `id`, `name`, `price`, `limits`, `features`, `status`.

Subscription fields: `id`, `tenant_id`, `plan_id`, `status`, `trial_ends_at`, `current_period_end`, `last_payment_at`, `notes`.

Rules:

- MVP billing is manual.
- Expired tenants get a 7-day grace period.
- Suspended tenants can only sync pending data and access billing.

### 8.14 AuditLog and SyncLog

AuditLog fields: `id`, `tenant_id`, `actor_user_id`, `actor_type`, `action`, `target_type`, `target_id`, `metadata`, `ip`, `created_at`.

SyncLog fields: `id`, `tenant_id`, `outlet_id`, `device_id`, `entity_type`, `local_id`, `result`, `error_code`, `error_message`, `created_at`.

Rules:

- Audit log records sensitive actions.
- Sync log powers support troubleshooting and internal monitoring.

## 9. Offline Sync Requirements

### 9.1 Local Storage

Mobile local DB stores:

- Master data: products, categories, user PIN hashes, outlet settings, QRIS, bank accounts.
- Shift state.
- Transactions, items, and payments.
- Cash movements.
- Local stock movements.
- Sync queue.
- Last sync cursor and last known server time.

### 9.2 Sync Queue

Every syncable local event has:

- `local_id`
- `entity_type`
- `payload`
- `depends_on`
- `status`
- `attempt_count`
- `last_error_code`
- `last_error_message`
- `next_retry_at`
- `created_at`
- `updated_at`

Statuses: `pending`, `syncing`, `synced`, `sync_failed`, `dependency_missing`.

### 9.3 Sync Behavior

- Sync automatically when online and manually from Sync Center.
- Retry uses exponential backoff.
- Batch API returns per-item acknowledgements.
- Mobile marks only successful items as synced.
- Failed items remain locally and can be retried.
- Child entities wait for parent transaction if needed.

### 9.4 Idempotency

Idempotency key uses:

- tenant
- outlet
- device or device install
- entity type
- local id

Server must return the previous successful result for repeated keys and must not create duplicates.

### 9.5 Master Pull

Master data pull uses cursor or `updated_at` watermark and includes:

- products
- categories
- users and PIN hashes
- roles and permissions relevant to mobile
- outlet settings
- subscription local enforcement fields
- server time

## 10. Mobile POS UX Requirements

The PNG templates in `C:\laragon\www\pos\template` are mobile visual references only. They are not web dashboard references.

### 10.1 Visual Direction

- Clean white surfaces.
- Navy/blue brand color.
- Green for success and safe states.
- Large touch targets.
- Strong total and change display.
- Clear online/offline and pending sync badges.
- Indonesian copy that reduces anxiety.

### 10.2 Remove F&B Concepts

The MVP must not include:

- Dine in.
- Table.
- Split bill.
- Customer management as a cashier requirement.
- Kitchen or menu modifier concepts.

### 10.3 Adaptive Layout

Tablet landscape:

- Left: product catalog, search, category, barcode button.
- Right: cart, summary, total, payment CTA.
- Persistent offline and pending sync indicators.

Phone portrait:

- Single-column catalog.
- Floating cart button or bottom sheet.
- Barcode/search always reachable.
- Payment flow optimized for one-handed use.

Tablet portrait and large phones:

- Hybrid catalog with cart summary panel or bottom sheet.

### 10.4 Mobile Screen List

Required screens and states:

1. Hubungkan Perangkat.
2. Sinkronisasi Awal.
3. Pilih User dan Masuk PIN.
4. Buka Shift.
5. Home POS.
6. Pencarian dan Barcode Scanner.
7. Keranjang.
8. Pilih Pembayaran.
9. Pembayaran Tunai.
10. Pembayaran QRIS Manual.
11. Pembayaran Transfer.
12. Transaksi Berhasil.
13. Gagal Cetak.
14. Riwayat Transaksi.
15. Detail Transaksi.
16. Void dengan Approval.
17. Kas Masuk/Keluar.
18. Tutup Shift.
19. Sync Center.
20. Pengaturan Device dan Printer.

### 10.5 Copy Examples

Use:

- `Hubungkan Perangkat`
- `Masuk Kasir`
- `Buka Shift`
- `Mulai Transaksi`
- `Pilih Pembayaran`
- `Uang Diterima`
- `Kembalian`
- `Pembayaran Diterima`
- `Transaksi Berhasil`
- `Ada transaksi belum terkirim`
- `Internet putus, transaksi tetap bisa berjalan`
- `Data aman dan akan dikirim saat online`

Avoid:

- Technical jargon such as queue, payload, idempotency, mutation.
- English labels in cashier flow unless no good Indonesian equivalent exists.

## 11. API Requirements

All customer API endpoints are tenant-aware and derive tenant from the authenticated token.

### 11.1 Auth and Device

- `POST /api/v1/auth/login`
- `POST /api/v1/auth/logout`
- `POST /api/v1/auth/device/activate`
- `POST /api/v1/auth/pin/verify`

### 11.2 Master Data

- `GET /api/v1/sync/master`
- `GET /api/v1/products`
- `POST /api/v1/products`
- `PUT /api/v1/products/{id}`
- `DELETE /api/v1/products/{id}`
- `POST /api/v1/products/import`
- `GET /api/v1/categories`
- `POST /api/v1/categories`
- `PUT /api/v1/categories/{id}`

### 11.3 Sync

- `POST /api/v1/sync/batch`
- `POST /api/v1/sync/transactions`
- `POST /api/v1/sync/voids`
- `POST /api/v1/sync/shifts`
- `GET /api/v1/sync/status`

The preferred endpoint is `POST /sync/batch` with per-item acknowledgement. Specific endpoints can be implemented internally or kept for compatibility if simpler.

### 11.4 Reports

- `GET /api/v1/reports/sales-summary`
- `GET /api/v1/reports/transactions`
- `GET /api/v1/reports/shifts`
- `GET /api/v1/reports/stock-movements`
- `POST /api/v1/reports/export`

### 11.5 Dashboard and Settings

- `GET /api/v1/outlets`
- `PUT /api/v1/outlets/{id}/settings`
- `GET /api/v1/users`
- `POST /api/v1/users`
- `PUT /api/v1/users/{id}`
- `POST /api/v1/users/{id}/reset-pin`
- `GET /api/v1/devices`
- `POST /api/v1/devices/activation-code`
- `POST /api/v1/devices/{id}/revoke`

### 11.6 Internal Admin

- `POST /api/v1/internal/tenants`
- `GET /api/v1/internal/tenants`
- `GET /api/v1/internal/tenants/{id}`
- `PATCH /api/v1/internal/tenants/{id}/status`
- `PUT /api/v1/internal/tenants/{id}/subscription`
- `GET /api/v1/internal/tenants/{id}/devices`
- `GET /api/v1/internal/audit-logs`

## 12. Security Requirements

- Web passwords use bcrypt or argon2.
- PIN hashes are never stored in plaintext.
- Mobile stores device token securely.
- Login, PIN verify, sync, and export endpoints are rate-limited through Redis.
- RBAC is enforced server-side.
- `tenant_id` from request body/query is ignored or rejected.
- Internal support access is read-only by default.
- Sensitive actions require audit log.
- HTTPS is required in production.
- File upload is controlled by storage rules or presigned URL.
- Cross-tenant test cases are mandatory.

## 13. Subscription Requirements

Plans:

- Basic: 1 outlet, up to 3 users, basic reports, CSV export.
- Pro: 1 outlet in MVP, up to 10 users, fuller reports, CSV and Excel export if available, priority support.

Trial:

- 14 days.
- Trial acts like Pro for adoption.

Expired:

- 7-day grace period.
- Cashier can keep transacting during grace.
- Dashboard becomes read-friendly with billing access for Owner.

Suspended:

- New shift and new transaction blocked.
- Pending sync remains allowed.
- Data remains safe.

Offline enforcement:

- Mobile stores `subscription_valid_until`, `last_subscription_sync_at`, and last known server time.
- Gate is evaluated when opening shift, not on every transaction.
- Use `max(device_time, last_known_server_time)` to reduce clock manipulation.

## 14. Reporting Requirements

MVP reports:

- Sales today.
- Transaction count.
- Average transaction value.
- Payment method breakdown.
- Best-selling products.
- Transactions by cashier.
- Shift recap.
- Stock minimum.
- Stock movement.
- Void report.

Rules:

- Dashboard reports only synced data.
- Reports are tenant and outlet scoped.
- Export is async if dataset is large.
- Gross profit is not shown in MVP unless cost data is reliable; defer profit reporting to V1.1.

## 15. Non-Functional Requirements

- API p95 under 400 ms for common endpoints.
- Mobile app open under 3 seconds on target devices after first install.
- Adding item to cart feels instant.
- Standard checkout under 15 seconds.
- Offline transaction never lost after local save.
- Sync is idempotent and duplicate-safe.
- Printer failure does not cancel transaction.
- Backend has daily database backups and restore testing.
- Sync failures produce structured logs.
- Migrations are versioned and reversible when practical.

## 16. Acceptance Criteria

1. Device activation succeeds only with a valid activation code and online connection.
2. Initial sync downloads master data and allows offline use.
3. Cashier PIN login works offline after initial sync.
4. Transaction is blocked until shift is open.
5. Cash transaction calculates change correctly.
6. QRIS manual requires cashier confirmation and is reported separately.
7. Transfer manual requires reference or reason.
8. Offline transaction gets `pending_sync` status and local invoice number.
9. Duplicate sync request does not create duplicate server transaction.
10. Partial batch sync marks successful items synced and keeps failed items retryable.
11. Void requires approver PIN and reason.
12. Void reversal restores stock through StockMovement.
13. Shift close calculates expected cash and difference.
14. Closed shift with pending sync is provisional until all related items sync.
15. Dashboard does not show unsynced local-only data.
16. User from tenant A cannot read or write tenant B data.
17. Expired tenant follows grace behavior.
18. Suspended tenant can sync pending data but cannot create new transactions.
19. Support access is audited.
20. Printer failure does not undo completed transaction.

## 17. Milestones

Phase 0 - Product Foundation:

- PRD, architecture, ERD, API contract, repo structure, CI baseline, design tokens.

Phase 1 - Backend Core:

- Laravel app, tenant isolation, auth, RBAC, core migrations, seed data, audit log.

Phase 2 - Sync Foundation:

- Device activation, master sync, sync queue contract, idempotency storage, sync logs.

Phase 3 - Mobile POS Core:

- Flutter shell, local Drift schema, PIN login, shift, catalog, cart, payments, local transaction save.

Phase 4 - Transaction Sync:

- Offline transaction sync, payments, voids, shifts, cash movement, stock movements.

Phase 5 - Customer Dashboard:

- Reports, products, import, stock, users, settings, activation code, subscription view.

Phase 6 - Internal Admin:

- Tenant, subscription, support sync monitor, audit log.

Phase 7 - QA and Beta:

- End-to-end testing, UAT with 5 to 15 merchants, bug fixes, Play Store readiness.

## 18. Task Decomposition Guidance

This is a team/startup-scale MVP. The initial task graph should contain 16 to 20 tasks with 3 to 5 subtasks each. Implementation should start with backend and sync contracts, then mobile shell and local schema, then transaction flow, then dashboard/admin.

## 19. Open Decisions

Default decisions for MVP unless changed:

- Use Laravel Sanctum for first-party API token auth.
- Reserve PostgreSQL RLS as defense-in-depth after application tenant isolation tests pass.
- Include pay-later as a Should Have, not blocking MVP launch.
- Implement Bluetooth printing after transaction local save and sync architecture are stable.
- Use mobile PNG templates only as mobile visual direction.
- Web dashboard and internal admin get their own admin-friendly UI later.

