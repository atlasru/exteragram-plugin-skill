# Java/Kotlin, DEX и R8

DEX хранит байткод JVM-реализации, преобразованный в формат Android. Прямого преобразования Python-плагина в DEX в ElyxBuilder нет. Для переноса выдели логику подходящим интерфейсом и реализуй её на Java/Kotlin. D8 выполняет конвертацию. R8 может удалить недостижимый код, оптимизировать и обфусцировать внутренние имена. DEX можно проанализировать и декомпилировать. Для релиза без исходников отдавай только штатный установочный артефакт, сохраняя исходники и R8 mapping в хранилище проекта.

## Выбери совместимый способ доставки

| Как устроен проект | Что выпускать | Что проверить |
| --- | --- | --- |
| Python-мост с DEX | `.plugin` с видимыми метаданными и мостом, собранный проектным упаковщиком | Совместимость Chaquopy, способ передачи DEX и сигнатуры вызовов |
| Elyx с JVM-реализацией | `.elyx` с Python-мостом и локальным `classes.dex` в assets | Совместимость загрузчика, `refmap`, разрешение DEX-ресурса и, при необходимости, `.pyc` |
| JAR-формат Gradle-инструментария | JAR с DEX и манифестом согласно контракту проекта | Поддержку JAR-установки целевой версией, `Plugin-Id`, `Plugin-Class`, версию, зависимости и подпись при наличии |

Первые два варианта используют Python-точку входа. Варианты DEX зависят от загрузчика проекта и клиента. На сайте Python SDK нет универсального установочного контракта для отдельного файла `.dex` или JAR. Для JAR сверяй реализацию целевого клиента и испытывай установку. Для Python-моста ориентируйся на исходники используемого шаблона.

## Сборка и мост

1. Определи Gradle wrapper, JDK, Android SDK/Build Tools и `minSdk` фактического проекта. Выбери совместимый API хоста для компиляции. Классы Telegram, exteraGram и платформы, предоставляемые хостом, держи в classpath/`compileOnly`. В DEX включай собственные классы и обязательные runtime-зависимости. Relocation применяй к частным библиотекам при коллизиях; типы хоста на границе моста сохраняй.
2. Используй существующую сборку. Для `io.github.exterastuff.plugin` проверь задачи через `./gradlew tasks --all` или `.\gradlew.bat tasks --all`. При поддержке Gradle-модулем `buildDexRelease` собирает `classes.dex`. `packagePluginJarRelease` появляется при конфигурации `bundle`, а `signPluginJarRelease` — при наличии настроек подписи. В проекте на основе `pydex-plugin-template` рецепт `just dex` собирает debug DEX, `just ci-release 1.2.3` даёт `dist/<id>.plugin`. Сверяй команды и версии по checkout проекта.
3. Для минимального Java-проекта без Gradle можно скомпилировать `.class` в jar и затем вызвать D8. Подставь реальный номер API и путь к SDK. Дополнительные compile-time зависимости перечисляй как `--classpath`, а те, которые должны ехать с плагином, передавай на вход как program files.
   ```text
   d8 --release --min-api <API> --lib <android.jar> --output <existing-output-dir> <plugin-classes.jar>
   ```
   Проверь ожидаемое число dex-файлов. Упаковщик или мост для одного `classes.dex` не подхватит `classes2.dex` автоматически.
4. Если нужен Python-мост, используй встроенный в релиз локальный DEX или ресурс Elyx. В Elyx бери `assets["classes.dex"].content_bytes()`. Для `InMemoryDexClassLoader` с одним `ByteBuffer` нужен Android API 26+. Parent class loader должен видеть классы хоста, например loader контекста приложения в проверенном шаблоне exteraStuff. Загружай класс по полному имени, вызывай реально объявленные методы с точными Java-типами. `BasePlugin` управляет запуском и очисткой моста. Явно отменяй зарегистрированные из JVM слушатели и хуки, завершай задачи и разрывай ссылки при выгрузке. Для релиза без `.py` в Elyx храни DEX отдельным ассетом: шаблон, который читает Base64 из комментариев собственного файла, зависит от исходника.
5. Для подписи используй механизм поддерживаемого формата и храни keystore/пароли вне релиза. Публикуй контрольную сумму финального файла. Используй штатный путь установки и покажи, какие компоненты исполняются.

## Правила для R8

Оставь узкие keep rules для класса и методов, на которые ссылается Python, манифест или рефлексия. Например, если Java-мост вызывает именно эти статические методы:

```proguard
-keep class com.example.compact.EntryPoint {
    public static void start();
    public static void stop();
}
```

Добавляй конструктор, callback и сигнатурные типы по необходимости. В Kotlin проверяй сгенерированную сигнатуру: при вызове статического метода объекта может понадобиться `@JvmStatic`. Загружаемые по строковым именам классы проверь после R8. Храни mapping каждой версии для восстановления трассировок. Убери из релиза исходные `.java/.kt`, отладочные JAR и случайные копии SDK/классов приложения.

## Диагностика релиза

- `ClassNotFoundException` — проверь имя entrypoint, R8 mapping, наличие класса в DEX, parent loader и переданный артефакт.
- `NoSuchMethodException` — проверь рефлексивные типы, `static` или экземпляр, видимость, keep rules и Kotlin/JVM-имя.
- `NoClassDefFoundError`/ошибка линковки — проверь dependency classpath, несовпадение версии класса хоста, relocation и реальный `minSdk`.
- Сравни debug и release на одном устройстве. Проверь размер, наличие ожидаемого DEX/manifest, загрузку, вызов основной функции и disable → enable → reload. При подписи проверь JAR через `jarsigner -verify` и сверяй ожидаемого издателя. Для Bridge-артефакта проверь доступность метаданных со штатного экрана установки.

[Gradle-проект exteraStuff](https://github.com/exteraStuff/gradle-plugin) · [Шаблон Python + DEX](https://github.com/exteraStuff/pydex-plugin-template) · [D8](https://developer.android.com/tools/d8) · [R8 keep rules](https://developer.android.com/topic/performance/app-optimization/keep-rules-overview) · [InMemoryDexClassLoader](https://developer.android.com/reference/dalvik/system/InMemoryDexClassLoader) · [Android Dynamic Code Loading](https://developer.android.com/privacy-and-security/risks/dynamic-code-loading)

