# self-contained

[中文](doc/README_CN.md)

### Description

Dynamic linking is an unavoidable topic in native development. Even if you can ensure static linking for third-party libraries, the system's libc version can still be a major obstacle (musl libc can be used as an alternative approach).

- Running programs compiled with newer versions on older Linux distributions may report errors like `GLIBC_XXX not found` or `GLIBCXX_XXX not found`
- Software that runs normally during development may report `load shared object no such file` errors after deployment

These are common dynamic linking dependency issues.

Self-contained means the application bundles all required dependencies for execution. This project helps developers package their binary executables as self-contained bundles to resolve dynamic linking dependencies.

### Usage

Given a binary executable `a.out`, generate a self-contained directory using:

```bash
chmod +x make-selfcontained
./make-selfcontained a.out
```

After successful execution, a `selfcontained` directory will be created:

```
selfcontained/
├── bin
│   └── a.out
├── lib
│   ├── ld-linux-x86-64.so.2
│   ├── libc.so.6
│   ├── libgcc_s.so.1
│   ├── libm.so.6
│   └── libstdc++.so.6
└── run.sh
```

Explanation:
- `bin` stores a copy of the input `a.out`
- `lib` contains all dynamically linked dependencies (including `libc`)
- `run.sh` is the execution script for `a.out` (filename can be modified, but path must remain unchanged)

After generating the `selfcontained` directory, you may add other runtime resources. The directory name `selfcontained` can be modified.

**Note: If using `dlopen` to dynamically load shared libraries, you must manually manage the shared library files.**

### Mechanism

The `make-selfcontained` script itself detects dynamic library dependencies of the target executable, copies them to the `selfcontained` directory, and generates a `run.sh` script that launches the executable via direct invocation of the loader `ld-linux-x86-64.so.2`. This ensures all dynamic libraries (starting from `libc`) are preferentially loaded from the bundled set.

Reference: https://www.cnblogs.com/pengdonglin137/p/17623177.html

### Dependencies

The `make-selfcontained` script requires:

```bash
ldd
awk
```