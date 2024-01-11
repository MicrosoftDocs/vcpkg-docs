---
title: Build system integration
description: This article explains the different mechanisms vcpkg uses to integrate with build systems.
author: vicroms
ms.author: viromer
ms.date: 01/10/2024
ms.topic: concept-article
---
# Build system integration

vcpkg integrates seamlessly with MSBuild and CMake to restore your project's
dependencies at build time. vcpkg also makes it easy to integrate with other
build systems by providing a standard installation layout.

## MSBuild integration

vcpkg provides user-wide MSBuild integration via the
[`vcpkg integrate install`](../commands/integrate.md#vcpkg-integrate-install)
command and per-project integration via the `vcpkg.props` and `vcpkg.target`
files.

When you enable this integration, all MSBuild and Visual Studio projects will automatically link their dependencies.

Read more about how this integration works in the [MSBuild integration
documentation](../users/buildsystems/msbuild-integration.md).

## CMake integration

vcpkg provides seamless `find_package()` integration for your projects via a
custom toolchain file. You can enable this integration by adding the provided
toolchain file in `<vcpkg root>/scripts/buildsystems/vcpkg.cmake` in your CMake
configure calls via `CMAKE_TOOLCHAIN_FILE` or with a `CMakePresets.json` file.

Once this integration is activated, functions like `find_package()`, `find_library()`, and `find_path()` will automatically search through the vcpkg installation directories to find the required dependencies. Furthermore, vcpkg is also able to install your dependencies
declared in a manifest file automatically.

Read more about how this integration works in the [CMake integration
documentation](../users/buildsystems/cmake-integration.md).

## Manual integration

vcpkg makes it easy to integrate with other build systems by providing a standardized documented [installation layout](../reference/installation-tree-layout.md).

Read more about how to integrate vcpkg with your build system using [manual
integration](../users/buildsystems/manual-integration.md).
