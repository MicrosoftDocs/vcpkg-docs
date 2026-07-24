---
title: Provide usage documentation for your ports
description: Guidance for adding usage documentation to vcpkg ports
author: BillyONeal
ms.author: bion
ms.date: 07/09/2026
ms.topic: concept-article
---
# Provide usage documentation for your ports

## Overview

Providing accurate usage documentation for ports allows users to easily adopt
them in their projects. vcpkg automatically generates usage information by
inspecting the files installed by a port. Only provide a custom `usage` file
when the generated usage information is incorrect or incomplete.

After installing a port, run [`vcpkg print-usage`](../commands/print-usage.md)
to inspect the usage information:

```console
vcpkg print-usage <port>:<triplet>
```

If the port already installs a custom `usage` file, you can use `--generated` to
inspect what vcpkg generates without that file:

```console
vcpkg print-usage --generated <port>:<triplet>
```

### Affirming generated usage

When the generated usage information is correct, install an empty
`usage-accurate` marker file:

```cmake
file(TOUCH "${CURRENT_PACKAGES_DIR}/share/${PORT}/usage-accurate")
```

This marker affirms that the generated instructions are accurate, so vcpkg
omits the warning that the instructions are heuristically generated and might
be incorrect.

Before adding the marker, you can use `--affirm` to preview the generated output
without the warning:

```console
vcpkg print-usage --generated --affirm <port>:<triplet>
```

### Supplying a usage file

When generated usage information doesn't correctly describe how to consume the
package, create a text file named `usage` in the port directory and install it
to the port's `share` directory. The recommended method is to call the
`file(INSTALL ...)` function in `portfile.cmake`.

For example:

```cmake
file(INSTALL "${CMAKE_CURRENT_LIST_DIR}/usage" DESTINATION "${CURRENT_PACKAGES_DIR}/share/${PORT}")
```

After installing a port, vcpkg detects the file installed to
`${CURRENT_PACKAGES_DIR}/share/${PORT}/usage` and prints its instructions
instead of generating usage information.

### Content format

Provide clear instructions on how to use the package. The content should be concise, well-structured, and emphasize the minimum build system integration required to use the library.

Be clear and concise about how to utilize the package effectively. Avoid
overwhelming users with code snippets, command-line instructions, or
configuration details. Instead, use the [`"documentation"` property in the
port's `vcpkg.json` file](../concepts/manifest-mode.md) so users can learn more
about your library.

Use the following templates as a pattern for your `usage` files:

Packages with CMake targets:

```text
<port> provides CMake targets:

  <instructions>
```

Header-only libraries:

```text
<port> is header-only and can be used from CMake via:

  <instructions>
```

#### Example of `usage` file

```text
proj provides CMake targets:

  find_package(PROJ CONFIG REQUIRED)
  target_link_libraries(main PRIVATE PROJ::proj)
```
