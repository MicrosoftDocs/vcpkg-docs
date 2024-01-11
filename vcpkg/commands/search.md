---
title: vcpkg search
description: Command line reference for the vcpkg search command. Search for available packages by name and description.
ms.date: 01/10/2024
---

# vcpkg search

## Synopsis

```console
vcpkg search [options] [query]
```

## Description

Search for available packages by name and description.

Search performs a case-insensitive search through all available package names and descriptions. The results are displayed in a tabular format.

## Example

```console
$ vcpkg search zlib
miniz                    2.2.0#1          Single C source file zlib-replacement library
zlib                     1.2.12#1         A compression library
zlib-ng                  2.0.6            zlib replacement with optimizations for 'next generation' systems
```

## Options

All vcpkg commands support a set of [common options](common-options.md).

### `--x-full-desc`

[!INCLUDE [experimental](../../includes/experimental.md)]

Do not truncate long descriptions.

By default, long descriptions will be truncated to keep the tabular output browsable.
