---
name: mysql-best-practices
description: >-
  Use when writing or reviewing MySQL schemas, queries, migrations, or app DB
  access. Covers InnoDB schema design, indexes, queries, transactions,
  connections, security, and operations. Triggers on MySQL/.sql for MySQL,
  designing MySQL tables, ORMs against MySQL, index tuning, InnoDB, or MySQL
  migrations. Do not use for Postgres, SQLite, or generic SQL that is not MySQL.
license: MIT
metadata:
  author: mengtaoxin
  version: "1.1.0"
  docs: https://dev.mysql.com/doc/
---

# MySQL best practices

Apply these practices when writing or changing MySQL schemas, queries, migrations, or app DB access. Prefer project conventions when they conflict; discover them first (existing migrations, ORM config, charset/collation, naming).

## Defaults

1. Prefer **InnoDB** and **utf8mb4** (with an explicit collation the project already uses).
2. Prefer **parameterized queries** / bound parameters everywhere; never concatenate untrusted input into SQL.
3. Prefer **small, reviewable migrations** over large opaque dumps.
4. Prefer **explicit transactions** for multi-statement writes that must succeed or fail together.
5. Match the project's MySQL version features; do not require syntax the deployed server lacks.

## Schema design

- Give every table a clear primary key. Prefer surrogate `BIGINT`/`BINARY(16)` keys when natural keys change or are wide; keep natural uniqueness with `UNIQUE`.
- Prefer `NOT NULL` unless `NULL` is meaningful. Avoid sentinel values that encode "missing".
- Prefer precise types: `DECIMAL` for money, `DATETIME`/`TIMESTAMP` with documented timezone policy, `TINYINT(1)` or `BOOLEAN` for flags only when that matches the codebase.
- Prefer `utf8mb4` for text columns; avoid legacy `utf8` (3-byte) unless the project is stuck on it.
- Prefer foreign keys when the app and ops model allow them; if FKs are intentionally omitted, enforce integrity in app code and document why.
- Prefer soft-delete only when the product requires it; otherwise hard-delete and retain history elsewhere.
- Name tables/columns consistently with the repo (`snake_case` is typical for MySQL). Avoid reserved words as identifiers.

```sql
CREATE TABLE orders (
  id BIGINT UNSIGNED NOT NULL AUTO_INCREMENT,
  user_id BIGINT UNSIGNED NOT NULL,
  amount DECIMAL(12, 2) NOT NULL,
  status VARCHAR(32) NOT NULL,
  created_at DATETIME(6) NOT NULL DEFAULT CURRENT_TIMESTAMP(6),
  PRIMARY KEY (id),
  KEY idx_orders_user_id_created_at (user_id, created_at),
  CONSTRAINT fk_orders_user FOREIGN KEY (user_id) REFERENCES users (id)
) ENGINE=InnoDB DEFAULT CHARSET=utf8mb4;
```

## Indexes & query shape

- Index columns used in `WHERE`, `JOIN`, and `ORDER BY` when selective enough to help; do not index every column.
- Prefer **composite indexes** that match left-prefix usage (`(user_id, created_at)` for filters on `user_id` then sort/filter by time).
- Prefer covering indexes only when profiling shows they help and write cost is acceptable.
- Avoid functions on indexed columns in predicates (`WHERE DATE(created_at) = ...`); rewrite to range predicates.
- Prefer `EXPLAIN` / `EXPLAIN ANALYZE` (MySQL 8.0.18+) before shipping non-trivial queries.
- Prefer selective filters first in mental index design; low-cardinality columns alone rarely deserve a standalone index.
- Avoid `SELECT *` in application hot paths; select only needed columns.
- Prefer `LIMIT` with a stable order for pagination; for deep pages, prefer keyset/cursor pagination over large `OFFSET`.

```sql
-- Prefer range on the indexed column
WHERE created_at >= '2024-01-01' AND created_at < '2024-02-01'

-- Prefer keyset pagination
WHERE (created_at, id) < (?, ?)
ORDER BY created_at DESC, id DESC
LIMIT 20
```

## Writes, transactions & concurrency

- Keep transactions **short**: no network/user waits inside a transaction.
- Prefer a single round-trip batch when inserting/updating many rows (`INSERT ... VALUES (...), (...)` or multi-row upserts the project already uses).
- Prefer `INSERT ... ON DUPLICATE KEY UPDATE` or `REPLACE` only when semantics are understood; `REPLACE` deletes then inserts (FK/trigger side effects).
- Choose isolation deliberately; default `REPEATABLE READ` (InnoDB) is fine unless the app needs otherwise. Document any non-default choice.
- Prefer optimistic patterns or `SELECT ... FOR UPDATE` only on the minimal row set when race conditions require it.
- Avoid long-held locks and gap-lock surprises on wide ranges; narrow the predicate and use appropriate indexes.

## Application access

- Always use prepared statements / ORM parameter binding. Never build SQL with string interpolation of user input.
- Prefer connection pooling with sensible limits (`max` connections, idle timeout) matching MySQL `max_connections` and deployment topology.
- Set explicit timeouts for connect/query when the driver supports them.
- Prefer migrations tools already in the project (Flyway, Liquibase, golang-migrate, Prisma, Knex, etc.); do not hand-edit production schemas.
- Prefer read replicas only with clear consistency expectations; do not read-your-writes from a replica without a plan.
- Map DB errors to app errors carefully; do not leak internal SQL or schema details to clients.

## Migrations & ops

- Make migrations **forward-compatible** when zero-downtime is required: add nullable/new columns first, backfill, then switch reads/writes, then drop old columns in a later change.
- Prefer online-friendly changes for large tables; be wary of full table rebuilds on hot tables without a plan.
- Never put secrets in migration files or SQL comments that get committed.
- Prefer idempotent or clearly versioned migrations; do not edit already-applied migrations on shared environments.
- Take backups / confirm restore before destructive production DDL when the change is risky.

## Security

- Principle of least privilege: app DB users get only needed DML/DDL on required schemas—not `ROOT`.
- Never log full queries with secrets (tokens, passwords, PII beyond policy).
- Prefer stored procedures only when the project already relies on them; do not move business logic into MySQL by default.
- Treat dynamic `ORDER BY` / column names as allowlists, not free strings.

## Anti-patterns (do not)

- String-concatenated SQL with request/user data.
- Unindexed foreign-key join columns on large tables.
- `OFFSET` into millions of rows for pagination hot paths.
- Holding transactions open across HTTP calls or external APIs.
- `SELECT *` plus client-side filtering for large result sets.
- Changing column types / charsets on huge tables without an online strategy.
- Using `MyISAM` for new transactional tables.
- Relying on implicit default charset/collation differences across environments.

## Agent checklist

Before finishing MySQL-related work:

1. SQL uses bound parameters; no injection risk from concatenation.
2. New filters/joins have a sensible index story (`EXPLAIN` when non-trivial).
3. Multi-write paths use an explicit transaction with a clear boundary.
4. Schema/migration matches existing naming, charset, engine, and migration tool.
5. Destructive or locking DDL has a rollout plan appropriate to table size.
