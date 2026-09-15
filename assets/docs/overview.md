# Overview & Architecture

## The Cross-Database Federation Dilemma

In production software engineering, application state rarely lives in a single database engine. Architectures typically distribute workloads across specialized storage technologies:

| Engine | Typical Production Role | Protocol / Default Scheme |
| :--- | :--- | :--- |
| **PostgreSQL** | Primary user accounts, multi-tenant relational models, transactional data | `postgresql://` |
| **MySQL / MariaDB** | E-commerce transactions, operational relational stores | `mysql://` |
| **MongoDB** | Unstructured event streams, activity logs, dynamic document payloads | `mongodb://` |
| **Redis** | In-memory session states, distributed locks, ephemeral cache tokens | `redis://` |
| **SQLite** | Local embedded storage, desktop client state, offline caches | File Path / `:memory:` |
| **DuckDB** | Fast local analytical column-store execution | File Path / `:memory:` |
| **Microsoft SQL Server** | Enterprise T-SQL operational databases and transactional stores | `mssql://` |
| **ClickHouse** | High-throughput columnar real-time analytics engine | `http://` |
| **PlanetScale** | Cloud-native serverless MySQL-compatible cluster stores | `planetscale://` |
| **CockroachDB** | Distributed resilient cloud PostgreSQL-compatible SQL engine | `postgresql://` |
| **YugabyteDB** | Cloud-native distributed YSQL relational database | `postgresql://` |
| **Oracle Database** | Enterprise PL/SQL mission-critical relational infrastructure | `oracle://` |
| **Cassandra / ScyllaDB** | Distributed high-speed NoSQL CQL keyspace stores | `cassandra://` |

---

## Example E-Commerce Federation Architecture

Consider a typical e-commerce infrastructure where business domain entities are distributed across multiple specialized database engines linked via shared join keys:

- **`customer_id`** (`cust_101` – `cust_105`): Links customer profiles, orders, shipments, activity logs, Redis user sessions, and ClickHouse web events.
- **`order_id`** (`ord_901` – `ord_906`): Links SQL order records, physical shipments, SQLite warehouse inventory stock, Redis carts, and web events.

```mermaid
graph TD
    subgraph PostgreSQL ["PostgreSQL (ecommerce_pg)"]
        C["customers<br/>(customer_id, full_name, email, country)"]
        O["orders<br/>(order_id, customer_id, order_total, status)"]
        C -->|1 : N| O
    end

    subgraph MySQL ["MySQL (inventory_mysql)"]
        S["shipments<br/>(shipment_id, order_id, customer_id, carrier, tracking_code)"]
    end

    subgraph SQLite ["SQLite (related_warehouse.sqlite)"]
        W["warehouse_stock<br/>(item_sku, order_id, customer_id, warehouse_bin, quantity)"]
    end

    subgraph MongoDB ["MongoDB (analytics_mongo)"]
        M["user_activity_logs<br/>(_id, customer_id, order_id, event_type, platform)"]
    end

    subgraph Redis ["Redis (Session & Cart Cache)"]
        R1["session:{customer_id}<br/>(customer_id, role, active_cart_id)"]
        R2["cart:{order_id}<br/>(items, currency)"]
    end

    subgraph ClickHouse ["ClickHouse (analytics_db)"]
        CH["analytics_db_web_events<br/>(event_id, customer_id, order_id, event_type, duration_ms)"]
    end

    O -.->|order_id| S
    O -.->|order_id| W
    C -.->|customer_id| M
    C -.->|customer_id| R1
    O -.->|order_id| R2
    C -.->|customer_id| CH
```

---

## The TTQL Approach

**TableTalk Query Language (TTQL)** is a declarative domain-specific language designed for local, cross-database analytical queries. Instead of managing ETL infrastructure for routine queries, developers can express queries across multiple database engines in a single script.

TTQL executes data retrieval in parallel background isolates and stages intermediate results into an ephemeral, in-memory SQLite database (`sqlite3.openInMemory()`) on the local workstation. Relational joins and final filtering occur directly in local memory.

```
[TTQL Script] ──► [Lexer & Security Gate] ──► [DAG Query Planner] ──► [Parallel Isolates (≤3)]
                                                                               │
                                                                               ▼
[Local Result Grid] ◄── [Synthetic Relational Joins] ◄── [Ephemeral SQLite :memory:]
```

### Core Architectural Invariants

- **100% Read-Only Safety Gate:** All write operations (`INSERT`, `UPDATE`, `DELETE`, `DROP`, `ALTER`, etc.) are rejected during lexical analysis and parsing. The engine refuses to generate or dispatch mutating commands.
- **OS Keychain Credential Airgap:** Query scripts never contain raw passwords or connection strings. Queries reference abstract pool aliases (e.g. `environment.connections.pg_main`), and TableTalk resolves credentials on demand via operating system keychains (Windows DPAPI, macOS Keychain, Linux Secret Service).
- **FIFO DAG Concurrency Throttling:** To protect production databases from connection exhaustion, parallel query batches declaring more than 3 simultaneous evaluations are partitioned into sequential stages of at most 3 concurrent isolate queries.
- **Deterministic Resource Disposal:** Staging tables exist exclusively in ephemeral memory and are disposed immediately upon query completion via a guaranteed `finally` block (`stagingDb.dispose()`).

---

## Architecture Diagrams

### FIFO Thread Throttling & Isolate Scheduling

When a query declares multiple simultaneous evaluations in `declare asyncRun`, the DAG planner partitions isolate dispatch into sequential batches of at most 3 concurrent threads:

```mermaid
flowchart TD
    subgraph Client["TableTalk Query Engine (Local Isolate)"]
        Q["TTQL Script: declare asyncRun(Q1..Q5)"]
        DAG["DAG Query Planner & Throttler"]
        Q --> DAG
    end

    subgraph Batch1["FIFO Stage 1 (Max 3 Concurrent Isolates)"]
        I1["Isolate 1: PostgreSQL (Customers)"]
        I2["Isolate 2: MySQL (Orders)"]
        I3["Isolate 3: SQLite (Local Warehouse)"]
    end

    subgraph Batch2["FIFO Stage 2 (Sequential Relay)"]
        I4["Isolate 4: MongoDB (Activity Logs)"]
        I5["Isolate 5: Redis (Session Cache)"]
    end

    DAG -->|"Stage 1 Dispatch"| I1 & I2 & I3
    I1 & I2 & I3 -->|"FIFO Queue Drain"| DAG
    DAG -->|"Stage 2 Dispatch"| I4 & I5

    subgraph Staging["Ephemeral In-Memory Staging"]
        MEM[("sqlite3 :memory: Staged Tables")]
        SYN["Synthetic Relational Engine<br/>(INNER / LEFT / FULL JOINs)"]
        MEM --> SYN
    end
    I1 & I2 & I3 --> MEM
    I4 & I5 --> MEM
    SYN --> OUT["Federated Analytical Result (Grid / JSON)"]
```

---

## Pragmatic Design Trade-Offs

To maintain predictable behavior, TTQL is deliberately scoped around the following trade-offs:

1. **Designed for Working Sets, Not Petabyte Warehousing:** Ephemeral staging uses your local workstation's RAM. It is optimized for query result sets ranging from tens of rows to tens of thousands of rows.
2. **Read-Only Scope:** TTQL does not execute cross-database transactions (`2PC`) or multi-source writes. It is strictly an analytical and diagnostic querying tool.
3. **Safe Concurrency Bounds:** By enforcing a strict ceiling of 3 concurrent worker isolates per batch, TTQL prioritizes database stability over raw saturated network throughput.
