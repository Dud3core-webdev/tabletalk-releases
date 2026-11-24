# TableTalk Public Release Distribution

Official binary releases and distribution manifests for [TableTalk](https://docs.tabletalkapp.net).

## Latest Production Release: v1.2.10

- **Windows Installer (x64):** [TableTalkSetup.exe](https://github.com/Dud3core-webdev/tabletalk-releases/releases/download/v1.2.10/TableTalkSetup.exe)
- **Linux Tarball (x64):** [TableTalk_Linux_v1.2.10.tar.gz](https://github.com/Dud3core-webdev/tabletalk-releases/releases/download/v1.2.10/TableTalk_Linux_v1.2.10.tar.gz)
- **Release Date:** 2026-11-24

### Release Notes
### Dependencies & Compatibility
- **TTQL `^1.0.3` Upgrade:** Updated the `ttql` query engine dependency from `^1.0.2` to `^1.0.3`, bringing full `sqlite3` `2.x` and `3.x` compatibility and a 160/160 pub.dev package score.

### Security & Testing
- **Chat Session Credential Scrubbing Verification:** Updated `multi_connection_test` to assert that plaintext `connectionString` values are stripped from `chat_sessions_table` after OS keychain migration.


---

## Update Manifests
- [manifest.json](manifest.json)
- [version.json](version.json)

## Documentation & Showcase
Visit our documentation and showcase portal at [docs.tabletalkapp.net](https://docs.tabletalkapp.net).