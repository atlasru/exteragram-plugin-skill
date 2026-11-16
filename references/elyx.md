# Structured plugins with Elyx

Use Elyx for multiple modules, assets, localization, bundled wheels or Python bytecode releases. A `.elyx` or `.eaf` is a ZIP with one root `refmap.yaml`, `refmap.yml` or `refmap.json`. Paths are relative to the archive root. Preserve exact case and avoid duplicate paths or a wrapper directory.

## Layout

```text
refmap.yml
meta.yml
main.py
features/
  __init__.py
  render.py
locales/
  strings_en.yml
  strings_ru.yml
res/
  help.txt
```

```yaml
# refmap.yml
metainfo: meta.yml
main: main.py
strings: locales
assets: res
```

```yaml
# meta.yml
id: compact_preview
name: Compact Preview
version: "1.0.0"
app_version: ">=12.5.1"
sdk_version: ">=1.4.5.3"
```

Keep one concrete `BasePlugin` subclass in the entry module. Import helpers with paths such as `from features.render import ...`. Preserve the layout referenced by existing `refmap` entries. On migration from a single-file plugin, preserve its ID if valid. Elyx IDs use 2–32 ASCII letters, digits and underscores. If the existing ID is incompatible, agree on settings migration and test the upgrade on device. Move metadata to its own file and verify settings keys and enabled state.

Set `strings`, `assets` and `wheels` when those directories exist. Keep version constraints as strings. Express plugin dependencies in `requires`, for example `"shared_plugin (2.1.0)": "https://…"`. The required plugin must be installed and enabled.

## Imports and resources

- Use the public `elyx` facade. For dynamic local imports use `elyx.import_module(name)` to keep plugin scope. Relative imports work within packages. Avoid standard-library names and reserved SDK roots such as `elyx`, `android`, `java`, `org`, `ui`, and `base_plugin`.
- Access assets by exact name with `assets["help.txt"]`, or by normalized stem when unique. For optional assets use `get_environment().get("assets")`. Choose an explicit read method such as `content_string()` or `content_bytes()`.
- Put translations in `strings_<locale>.*`. Keep placeholder names consistent across languages. Lookup falls back from the selected locale to English and then to the key itself. Use named parameters, such as `strings("greeting", name=name)`. Metadata supports localized placeholders.
- `elyx.settings` uses the current plugin ID and the same keys as settings rows. Read with `settings.get(key, default)` and write with `settings.set(key, value)`. Pass `reload_settings=True` when the open page's structure changes.
- Declare PEP 508 requirements in `requirements`. Put bundled pure-Python wheels directly in `wheels`. Change the wheel version and filename when changing its contents because Elyx tracks extracted wheels by filename stem. Small helpers can be local Python packages.
- Reload clears Elyx modules. Stop custom threads and listeners on unload and make settings migration safe to rerun.

## Build and iterate

Run `elyb new` in an empty directory to scaffold, or create the layout manually. Run `elyb build --ast -v -nf` in the project root for a source build. For manual packaging, zip the project contents and use the `.elyx` extension. Install and enable an initial build before using the [development server](testing-release.md). See [Python bytecode](pyc.md) for compiled releases.

[Quick Start](https://plugins.exteragram.app/docs/elyx/quick-start) · [Project Structure](https://plugins.exteragram.app/docs/elyx/project-structure) · [Metadata](https://plugins.exteragram.app/docs/elyx/metadata) · [Modules and Imports](https://plugins.exteragram.app/docs/elyx/modules-and-imports) · [Public API](https://plugins.exteragram.app/docs/elyx/public-api)
