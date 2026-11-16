# Debug, test and distribute

## Device development

1. Enable the plugin engine and developer mode in a test build. Run `adb devices` and use `adb -s <serial>` for multiple devices. Forward the local DevServer with `adb forward tcp:42690 tcp:42690`. Use loopback, and disable the server and remove your forwarding rule when finished as appropriate.
2. Open a single-file `.plugin` in exteraGram or use an existing DevServer client. The connection carries consecutive UTF-8 JSON objects. `@` names the action and `#` is a unique request ID echoed in its response. `write_plugin` takes `plugin_id` and complete Python source as `content`. `reload_plugin` reloads a single-file plugin.
3. Install and enable an Elyx archive once and verify its ID against local metadata. Use the project's `elyx_dev_client.py` or compatible client and check `--help`. Actions include `elyx_ping`, `get_elyx_plugins`, `elyx_compare_folder` and `elyx_changes`. Send zlib-compressed JSON encoded in base64 as `data`, with paths relative to the project root. Match the response `#` and check `success` and `errors`.
4. Capture logcat with the `plugin.<id>` tag, for example `adb logcat -s 'plugin.compact_preview:D' '*:S'`. Use `self.logger.exception` for exceptions and omit message content, tokens and personal data. Configure the IDE debugger separately, for example on port 5678, and choose ADB forwarding direction based on which side connects. The DevServer port defaults to 42690.

## Regression checks

| Scenario | Expected observation |
| --- | --- |
| Static checks | Python 3.11 syntax, literal metadata, archive layout, target Java signatures and SDK minimum |
| Clean install | Correct ID, name and version, and expected behavior with default settings |
| Upgrade | Same ID, old settings still work, migrations can rerun, and the catalog offers the release when relevant |
| Two disable/enable cycles and reload | No duplicate menus, hooks, threads, observers or stale callbacks |
| Multiple accounts for Telegram actions | Background work uses the triggering account across UI account switches |
| Failure paths | Missing context, network, dependencies or signatures do not hang the host |
| Release artifact | The actual `.plugin` or Elyx archive installs and runs, including any `.pyc` or DEX |

Report device tests as unrun when no target is available and provide exact reproduction steps. AST checks validate syntax. Test loading and Java binding in the target client.

## Plugin catalog

- For public Telegram channels, publish supported plugin files as documents. The catalog checks up to 2,000 recent documents, up to 2 MB per `.plugin` and 10 MB per archive. Publish a new document or replace the prior file to update. The catalog groups releases by ID.
- For Git, generate UTF-8 `extera-plugins.json` with `schema: 1` and at most 1 MiB. The root needs `name` and `plugins`. Keep one current entry per ID with `id`, `name`, `version`, `file`, `sha256` and `size`. Use paths relative to the index URL or absolute HTTPS URLs. Size is in bytes and the hash is lowercase hex. ID and version must match the artifact. Set a suitable category and regenerate the index whenever a file changes.
- Increase the numeric release version, for example from `1.9.1` to `1.10.0`. The catalog ignores suffixes like `-beta` for ordering. Verify index and download URLs, sizes and hashes. Use built-in catalog refresh when cached. Git installations use the host's unknown-source settings and installation analysis.
- For JAR, follow the target loader's distribution path and test independently of the Python/Elyx catalog.
- Prepare artifacts and indexes locally. Publish them only when requested.

[Setup](https://plugins.exteragram.app/docs/setup) · [Development Server](https://plugins.exteragram.app/docs/dev-server) · [Elyx Development](https://plugins.exteragram.app/docs/elyx/development) · [Logging](https://plugins.exteragram.app/docs/logging) · [Plugin Repositories](https://plugins.exteragram.app/docs/plugin-repositories)
