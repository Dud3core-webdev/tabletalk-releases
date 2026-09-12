# Quick Start Guide & Production Cookbook

A guide to writing cross-database federated queries and common query patterns with TTQL.

---

## 3-Step Quick Start

### Step 1: Connect Databases in TableTalk
In the TableTalk Connection Pool, add your database instances (PostgreSQL, MySQL, SQLite, MongoDB, Redis). Credentials are saved securely inside your operating system's Credential Vault (Windows DPAPI / macOS Keychain).

### Step 2: Open the TTQL Worksheet
Press `Ctrl` + `Shift` + `T` or click **TTQL Studio** in the main navigation. The Environment Explorer on the right displays all active connection aliases.

### Step 3: Run Your Federated Query
Write your query in the worksheet editor and press `Ctrl` + `Enter` (or `F5`). The DAG query planner compiles the execution stages, partitions concurrent requests across worker isolates, and evaluates joins in ephemeral SQLite memory.

---

## Production Cookbook

### 1. Cross-Database Relay (PostgreSQL -> MySQL -> SQLite)

Correlate customer accounts in PostgreSQL with active MySQL orders and embedded SQLite warehouse stock:

```javascript
const pg = environment.connections.pg_main;
const mysql = environment.connections.mysql_orders;
const sqlite = environment.connections.sqlite_inventory;

select(() => {
    PostgreSql(pg.id, ['customer_id', 'email', 'full_name'], pg.tables['customers'])
})
// Stage 1: Relay customer_id to MySQL orders
.innerJoin((cust) => {
    MySql(mysql.id, ['order_id', 'customer_id', 'order_total'], mysql.tables['orders'])
        .where((order) => order.customer_id is cust.customer_id)
})
// Stage 2: Relay order_id to SQLite warehouse bins
.innerJoin((order) => {
    Sqlite(sqlite.id, ['item_sku', 'order_id', 'warehouse_bin', 'quantity'], sqlite.tables['warehouse_stock'])
        .where((stock) => stock.order_id is order.order_id)
})
.orderBy((stock) => stock.quantity, 'DESC');
```

---

### 2. Relational to Document Correlation (PostgreSQL + MongoDB)

Correlate relational customer profiles with high-throughput MongoDB activity logs without an external ETL:

```javascript
const pg = environment.connections.pg_main;
const mongo = environment.connections.mongo_logs;
const vipEmail = 'alex.morgan@enterprise.org';

select(() => {
    PostgreSql(pg.id, ['customer_id', 'email'], pg.tables['customers'])
        .where((c) => c.email is vipEmail)
})
.innerJoin((c) => {
    MongoDb(mongo.id, 'user_activity_logs')
        .where((log) => log.customerId is c.customer_id AND log.action LIKE '%checkout%')
});
```

---

### 3. Inventory Discrepancy Detection (leftJoin + is null)

Identify confirmed customer orders that have missing warehouse allocations across independent database systems:

```javascript
const mysql = environment.connections.mysql_orders;
const sqlite = environment.connections.sqlite_inventory;

select(() => {
    MySql(mysql.id, ['order_id', 'customer_id', 'status'], mysql.tables['orders'])
        .where((o) => o.status is 'CONFIRMED')
})
.leftJoin((order) => {
    Sqlite(sqlite.id, ['order_id', 'item_sku', 'quantity'], sqlite.tables['warehouse_stock'])
        .where((stock) => stock.order_id is order.order_id)
})
.where((row) => row.item_sku is null);
```

---

### 4. High-Concurrency Parallel Batching (declare asyncRun)

Query multiple database engines concurrently across separate background Dart isolates:

```javascript
const pg = environment.connections.pg_main;
const mysql = environment.connections.mysql_orders;

const priorityCarriers = ['FedEx', 'DHL', 'UPS'];
const targetFields = ['customer_id', 'email'];

select(() => {
    declare asyncRun(() => {
        PostgreSql(pg.id, [...targetFields], pg.tables['customers']),
        MySql(mysql.id, ['shipment_id', 'customer_id', 'carrier'], mysql.tables['shipments'])
    })
    .where((row) => row.carrier IN priorityCarriers)
})
.limit(50);
```

---

### 5. Real-Time Session Cache Enrichment (PostgreSQL + Redis)

Correlate persistent PostgreSQL database records with live Redis in-memory session keys:

```javascript
const pg = environment.connections.pg_main;
const redis = environment.connections.redis_cache;

select(() => {
    PostgreSql(pg.id, ['customer_id', 'email'], pg.tables['customers'])
})
.innerJoin((cust) => {
    Redis(redis.id, 'session:user:*')
});
```
