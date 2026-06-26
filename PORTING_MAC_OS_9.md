# QuickJS Mac OS 9 / CodeWarrior 8 Port

This repository contains a natively ported version of the [QuickJS](https://bellard.org/quickjs/) JavaScript Engine for Classic Mac OS 9 and PowerPC, specifically designed to be compiled with Metrowerks CodeWarrior 8.

## The Porting Journey & Major Changes

The original QuickJS engine is written in C99 and heavily relies on GCC/Clang builtins and POSIX extensions. CodeWarrior 8 for Mac OS 9 strictly enforces C89 standards and uses the Metrowerks Standard Library (MSL), which lacks many modern math functions. 

To achieve a clean, working compilation, the following structural changes were made to the codebase:

### 1. Strict C89 Compliance
CodeWarrior 8 immediately aborts on C99 mixed declarations.
- **Variable Hoisting:** Over 63,000 lines of code in `quickjs.c` were refactored. Every single nested block was scanned, and all inline variable declarations (e.g., `int i = 0;`) were hoisted to the top of their respective lexical scopes before any statements were executed.
- **Designated Initializers:** CodeWarrior does not support C99 designated struct initializers (e.g., `.field = value`). These were expanded into standard C89 assignments or ordered initializations.
- **`inline` Keyword:** The C99 `inline` keyword was stubbed out or replaced with compiler-specific macros where CodeWarrior failed to recognize it.

### 2. Missing C99 Math Polyfills
QuickJS relies heavily on modern math operations (`log2`, `cbrt`, `hypot`, `trunc`, `expm1`). MSL's `MathLib` for CodeWarrior only provides standard C89 math functions.
- A custom prefix file, `QuickJSPrefix.h`, was created.
- Simple, static C89 polyfills for these functions were injected via the prefix file to seamlessly bridge the gap without modifying the core `quickjs.c` calls.

### 3. Compiler Built-ins Replacement
QuickJS utilizes GCC builtins for fast bit-manipulation (`__builtin_clz`, `__builtin_clzll`, `__builtin_ctz`) which CodeWarrior PPC does not possess natively.
- Standard C fallback implementations were injected via the prefix file.
- Branch prediction hints (`__builtin_expect`) were stubbed out via macros (`#define __builtin_expect(x, expected_value) (x)`).

### 4. Memory Mapping & Line Endings Bug
CodeWarrior 8 contains a known bug: If a massive file uses UNIX line endings (`\n`), CodeWarrior maps the file into read-only memory for optimization, but then its internal text parser attempts to dynamically rewrite the `\n` to `\r` in-place. This triggers a hardware memory protection fault and crashes the entire IDE.
- All source files (`quickjs.c`, `main.c`, `QuickJSPrefix.h`, `cutils.h`) were permanently converted to native Mac OS `CR` (`\r`) line terminators.

### 5. `JS_IsError` Compiler Bug Fix
In the test harness, an implicit conversion bug was introduced by accidentally passing two arguments to `JS_IsError` instead of one. The C compiler attempted to cast the `JSContext *` pointer down to a `uint64_t` (QuickJS's NaN-boxed `JSValue` representation). This was corrected to properly use single-argument evaluation.

## Compiling for Mac OS 9

If you are using this repository to build a standalone QuickJS executable or embed it into a Carbon application, you must adhere to the following CodeWarrior project constraints:

### Library Selection (Crucial!)
You must **not** mix Classic Mac OS libraries with Carbon libraries. If your host application uses Carbon (or if you are building the standalone console test harness included here), your project window **must** contain exactly these 4 MSL libraries, and no others:
1. `CarbonLib`
2. `MSL C.Carbon.lib`
3. `MSL SIOUX.Carbon.lib` (if you want the console text window)
4. `MSL Runtime Carbon.lib`

*Do not include `MathLib` or `MSL_SIOUX_PPC.Lib`, as they will collide with CarbonLib and cause hundreds of `ignored descriptor` linker warnings and fatal undefined symbol errors for `InitGraf`.*

### Memory Requirements (Crucial!)
QuickJS utilizes a recursive descent parser for reading JavaScript code. The default CodeWarrior stack size of 64KB (`0x10000`) will immediately stack overflow.
- **Stack Size:** Minimum `1024` KB (1 MB) / `0x100000`. Set this in the **PPC PEF** or **PPC Target** settings panel.
- **Preferred Heap:** Minimum `16384` KB (16 MB) for standalone JS execution. If embedding within a browser, ~195 MB is recommended.
- **Minimum Heap:** Minimum `8192` KB (8 MB).

## The Included Test Harness (`main.c`)

This repository includes a standalone test harness (`main.c`) designed to validate the PowerPC port. It completely bypasses SIOUX's internal logging conflicts by routing a custom `print()` function directly into the JS global scope, and includes a "Sanity Check Suite" that actively validates:
- Native MathLib integration (`Math.sqrt`)
- C99 Polyfill integrity (`Math.cbrt`)
- ES6 Engine features (Arrow Functions, Array mappings, JSON Parsing, Exceptions, Object Destructuring)

Enjoy running modern ES6 JavaScript on a classic Power Mac!
