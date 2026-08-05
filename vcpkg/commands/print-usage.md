---
title: vcpkg print-usage
description: Command line reference for the vcpkg print-usage command. Print usage information for an installed port.
author: BillyONeal
ms.author: bion
ms.date: 07/09/2026
---
# vcpkg print-usage

## Synopsis

```console
vcpkg print-usage [options] <package>
```

## Description

The `vcpkg print-usage` command prints usage information for one installed port.
The package must be installed for the selected triplet. See the
[`install` command documentation](install.md#package-syntax) for the accepted
`<package>` syntax.

Port authors can use this command to inspect custom usage information or preview
the usage information that vcpkg generates from the installed files.

## Examples

Print the usage information that users receive for an installed package:

```console
vcpkg print-usage zlib:x64-windows
```

Ignore any custom `usage` file and preview the generated usage information:

```console
vcpkg print-usage --generated zlib:x64-windows
```

Preview generated usage information without the heuristic warning:

```console
vcpkg print-usage --generated --affirm zlib:x64-windows
```

## Options

All vcpkg commands support a set of [common options](common-options.md).

### `--generated`

Print generated usage information even if the port installs a custom `usage`
file.

### `--affirm`

Affirm that the generated usage information is correct and omit the warning
that it might be inaccurate from the output.

This option only changes the command's output. To affirm generated usage
information in a port, install a
[`usage-accurate` marker file](../maintainers/handling-usage-files.md#affirming-generated-usage).
