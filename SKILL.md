---
name: exteragram-plugin-skill
description: Create, edit, debug, and package exteraGram plugins. Use for BasePlugin and Chaquopy APIs, single-file .plugin and Elyx projects, Java hooks, Python .pyc releases, and Java/Kotlin DEX builds with R8.
---

# exteraGram plugins

1. Identify the format and preserve it unless the task calls for migration. A single-file `.plugin` is Python source with literal metadata and one `BasePlugin` subclass. An Elyx project has a root `refmap.*`. For DEX, find the build, entry point, and loader contract.
2. Check the target exteraGram build, SDK and Android API level. The SDK introduction lists Python 3.11, Chaquopy v16, SDK 1.4.4.3 and exteraGram 12.5.1+. Account-bound helpers require SDK 1.4.5.0+, and the Elyx Quick Start example uses 1.4.5.3+. Verify symbols and Java signatures against the actual target.
3. Preserve the plugin ID, settings keys, menu IDs and dependency contracts. Make migrations safe to rerun and test with prior-version data. Bump the version when preparing an update.
4. Register hooks on load and release plugin-owned resources on unload. Keep I/O off the Android UI thread. Use the account from the triggering event. Prefer public SDK APIs when they cover the operation.
5. Check the final package. When a target device is available, test clean install, update, enable, disable, re-enable and the changed behavior. Report the artifact, target versions and which checks ran on device.

## References

- [Python SDK](references/python-sdk.md) for metadata, lifecycle, UI, accounts and Java interop.
- [Elyx](references/elyx.md) for project layout, imports, assets, localization and dependencies.
- [Python bytecode](references/pyc.md) for ElyxBuilder, manual compilation and release checks.
- [DEX and R8](references/dex.md) for Java/Kotlin release formats, loader contracts and keep rules.
- [Testing and distribution](references/testing-release.md) for DevServer, ADB, regression tests and the catalog.
- [Documentation map](references/sources.md) for official documentation and tool sources.

Use Elyx bytecode to ship Python logic without `.py` files. Use DEX for JVM logic or an explicitly requested port to Java/Kotlin. Both forms can be analyzed. Keep credentials and logic that must remain confidential from device owners on a server. Preserve the host's normal installation and inspection flow.
