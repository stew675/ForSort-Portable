# ForSort Minimal - Agent Instructions

## Build Commands

```bash
# Build the project (uses clang by default)
make

# Clean build artifacts
make clean

# Build with gcc instead of clang
CC=gcc make
```

## Testing

The `ts` (TestSort) executable tests various sorting algorithms.

```bash
# Run a single test with specific algorithm and size
./ts <sorttype> <num_items>

# Common sort types:
#   fb - Basic ForSort In-Place (Stable)
#   fi - Adaptive ForSort In-Place (Unstable)
#   fs - Stable ForSort In-Place (Stable)
#   is - Insertion Sort (Stable)

# Examples:
./ts fs 1000000           # Test stable sort with 1M items
./ts fb 10000             # Test basic sort with 10K items
./ts fi 50000 -w 6250     # Test adaptive with workspace

# Test options:
./ts -a 5 -d 5 -r fs 1000000   # Seed=5, 5% disorder, reversed data
./ts -o -u fs 100000           # Ordered unique data
./ts -v fs 100                 # Verbose (show data before sorting)
./ts -x fs 10000               # Extended test (30s or 10+ runs)
```

## Code Style Guidelines

### File Organization
- `.h` files contain template code that is included multiple times with different `VAR` type definitions
- `.c` files contain the main logic and public API wrappers
- Header files use include guards and are organized by functionality

### Includes
- Standard C headers first (`<stddef.h>`, `<stdint.h>`, `<stdlib.h>`, etc.)
- Project headers last (`"forsort.h"`)
- Use double quotes for project headers, angle brackets for system headers

### Comment Style
- File headers: Block comments at top describing algorithm, author, copyright
- Function documentation: Block comments before function definitions
- Tuning parameters: Explain the purpose and experimental basis for constants
- Use tab indentation in comments, matching code style

### Naming Conventions
- Functions: `snake_case` (e.g., `forsort_basic`, `shift_merge_in_place`)
- Variables: `snake_case` (e.g., `is_lt`, `work_size`, `nitems`)
- Macros/Constants: `UPPER_SNAKE_CASE` (e.g., `BASIC_INSERT_MAX`, `WSRATIO`)
- Types: `snake_case_t` suffix for typedefs (e.g., `swap_type_t`)
- Static functions: No prefix, file-local scope implied

### Type System
- Use `<stdint.h>` fixed-width types (`uint32_t`, `uint64_t`, `size_t`)
- Prefer `const` for parameters that won't be modified
- Use `restrict` pointer qualifier for performance-critical code
- Define 128-bit types via `typedef unsigned __int128`

### Function Parameters
- Size parameters: `size_t n` (count), `size_t es` (element size)
- Comparison function: `int (*is_lt)(const void *, const void *)`
- Workspace: `void *workspace, size_t worksize`
- Use meaningful names: `is_lt` instead of `cmp` for less-than comparisons

### Error Handling
- Use `assert()` for internal consistency checks (can be disabled with `#if 0`)
- Return values indicate success/failure where appropriate
- Use standard errno codes when applicable

### Optimization Patterns
- Define `likely(x)` and `unlikely(x)` using `__builtin_expect`
- Use `__attribute__((noinline))` for functions that should not be inlined
- Compiler-specific optimizations gated with `#ifdef __clang__`
- Tuning constants at top of file with experimental justification

### Memory Management
- Use `malloc()`/`free()` for dynamic workspace allocation
- Document ownership and lifetime of allocated memory
- Prefer stack allocation where possible for performance

### Code Structure
- Put function definitions before their first use, or provide forward declarations
- Group related functions together (e.g., all insertion sort variants)
- Use `#pragma GCC diagnostic` to suppress warnings for intentionally unused code
- Separate public API functions at end of file

### Build Configuration
- Debug flags: `-Wall -g` (controlled via `DEBUG_FLAGS` in Makefile)
- Optimization: `-O3 -mtune=native` (controlled via `CC_OPT_FLAGS`)
- Conditional compilation via `#if 0` / `#if 1` blocks for debugging/features

### Testing Conventions
- Comparison functions should count comparisons for benchmarking
- Test data can be ordered, reversed, unique, or randomly disordered
- Verify both correctness and stability (for stable sorts)
- Use `-x` flag for extended testing