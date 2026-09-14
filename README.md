# TableTalk Public Release Distribution

Official binary releases and distribution manifests for [TableTalk](https://docs.tabletalkapp.net).

## Latest Production Release: v1.2.5

- **Windows Installer (x64):** [TableTalkSetup.exe](https://github.com/Dud3core-webdev/tabletalk-releases/releases/download/v1.2.5/TableTalkSetup.exe)
- **Linux Tarball (x64):** [TableTalk_Linux_v1.2.5.tar.gz](https://github.com/Dud3core-webdev/tabletalk-releases/releases/download/v1.2.5/TableTalk_Linux_v1.2.5.tar.gz)
- **Release Date:** 

### Release Notes
### Security & Reliability
- **Hardened Safe Read-Only Mode**: Fixed a bug where AI chat entity relationship analysis and query profiling could execute database DDL modifications (`CREATE INDEX`).
- **Strict Read-Only Handle Discipline**: Removed silent defensive fallback to read-write mode in SQLite database adapter (`SqliteDbAdapter`) to enforce strict local database immutability and fail-fast exception handling.
- **Session Read-Only Enforcement**: Added active session read-only transaction configuration for PostgreSQL (`SET default_transaction_read_only = on`) and MySQL (`SET SESSION TRANSACTION READ ONLY`).
- **Expanded Keyword Guard**: Added DDL and mutation keywords (`INDEX`, `REINDEX`, `MERGE`, `CALL`, `RENAME`, `COMMENT`, `LOCK`, `UNLOCK`, `COPY`, `FLUSH`, `KILL`) to `SqlValidator` protection rules.
- **Catastrophic Command Protection**: Added `SqlValidator.validateCatastrophicSql` safeguard blocking nuclear instance-destruction statements (`DROP DATABASE`, `DROP SCHEMA`, `SHUTDOWN`, `FLUSHALL`, `FLUSHDB`, `ATTACH DATABASE`) even when operating in Direct Write / YOLO Mode.


---

## Update Manifests
- [manifest.json](manifest.json)
- [version.json](version.json)

## Documentation & Showcase
Visit our documentation and showcase portal at [docs.tabletalkapp.net](https://docs.tabletalkapp.net).