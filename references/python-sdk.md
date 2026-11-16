# Develop and edit Python plugins

## Entry point

Develop in `<id>.py` and ship the same Python text as `<id>.plugin`. The loader reads metadata via AST. Use top-level literals and one concrete `BasePlugin` subclass. Required fields are `__id__` and `__name__`. IDs match `[A-Za-z][A-Za-z0-9_-]{1,31}`.

Optional fields include `__description__`, `__author__`, `__version__`, `__icon__`, `__app_version__`, `__sdk_version__`, `__requirements__` and `__category__`. Version constraints are strings such as `">=12.5.1"`. Legacy `__min_version__` specifies a minimum app version. Icons use a sticker-pack name and index, such as `exteraPlugins/1`. Catalog categories accept one to three values from `appearance`, `chats`, `media`, `privacy`, `tools`, `integrations`, `fun`, and `libraries`.

```python
from base_plugin import BasePlugin
from ui.settings import Switch

__id__ = "compact_preview"
__name__ = "Compact Preview"
__version__ = "1.0.0"
__app_version__ = ">=12.5.1"
__sdk_version__ = ">=1.4.4.3"
__category__ = "appearance"


class CompactPreviewPlugin(BasePlugin):
    def on_plugin_load(self):
        self.logger.info("Compact Preview enabled")

    def create_settings(self):
        return [
            Switch(key="compact_mode", text="Compact mode", default=False),
        ]
```

## Lifecycle and UI

- Register hooks, listeners, and background work in `on_plugin_load`. `on_app_event` receives `AppEvent.START/STOP/PAUSE/RESUME`. Make `on_plugin_unload` safe after partial initialization. Cancel owned tasks, listeners, observers and timers.
- Return `ui.settings` rows from `create_settings()`: `Header`, `Divider`, `Switch`, `Selector`, `Input`, `Text`, `EditText`, or `Custom`. A `Selector` persists an index, so preserve option order or migrate the saved value. Use `get_setting` and `set_setting`. Use `reload_settings=True` when page structure changes. Reuse a `SimpleSettingFactory` across page builds and `release()` a temporary one after its screens close. Check each control's callback signature.
- Use `MenuItemData` and `MenuItemType`. Verify the menu context and handle absent optional values. The SDK removes menus and Java hooks registered through its API on unload. Clean up direct registrations yourself.
- Use `android_utils.run_on_ui_thread` for UI and `client_utils.run_on_queue` for expensive work. Before applying a background result, check that the plugin is still active and the screen is current. Use `ui.alert` and `ui.bulletin` helpers with the target SDK signatures.

## Hooks and accounts

- Register outgoing message handling with `self.add_on_send_message_hook()` and implement `on_send_message_hook(self, account, params)`. Use `self.add_hook(...)` with the relevant TL names for requests, responses and updates, and implement the matching callbacks.
- Return `HookResult()` outside the target case. Use `HookResult(strategy=HookStrategy.MODIFY, params=params)` for modified outgoing messages. Other hooks use `request`, `response`, `update` or `updates`. `CANCEL` stops the operation. `MODIFY_FINAL` stops further plugin processing for that event, so reserve it for cases that require exclusivity.
- For actions triggered by a hook, use `self.client(account)` or pass the event's `account` explicitly. Background events can belong to an account other than the one displayed. Pass the `NotificationCenterDelegate` account explicitly too. Account-bound helpers require SDK 1.4.5.0+. The selected account is appropriate for a UI action explicitly scoped to it.
- `send_request` takes a TLObject and a callback with `response` and `error`. Handle both outcomes and log error codes or sanitized diagnostics.

## Java, data and dependencies

- Resolve the exact Java class and overload in the target build before using `hook_utils` or reflection. Check primitive and boxed argument types. `MethodHook` wraps before/after callbacks. `MethodReplacement` replaces the implementation. For early removal call `self.unhook_method(handle)`. If a signature is missing, skip only the feature that relies on it and log the reason.
- Use the class proxy DSL in `extera_utils.classes` for Java subclasses, including `Base`, `java_subclass` and `joverride`. `new_instance()` returns the Python peer. Pass `.java` to Java where required. DexMaker generates the proxy class, while the Python implementation stays in Python.
- Resolve files through `file_utils` or `elyx`. Use `IntentsManager` from `intents` for incoming Android intents when needed. Filter by action or URI and remove owned registrations. Use `extera_utils.text_formatting.parse_text` for markup and pass the parsed entities with the plain text. For manually built entities, check UTF-16 offsets around emoji.
- List PEP 508 requirements in `__requirements__`. The PIP controller installs pure-Python wheels into shared storage. Check already provided packages, Python 3.11 compatibility and version conflicts with other plugins.

[Plugin Class](https://plugins.exteragram.app/docs/plugin-class) · [Multi-account](https://plugins.exteragram.app/docs/multi-account) · [Plugin Settings](https://plugins.exteragram.app/docs/plugin-settings) · [Client Utilities](https://plugins.exteragram.app/docs/client-utils) · [Class Proxy](https://plugins.exteragram.app/docs/class-proxy) · [Xposed](https://plugins.exteragram.app/docs/xposed-hooking)
