# Ship an Elyx package without .py source

Elyx loads `.pyc` modules and packages with `__init__.pyc`. Single-file `.plugin` metadata is read from Python source. To ship Python logic as bytecode, move metadata into an Elyx file and build separately from the editable source tree. Bytecode exposes names, strings, constants and algorithms to analysis. Keep all credentials and signing keys out of artifacts.

## ElyxBuilder

1. Use a pinned compatible version in a **CPython 3.11** environment. On Windows with the launcher, run `py -3.11 -m venv .venv` and `.\.venv\Scripts\python.exe -m pip install ElyxBuilder`. Confirm the actual Python version, `elyb --version` and `elyb build --help`. You can call `.\.venv\Scripts\elyb.exe` directly.
2. From the directory containing `refmap.yml`, check the source build with `elyb build --ast -v -nf`. Separately build bytecode with `elyb build --compile 0 -v -nf`. Output goes to `builds/`. The flags `--ast` and `--compile` are mutually exclusive. Optimization level 0 preserves normal semantics. Level 1 removes assertions and changes `__debug__`. Level 2 also strips docstrings. Check dependencies on these features before using levels 1 or 2.
3. If output appears stale, add `--reset` to the compiled build. Check `compilationIgnore` for Python files left as source and `ignoreAll` for paths excluded entirely. `-nf` omits the builder configuration directory. Inspect nested wheels, archives and resources when the release is meant to contain no source.

## Manual build

Copy required files to staging and compile every shipped module with CPython 3.11. Remove each corresponding source only from staging after compilation succeeds. Compile package initializers as `__init__.pyc`. Put bytecode beside its logical module path outside `__pycache__`. Update `refmap.main` to the actual entry, such as `main.pyc`, and explicitly point to the separate metadata file.

```python
import py_compile

py_compile.compile(
    "stage/main.py",
    cfile="stage/main.pyc",
    dfile="main.py",
    doraise=True,
    optimize=0,
    invalidation_mode=py_compile.PycInvalidationMode.CHECKED_HASH,
)
```

Use archive-relative paths as `dfile` for tracebacks and to keep developer paths out of code objects. Zip the staging contents with `refmap.*` at the archive root and name the result `.elyx` or `.eaf`.

## Release checks

- Run `python -m zipfile -t <archive>` and `python -m zipfile -l <archive>`. Verify the root refmap, mapped paths, metadata, locales, assets and the absence of source in the intended scope.
- Use the build's CPython 3.11 to check that each `.pyc` has at least a 16-byte header and begins with `importlib.util.MAGIC_NUMBER`. Static checks need no imports from the package. Header and CRC checks only verify file properties.
- Install the final artifact on the target app. Test metadata, all module imports, resources, settings, re-enable and upgrade. If imports fail, check case, package paths, `__init__.pyc`, runtime version and `refmap.main`.
- Record the app, SDK, Python, builder and release SHA-256. Keep the source commit and build configuration for diagnosis.

[ElyxBuilder](https://plugins.exteragram.app/docs/elyx/elyxbuilder) · [Development and Build](https://plugins.exteragram.app/docs/elyx/development) · [Modules and Imports](https://plugins.exteragram.app/docs/elyx/modules-and-imports) · [CPython py_compile](https://docs.python.org/3.11/library/py_compile.html)
