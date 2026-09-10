# Velox

Velox is an open-source chess engine written in pure C23.

The project aims to combine:

- high-performance chess search;
- highly optimized bitboard-based move generation;
- clean and maintainable C code;
- UCI protocol support;
- multi-threaded search;
- automated strength testing with SPRT;
- parameter tuning;
- opening book generation;
- Syzygy tablebase probing;
- NNUE evaluation.

## Status

Velox is currently in early development.

The engine is not yet usable for chess play.

## Requirements

- CMake 4.0 or newer
- Ninja
- GCC or Clang with C23 support
- CTest
- clang-format
- clang-tidy

## Building

### Debug

```bash
cmake --preset debug
cmake --build --preset debug
```

### Debug optimized

```bash
cmake --preset debug-opt
cmake --build --preset debug-opt
```

### Release

```bash
cmake --preset release
cmake --build --preset release
```

## Testing
```bash
ctest --preset debug
```

or:

```bash
ctest --test-dir build-debug --output-on-failure
```

## Project structure

```
include/    Public headers
src/        Engine sources
tests/      Automated tests
tools/      Development tools
bench/      Performance benchmarks
```
