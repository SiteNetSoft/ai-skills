# C/C++ Best Practices

Guidelines for writing safe, performant, and portable C and C++ code.

## Sub-Files

| File | When to read |
|------|-------------|
| [building.md](building.md) | Compilation, make, CMake, resource limits, cross-compilation |

## Key Principles

1. **Compile defensively** — enable warnings (`-Wall -Wextra -Werror`) and use sanitizers during development
2. **Limit build parallelism** — never run unbounded `make`; cap jobs to avoid freezing the system
3. **Prefer RAII in C++** — manage resources through constructors/destructors, not manual alloc/free
4. **Minimize undefined behavior** — use sanitizers (ASan, UBSan, TSan) to catch UB early
5. **Use static analysis** — `clang-tidy`, `cppcheck`, and compiler warnings are your first line of defense
6. **Isolate platform-specific code** — use `#ifdef` sparingly; prefer abstraction layers
7. **Validate at boundaries** — trust internal invariants, validate external input

## Sources

- [C++ Core Guidelines](https://isocpp.github.io/CppCoreGuidelines/CppCoreGuidelines)
- [SEI CERT C Coding Standard](https://wiki.sei.cmu.edu/confluence/display/c/SEI+CERT+C+Coding+Standard)
- [SEI CERT C++ Coding Standard](https://wiki.sei.cmu.edu/confluence/display/cplusplus)
- [GCC Warning Options](https://gcc.gnu.org/onlinedocs/gcc/Warning-Options.html)
- [CMake Best Practices](https://cmake.org/cmake/help/latest/guide/tutorial/)
- [Google C++ Style Guide](https://google.github.io/styleguide/cppguide.html)
