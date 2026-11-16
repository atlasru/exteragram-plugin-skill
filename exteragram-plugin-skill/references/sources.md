# Карта документации exteraGram

Сверено по навигации [официальной документации](https://plugins.exteragram.app/docs) на 15 ноября 2026 года. Сверяй меняющиеся интерфейсы с версией, установленной у пользователя, особенно для бета-сборок, классов Telegram и сторонней сборки DEX.

## Начало работы

- [Introduction](https://plugins.exteragram.app/docs) — рантайм, базовые версии и состав SDK.
- [Setup](https://plugins.exteragram.app/docs/setup) — среда разработки, устройство и базовый процесс.
- [First Plugin](https://plugins.exteragram.app/docs/first-plugin) — жизненный цикл и работа исходящего хука.

## Elyx

- [Elyx](https://plugins.exteragram.app/docs/elyx) — выбор структурированного формата.
- [Quick Start](https://plugins.exteragram.app/docs/elyx/quick-start) — минимальная структура, сборка и установка.
- [ElyxBuilder](https://plugins.exteragram.app/docs/elyx/elyxbuilder) — параметры CLI, уровни компиляции, исключения и cache reset.
- [Project Structure](https://plugins.exteragram.app/docs/elyx/project-structure) — поиск `refmap`, пути и корень архива.
- [Metadata](https://plugins.exteragram.app/docs/elyx/metadata) — ключи, идентификатор, версии и `requires`.
- [Modules and Imports](https://plugins.exteragram.app/docs/elyx/modules-and-imports) — локальная область импорта, байткод и очистка модулей.
- [Assets](https://plugins.exteragram.app/docs/elyx/assets) — файлы, имена и преобразование ресурсов для Android.
- [Localization](https://plugins.exteragram.app/docs/elyx/localization) — выбор языка, подстановки и fallback.
- [Settings Storage](https://plugins.exteragram.app/docs/elyx/settings) — хранение настроек Elyx.
- [Dependencies](https://plugins.exteragram.app/docs/elyx/dependencies) — локальные wheels и зависимость от другого плагина.
- [Development and Build](https://plugins.exteragram.app/docs/elyx/development) — начальная установка, протокол и сборка.
- [Public API](https://plugins.exteragram.app/docs/elyx/public-api) — интерфейсы фасада Elyx.
- [Troubleshooting](https://plugins.exteragram.app/docs/elyx/troubleshooting) — ошибки загрузки, импортов, ресурсов и reload.

## Базовый SDK

- [Plugin Class](https://plugins.exteragram.app/docs/plugin-class) — метаданные, жизненный цикл, хуки и меню.
- [Multi-account](https://plugins.exteragram.app/docs/multi-account) — привязка к аккаунту и версионный минимум.
- [Plugin Settings](https://plugins.exteragram.app/docs/plugin-settings) — строки настроек, callback и Java-фабрики.
- [Logging](https://plugins.exteragram.app/docs/logging) — уровни, теги, исключения и диагностика.
- [Class Proxy](https://plugins.exteragram.app/docs/class-proxy) — Java-подклассы, DSL и преобразования Python/Java.
- [Xposed Method Hooking](https://plugins.exteragram.app/docs/xposed-hooking) — before/after, replacement, фильтры и регистрация.

## Утилиты

- [Android Utilities](https://plugins.exteragram.app/docs/android-utils) — Android UI thread, слушатели, логирование и буфер обмена.
- [Client Utilities](https://plugins.exteragram.app/docs/client-utils) — очереди, запросы и клиенты аккаунта.
- [Text Formatting](https://plugins.exteragram.app/docs/text-formatting) — преобразование разметки и Telegram entities.
- [Hook Utilities](https://plugins.exteragram.app/docs/hook-utils) — поиск классов и доступ к полям.
- [File Utilities](https://plugins.exteragram.app/docs/file-utils) — директории, пути и файловые операции.
- [Intents](https://plugins.exteragram.app/docs/intents) — перехват входящих Android intents и фильтры по URI.
- [Alert Dialog Builder](https://plugins.exteragram.app/docs/alert-dialog-builder) — диалоги и их lifecycle.
- [Bulletin Helper](https://plugins.exteragram.app/docs/bulletin-helper) — нативные уведомления Telegram и выбор фрагмента.

## Распространение и окружение

- [Plugin Repositories](https://plugins.exteragram.app/docs/plugin-repositories) — Telegram, Git-индекс и верификация.
- [Development Server](https://plugins.exteragram.app/docs/dev-server) — команды для разработки и удалённый отладчик.
- [PIP & Dependencies](https://plugins.exteragram.app/docs/pip) — требования и ограничения менеджера пакетов.
- [Available Libraries](https://plugins.exteragram.app/docs/available-libraries) — предустановленные пакеты.
- [Common Telegram Classes](https://plugins.exteragram.app/docs/common-source-classes) — ориентиры в исходниках Telegram.

## Первичные источники инструментов

- [Публикации Python SDK](https://github.com/exteraSquad/plugins-pysdk-builds) — репозиторий команды exteraGram.
- [ElyxBuilder](https://github.com/shareui/ElyxBuilder) и [пакет PyPI](https://pypi.org/project/ElyxBuilder/) — исходники и версии CLI.
- [Исходники Gradle-плагина exteraStuff](https://github.com/exteraStuff/gradle-plugin) и [Gradle Plugin Portal](https://plugins.gradle.org/plugin/io.github.exterastuff.plugin) — DEX, R8 и JAR. Сверяй документацию с тегом своей версии.
- [Исходники Python-моста exteraStuff](https://github.com/exteraStuff/pydex-plugin-template) — пример проекта с Kotlin/DEX и Python-точкой входа.
- [Python 3.11 py_compile](https://docs.python.org/3.11/library/py_compile.html) и [Android D8](https://developer.android.com/tools/d8) — параметры инструментов компиляции.

