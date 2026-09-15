# TTQL Syntax & Grammar Reference

TableTalk Query Language (TTQL) is modeled on modern TypeScript functional DSL patterns, offering static type inference, first-class closures, and spread operations.

---

## 1. Connection Bindings

Declare abstract references to active databases registered in your local pool:

```javascript
const pg = environment.connections.postgres_prod;
const mysql = environment.connections.mysql_orders;
const sqlite = environment.connections.sqlite_warehouse;
const mongo = environment.connections.mongo_logs;
const redis = environment.connections.redis_cache;
const clickhouse = environment.connections.clickhouse_events;
```

---

## 2. Constants & Static Type Inference

TTQL infers types statically at compile time (`string`, `number`, `boolean`, `listString`, `listNumber`, `connectionRef`):

```javascript
const minOrderSpend = 250.00;
const priorityCountry = 'United States';
const allowedStatuses = ['CONFIRMED', 'SHIPPED', 'DELIVERED'];
const projectionColumns = ['order_id', 'customer_id', 'order_total'];
```

The linter validates operands before execution—preventing type mismatches like passing numbers to `LIKE` or strings to mathematical inequalities.

---

## 3. Spread Syntax (`...`)

TTQL provides first-class spread operators in arrays and projections:

```javascript
const baseCols = ['customer_id', 'created_at'];
const fullCols = [...baseCols, 'email', 'full_name'];

select(() => {
    PostgreSql(pg.id, fullCols, pg.tables['customers'])
});
```

> **Boundary Safety Invariant:** Spreading into `tables[...]` requires strictly one table name (`targetTable.length == 1`). Spreading multi-table arrays triggers an immediate `AMBIGUOUS_TABLE_SPREAD` parser error.

---

## 4. Engine Constructors

TTQL provides 13 native engine constructors matching all supported database engines in TableTalk:

| Engine | Dialect Target | Signature | Description |
| :--- | :--- | :--- | :--- |
| `PostgreSql()` | PostgreSQL | `PostgreSql(connId, columns, table)` | ANSI double-quoted relational schema query (`"col"`, `$1`) |
| `MySql()` | MySQL / MariaDB | `MySql(connId, columns, table)` | Backtick-escaped relational dialect query (``` `col` ```, `?`) |
| `Sqlite()` | SQLite | `Sqlite(connId, columns, table)` | Local file or in-memory SQLite table query (`"col"`, `?`) |
| `Mongo()` / `MongoDb()` | MongoDB | `Mongo(connId, columns, collection)` | Document collection scan with filter pushdown |
| `Redis()` | Redis | `Redis(connId, keyPattern)` | Key-value pattern scan with regex evaluation |
| `DuckDb()` | DuckDB | `DuckDb(connId, columns, table)` | Local analytical column-store table query (`"col"`, `?`) |
| `MsSql()` | Microsoft SQL Server | `MsSql(connId, columns, table)` | T-SQL relational table query (`"col"`, `?`) |
| `ClickHouse()` | ClickHouse | `ClickHouse(connId, columns, table)` | Columnar analytics engine table query (`"col"`, `?`) |
| `PlanetScale()` | PlanetScale | `PlanetScale(connId, columns, table)` | Distributed MySQL-compatible table query (``` `col` ```, `?`) |
| `CockroachDb()` | CockroachDB | `CockroachDb(connId, columns, table)` | Distributed PostgreSQL-compatible table query (`"col"`, `$1`) |
| `YugabyteDb()` | YugabyteDB | `YugabyteDb(connId, columns, table)` | Cloud-native YSQL table query (`"col"`, `$1`) |
| `Oracle()` | Oracle Database | `Oracle(connId, columns, table)` | Enterprise PL/SQL table query (`"col"`, `?`) |
| `Cassandra()` | Cassandra / ScyllaDB | `Cassandra(connId, columns, table)` | NoSQL CQL keyspace table query (`"col"`, `?`) |

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
    MySql(mysql.id, ['order_id', 'customer_id', 'carrier'], mysql.tables['shipments'])
        .where((ship) => ship.customer_id is cust.customer_id)
});
```

---

## 6. Predicates & Operators

```javascript
.where((entity) => <predicate>)
```

| Operator | Example | Description |
| :--- | :--- | :--- |
| `is` / `=` | `row.status is 'delivered'` | Equality check |
| `is not` / `!=` | `row.status is not 'cancelled'` | Inequality check |
| `<`, `<=`, `>`, `>=` | `row.order_total >= 500` | Numeric and chronological comparisons |
| `is null` / `is not null` | `row.deleted_at is null` | Nullability verification |
| `LIKE '%pat%'` | `row.email LIKE '%@tabletalk.io'` | Wildcard substring search |
| `IN [...]` | `row.carrier IN ['FedEx', 'UPS']` | Set membership inclusion |
| `NOT IN [...]` | `row.customer_id NOT IN ['cust_999']` | Set membership exclusion |
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
