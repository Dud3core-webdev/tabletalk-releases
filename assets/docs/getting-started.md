# Quick Start Guide & Connection Reference

TableTalk Query Language (TTQL) is a local, privacy-first functional DSL designed for cross-database analytical federation.

---

## 1. 3-Step Quick Start

### Step 1: Add Database Connections
In TableTalk, add your target databases or click **Import Server Pool** to discover all hosted server databases automatically. Credentials are securely encrypted inside your native OS Credential Vault (Windows DPAPI, macOS Keychain, Linux Secret Service).

### Step 2: Open TTQL Studio
Press `Ctrl` + `Shift` + `T` (or click **TTQL Studio** in the navigation bar). The **Environment Explorer** lists all active database connection aliases in `environment.connections.<alias>`.

### Step 3: Run Your Federated Query
Write your query script in the worksheet and press `Ctrl` + `Enter` (or `F5`). The DAG query planner orchestrates parallel isolate data retrieval and performs synthetic relational joins in local ephemeral C-memory (`sqlite3.openInMemory()`).

---

## 2. Connection Strings Reference (All 13 Engine Drivers)

| Engine | Protocol / URI Scheme | Connection String Format | Example Connection String |
| :--- | :--- | :--- | :--- |
| **PostgreSQL** | `postgresql://` | `postgresql://user:password@host:port/dbname` | `postgresql://admin:secret@db.company.com:5432/ecommerce` |
| **MySQL / MariaDB** | `mysql://` | `mysql://user:password@host:port/dbname` | `mysql://root:secret@mysql.company.com:3306/inventory` |
| **SQLite** | Local File Path | `C:\path\to\database.sqlite` or `/path/to/db.sqlite` | `/var/data/warehouse_stock.sqlite` |
| **MongoDB** | `mongodb://` | `mongodb://user:password@host:port/dbname?authSource=admin` | `mongodb://app:secret@mongo.company.com:27017/analytics?authSource=admin` |
| **Redis** | `redis://` | `redis://:password@host:port/dbIndex` or `redis://host:port/0` | `redis://:secret@cache.company.com:6379/0` |
| **DuckDB** | Local File Path | `C:\path\to\analytics.duckdb` or `:memory:` | `/var/data/analytics.duckdb` |
| **Microsoft SQL Server** | `mssql://` | `mssql://user:password@host:port/dbname` | `mssql://sa:SecretPass123!@mssql.company.com:1433/master` |
| **ClickHouse** | `http://` | `http://user:password@host:port/dbname` | `http://default:secret@clickhouse.company.com:8123/analytics_db` |
| **PlanetScale** | `planetscale://` | `planetscale://user:password@host:3306/dbname` | `planetscale://ps_user:ps_pass@aws.connect.psdb.cloud/production` |
| **CockroachDB** | `postgresql://` | `postgresql://user:password@host:26257/dbname` | `postgresql://root@cockroach.company.com:26257/defaultdb` |
| **YugabyteDB** | `postgresql://` | `postgresql://user:password@host:5435/dbname` | `postgresql://yugabyte:yugabyte@yugabyte.company.com:5435/yugabyte` |
| **Oracle Database** | `oracle://` | `oracle://user:password@host:1521/service_name` | `oracle://system:secret@oracle.company.com:1521/FREEPDB1` |
| **Cassandra / ScyllaDB** | `cassandra://` | `cassandra://user:password@host:9042/keyspace` | `cassandra://cassandra:cassandra@cassandra.company.com:9042/system` |

---

## 3. Example E-Commerce Federation Architecture

To illustrate cross-database federation, consider a typical modern e-commerce platform where services are split across multiple specialized database engines linked by standard domain identifiers (`customer_id` and `order_id`):

1. **`customer_id`** (`cust_101` – `cust_105`): Links customer profiles, orders, shipments, MongoDB logs, Redis sessions, and ClickHouse web events.
2. **`order_id`** (`ord_901` – `ord_906`): Links SQL orders, physical shipments, SQLite warehouse stock bins, and Redis carts.

---

## 4. Production Query Cookbook

### Recipe 1: Cross-Database Relay (PostgreSQL -> MySQL -> SQLite)
Correlate customer profiles in PostgreSQL with active MySQL orders and embedded SQLite warehouse stock bins:

```javascript
const pg = environment.connections.ecommerce_pg;
const mysql = environment.connections.inventory_mysql;
const sqlite = environment.connections.sqllite;

select(() => {
    PostgreSql(pg.id, ['customer_id', 'email', 'full_name'], pg.tables['customers'])
})
// Stage 1: Relay customer_id to MySQL shipments
.innerJoin((cust) => {
    MySql(mysql.id, ['shipment_id', 'order_id', 'customer_id', 'carrier'], mysql.tables['shipments'])
        .where((ship) => ship.customer_id is cust.customer_id)
})
// Stage 2: Relay order_id to SQLite warehouse bins
.innerJoin((ship) => {
    Sqlite(sqlite.id, ['item_sku', 'order_id', 'warehouse_bin', 'quantity'], sqlite.tables['warehouse_stock'])
        .where((stock) => stock.order_id is ship.order_id)
})
.orderBy((row) => row.quantity, 'DESC');
```

---

### Recipe 2: Relational to NoSQL Document Correlation (PostgreSQL + MongoDB)
Correlate relational customer accounts in PostgreSQL with high-throughput MongoDB user activity logs:

```javascript
const pg = environment.connections.ecommerce_pg;
const mongo = environment.connections.default_mongodb;

select(() => {
    PostgreSql(pg.id, ['customer_id', 'full_name', 'email'], pg.tables['customers'])
        .where((cust) => cust.country is 'US')
})
.innerJoin((cust) => {
    Mongo(mongo.id, ['_id', 'customer_id', 'order_id', 'event_type', 'platform'], mongo.collections['user_activity_logs'])
        .where((log) => log.customer_id is cust.customer_id)
});
```

---

### Recipe 3: High-Concurrency Parallel Batching (`declare asyncRun`)
Execute parallel query isolates across PostgreSQL, MySQL, and Redis simultaneously:

```javascript
const pg = environment.connections.ecommerce_pg;
const mysql = environment.connections.inventory_mysql;
const redis = environment.connections.redis;

const targetCustomer = 'cust_101';
const customerCols = ['customer_id', 'full_name', 'email'];

select(() => {
    declare asyncRun(() => {
        PostgreSql(pg.id, [...customerCols], pg.tables['customers']),
        MySql(mysql.id, ['shipment_id', 'order_id', 'carrier', 'tracking_code'], mysql.tables['shipments']),
        Redis(redis.id, 'session:cust_101')
    })
    .where((row) => row.customer_id is targetCustomer)
});
```
