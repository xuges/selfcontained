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
wget https://raw.githubusercontent.com/xuges/selfcontained/refs/heads/main/make-selfcontained
chmod +x make-selfcontained
./make-selfcontained a.out
```

After successful execution, a `selfcontained` directory will be created:

```
selfcontained/
├── bin
│   ├── ld-linux-x86-64.so.2
│   └── a.out
├── lib
│   ├── libc.so.6
│   ├── libgcc_s.so.1
│   ├── libm.so.6
│   └── libstdc++.so.6
└── run.sh
```

Explanation:
- `bin` stores a copy of the input `a.out` and dynamic link interpreter
- `lib` contains all dynamically linked dependencies (including `libc`)
- `run.sh` is the execution script for `a.out` (filename can be modified, but path must remain unchanged)

After generating the `selfcontained` directory, you may add other runtime resources. The directory name `selfcontained` can be modified.

**Note:**
- If using `dlopen` to dynamically load shared libraries, you must manually manage the shared library files.
- The working directory when launching the program with `run.sh` is the bin directory.

### Mechanism

The `make-selfcontained` script itself searches for the shared libraries required by the target executable, copies them along with the dynamic linker to the `selfcontained` directory, and modifies the dynamic linker path in the copied executable. The generated `run.sh` script sets `LD_LIBRARY_PATH` before execution, ensuring the executable prioritizes the bundled `ld-linux-x86-64.so.2` and all subsequent shared libraries, including libc, from its own directory.

Reference: <br/>
https://www.cnblogs.com/pengdonglin137/p/17623177.html
https://www.bytezonex.com/archives/Ivbb0Uz6.html

### Dependencies

The `make-selfcontained` script requires:

```bash
ldd
readelf
```