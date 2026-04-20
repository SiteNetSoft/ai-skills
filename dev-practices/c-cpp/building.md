# Building & Compilation

## Resource-Safe Compilation

Large C/C++ projects (especially Wine, Chromium, LLVM) can exhaust system resources during parallel builds. Always limit parallelism to prevent system freezes.

- **Never use bare `make`** without a `-j` flag on large projects — it may default to unlimited jobs
- **Never use `make -j$(nproc)`** — using all cores leaves nothing for the OS and desktop
- **Cap at half cores:**
  ```sh
  make -j$(( $(nproc) / 2 ))
  ```
- **Or use a fixed safe value:**
  ```sh
  make -j2   # safe on any machine
  make -j4   # reasonable for 8+ core systems
  ```
- **Lower scheduling priority for long builds:**
  ```sh
  nice -n 19 ionice -c3 make -j$(( $(nproc) / 2 ))
  ```
- **Limit memory per build with `ulimit`** if builds spawn memory-hungry linker jobs (e.g., LTO):
  ```sh
  # Limit virtual memory to 8 GB per process
  ulimit -v 8388608
  ```
- **CMake parallel builds:**
  ```sh
  cmake --build build -- -j$(( $(nproc) / 2 ))
  # Or set globally:
  export CMAKE_BUILD_PARALLEL_LEVEL=$(( $(nproc) / 2 ))
  ```
- **Ninja builds:**
  ```sh
  ninja -j$(( $(nproc) / 2 ))
  ```

## Compiler Warnings

Always compile with warnings enabled — catch bugs at compile time, not runtime:

```sh
# C
gcc -Wall -Wextra -Wpedantic -Werror -std=c17 -o output source.c

# C++
g++ -Wall -Wextra -Wpedantic -Werror -std=c++20 -o output source.cpp
```

Key warning flags beyond `-Wall -Wextra`:

| Flag | What it catches |
|------|-----------------|
| `-Wshadow` | Variable shadowing |
| `-Wconversion` | Implicit narrowing conversions |
| `-Wformat=2` | Printf/scanf format string issues |
| `-Wnull-dereference` | Potential null pointer dereferences |
| `-Wdouble-promotion` | Implicit float-to-double promotion |
| `-Wold-style-cast` | C-style casts in C++ code |

## Sanitizers (Development Builds)

Use sanitizers during development and testing — they catch bugs that warnings miss:

```sh
# Address sanitizer — buffer overflows, use-after-free, leaks
gcc -fsanitize=address -fno-omit-frame-pointer -g -o output source.c

# Undefined behavior sanitizer
gcc -fsanitize=undefined -g -o output source.c

# Thread sanitizer — data races (C++ multithreaded code)
g++ -fsanitize=thread -g -o output source.cpp

# Combine multiple sanitizers (ASan + UBSan)
gcc -fsanitize=address,undefined -fno-omit-frame-pointer -g -o output source.c
```

Do not ship with sanitizers enabled — they add significant runtime overhead.

## Build Systems

### Make

- Always provide a `-j` flag (see resource limits above)
- Use `$(MAKE)` for recursive make calls so `-j` propagates:
  ```make
  subsystem:
  	$(MAKE) -C subdir
  ```
- Use `.PHONY` for non-file targets:
  ```make
  .PHONY: all clean install test
  ```

### CMake

- Prefer out-of-source builds:
  ```sh
  cmake -S . -B build -DCMAKE_BUILD_TYPE=Release
  cmake --build build -- -j$(( $(nproc) / 2 ))
  ```
- Use `target_*` commands over global settings:
  ```cmake
  # Good
  target_compile_options(mylib PRIVATE -Wall -Wextra)
  target_include_directories(mylib PUBLIC include)

  # Avoid
  add_compile_options(-Wall -Wextra)
  include_directories(include)
  ```
- Set the C/C++ standard explicitly:
  ```cmake
  set(CMAKE_C_STANDARD 17)
  set(CMAKE_CXX_STANDARD 20)
  set(CMAKE_CXX_STANDARD_REQUIRED ON)
  ```

## Cross-Compilation

- Use a toolchain file for CMake cross-compilation:
  ```sh
  cmake -S . -B build -DCMAKE_TOOLCHAIN_FILE=toolchain-mingw.cmake
  ```
- For Wine builds (32-bit and 64-bit):
  ```sh
  # 64-bit
  ../configure --enable-win64 --prefix=/opt/wine
  nice -n 19 make -j$(( $(nproc) / 2 ))

  # 32-bit (after 64-bit is built)
  ../configure --with-wine64=../build64 --prefix=/opt/wine
  nice -n 19 make -j$(( $(nproc) / 2 ))
  ```
