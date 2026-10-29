# Quick Start Guide & Connection Reference

TableTalk Query Language (TTQL) is a local-first, privacy-native federated query language designed for cross-database analytical queries.

---

## 1. 3-Step Quick Start

### Step 1: Add Database Connections
In TableTalk, add your target databases manually or click **Import Server Pool** to discover all hosted server databases automatically. Credentials are encrypted inside your native OS Credential Vault (Windows DPAPI, macOS Keychain, Linux Secret Service).

### Step 2: Open TTQL Studio
Press `Ctrl` + `Shift` + `T` (or click **TTQL Studio** in the navigation bar). The **Environment Explorer** lists all active database connection aliases and their introspected tables/collections.

### Step 3: Run Your Federated Query
Write your query using TTQL's modern fluent pipeline and press `Ctrl` + `Enter` (or `F5`). The DAG query planner orchestrates parallel isolate data retrieval and performs synthetic relational joins in local ephemeral C-memory (`sqlite3.openInMemory()`).

```ttql
const pg = environment.connections.ecommerce_pg;
const mysql = environment.connections.inventory_mysql;

from(pg.customers as c)
  .innerJoin(mysql.shipments as s, on: c.customer_id == s.customer_id)
  .where(c.country == 'US')
  .select([c.customer_id, c.full_name, c.email, s.shipment_id, s.carrier])
  .orderBy(c.customer_id, 'ASC')
  .limit(20);
```

---

## 2. Connection Strings Reference (Supported Engine Drivers)

| Engine | Protocol / URI Scheme | Connection String Format | Example Connection String |
| :--- | :--- | :--- | :--- |
| **PostgreSQL** | `postgresql://` | `postgresql://user:password@host:port/dbname` | `postgresql://admin:secret@localhost:5432/ecommerce` |
| **MySQL / MariaDB** | `mysql://` | `mysql://user:password@host:port/dbname` | `mysql://root:secret@localhost:3306/inventory` |
| **SQLite** | Local File Path | `C:\path\to\database.sqlite` or `/path/to/db.sqlite` | `C:\projects\TableTalk\test_dbs\related_warehouse.sqlite` |
| **MongoDB** | `mongodb://` | `mongodb://user:password@host:port/dbname?authSource=admin` | `mongodb://app:secret@localhost:27017/analytics?authSource=admin` |
| **Redis** | `redis://` | `redis://:password@host:port/dbIndex` or `redis://host:port/0` | `redis://:secret@localhost:6379/0` |
| **DuckDB** | Local File Path | `C:\path\to\analytics.duckdb` or `:memory:` | `C:\projects\TableTalk\test_dbs\duckdb.db` |
| **Microsoft SQL Server** | `mssql://` | `mssql://user:password@host:port/dbname` | `mssql://sa:SecretPass123!@localhost:1433/master` |
| **ClickHouse** | `http://` | `http://user:password@host:port/dbname` | `http://default:secret@localhost:8123/analytics_db` |
| **PlanetScale** | `planetscale://` | `planetscale://user:password@host:3306/dbname` | `planetscale://ps_user:ps_pass@aws.connect.psdb.cloud/production` |
| **CockroachDB** | `postgresql://` | `postgresql://user:password@host:26257/dbname` | `postgresql://root@localhost:26257/defaultdb` |
| **YugabyteDB** | `postgresql://` | `postgresql://user:password@host:5435/dbname` | `postgresql://yugabyte:yugabyte@localhost:5435/yugabyte` |
| **Oracle Database** | `oracle://` | `oracle://user:password@host:1521/service_name` | `oracle://system:secret@localhost:1521/FREEPDB1` |
| **Cassandra / ScyllaDB** | `cassandra://` | `cassandra://user:password@host:9042/keyspace` | `cassandra://cassandra:cassandra@localhost:9042/system` |

---

## 3. Example E-Commerce Federation Architecture

In modern architectures, services partition data across specialized database engines linked by standard domain identifiers:

1. **`customer_id`** (`cust_101` – `cust_105`): Links customer profiles in PostgreSQL, shipments in MySQL, activity logs in MongoDB, and sessions in Redis.
2. **`order_id`** (`ord_901` – `ord_906`): Links SQL orders, shipments, warehouse stock bins, and invoices.
3. **`item_sku`** (`SKU-8821`, `SKU-1044`): Links warehouse stock in SQLite to products in PostgreSQL.

---

## 4. Production Query Cookbook

### Recipe 1: 3-Way Cross-Database Join (PostgreSQL -> MySQL -> SQLite)
Correlate customer profiles in PostgreSQL with active shipments in MySQL and warehouse bin locations in SQLite:

```ttql
const pg = environment.connections.ecommerce_pg;
const mysql = environment.connections.inventory_mysql;
const sqlite = environment.connections.related_warehouse;

from(pg.customers as c)
  .innerJoin(mysql.shipments as s, on: c.customer_id == s.customer_id)
  .innerJoin(sqlite.warehouse_stock as w, on: s.order_id == w.order_id)
  .where(c.country == 'US')
  .select([
    c.customer_id,
    c.full_name,
    c.email,
    s.shipment_id,
    w.warehouse_bin,
    w.quantity
  ])
  .orderBy(w.quantity, 'DESC')
  .limit(25);
```

---

### Recipe 2: Relational to NoSQL Activity Correlation (PostgreSQL + MongoDB)
Correlate relational customer accounts in PostgreSQL with high-throughput MongoDB user activity logs:

```ttql
const pg = environment.connections.ecommerce_pg;
const mongo = environment.connections.analytics_mongo;

from(pg.customers as c)
  .innerJoin(mongo.user_activity_logs as l, on: c.customer_id == l.customer_id)
  .where(c.country == 'US')
  .select([
    c.customer_id,
    c.full_name,
    l.event_type,
    l.platform
  ])
  .limit(50);
```

---

### Recipe 3: Federated Analytical Aggregation with GROUP BY & HAVING
Calculate total revenue and order volume per shipping carrier across PostgreSQL orders and MySQL shipments:

```ttql
const mysql = environment.connections.inventory_mysql;
const pg = environment.connections.ecommerce_pg;

from(mysql.shipments as s)
  .innerJoin(pg.orders as o, on: s.order_id == o.order_id)
  .groupBy(s.carrier)
  .select([
    s.carrier,
    count() as total_shipments,
    sum(o.order_total) as carrier_revenue,
    avg(o.order_total) as avg_order_value
  ])
  .having(total_shipments > 1)
  .orderBy(carrier_revenue, 'DESC');
```

---

### Recipe 4: Redis Session Enrichment
Enrich active customer profiles with their live session state cached in Redis:

```ttql
const pg = environment.connections.ecommerce_pg;
const redis = environment.connections.session_redis;

from(pg.customers as c)
  .innerJoin(redis.session as r, on: c.customer_id == r.customer_id)
  .select([
    c.customer_id,
    c.full_name,
    c.email,
    r.active_cart_id
  ])
  .limit(10);
```
