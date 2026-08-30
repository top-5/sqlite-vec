# MSVC Build for Windows

This fork adds Microsoft Visual C++ (MSVC) compatibility to sqlite-vec for native Windows builds.

## Changes Made

### 1. MSVC Alignment Syntax (sqlite-vec.c)

**Original (GCC-only):**
```c
#define PORTABLE_ALIGN32 __attribute__((aligned(32)))
```

**Fixed (GCC + MSVC):**
```c
#ifdef _MSC_VER
#define PORTABLE_ALIGN32 __declspec(align(32))
#define PORTABLE_ALIGN64 __declspec(align(64))
#else
#define PORTABLE_ALIGN32 __attribute__((aligned(32)))
#define PORTABLE_ALIGN64 __attribute__((aligned(64)))
#endif
```

### 2. Version Macro Expansion (sqlite-vec.h)

**Original (template with placeholders):**
```c
#define SQLITE_VEC_VERSION_MAJOR ${VERSION_MAJOR}
```

**Fixed (expanded values):**
```c
#define SQLITE_VEC_VERSION_MAJOR 0
#define SQLITE_VEC_VERSION_MINOR 1
#define SQLITE_VEC_VERSION_PATCH 7
```

## Build Instructions

### Prerequisites
- Visual Studio 2022 with C++ Desktop Development workload
- better-sqlite3 (provides sqlite3.h)

### Compile Command

```powershell
cl /LD /O2 /MT /DSQLITE_VEC_ENABLE_AVX /DNOMINMAX `
   /I"path\to\node_modules\better-sqlite3\deps\sqlite3" `
   sqlite-vec.c /Fe:dist\vec0.dll
```

### Flags Explained

- `/LD` - Build as DLL (loadable extension)
- `/O2` - Optimize for speed
- `/MT` - Static runtime (no external dependencies)
- `/DSQLITE_VEC_ENABLE_AVX` - Enable AVX SIMD (8x faster vectors)
- `/DNOMINMAX` - Prevent Windows min/max macro conflicts
- `/I<path>` - Include path for sqlite3.h
- `/Fe:` - Output filename

## Why MSVC?

1. **Native Windows toolchain** - Better integration with Windows SDK
2. **AVX optimization** - All modern x64 CPUs support AVX (since 2011)
3. **Easier debugging** - PDB symbols, Visual Studio integration
4. **No MinGW dependency** - Pure MSVC build

## Import Options

### Option 1: NPM Package (Default - MinGW)
```javascript
const sqliteVec = require('sqlite-vec');
sqliteVec.load(db);
```
Uses: `node_modules/sqlite-vec-windows-x64/vec0.dll` (MinGW-compiled)

### Option 2: Build from Submodule (MSVC)
```bash
git submodule add https://github.com/top-5/sqlite-vec external/sqlite-vec
npm run build:sqlite-vec
```
Produces: `external/sqlite-vec/dist/vec0.dll` (MSVC-compiled with AVX)

## Performance

Both MinGW and MSVC builds work correctly. MSVC build may have slight performance advantages:
- Native Windows calling convention
- MSVC-specific optimizations
- AVX-enabled vector operations

## Upstream Compatibility

This fork maintains compatibility with upstream asg017/sqlite-vec. The changes are minimal:
- Cross-platform alignment macros
- Pre-expanded version constants

These changes could be submitted as a PR to support MSVC builds officially.

## Testing

```powershell
# Verify DLL loads
node check-vector-mode.mjs

# Expected output:
# ✅ sqlite-vec extension LOADED successfully
# ✅ vec0 virtual table created successfully
# 🎉 PRODUCTION MODE: Using sqlite-vec indexed search
```

## Current Import

Memento uses the NPM package by default:
- **Source**: `npm install sqlite-vec` → `node_modules/sqlite-vec-windows-x64/vec0.dll`
- **Type**: MinGW-compiled, pre-built
- **Status**: ✅ Working in production

The submodule provides an alternative MSVC build option.
