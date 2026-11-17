# TableTalk Public Release Distribution

Official binary releases and distribution manifests for [TableTalk](https://docs.tabletalkapp.net).

## Latest Production Release: v1.2.8

- **Windows Installer (x64):** [TableTalkSetup.exe](https://github.com/Dud3core-webdev/tabletalk-releases/releases/download/v1.2.8/TableTalkSetup.exe)
- **Linux Tarball (x64):** [TableTalk_Linux_v1.2.8.tar.gz](https://github.com/Dud3core-webdev/tabletalk-releases/releases/download/v1.2.8/TableTalk_Linux_v1.2.8.tar.gz)
- **Release Date:** 2026-11-17

### Release Notes
### Fixed & Hardened
- **Non-Blocking Startup DB Pool Sync:** Moved `_syncSessionDatabaseConnection` to run asynchronously after local SQLite persistence loads in `AppState`, and updated `SessionPoolSyncService.syncSessionConnectionPool` to attach pooled connections concurrently with a 6-second per-engine timeout so offline database clusters never block the splash screen.
- **License UI Key Hygiene:** Replaced the real voucher key example in `LicenseSettingsCard` with the synthetic placeholder `TTLK-PRO1-XXXX-XXXX-XXXX-XXXX`.


---

## Update Manifests
- [manifest.json](manifest.json)
- [version.json](version.json)

## Documentation & Showcase
Visit our documentation and showcase portal at [docs.tabletalkapp.net](https://docs.tabletalkapp.net).