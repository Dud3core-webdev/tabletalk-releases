# TableTalk Changelog & Release Notes

All notable changes, architectural milestones, and engine updates for TableTalk are documented here.

---

## [v1.2.11] — Current Release

### Fixed & Hardened
- **Non-Blocking DB Pool Connection Recovery:** Updated `ConnectionRecoveryDialog` to connect pool databases concurrently with a 6-second per-connection timeout, added always-available **Cancel**, **Retry Failed**, and **Continue Offline** controls, and guarded against stale post-cancel async callbacks so unreachable database pools never hang or lock out the application.

---

## [v1.2.10] — Previous Release

### Dependencies & Compatibility
- **TTQL `^1.0.3` Upgrade:** Updated the `ttql` query engine dependency from `^1.0.2` to `^1.0.3`, bringing full `sqlite3` `2.x` and `3.x` compatibility and a 160/160 pub.dev package score.

### Security & Testing
- **Chat Session Credential Scrubbing Verification:** Updated `multi_connection_test` to assert that plaintext `connectionString` values are stripped from `chat_sessions_table` after OS keychain migration.

---

## [v1.2.9] — Previous Release

### Query Profiler & Diagnostics
- **13-Engine Query Profiler & Dialect Registry:** Added a modular `QueryProfilerDialect` architecture (`QueryProfilerDialectRegistry` and `ProfilerDialectConstants`) with native execution-plan profiling across PostgreSQL, MySQL/MariaDB, SQLite, DuckDB, Microsoft SQL Server, Oracle, ClickHouse, MongoDB, Redis, Cassandra/ScyllaDB, PlanetScale, CockroachDB, and YugabyteDB.
- **Saved Query Plans & Profiler UI:** Upgraded `ChatProfilerPane`, `QueryProfilerDialog`, `CustomSqlInputCard`, and `RecentSessionsTable` to support saved execution plans, custom SQL profiling, and clearer diagnostic output.

### Pool Routing, Packaging & Security
- **Pooled Connection Routing & Server Pool Discovery:** Enhanced `ConnectionPoolService` and `SessionPoolSyncService` with improved pooled routing and multi-engine server pool discovery (including Redis logical database discovery).
- **Published `ttql` Package Integration:** Transitioned TableTalk from a local path dependency to the published `ttql` package on pub.dev.
- **Keychain Credential Leak Fix & Test Coverage:** Hardened `CredentialMigrationService`, `ChatRepository`, and `SessionController` to strip legacy plaintext connection strings from local SQLite chat session records after OS keychain migration, and added dedicated test suites for server connection pool discovery, keychain migration, and multi-engine query profiling.

---

## [v1.2.8] — Previous Release

### Fixed & Hardened
- **Non-Blocking Startup DB Pool Sync:** Decoupled active session database/pool synchronization from the initial app splash screen in `AppState` and parallelized `SessionPoolSyncService` connection attachment with a 6-second per-engine timeout so unreachable/stopped databases never stall app launch.
- **License UI Key Hygiene:** Replaced the voucher key example in `LicenseSettingsCard` with `TTLK-PRO1-XXXX-XXXX-XXXX-XXXX`.

---

## [v1.2.7] — Previous Release

### Licensing & Payments
- **Perpetual Pass & Community TTQL:** Introduced a one-time Perpetual License (£100.00, lifetime offline access) alongside free Community Edition TTQL federation across up to 2 connections.
- **Stripe Checkout & Webhook Reliability:** Added automatic Stripe Payment Link provisioning in `StripeCatalogService`, `CancellationToken` propagation in `ProductsFunction`, comma-separated webhook secret rotation support in `StripeSignatureVerifier`, and dev configuration parity in `EnvironmentConfig`.

### Architecture & TTQL Studio Stability
- **Modular State Services:** Decomposed `AppState` into focused single-responsibility services and controllers (`LicenseStateService`, `PromptSuggestionStateService`, `ConnectionPoolLifecycleCoordinator`, `UserPoolController`, `SchemaIngestionController`, `SettingsController`, and `TtqlWorksheetController`) while retaining `AppState` as the top-level coordinator.
- **Multi-Tab Worksheet Hardening:** Eliminated unmodifiable list and index-clamping crashes when managing more than 5 tabs, and added horizontal tab bar scrolling plus a quick-jump tab menu.
- **Website & Open-Source TTQL Alignment:** Updated showcase examples to canonical fluent TTQL syntax and finalized standalone open-source packaging for the `ttql` compiler, LSP daemon, CLI runner, and VS Code extension.

### TTQL Engine & Multi-Engine Adapter Fixes
- **Parser & Spread Operator Guardrails:** Added qualified `alias.column` support in array constants, `.select([...])` and `IN [...]` spread expansion, and zero-progress loop guards preventing inline-linter freezes.
- **Staging Precision & Defense-in-Depth Filtering:** Fixed lexicographic ordering on `NUMERIC`/`DECIMAL` strings (`settled_amount`), case-insensitive staged column resolution, and stage-level `WHERE` / `LEFT JOIN` predicate enforcement in SQLite staging.
- **Oracle, ScyllaDB, MongoDB & ClickHouse Fixes:** Added unquoted Oracle/Cassandra identifier generation and lowercase Oracle metadata/row normalization, automatic `ALLOW FILTERING` and fail-fast CQL v4 error/`timestamp` handling in `CassandraDbAdapter`, `$query` wrapper pushdown in `MongoDbAdapter`, and ClickHouse Docker access configuration.

---

## [v1.2.6] — Previous Release

### TTQL Analytical Engine & Fluent Pipeline
- **Fluent Pipeline Syntax:** First-class support for method-chained queries (`from()`, `.select()`, `.where()`, `.innerJoin()`, `.leftJoin()`, `.groupBy()`, `.having()`, `.orderBy()`, `.limit()`, `.offset()`, `.skip()`).
- **Analytical Aggregations:** Integrated aggregate engine supporting `count()`, `sum()`, `avg()`, `min()`, `max()` with deterministic column aliasing via `.as()`.
- **TTQL Studio Autocomplete & Highlighting:** Real-time syntax highlighting for pipeline methods, analytical functions, SQL `--` and C-style `//` comments, and context-aware dot autocomplete.
- **Decoupled Language Server (LSP):** Standalone JSON-RPC LSP daemon providing schema-aware diagnostics, hover documentation, and auto-completions for VS Code and desktop IDEs.

### Poly-Store Expansion & Pool Automation
- **13 Engine TTQL Support:** Query federation across SQLite, PostgreSQL, MySQL / MariaDB, MongoDB, Redis, DuckDB, Microsoft SQL Server, ClickHouse, PlanetScale, CockroachDB, YugabyteDB, Oracle Database, and Cassandra / ScyllaDB.
- **Automated Server Pool Import:** One-click catalog discovery and pool creation for entire database clusters (scanning `pg_database`, `information_schema.SCHEMATA`, `sys.databases`, `system.databases`).
- **Periodic Background Schema Indexing:** Configurable background synchronization (`AutoSchemaSyncService`) maintaining live synthetic foreign key graphs across active pools.

---

## [v1.2.5] — Previous Release

### Security & Reliability
- **Hardened Safe Read-Only Mode:** Fixed a bug where AI chat entity relationship analysis and query profiling could execute database DDL modifications (`CREATE INDEX`).
- **Strict Read-Only Handle Discipline:** Removed silent defensive fallback to read-write mode in SQLite database adapter (`SqliteDbAdapter`) to enforce strict local database immutability and fail-fast exception handling.
- **Session Read-Only Enforcement:** Added active session read-only transaction configuration for PostgreSQL (`SET default_transaction_read_only = on`) and MySQL (`SET SESSION TRANSACTION READ ONLY`).
- **Expanded Keyword Guard:** Added DDL and mutation keywords (`INDEX`, `REINDEX`, `MERGE`, `CALL`, `RENAME`, `COMMENT`, `LOCK`, `UNLOCK`, `COPY`, `FLUSH`, `KILL`) to `SqlValidator` protection rules.
- **Catastrophic Command Protection:** Added `SqlValidator.validateCatastrophicSql` safeguard blocking nuclear instance-destruction statements (`DROP DATABASE`, `DROP SCHEMA`, `SHUTDOWN`, `FLUSHALL`, `FLUSHDB`, `ATTACH DATABASE`) even when operating in Direct Write / YOLO Mode.

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
