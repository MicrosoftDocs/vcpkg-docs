---
title: Host dependencies
description: Describes host dependencies, tools packaged by vcpkg for use by ports or custom build systems.
author: vicroms
ms.author: viromer
ms.date: 10/30/2026
ms.topic: concept-article
---
# Host dependencies

Tools used at build time by other ports to generate code or implement a custom build system can be packaged inside vcpkg.

## Consuming

When consuming a port as a tool, you must set the dependency's `"host"` field to true. For example:

```json
{
    "name": "contoso-http-library",
    "version-string": "1.0.0",
    "description": "Contoso's http runtime library",
    "dependencies": [
        "contoso-core-library",
        {
            "name": "contoso-code-generator",
            "host": true
        },
        {
            "name": "contoso-build-system",
            "host": true
        }
    ]
}
```

In this case, the `contoso-code-generator` and `contoso-build-system` (including any transitive dependencies) will be built and installed for the host triplet before `contoso-http-library` is built.

> [!NOTE]
> Consumers must use `vcpkg.json` instead of `CONTROL` as their metadata format. You can easily convert an existing `CONTROL` file using `vcpkg format-manifest /path/to/CONTROL`.

Then, within the portfile of the consumer (`contoso-http-library` in the example), the CMake variable `CURRENT_HOST_INSTALLED_DIR` will be defined to `installed/<host-triplet>` and should be used to locate any required assets. In the example, `contoso-code-generator` might have installed `tools/contoso-code-generator/ccg.exe` which the consumer would add to its local path via:

```cmake
# ports/contoso-http-library/portfile.cmake
vcpkg_add_to_path(${CURRENT_HOST_INSTALLED_DIR}/tools/contoso-code-generator)
```

## Specifying the Host Triplet

The default host triplets are chosen based on the host architecture and operating system, for example `x64-windows`, `x64-linux`, or `x64-osx`. They can be overridden via:

1. In CMake-based manifest mode, calling `set(VCPKG_HOST_TRIPLET "<triplet>" CACHE STRING "")` before the first `project()` directive
1. In MSBuild-based manifest mode, setting the `VcpkgHostTriplet` property
1. On the command line, via the flag `--host-triplet=...`
1. The `VCPKG_DEFAULT_HOST_TRIPLET` environment variable

## Producing

Producing a tool has no special requirements; tools should be authored as a standard port, following all the normal policies and practices. Notably, they should build against `TARGET_TRIPLET`, not `HOST_TRIPLET` within the context of their portfile.

If the current context is cross-compiling (`TARGET_TRIPLET` is not `HOST_TRIPLET`), then `VCPKG_CROSSCOMPILING` will be set to a truthy value.

```cmake
if(VCPKG_CROSSCOMPILING)
    # This is a native build
else()
    # This is a cross build
endif()
```

## Host-only ports

Some ports should only be depended upon via a host dependency; script ports and
tool ports are common examples. In this case, you can use the `"native"`
supports expression to describe this. This supports expression is true when
`VCPKG_CROSSCOMPILING` is false (implying that `TARGET_TRIPLET ==
HOST_TRIPLET`).

## VCPKG_PREFER_HOST_TOOLS

This CMake option adds executables built by vcpkg to
[`CMAKE_PROGRAM_PATH`](https://cmake.org/cmake/help/latest/variable/CMAKE_PROGRAM_PATH.html)
and makes them available during your build process. Tools for the host
triplet are searched first, then tools for the target triplet. This finds
build tools that come with a target port and are not available as host tools,
such as `wxrc` from `wxwidgets`.

This option is on by default when `VCPKG_HOST_TRIPLET` is set. For it to take
effect, it and `VCPKG_HOST_TRIPLET` must be set before any call to `project()`
in your `CMakeLists.txt` file. [`VCPKG_USE_HOST_TOOLS`](#vcpkg_use_host_tools)
takes precedence over it.

Tools installed in the [`tools` folder](../reference/installation-tree-layout.md#layout-tools)
are added to `CMAKE_PROGRAM_PATH` in this order:

* `${VCPKG_INSTALLED_DIR}/${VCPKG_HOST_TRIPLET}/tools`
* `${VCPKG_INSTALLED_DIR}/${VCPKG_HOST_TRIPLET}/tools/*` or `tools/*/bin`
* `${VCPKG_INSTALLED_DIR}/${VCPKG_TARGET_TRIPLET}/tools`
* `${VCPKG_INSTALLED_DIR}/${VCPKG_TARGET_TRIPLET}/tools/*` or `tools/*/bin`

When the host and target triplets differ and
[`CMAKE_FIND_ROOT_PATH_MODE_PROGRAM`](https://cmake.org/cmake/help/latest/variable/CMAKE_FIND_ROOT_PATH_MODE_PROGRAM.html)
is not already set, it is set to `NEVER`. Otherwise `find_program()` would
search the target triplet's installed tree first and find the target copy of a
tool that is installed for both triplets.

> [!NOTE]
> Tools found for the target triplet, such as `wxrc`, are built for the target
> and may fail to run on the build machine when cross compiling.

## VCPKG_USE_HOST_TOOLS

Set this CMake option to add only executables built by vcpkg for the host
triplet to
[`CMAKE_PROGRAM_PATH`](https://cmake.org/cmake/help/latest/variable/CMAKE_PROGRAM_PATH.html)
and make them available during your build process. Tools for the target
triplet, such as `wxrc` from `wxwidgets`, are not found. This option is off by
default; when on, it takes precedence over
[`VCPKG_PREFER_HOST_TOOLS`](#vcpkg_prefer_host_tools).

For this option to take effect it must be set before any call to `project()` in
your `CMakeLists.txt` file. The `VCPKG_HOST_TRIPLET` variable must also be set.

Only tools installed in the [`tools` folder](../reference/installation-tree-layout.md#layout-tools)
for the host triplet are added to `CMAKE_PROGRAM_PATH`.
Effectively, the following locations are added:

* `${VCPKG_INSTALLED_DIR}/${VCPKG_HOST_TRIPLET}/tools`
* `${VCPKG_INSTALLED_DIR}/${VCPKG_HOST_TRIPLET}/tools/*` or `tools/*/bin`
