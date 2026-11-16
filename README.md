# TableTalk Public Release Distribution

Official binary releases and distribution manifests for [TableTalk](https://docs.tabletalkapp.net).

## Latest Production Release: v1.2.7

- **Windows Installer (x64):** [TableTalkSetup.exe](https://github.com/Dud3core-webdev/tabletalk-releases/releases/download/v1.2.7/TableTalkSetup.exe)
- **Linux Tarball (x64):** [TableTalk_Linux_v1.2.7.tar.gz](https://github.com/Dud3core-webdev/tabletalk-releases/releases/download/v1.2.7/TableTalk_Linux_v1.2.7.tar.gz)
- **Release Date:** 2026-11-16

### Release Notes
### Licensing & Payments
- **Perpetual License & Free Community TTQL:** Transitioned to a one-time Perpetual License (£100.00, lifetime offline Ed25519 access) with Community Edition access to TTQL federation for up to 2 connections (`EntitlementService` and `TtqlWorksheetScreen`).
- **Automatic Stripe Payment Link Provisioning:** Updated `StripeCatalogService` to match Perpetual / Pro Studio catalog products and auto-provision Stripe Payment Links (`POST /v1/payment_links`) when missing, with full `CancellationToken` propagation through `ProductsFunction`.
- **Multi-Secret Webhook Rotation Support:** Updated `StripeSignatureVerifier` (`VerifyStripeSignature` and `VerifyGenericHmacSignature`) to support comma-separated signing secrets for seamless CLI/rotated secret transitions.
- **Dev Config & Launch Alignment:** Aligned fallback defaults in `EnvironmentConfig` (`config/dev.json` parity) and added `.vscode/launch.json` profiles for `TableTalk (Dev)` and `TableTalk (Prod)`.

### Architecture & State Decomposition
- **Single-Responsibility State Extraction:** Decomposed `AppState` into dedicated single-responsibility services and controllers (`LicenseStateService`, `PromptSuggestionStateService`, `ConnectionPoolLifecycleCoordinator`, `UserPoolController`, `SchemaIngestionController`, `SettingsController`, `TtqlWorksheetController`) while keeping `AppState` as the main reactive facade.

### TTQL Compiler, Parser & Staging Engine
- **Qualified Arrays & Spread Operator Fixes:** Added support for qualified `alias.column` entries in constant arrays (`const cols = [p.product_id, ...]`), `...spread` expansion inside `.select([...])` (preserving `entityAlias`) and `IN [...]`, plus fail-fast `TtqlParseException` checks and zero-progress loop guards in `TtqlParser` preventing inline-linter UI freezes.
- **Numeric Sorting & Staging Filter Enforcement:** Coerced decimal strings (`NUMERIC`/`DECIMAL`) to SQLite `REAL` values, added numeric-aware `ORDER BY` sorting (preserving aggregate projection aliases), case-insensitive column staging lookup, and defense-in-depth stage `WHERE` / `LEFT JOIN ... ON` filtering in `TtqlEphemeralStagingDatabase`.
- **Open-Source TTQL Package & Website Syntax:** Prepared the standalone `ttql` repository for open-source publication (`pub.dev` CLI executables, LSP server, VS Code extension workflows) and updated website examples to canonical fluent TTQL syntax.

### Multi-Engine Database Adapters & Studio Stability
- **TTQL Studio Multi-Tab Stability:** Fixed unmodifiable list mutation and tab index clamping crashes when opening/closing 5+ worksheet tabs, and added horizontal tab bar scrolling with a quick-jump tab selector menu.
- **Oracle Identifier & Case Normalization:** Disabled double-quoting for Oracle/Cassandra identifiers in `SqlQueryDialect` and `SqlExpressionVisitor`, added case-insensitive staging column lookup, and normalized `OracleDbAdapter` schema metadata and result row keys to lowercase (`ORA-00942` fix).
- **ScyllaDB / Cassandra `ALLOW FILTERING` & TCP CQL v4 Fixes:** Auto-appended `ALLOW FILTERING` for non-partition-key `WHERE` queries in `SqlQueryDialect` and `CassandraDbAdapter`, surfaced native CQL v4 server `ERROR` frames (`0x00`) fail-fast to eliminate 5-second CLI fallback timeouts, and added CQL `timestamp` (`0x000B`) decoding.
- **MongoDB & ClickHouse Fixes:** Wrapped MQL filters under `r'$query'` in `MongoDbAdapter.executeQuery` so `mongo_dart` applies pushed-down filters instead of scanning all documents, and configured ClickHouse Docker IPv4/IPv6 HTTP access.


---

## Update Manifests
- [manifest.json](manifest.json)
- [version.json](version.json)

## Documentation & Showcase
Visit our documentation and showcase portal at [docs.tabletalkapp.net](https://docs.tabletalkapp.net).