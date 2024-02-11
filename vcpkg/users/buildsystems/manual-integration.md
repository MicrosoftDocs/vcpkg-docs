---
title: Manual Integration
description: Integrate vcpkg into any buildsystem, such as meson or autoconf.
ms.date: 01/10/2024
ms.topic: concept-article
---
# Manual Integration

When installing libraries, vcpkg creates a single common layout partitioned by triplet.

The root of the tree in [Classic mode](../../concepts/classic-mode.md) is
`<vcpkg root>/installed`. The root of the tree in [Manifest
mode](../../concepts/manifest-mode.md) is `<vcpkg.json
directory>/vcpkg_installed`.

Underneath this root, in a subfolder named after the triplet:

- Header files: `include/`
- Release `.lib`, `.a`, and `.so` files: `lib/` or `lib/manual-link/`
- Release `.dll` files: `bin/`
- Release `.pc` files: `lib/pkgconfig/` or `share/pkgconfig/`
- Debug `.lib`, `.a`, and `.so` files: `debug/lib/` or `debug/lib/manual-link/`
- Debug `.dll` files: `debug/bin/`
- Debug `.pc` files: `debug/lib/pkgconfig/` or `debug/share/pkgconfig/`
- Tools: `tools/<port>/`

For example, `zlib.h` for `zlib:x64-windows` in classic mode is located at `<vcpkg root>/installed/x64-windows/include/zlib.h`.

See your build system specific documentation for how to use prebuilt binaries. For example, Makefile projects often accept environment variables:

```sh
export CXXFLAGS=-I$(pwd)/installed/x64-linux/include
export CFLAGS=-I$(pwd)/installed/x64-linux/include
export LDFLAGS=-L$(pwd)/installed/x64-linux/lib
export PKG_CONFIG_PATH=$(pwd)/installed/x64-linux/lib/pkgconfig:$(pwd)/installed/x64-linux/share/pkgconfig:$PKG_CONFIG_PATH
```

On Windows dynamic triplets (such as x64-windows) you will also need to either copy the needed DLL files to the same folder as your executable or *prepend* the correct `bin\` directory to your path to run any produced executables.
