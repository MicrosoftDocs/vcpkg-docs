---
title: vcpkg_cmake_build
description: Use vcpkg_cmake_build to build a CMake project with a custom install target.
ms.date: 01/10/2024
---
# vcpkg_cmake_build

Build a cmake project with a custom install target.

Conventionally, CMake uses the target `install` to build and copy binaries into the [`CMAKE_INSTALL_PREFIX`](https://cmake.org/cmake/help/latest/variable/CMAKE_INSTALL_PREFIX.html). In rare circumstances, a project might have more specific targets that should be used instead.

Ports should prefer calling [`vcpkg_cmake_install()`](vcpkg_cmake_install.md) when possible.

## Usage

```cmake
vcpkg_cmake_build(
    [TARGET <target>]
    [LOGFILE_BASE <base>]
    [DISABLE_PARALLEL]
    [ADD_BIN_TO_PATH]
)
```

To use this function, you must depend on the helper port `vcpkg-cmake`:

```json
"dependencies": [
  {
    "name": "vcpkg-cmake",
    "host": true
  }
]
```

## Parameters

All supported parameters to `vcpkg_cmake_install()` are supported by `vcpkg_cmake_build()`. For more information on its parameters, see [`vcpkg_cmake_install()`](vcpkg_cmake_install.md#parameters).

### TARGET

The CMake target to build.

If this parameter is not passed, no target will be passed to the build.

### LOGFILE_BASE

An alternate root name for the logs.

Defaults to `build-${TARGET_TRIPLET}`. It should not contain any path separators. Logs will be generated matching the pattern `${CURRENT_BUILDTREES_DIR}/${LOGFILE_BASE}-<suffix>.log`

## Examples

```cmake
vcpkg_from_github(OUT_SOURCE_PATH source_path ...)
vcpkg_cmake_configure(
    SOURCE_PATH "${source_path}"
    OPTIONS
        -DBUILD_EXAMPLES=OFF
        -DBUILD_TESTS=OFF
)
vcpkg_cmake_build(TARGET my.install.target)
```

[Search microsoft/vcpkg for Examples](https://github.com/microsoft/vcpkg/search?q=vcpkg_cmake_build+path%3A%2Fports)

## Remarks

This command replaces [`vcpkg_build_cmake()`](vcpkg_build_cmake.md).

## Source

[ports/vcpkg-cmake/vcpkg\_cmake\_build.cmake](https://github.com/Microsoft/vcpkg/blob/master/ports/vcpkg-cmake/vcpkg_cmake_build.cmake)
