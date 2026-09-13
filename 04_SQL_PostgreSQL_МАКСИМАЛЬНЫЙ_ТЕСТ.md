# МАКСИМАЛЬНЫЙ ТЕСТ: SQL / PostgreSQL

TEST_VERSION: 2.0

## ПРАВИЛА

Выполни 32 задания одним ответом. Без интернета, IDE, выполнения SQL и уточняющих вопросов.

Требуется:
- PostgreSQL-семантика;
- не выдумывать синтаксис;
- учитывать MVCC, locks, isolation, indexes, planner, constraints, bloat, migration safety;
- SQL должен быть синтаксически правдоподобным;
- если точный план зависит от статистики/данных — сказать это;
- объяснять не только «что», но и «почему».

Начало:
```text
MODEL: <название модели или UNKNOWN>
MODE: <thinking|non-thinking|UNKNOWN>
TEST_VERSION: 2.0
```

Конец:
```text
FINAL
Выполнено заданий: <число>
Пропущено заданий: <число>
Использованы внешние инструменты: НЕТ
END_OF_TEST
```

Всего заданий: 32.

---

## КОНТЕКСТ ПРОЕКТА POLARIS DB

Запомни:
- PostgreSQL 16;
- primary port 5440;
- multi-tenant SaaS;
- tenant column `tenant_id uuid NOT NULL` почти во всех business tables;
- orders: ~1.5 billion rows;
- events: ~8 billion rows;
- event retention 120 days;
- public API `/api/v6`;
- legacy upload `/api/v5/files/upload`;
- payment idempotency key уникален внутри tenant;
- money хранится в bigint minor units;
- production требует online/zero-downtime migrations;
- Redis private only;
- secrets forbidden in Git;
- logs must not contain full card number or bearer token.

---

## 1. INDEXING / QUERY DESIGN

### SQLX-1
Запрос:

```sql
SELECT id, total_cents, created_at
FROM orders
WHERE tenant_id = $1
  AND status = 'paid'
  AND created_at >= $2
ORDER BY created_at DESC, id DESC
LIMIT 100;
```

Предложи основной индекс, возможный partial index и объясни порядок колонок, INCLUDE и trade-offs.

### SQLX-2
Почему индекс `(status, tenant_id, created_at)` обычно хуже для этого запроса, чем `(tenant_id, status, created_at)`? Когда статистика может изменить вывод?

### SQLX-3
`OFFSET 10000000 LIMIT 100`: объясни стоимость и напиши корректную keyset pagination с `(created_at,id)` tie-breaker.

### SQLX-4
Когда BRIN лучше B-tree для events table? Какие свойства физического порядка данных нужны?

### SQLX-5
GIN index на `jsonb`: сравни `jsonb_ops` и `jsonb_path_ops` концептуально. Для чего каждый подходит?

---

## 2. TRANSACTIONS / LOCKS / MVCC

### SQLX-6
Две транзакции:
T1: lock row A, потом B.
T2: lock row B, потом A.
Объясни deadlock detection PostgreSQL, victim, retry и как снизить вероятность.

### SQLX-7
Сравни READ COMMITTED, REPEATABLE READ и SERIALIZABLE в PostgreSQL. Назови минимум по одной аномалии/особенности каждого.

### SQLX-8
Покажи lost update на balance и два решения: `SELECT ... FOR UPDATE` и атомарный conditional `UPDATE ... RETURNING`.

### SQLX-9
Что такое write skew и почему REPEATABLE READ может его допустить? Как SERIALIZABLE PostgreSQL это предотвращает?

### SQLX-10
Объясни MVCC visibility, dead tuples и почему долгоживущая транзакция мешает VACUUM освобождать пространство.

---

## 3. CONSTRAINTS / DATA MODEL

### SQLX-11
Спроектируй `payments` table:
- tenant_id;
- id;
- idempotency_key;
- request_hash;
- amount_minor bigint;
- currency char/varchar;
- status;
- timestamps;
- unique tenant+idempotency;
- check amount >0;
- CHECK для статусов или enum — сравни trade-off.

### SQLX-12
Как гарантировать, что период бронирования комнаты не пересекается для одного room_id? Покажи PostgreSQL exclusion constraint с range type и GiST.

### SQLX-13
Почему foreign key не гарантирует tenant isolation сам по себе? Дай schema pattern с composite key `(tenant_id,id)`.

### SQLX-14
Soft delete: плюсы/минусы, partial unique index «email уникален среди не удалённых», влияние на FK и queries.

---

## 4. ADVANCED SQL

### SQLX-15
Дана таблица `transactions(account_id, created_at, amount)`. Напиши запрос с running balance по account, не схлопывая строки.

### SQLX-16
Дана `employees(id, manager_id, name)`. Напиши recursive CTE, возвращающий дерево подчинённых от `$root`, depth и path; защити от цикла.

### SQLX-17
Для каждого tenant получить топ-3 заказа по total_cents. Используй window function.

### SQLX-18
События `(tenant_id,event_id,aggregate_id,seq,payload)`. Напиши запрос, находящий пропуски sequence по aggregate без генерации миллиардов строк.

### SQLX-19
UPSERT counter: атомарно увеличить существующее значение или создать строку. Верни новое значение.

---

## 5. OUTBOX / QUEUES / CONCURRENCY

### SQLX-20
Спроектируй outbox table и worker query на батч 100 строк с `FOR UPDATE SKIP LOCKED`. Объясни, где ставить `published_at` и почему нельзя считать broker publish и DB commit одной атомарной операцией.

### SQLX-21
Два workers могут повторно отправить одно событие после crash. Как хранить event_id/dedupe и почему exactly-once не появляется автоматически?

### SQLX-22
Нужно очистить events старше 120 дней из 8 млрд строк. Сравни massive DELETE и time partitioning + DROP/DETACH old partitions.

---

## 6. PERFORMANCE / PLANNER / MAINTENANCE

### SQLX-23
`EXPLAIN ANALYZE` показывает estimated rows=100, actual rows=10,000,000. Назови минимум 6 причин/действий: stale stats, correlations, extended statistics, parameter skew, casts, functions, bad predicates.

### SQLX-24
Почему `SELECT *` может помешать index-only scan? Объясни visibility map и heap fetches.

### SQLX-25
Что такое table/index bloat, откуда он берётся, чем VACUUM отличается от VACUUM FULL и REINDEX, почему VACUUM FULL опасен online.

### SQLX-26
Автовакуум не успевает на hot table. Какие параметры/метрики ты проверишь до «просто увеличить autovacuum»?

### SQLX-27
Почему `CREATE INDEX` может блокировать production и чем отличается `CREATE INDEX CONCURRENTLY`? Назови ограничения/риски concurrent build.

---

## 7. MIGRATIONS / RLS / RELIABILITY

### SQLX-28
Нужно добавить NOT NULL колонку с default на таблицу 1.5 млрд rows с минимальным риском. Дай online migration sequence для PostgreSQL 16, включая backfill, CHECK NOT VALID / VALIDATE и final NOT NULL.

### SQLX-29
Спроектируй Row Level Security для `orders` по `tenant_id`, где tenant id передаётся в session setting. Объясни риски connection pooling и `SET LOCAL`.

### SQLX-30
Почему читать replica immediately after write может дать stale read? Какие стратегии read-your-writes существуют?

---

## 8. LONG CONTEXT / SYSTEM DESIGN

### SQLX-31
Перечисли по памяти:
- PostgreSQL version;
- port;
- orders rows;
- events rows;
- retention;
- public API prefix;
- legacy upload endpoint;
- money representation;
- scope idempotency uniqueness.

### SQLX-32
Спроектируй schema + indexes + transaction flow для idempotent payment creation в multi-tenant SaaS. Нужно покрыть concurrent duplicate requests, request hash mismatch, status transitions, outbox, retry, audit trail, partitioning strategy для audit/events и zero-downtime evolution schema.

