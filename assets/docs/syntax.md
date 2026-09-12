# TTQL Syntax & Grammar Reference

TableTalk Query Language (TTQL) is modeled on modern TypeScript functional DSL patterns, offering static type inference, first-class closures, and spread operations.

---

## 1. Connection Bindings

Declare abstract references to active databases registered in your local pool:

```javascript
const pg = environment.connections.pg_main;
const mysql = environment.connections.mysql_orders;
const sqlite = environment.connections.sqlite_warehouse;
const mongo = environment.connections.mongo_events;
const redis = environment.connections.redis_cache;
```

---

## 2. Constants & Static Type Inference

TTQL infers types statically at compile time (`string`, `number`, `boolean`, `listString`, `listNumber`, `connectionRef`):

```javascript
const minOrderSpend = 250.00;
const priorityCountry = 'United States';
const allowedStatuses = ['CONFIRMED', 'SHIPPED', 'DELIVERED'];
const projectionColumns = ['order_id', 'customer_id', 'total'];
```

The linter validates operands before execution—preventing type mismatches like passing numbers to `LIKE` or strings to mathematical inequalities.

---

## 3. Spread Syntax (`...`)

TTQL provides first-class spread operators in arrays and projections:

```javascript
const baseCols = ['id', 'created_at'];
const fullCols = [...baseCols, 'email', 'display_name'];

select(() => {
    PostgreSql(pg.id, fullCols, pg.tables['users'])
});
```

> **Boundary Safety Invariant:** Spreading into `tables[...]` requires strictly one table name (`targetTable.length == 1`). Spreading multi-table arrays triggers an immediate `AMBIGUOUS_TABLE_SPREAD` parser error.

---

## 4. Engine Constructors

| Engine | Dialect Target | Signature | Description |
| :--- | :--- | :--- | :--- |
| `PostgreSql()` | PostgreSQL | `PostgreSql(connId, columns, table)` | ANSI double-quoted relational schema query |
| `MySql()` | MySQL / MariaDB | `MySql(connId, columns, table)` | Backtick-escaped relational dialect query |
| `Sqlite()` | SQLite | `Sqlite(connId, columns, table)` | Local file or in-memory SQLite table query |
| `MongoDb()` | MongoDB | `MongoDb(connId, collection)` | Document collection scan with state injection |
| `Redis()` | Redis | `Redis(connId, keyPattern)` | Key-value pattern scan with regex evaluation |

---

## 5. Relational Join Methods

Sequential dependency chaining links disparate databases across execution stages:

- **`.innerJoin((parent) => { ... })`**: Returns matched rows existing in both databases.
- **`.leftJoin((parent) => { ... })`**: Preserves all rows from upstream stage, filling unlinked downstream columns with nulls.
- **`.rightJoin((parent) => { ... })`**: Preserves all rows from joined database, matching upstream rows where available.
- **`.fullOuterJoin((parent) => { ... })`**: Preserves all records from both stages.

```javascript
select(() => {
    PostgreSql(pg.id, ['customer_id', 'full_name'], pg.tables['customers'])
})
.innerJoin((cust) => {
    MySql(mysql.id, ['order_id', 'customer_id', 'order_total'], mysql.tables['orders'])
        .where((order) => order.customer_id is cust.customer_id)
});
```

---

## 6. Predicates & Operators

```javascript
.where((entity) => <predicate>)
```

| Operator | Example | Description |
| :--- | :--- | :--- |
| `is` / `=` | `row.status is 'CONFIRMED'` | Equality check |
| `is not` / `!=` | `row.status is not 'CANCELLED'` | Inequality check |
| `<`, `<=`, `>`, `>=` | `row.order_total >= 500` | Numeric and chronological comparisons |
| `is null` / `is not null` | `row.deleted_at is null` | Nullability verification |
| `LIKE '%pat%'` | `row.email LIKE '%@enterprise.org'` | Wildcard substring search |
| `IN [...]` | `row.carrier IN ['FedEx', 'UPS']` | Set membership inclusion |
| `NOT IN [...]` | `row.customer_id NOT IN [9999, 8888]` | Set membership exclusion |
| `AND` / `OR` / `NOT` | `(a OR b) AND NOT c` | Logical composition |

---

## 7. Ordering & Paging Modifiers

```javascript
select(() => { ... })
.orderBy((row) => row.order_total, 'DESC')
.limit(50);
```

---

## 8. Linter Error Codes

| Error Code | Cause |
| :--- | :--- |
| `FORBIDDEN_MUTATION` | Non-read-only statement detected (e.g. `INSERT`, `UPDATE`, `DROP`) |
| `MISSING_CONNECTION` | Referenced connection alias is not active in connection pool |
| `TABLE_NOT_FOUND` | Queried entity does not exist in schema |
| `TYPE_MISMATCH` | Incompatible operand in comparison |
| `INVALID_SPREAD` | Attempting to spread non-list variable |
| `AMBIGUOUS_TABLE_SPREAD` | Multi-table list passed to `tables[...]` |
