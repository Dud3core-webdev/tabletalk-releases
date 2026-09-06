# TableTalk Releases

Welcome to the TableTalk Release Distribution Repository! 
This repository contains the latest compiled binaries, update manifests, and the Angular showcase landing page.

## Latest Release: v1.0.1 (September 2026)

### What's New
- **Improved Deployment Pipelines**: CI/CD and PowerShell publish scripts have been fortified to safely deploy new releases without overwriting or deleting historical versions (like v1.0.0).
- **Roadmap Updates**: Native Linux and MacOS packaging support has been formally added to our Upcoming roadmap.
- **Enhanced Test Stability**: Fixed a critical Dart test runner CI hang related to fake-time clock starvation during asynchronous local LLM health checks.
- **Clean Artifacts**: Reduced release payload size by stripping unneeded `.pdb`, `.lib`, and `.exp` debug symbols from the installer and standalone zip bundles.

---

### Previous Releases

#### v1.0.0
- Initial production launch.
- Included TableTalk Angular Showcase site.
- Integrated .NET 10 Isolated Worker backend for Stripe webhooks and licensing.
- Local Ollama AI, SQLite, PostgreSQL, and secure Ed25519 licensing support.
