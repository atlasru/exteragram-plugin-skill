# Разработка и изменение Python-плагинов

## Входной файл

Для разработки используй `<id>.py`. Однофайловый релиз `<id>.plugin` содержит тот же Python-текст. Загрузчик читает метаданные через AST, поэтому поля задаются литералами на верхнем уровне. Обязательны `__id__` и `__name__`. Для ID подходит шаблон `[A-Za-z][A-Za-z0-9_-]{1,31}`. Оставляй ровно один реализуемый класс плагина в точке входа.

Дополнительные поля: `__description__`, `__author__`, `__version__`, `__icon__`, `__app_version__`, `__sdk_version__`, `__requirements__`, `__category__`. Совместимость задаётся выражением вроде `">=12.5.1"`. Старое `__min_version__` соответствует нижней границе версии приложения. Иконка имеет вид `exteraPlugins/1`. Для каталога укажи от одной до трёх подходящих категорий: `appearance`, `chats`, `media`, `privacy`, `tools`, `integrations`, `fun`, `libraries`.

Минимальная точка входа с безопасной локальной проверкой через настройки и logcat.

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

## Жизненный цикл и интерфейс

- `on_plugin_load` срабатывает при включении и восстановлении активного плагина. Регистрацию, запуск фоновых задач и создание слушателей выполняй в жизненном цикле. `on_app_event` получает `AppEvent.START/STOP/PAUSE/RESUME`. `on_plugin_unload` должен переносить частично завершённую инициализацию и отменять собственные задачи, слушатели, наблюдателей и таймеры.
- `create_settings()` возвращает строки `ui.settings`: `Header`, `Divider`, `Switch`, `Selector`, `Input`, `Text`, `EditText`, `Custom`. `Selector` хранит индекс, поэтому сохраняй порядок вариантов или мигрируй сохранённое значение. Используй `get_setting` и `set_setting`. Включай `reload_settings=True`, когда меняется состав страниц. `SimpleSettingFactory` переиспользуй между вызовами `create_settings`; временную фабрику освободи через `release()` после закрытия использующих её экранов. Подписки и колбэки проверяй по сигнатуре конкретного контрола.
- Собирай меню через `MenuItemData` и `MenuItemType`. Проверь контекст конкретного меню в SDK и обрабатывай отсутствие необязательных значений. SDK сам убирает зарегистрированные через него меню и Java-хуки при выгрузке. Самостоятельные регистрации чисти вручную.
- UI меняй через `android_utils.run_on_ui_thread`, тяжёлую работу передавай в `client_utils.run_on_queue`. При возврате из фона проверь, что плагин ещё включён и нужный экран актуален. Для диалогов и уведомлений используй `ui.alert` / `ui.bulletin` по сигнатурам целевого SDK.

## Хуки и аккаунты

- Для исходящих сообщений зарегистрируй `self.add_on_send_message_hook()`, затем реализуй `on_send_message_hook(self, account, params)`. Для TL-запросов, ответов и обновлений зарегистрируй нужные имена через `self.add_hook(...)` и реализуй соответствующий callback.
- Вне целевой ветки верни `HookResult()`. Для изменения верни `HookResult(strategy=HookStrategy.MODIFY, params=params)` для исходящего сообщения. Для остальных типов передавай соответствующее поле `request`, `response`, `update` или `updates`. `CANCEL` прекращает операцию. `MODIFY_FINAL` прекращает дальнейшую обработку события другими плагинами и подходит только при необходимости такого поведения.
- При чтении данных, отправке сообщения или выполнении запроса из хука передай его `account` явно либо возьми `self.client(account)`. Аккаунт на экране может отличаться от аккаунта фонового события. У `NotificationCenterDelegate` свой параметр `account`, передай его явно. Эти хелперы привязки к аккаунту требуют SDK ≥ 1.4.5.0.
- `send_request` получает TLObject и callback с `response` и `error`. Проверяй обе ветки. В логах оставляй коды ошибок и обезличенные диагностические данные. Для UI, открытого пользователем, выбор текущего аккаунта допустим, когда это часть поведения функции.

## Java, файлы и интеграции

- Если публичного метода SDK недостаточно, найди точное имя Java-класса и перегрузку в целевой сборке. Для отражения используй `hook_utils` и Java reflection. Проверь примитивы и boxed-типы. `MethodHook` даёт обработчики до и после вызова, `MethodReplacement` заменяет исходный метод. Для раннего ручного снятия используй `self.unhook_method(handle)`. При отсутствии класса или сигнатуры пропусти только зависимую функцию и запиши причину.
- Для Java-подклассов используй DSL из `extera_utils.classes`: `Base`, `java_subclass`, `joverride` и другие методы по паспорту API. `new_instance()` даёт Python-объект. Для передачи экземпляра в Java используй `.java`, если его требует API. DexMaker в Class Proxy создаёт Java-прокси во время работы, Python-реализация методов остаётся Python-кодом.
- Для путей используй `file_utils` или `elyx`. Входящие Android intents перехватывай через `from intents import IntentsManager`, если нужен именно такой сценарий. Сужай фильтры по action/URI и освобождай собственные регистрации. Для разметки используй `extera_utils.text_formatting.parse_text` и передавай полученные `entities` вместе с очищенным текстом. При ручном построении сущностей проверяй смещения в UTF-16 на emoji.
- `__requirements__` принимает список PEP 508. PIP-подсистема ставит pure-Python wheels в общее хранилище, поэтому несовместимые ограничения версий влияют на другие плагины. Сверяй список предустановленных библиотек и доступность пакета для Python 3.11.

[Plugin Class](https://plugins.exteragram.app/docs/plugin-class) · [Multi-account](https://plugins.exteragram.app/docs/multi-account) · [Plugin Settings](https://plugins.exteragram.app/docs/plugin-settings) · [Client Utilities](https://plugins.exteragram.app/docs/client-utils) · [Class Proxy](https://plugins.exteragram.app/docs/class-proxy) · [Xposed](https://plugins.exteragram.app/docs/xposed-hooking)

