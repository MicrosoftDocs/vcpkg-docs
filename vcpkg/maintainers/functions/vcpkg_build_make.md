---
title: vcpkg_build_make
description: Use vcpkg_build_make to build a Makefile project.
ms.date: 01/10/2024
---
# vcpkg_build_make

Build a Makefile project.

## Usage

```cmake
vcpkg_build_make([BUILD_TARGET <target>]
                 [INSTALL_TARGET <target>]
                 [ADD_BIN_TO_PATH]
                 [ENABLE_INSTALL]
                 [MAKEFILE <makefileName>]
                 [LOGFILE_ROOT <logfileroot>]
                 [DISABLE_PARALLEL]
                 [SUBPATH <path>])
```

### BUILD_TARGET

The target passed to the make build command (`./make <target>`). If not specified, the 'all' target will
be passed.

### INSTALL_TARGET

The target passed to the make build command (`./make <target>`) if `ENABLE_INSTALL` is used. Defaults to 'install'.

### ADD_BIN_TO_PATH

Adds the appropriate Release and Debug `bin\` directories to the path during the build such that executables can run against the in-tree DLLs.

### ENABLE_INSTALL

If the port supports the `install` target, use [`vcpkg_install_make()`](vcpkg_install_make.md) instead.

### MAKEFILE

Specifies the Makefile as a relative path from the root of the sources passed to [`vcpkg_configure_make()`](vcpkg_configure_make.md).

### LOGFILE_ROOT

Specifies a log file prefix.

### DISABLE_PARALLEL

The underlying buildsystem will be instructed to not parallelize.

### SUBPATH

Additional subdirectory to invoke `make` in. Useful if only parts of a port should be built.

## Notes

This command should be preceded by a call to [`vcpkg_configure_make()`](vcpkg_configure_make.md).

You can use the alias [`vcpkg_install_make()`](vcpkg_install_make.md) function if your makefile supports the "install" target.

## Source

[scripts/cmake/vcpkg\_build\_make.cmake](https://github.com/Microsoft/vcpkg/blob/master/scripts/cmake/vcpkg_build_make.cmake)
