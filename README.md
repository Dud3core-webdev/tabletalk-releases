# TableTalk Public Release Distribution

Official binary releases and distribution manifests for [TableTalk](https://docs.tabletalkapp.net).

## Latest Production Release: v1.2.6

- **Windows Installer (x64):** [TableTalkSetup.exe](https://github.com/Dud3core-webdev/tabletalk-releases/releases/download/v1.2.6/TableTalkSetup.exe)
- **Linux Tarball (x64):** [TableTalk_Linux_v1.2.6.tar.gz](https://github.com/Dud3core-webdev/tabletalk-releases/releases/download/v1.2.6/TableTalk_Linux_v1.2.6.tar.gz)
- **Release Date:** 2026-10-29

### Release Notes
### Added & Expanded
- **TTQL Analytical Engine & Fluent Pipeline**: Introduced method-chained fluent pipeline (`from()`, `.select()`, `.where()`, `.innerJoin()`, `.leftJoin()`, `.groupBy()`, `.having()`, `.orderBy()`, `.limit()`, `.offset()`, `.skip()`) with analytical aggregations (`count()`, `sum()`, `avg()`, `min()`, `max()`, `.as()`).
- **TTQL Studio Autocomplete & Syntax Highlighter**: Real-time syntax highlighting for pipeline methods, analytical functions, SQL `--` and C-style `//` comments, and context-aware dot autocomplete.
- **Standalone TTQL Architecture**: Decoupled open-source TTQL compiler, runtime, and JSON-RPC Language Server Protocol (LSP) daemon for IDE integration.
- **13 Engine TTQL Poly-Store Support**: Query federation across SQLite, PostgreSQL, MySQL / MariaDB, MongoDB, Redis, DuckDB, Microsoft SQL Server, ClickHouse, PlanetScale, CockroachDB, YugabyteDB, Oracle Database, and Cassandra / ScyllaDB.
- **Automated Server Pool Import**: One-click catalog discovery and pool creation for entire database clusters.


---

## Update Manifests
- [manifest.json](manifest.json)
- [version.json](version.json)

## Documentation & Showcase
Visit our documentation and showcase portal at [docs.tabletalkapp.net](https://docs.tabletalkapp.net).