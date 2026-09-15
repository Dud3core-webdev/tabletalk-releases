# TableTalk Changelog & Release Notes

All notable changes, architectural milestones, and engine updates for TableTalk are documented here.

---

## [v1.2.6] — Current Release

### Added & Expanded
- **13 Engine TTQL Poly-Store Support**: First-class TTQL query compilation and isolate execution across **SQLite, PostgreSQL, MySQL / MariaDB, MongoDB, Redis, DuckDB, Microsoft SQL Server, ClickHouse, PlanetScale, CockroachDB, YugabyteDB, Oracle Database, and Cassandra / ScyllaDB**.
- **Automated Server Pool Import**: One-click catalog discovery and pool creation for entire database clusters (scanning `pg_database`, `information_schema.SCHEMATA`, `sys.databases`, `system.databases`, etc.).
- **Periodic Background Schema Indexing**: Configurable background job (`AutoSchemaSyncService`) re-indexing schema metadata and synthetic foreign key graphs across active pools.
- **Dialect-Aware TTQL Translators**: Dialect parameter binding (`$1` for PG/Cockroach/Yugabyte; `?` for SQL/MySQL/ClickHouse/MSSQL/Oracle/Cassandra), backtick quoting for MySQL/PlanetScale, and native MQL/Redis pipeline compilation.

---

## [v1.2.5] — Previous Release

---

## [v1.2.0] — Previous Release

### Added
- **Pooled Database Connections:** Enterprise connection pooling across PostgreSQL, MySQL, SQLite, MongoDB, and Redis with automatic lifecycle discipline and zero socket leakage.
- **TTQL Query Optimizer:** SQLite staged join optimization with automated foreign key indexing on intermediate staging tables.
- **Foreign Key Staging Enforcement:** Strict foreign key constraint propagation during cross-database isolate ingestion.
- **Schema Ingestion via Graph DB Table:** Visual relational schema graph ingestion supporting >1,000 tables.
- **Advanced Prompt Management:** Fine-tune local model context prompts and system directives directly inside the UI.
- **In-App License Purchase & Management:** Seamless checkout and instant cryptographic token activation.
- **Multi-DB Driver Stabilization:** Fixed connection timeouts and driver handshake quirks across MongoDB, Redis, and MySQL.

---

## [v1.1.0] — Previous Release

### Added
- **In-App Semantic Version Update Checker:** Non-intrusive notification banner for latest GitHub releases.
- **Ed25519 Cryptographic Licensing:** Mathematical offline validation with Crockford Base32 decoding. Zero telemetry.
- **LLM Fine-Tuning Directives:** Customizable system prompts for local Ollama and LM Studio endpoints.
- **UI Improvements:** High-contrast dark theme polish and keyboard shortcuts for query execution.

---

## [v1.0.0] — Initial Production Release

### Added
- **Local AI Studio:** Natural language SQL querying directly against active schemas.
- **Local LLM Integrations:** Seamless zero-setup integration with Ollama and LM Studio.
- **Multi-Database Support:** Native client drivers for SQLite, MySQL, PostgreSQL, and SQL Server.
- **Safe Read-Only Mode:** Strict lexical and driver-level guardrails preventing data mutations.
- **Incremental Schema Caching:** High-speed schema inspection designed for large production schemas (>1k tables).
- **AI Chat:** Natural conversation with local databases.
