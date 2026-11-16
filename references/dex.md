# Java/Kotlin, DEX and R8

DEX is Android bytecode. ElyxBuilder compiles Python to `.pyc`; moving logic to DEX requires implementing that logic in Java or Kotlin behind a defined interface. D8 converts JVM bytecode to DEX. R8 can shrink, optimize and obfuscate JVM implementations. DEX remains analyzable and decompilable. Keep source and per-release R8 mappings in the project.

## Choose a packaging format

| Project | Release artifact | Verify |
| --- | --- | --- |
| Python bridge with embedded DEX | `.plugin` built by the project's packager, with literal metadata and a Python bridge | Chaquopy compatibility, DEX delivery, and bridge signatures |
| Elyx with a JVM implementation | `.elyx` with a Python bridge and local `classes.dex` asset | Loader behavior, `refmap`, asset lookup, and `.pyc` when used |
| Gradle JAR plugin | JAR with DEX and the project's manifest | Target client support for JAR installation, `Plugin-Id`, `Plugin-Class`, version, dependencies, and signature if used |

The first two formats use a Python entry point. The Python SDK docs leave standalone DEX and JAR installation contracts to the target loader. Verify the actual client for JAR loading and consult the bridge template used by the project.

## Build and load

1. Check the Gradle wrapper, JDK, Android SDK/Build Tools and target `minSdk`. Compile against the actual host API. Keep host-supplied Telegram, exteraGram and platform classes on the classpath or in `compileOnly`. Package your classes and required runtime dependencies. Relocate private libraries only as needed for collisions and preserve host types at the bridge boundary.
2. Reuse the project's build. For `io.github.exterastuff.plugin`, inspect `./gradlew tasks --all` or `.\gradlew.bat tasks --all`. Where configured, `buildDexRelease` creates `classes.dex`, `packagePluginJarRelease` requires `bundle`, and `signPluginJarRelease` requires signing configuration. In the pydex template, `just dex` builds debug DEX and `just ci-release 1.2.3` builds `dist/<id>.plugin`. Confirm commands in the checkout.
3. For a minimal Java project, package compiled classes in a JAR and invoke D8 with the correct Android API and SDK path. Supply compile-time dependencies as `--classpath` and packaged dependencies as program inputs.
   ```text
   d8 --release --min-api <API> --lib <android.jar> --output <existing-output-dir> <plugin-classes.jar>
   ```
   Verify the number of DEX files. A loader that expects one `classes.dex` needs explicit support for additional files.
4. Use a release-bundled DEX or an Elyx asset. Read an Elyx asset with `assets["classes.dex"].content_bytes()`. The single-`ByteBuffer` `InMemoryDexClassLoader` constructor requires Android API 26+. Its parent must resolve host classes, as the application context class loader does in the exteraStuff template. Load by fully qualified class name and match exact Java argument types. Manage bridge start and cleanup from `BasePlugin`. On unload, unregister JVM hooks and listeners, stop work and release references. For an Elyx release without `.py`, use a separate DEX asset because bridges that read their own source comments require that source file.
5. Sign through the supported packaging workflow if needed. Keep keystore material and passwords outside the release. Publish the final artifact checksum and describe its executable components. Use normal host installation.

## R8 keep rules

Keep the exact classes and members reached through a manifest, Python bridge or reflection. For example, when the bridge calls these two static methods:

```proguard
-keep class com.example.compact.EntryPoint {
    public static void start();
    public static void stop();
}
```

Add constructors, callbacks and signature types as required. For Kotlin, inspect the generated JVM signature and use `@JvmStatic` when the bridge needs a static object method. Check string-resolved names after R8. Save the mapping for retracing. Exclude Java/Kotlin source, debug JARs and accidental copies of host classes from release packages.

## Diagnose release builds

- `ClassNotFoundException`: check entry name, mapping, DEX contents, parent loader and the artifact actually passed.
- `NoSuchMethodException`: check reflection argument types, static vs. instance access, visibility, keep rules and JVM names.
- `NoClassDefFoundError` or linkage failure: check runtime dependencies, host class versions, relocation and Android API level.
- Compare debug and release on the same device. Inspect size, DEX and manifest, then test install, main behavior, disable, re-enable and reload. For a signed JAR, run `jarsigner -verify` and check the expected signer. In bridge builds, verify metadata in the standard install sheet.

[exteraStuff Gradle project](https://github.com/exteraStuff/gradle-plugin) · [Python and DEX template](https://github.com/exteraStuff/pydex-plugin-template) · [D8](https://developer.android.com/tools/d8) · [R8 keep rules](https://developer.android.com/topic/performance/app-optimization/keep-rules-overview) · [InMemoryDexClassLoader](https://developer.android.com/reference/dalvik/system/InMemoryDexClassLoader) · [Android Dynamic Code Loading](https://developer.android.com/privacy-and-security/risks/dynamic-code-loading)
