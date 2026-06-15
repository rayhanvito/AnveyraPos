# Planning Audit - 2026-06-16

## Scope

This audit reviews the current planning artifacts for Anveyra POS before implementation begins:

- `.taskmaster/docs/prd.md`
- `.taskmaster/tasks/tasks.json`
- `docs/architecture/anveyra-pos-saas-mvp-design.md`
- `README.md`
- `AGENTS.md`

## Verdict

Status: ready for implementation planning.

The product direction is coherent: rebuild from scratch as an offline-first SaaS POS for Indonesian retail UMKM, using Laravel, PostgreSQL, Redis, Flutter, and Next.js. The PRD, task graph, and architecture blueprint agree on the critical decisions: multi-tenant backend, offline mobile sync, stock movement, shift reconciliation, QRIS manual, subscription grace behavior, and Indonesia-friendly mobile UX.

The project is not yet ready for feature coding until Task 1 to Task 3 are completed: monorepo foundation, ERD, and API/sync contract. Those tasks are correctly positioned before backend/mobile implementation.

## Validation Performed

Local checks:

- Task graph JSON parses successfully.
- Task graph contains 18 tasks.
- Task graph contains 69 subtasks.
- Every task has at least 2 subtasks.
- Task dependencies point only to existing task IDs.
- Artifact scan found no unresolved template markers in the new PRD/task graph/README.
- Keyword coverage check confirms PRD and tasks both cover tenant, offline, sync, idempotency, shift, QRIS, transfer, stock movement, RBAC, subscription, audit, adaptive mobile, Laravel, PostgreSQL, Redis, Flutter, and Next.js.
- F&B-related words appear only as exclusions/non-goals, not as implementation scope.

## Strong Decisions

1. Rebuild from scratch instead of mutating the old Flutter POS.
2. Treat the old Flutter app as reference only.
3. Make backend the source of truth.
4. Keep mobile offline-first.
5. Make sync/idempotency a core foundation, not a late feature.
6. Use stock movement as the audit source.
7. Keep the MVP retail-only and exclude F&B.
8. Use mobile PNG templates only as mobile visual direction.
9. Make both phone and tablet first-class mobile targets.
10. Keep web dashboard/admin visually separate from mobile POS.

## Product Risks To Watch

### R1 - MVP Still Large

Even with good scope cuts, this is a platform MVP with backend, mobile, dashboard, admin, and sync. The team must avoid starting all apps at once before ERD and API contracts are stable.

Mitigation: complete Task 1 to Task 3 first; implement backend and sync core before dashboard polish.

### R2 - Sync Complexity

Offline-first POS sync is the hardest technical risk. Failure here can create duplicate transactions, stock mismatch, or lost trust.

Mitigation: prioritize idempotency table, per-item batch ack, dependency handling, SyncLog, and automated sync tests early.

### R3 - Tenant Isolation

Multi-tenant SaaS can fail catastrophically if tenant scoping is inconsistent.

Mitigation: tenant context middleware, server-side policy tests, and cross-tenant negative tests are mandatory before customer dashboard work.

### R4 - Mobile UI Drift

The provided mobile template contains F&B ideas. If copied directly, it will conflict with retail MVP.

Mitigation: AGENTS.md explicitly forbids F&B concepts; mobile design must replace them with retail scan/search/cart/payment/shift states.

### R5 - Printer Scope

Bluetooth printing can consume time because cheap printers vary widely.

Mitigation: define a receipt adapter and ensure failed print never affects completed transactions. Full device compatibility can harden after transaction sync.

## Recommended Implementation Gate

Before writing feature code:

1. Create monorepo layout.
2. Write ERD in `docs/database/erd.md`.
3. Write API/sync contract in `docs/api/`.
4. Decide auth default: Laravel Sanctum unless changed.
5. Decide whether initial dashboard/admin apps are scaffolded immediately or after backend core.

## Professional Recommendation

Proceed with the task graph as written.

Do not start by building UI screens. Start with foundation and contracts:

- Task 1: repository foundation.
- Task 2: ERD.
- Task 3: API and sync contract.
- Task 4 to Task 9: backend and sync core.
- Task 10 onward: mobile shell and local sync implementation.

This order protects the product from the most expensive failure mode: a beautiful POS UI with weak sync and tenant isolation underneath.
