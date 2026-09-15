# TableTalk Changelog & Release Notes

All notable changes, architectural milestones, and engine updates for TableTalk are documented here.

---

## [1.2.5] - 2026-09-14

### Security & Reliability
- **Hardened Safe Read-Only Mode**: Fixed a bug where AI chat entity relationship analysis and query profiling could execute database DDL modifications (`CREATE INDEX`).
- **Strict Read-Only Handle Discipline**: Removed silent defensive fallback to read-write mode in SQLite database adapter (`SqliteDbAdapter`) to enforce strict local database immutability and fail-fast exception handling.
- **Session Read-Only Enforcement**: Added active session read-only transaction configuration for PostgreSQL (`SET default_transaction_read_only = on`) and MySQL (`SET SESSION TRANSACTION READ ONLY`).
- **Expanded Keyword Guard**: Added DDL and mutation keywords (`INDEX`, `REINDEX`, `MERGE`, `CALL`, `RENAME`, `COMMENT`, `LOCK`, `UNLOCK`, `COPY`, `FLUSH`, `KILL`) to `SqlValidator` protection rules.
- **Catastrophic Command Protection**: Added `SqlValidator.validateCatastrophicSql` safeguard blocking nuclear instance-destruction statements (`DROP DATABASE`, `DROP SCHEMA`, `SHUTDOWN`, `FLUSHALL`, `FLUSHDB`, `ATTACH DATABASE`) even when operating in Direct Write / YOLO Mode.

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
