---
title: vcpkg_cmake_config_fixup
description: Use vcpkg_cmake_config_fixup to support multiconfig generators.
ms.date: 01/10/2024
---
# vcpkg_cmake_config_fixup

Merge and correct Release and Debug CMake targets and configs to support multiconfig generators.

## Usage

```cmake
vcpkg_cmake_config_fixup(
    [PACKAGE_NAME <name>]
    [CONFIG_PATH <config-directory>]
    [TOOLS_PATH <tools/${PORT}>]
    [DO_NOT_DELETE_PARENT_CONFIG_PATH]
    [NO_PREFIX_CORRECTION]
)
```

To use this function, you must depend on the helper port `vcpkg-cmake-config`:

```json
"dependencies": [
  {
    "name": "vcpkg-cmake-config",
    "host": true
  }
]
```

Additionally corrects common issues with targets, such as absolute paths and incorrectly placed binaries.

For many ports, `vcpkg_cmake_config_fixup()` on its own should work,
as `PACKAGE_NAME` defaults to `${PORT}` and `CONFIG_PATH` defaults to `share/${PACKAGE_NAME}`.
For ports where the package name passed to `find_package` is distinct from the port name,
`PACKAGE_NAME` should be changed to be that name instead.
For ports where the directory of the `*config.cmake` files cannot be set,
use the `CONFIG_PATH` to change the directory where the files come from.

By default the parent directory of CONFIG_PATH is removed if it is named "cmake".
Passing the `DO_NOT_DELETE_PARENT_CONFIG_PATH` option disable such behavior,
as it is convenient for ports that install
more than one CMake package configuration file.

The `NO_PREFIX_CORRECTION` option disables the correction of `_IMPORT_PREFIX`
done by vcpkg due to moving the config files.
Currently the correction does not take into account how the files are moved,
and applies a rather simply correction which in some cases will yield the wrong results.

## How it works

1. Moves `/debug/<CONFIG_PATH>/*targets-debug.cmake` to `/share/${PACKAGE_NAME}`.
1. Transforms all references matching `/bin/*.exe` to `/${TOOLS_PATH}/*.exe` on Windows.
1. Transforms all references matching `/bin/*` to `/${TOOLS_PATH}/*` on other platforms.
1. Fixes `${_IMPORT_PREFIX}` in auto generated targets.
1. Replaces `${CURRENT_INSTALLED_DIR}` with `${_IMPORT_PREFIX}` in configs.
1. Merges INTERFACE_LINK_LIBRARIES of release and debug configurations.
1. Replaces `${CURRENT_INSTALLED_DIR}` with `${VCPKG_IMPORT_PREFIX}` in targets.
1. Removes `/debug/<CONFIG_PATH>/*config.cmake`.

## Examples

- [concurrentqueue](https://github.com/Microsoft/vcpkg/blob/master/ports/concurrentqueue/portfile.cmake)
- [curl](https://github.com/Microsoft/vcpkg/blob/master/ports/curl/portfile.cmake)
- [nlohmann-json](https://github.com/Microsoft/vcpkg/blob/master/ports/nlohmann-json/portfile.cmake)

## Source

[ports/vcpkg-cmake-config/vcpkg\_cmake\_config\_fixup.cmake](https://github.com/Microsoft/vcpkg/blob/master/ports/vcpkg-cmake-config/vcpkg_cmake_config_fixup.cmake)
