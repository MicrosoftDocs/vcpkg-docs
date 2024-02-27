---
title: vcpkg_from_bitbucket
description: Learn how to use vcpkg_from_bitbucket.
ms.date: 01/10/2024
---
# vcpkg_from_bitbucket

Download and extract a project from Bitbucket.

## Usage

```cmake
vcpkg_from_bitbucket(
    OUT_SOURCE_PATH <SOURCE_PATH>
    REPO <blaze-lib/blaze>
    [REF <v3.8.1>]
    [SHA512 <45d0d7f8cc350...>]
    [HEAD_REF <master>]
    [PATCHES <patch1.patch> <patch2.patch>...]
)
```

## Parameters

### OUT_SOURCE_PATH

Specifies the out-variable that will contain the extracted location.

This should be set to `SOURCE_PATH` by convention.

### REPO

The organization or user and repository on Bitbucket.

### REF

A stable git commit-ish (ideally a tag) that will not change contents. **This should not be a branch.**

For repositories without official releases, this can be set to the full commit id of the current latest master. `vcpkg_from_bitbucket()` will download a stable snapshot of the commit without any history information at `https://bitbucket.com/<REPO>/get/<REF>.tar.gz`.

If `REF` is specified, `SHA512` must also be specified.

### SHA512

The SHA512 hash of the source archive.

This is most easily determined by first setting it to `0`, then trying to build the port. The error message will contain the full hash, which can be copied back into the portfile.

### HEAD_REF

The unstable git commit-ish (ideally a branch) to pull for `--head` builds.

For most projects, this should be `master`. The chosen branch should be one that is expected to be always buildable on all supported platforms.

### PATCHES

A list of patches to be applied to the extracted sources.

Relative paths are based on the port directory.

## Notes

At least one of `REF` and `HEAD_REF` must be specified, however it is preferable for both to be present.

This exports the `VCPKG_HEAD_VERSION` variable during head builds.

## Examples

- [blaze](https://github.com/Microsoft/vcpkg/blob/master/ports/blaze/portfile.cmake)

## Source

[scripts/cmake/vcpkg\_from\_bitbucket.cmake](https://github.com/Microsoft/vcpkg/blob/master/scripts/cmake/vcpkg_from_bitbucket.cmake)
