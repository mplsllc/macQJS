# macQJS: QuickJS for Classic Mac OS 9

**macQuick** is a natively ported version of the [QuickJS JavaScript Engine](https://bellard.org/quickjs/) for Classic Mac OS 9 and PowerPC, specifically designed to be compiled with Metrowerks CodeWarrior 8.

## Overview
QuickJS is a small and embeddable Javascript engine created by Fabrice Bellard. It natively supports the ES2020 specification including modules, asynchronous generators, proxies, BigInt, and Arrow Functions. 

Because QuickJS is written in modern C99 and relies heavily on GCC builtins and POSIX extensions, compiling it on legacy Mac OS 9 toolchains (which enforce strict C89 standards and lack many modern MathLib functions) is extremely difficult. 

This repository successfully accomplishes the complete backporting and compilation of the QuickJS engine for PowerPC (G3/G4) under CodeWarrior 8, proving that modern ES6 JavaScript can execute perfectly on classic Macintosh hardware.

## Features
- **Strict C89 Compliance:** Over 63,000 lines of engine code successfully refactored to comply with C89 block-scoping constraints.
- **Native Math Polyfills:** Seamless injection of missing C99 math functions (`cbrt`, `hypot`, `log2`, etc.) via a custom Prefix header.
- **Compiler Built-ins Replaced:** Standard C fallbacks for GCC-specific bit manipulations (`__builtin_clz`, `__builtin_ctz`, etc.).
- **Sanity Check Test Harness:** Includes a standalone `main.c` testing suite which validates Native Math integrations, Arrow Functions, Array mappings, JSON parsing, Object Destructuring, and Exception Handling directly through SIOUX output.

## Documentation
For complete technical details regarding how this engine was ported, known CodeWarrior IDE compiler bugs with large files, memory requirements, and library configurations, please read the included [PORTING_MAC_OS_9.md](PORTING_MAC_OS_9.md).

## Usage & Integration
1. Add `quickjs.c` and `quickjs.h` to your CodeWarrior 8 Project.
2. In your **Target Settings**, make sure the Prefix File is set to `#include "QuickJSPrefix.h"`.
3. Use the MSL Carbon Libraries (`CarbonLib`, `MSL C.Carbon.lib`, `MSL Runtime Carbon.lib`). *Do NOT use Classic PPC MathLib or MSL PPC.*
4. Ensure your memory partition (Heap & Stack) is appropriately large (e.g., >1MB stack, >16MB heap) as QuickJS has a deep recursive descent parser.

## License
QuickJS is released under the MIT license. The custom backported polyfills and modifications in this repository are also provided under the MIT license.
