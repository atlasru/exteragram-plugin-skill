# Структурированные плагины Elyx

Используй Elyx для нескольких модулей, ресурсов, локализаций, локальных wheels и релизов с Python-байткодом. `.elyx` и `.eaf` — ZIP-контейнеры. В корне архива лежит один `refmap.yaml`, `refmap.yml` или `refmap.json`. Все указанные пути относительны к корню архива. Устрани лишний внешний каталог, разницу регистра и неоднозначные дубли.

## Минимальная структура

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

Оставь один наследник `BasePlugin` в `main.py` и используй `from features.render import ...`. При изменении существующей структуры сохраняй расположение файлов, указанное в `refmap`. Если переносишь обычный плагин, сохрани ID, когда он допустим для Elyx. Elyx-ID допускает 2–32 ASCII-буквы, цифры и подчёркивания. При несовместимом старом ID согласуй способ переноса настроек и проверь обновление на устройстве. Метаданные перенеси в отдельный файл и проверь сохранённые ключи настроек и состояние включения.

Добавляй ключи `strings`, `assets` и `wheels` при наличии этих каталогов. Сохраняй поля совместимости строками. Для другого плагина используй `requires` вида `"shared_plugin (2.1.0)": "https://…"`; зависимый плагин должен быть установлен и включён.

## Импорт и ресурсы

- Используй публичный фасад `elyx`, например `from elyx import strings, settings`. Для динамического импорта локального Python-модуля используй `elyx.import_module(name)`. Обычный `importlib.import_module` может потерять контекст плагина. Для соседей внутри пакета доступны относительные импорты. Избегай имён из stdlib и зарезервированных корней SDK (`elyx`, `android`, `java`, `org`, `ui`, `base_plugin`).
- Ресурс бери по точному имени через `assets["help.txt"]` или нормализованному stem при отсутствии коллизий. Для необязательного корня проверяй `get_environment().get("assets")`. Используй явный метод чтения, например `content_string()` или `content_bytes()`.
- Храни ключи локализации в `strings_<locale>.*`. Проверяй наличие ключей и совпадение имён подстановок между языками. Поиск идёт от выбранного языка к английскому, затем возвращает ключ. Вызывай строки с именованными аргументами, например `strings("greeting", name=name)`. Метаданные тоже поддерживают локализуемые подстановки.
- `elyx.settings` связан с текущим ID и теми же ключами, которые используют строки настроек. `settings.get(key, default)` читает, `settings.set(key, value)` пишет. Для изменения структуры открытой страницы передай `reload_settings=True`.
- Для внешних пакетов используй `requirements` с совместимыми PEP 508-ограничениями. Для локальных чистых wheels укажи `wheels` и положи файлы прямо в этот каталог. Elyx использует stem файла как идентификатор извлечённой версии. При обновлении содержимого меняй версию wheel и его имя. Для небольшого helper-модуля удобен локальный Python-пакет.
- Перезагрузка очищает модули пространства Elyx, но собственные потоки и Java-слушатели нужно завершить в `on_plugin_unload`. Код обновления настроек должен выдерживать повторный запуск.

## Быстрый цикл

Для нового проекта используй `elyb new` из пустой папки либо создай структуру вручную. Для сборки из исходников запусти `elyb build --ast -v -nf` в корне проекта. При ручной упаковке добавляй в ZIP содержимое проекта и укажи расширение `.elyx`. Осмотри архив перед установкой. Для разработки загрузи и включи начальную сборку, затем синхронизируй правки через DevServer по [сценарию отладки](testing-release.md). Для релиза с байткодом смотри [сборку .pyc](pyc.md).

[Quick Start](https://plugins.exteragram.app/docs/elyx/quick-start) · [Project Structure](https://plugins.exteragram.app/docs/elyx/project-structure) · [Metadata](https://plugins.exteragram.app/docs/elyx/metadata) · [Modules and Imports](https://plugins.exteragram.app/docs/elyx/modules-and-imports) · [Public API](https://plugins.exteragram.app/docs/elyx/public-api)

