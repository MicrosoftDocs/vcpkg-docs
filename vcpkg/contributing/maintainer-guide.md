---
title: vcpkg Maintainer Guide
description: This document describes policies, guidelines, and best practices to follow when making contributions to vcpkg.
author: BillyONeal
ms.author: bion
ms.date: 9/30/2026
ms.topic: concept-article
ai-usage: ai-assisted
---
# Maintainer guide

This document describes policies, guidelines, and best practices to follow when making contributions to vcpkg.

It is intended to serve the same role as:

- [Debian's Policy Manual](https://www.debian.org/doc/debian-policy/),
- [Homebrew's Maintainer Guidelines](https://docs.brew.sh/Maintainer-Guidelines), and
- [Homebrew's Formula Cookbook](https://docs.brew.sh/Formula-Cookbook).

## Curated registry design goals

### Ports must install simultaneously

A port included in the curated registry must build successfully in the presence of all the other ports in the registry.
The entire combination of ports is tested regularly by vcpkg's automated CI.

A port must not do any of the following:

- [Install files that conflict with another port.](#unique-port-attribution-rule)
- Install symbols and definitions owned by another package, except for narrowly scoped, explicitly linked libraries
  described in [Place conflicting libs in a `manual-link` directory](#place-conflicting-libs-in-a-manual-link-directory).
- Test for the presence or absence of another port. Except for declaring a dependency.

Exceptions are made for:

- Ports that replace another port in an official capacity. This includes cases where:
  - The same project publishes a successor component.
  - Another project takes over maintenance of an abandoned component, including through a fork.
  - Another project publishes a successor component.
- Ports where there is no platform overlap. For example, a Windows-only replacement to a Linux component that doesn't
  support Windows.
- Ports that precede this policy and have a demonstrably significant number of users and contributors.

vcpkg maintainers assess grandfathering exceptions case by case. For example, `boringssl` was added before custom
registries were available and remains in the curated registry because it still has meaningful users, even though
`openssl` is the canonical provider for the OpenSSL fork family. Prior acceptance alone doesn't guarantee an exception.

Ports that don't conform to this policy can't be accepted into the curated registry.

[!INCLUDE [registry-or-overlay](../../includes/registry-or-overlay.md)]

### Ports must be tested in at least one official triplet

All ports in the curated registry must be tested in CI for at least one official [triplet](../concepts/triplets.md).

Exceptions are made for:

- Temporary circumstances, such as:
  - The port doesn't build without an update to the CI infrastructure.
  - The port doesn't build because of a recent breaking change in an upstream dependency and there is work in progress
    to resolve the situation.
- Ports that precede this policy and have a demonstrably significant number of users and contributors.

New ports that can't be tested in at least one official triplet aren't accepted into the curated registry.
Existing ports that no longer conform to this policy and don't qualify for an exception listed above are delisted from
the registry. For example, if CI testing has been disabled on every official triplet a port supported, and no exception
applies, the port is delisted.

[!INCLUDE [registry-or-overlay](../../includes/registry-or-overlay.md)]

### <a name="packaged-projects-should-be-stable-and-actively-maintained"></a> Packaged components should be stable and actively maintained

Components packaged in the curated registry must be in active maintenance. Ports for inactive components may be delisted.

A component is considered inactive if:

- Its maintainers have declared the component abandoned.
- Its source repository is archived, or no longer accepting contributions.
- Maintainers are unresponsive or unreachable.
- No meaningful changes have been made in a long time.

Exceptions are made for foundational components that are considered mature and stable and don't receive changes often.
For example: `zlib`.

### <a name="packaged-projects-should-be-mature"></a> Packaged components should be mature

Components packaged in the curated registry must be mature and intended for consumption by users of vcpkg. Components intended
for personal use should be [published to custom registries](../produce/publish-to-a-git-registry.md).

A component is considered mature enough for the curated registry, if one of these statements is true:

- The component has a release that is at least six months old.
- The component demonstrates at least six months of active public development.
- The component is an official part of a project with at least six months of releases or active public development.
  For example, a new Boost library or Qt component.
- The component demonstrates equivalent maturity to the previous requirements in some other capacity.

Some indicators of component immaturity are:

- The component doesn't show up in search engines.
- The component has frequent renames.
- Conflicts with other libraries.

[!INCLUDE [registry-or-overlay](../../includes/registry-or-overlay.md)]

## PR structure

### Make separate pull requests per port

Smaller pull requests (PRs) are easier to review. Pull requests should make changes to a single port. This also reduces
the wait times for Continuous Integration (CI) results.

Limiting PRs to a single port may not be possible in some cases. For example, when changing a port requires downstream
consumers to be updated or patched.

### <a name="avoid-trivial-changes-in-untouched-files"></a> Avoid unrelated cosmetic changes

Don't make unrelated cosmetic changes to an otherwise unmodified port, such as reformatting, renaming variables, or
fixing typos. These changes consume compute time better used elsewhere. Changes that improve build correctness,
compatibility, reliability, or reproducibility aren't considered trivial solely because the installed files remain
unchanged.

<!-- The <a> tag is required to preserve old external links that use the previous header text-->
### <a name="check-names-against-other-repositories"></a> Use distinctive port names

A port's name should be indicative of its contents.

Searching the port's name in a search engine or specialized package browsers, like [Repology](<https://repology.org>),
should lead to its corresponding component.

Ports with short names or named after common words require disambiguation. This applies only to the name of the port in
the curated registry, the packaged component's name and its contents are not required to conform with this policy.

Exceptions are made for ports that package a component with a strong association to its port's name. For example:  `libpng`,
`openssl`, or `zlib`.

To conform with this policy, new ports with ambiguous names can use a prefix, such as:

- The repository's owner, username, or organization. Example: `google-cloud-cpp`.
  For ports of components hosted on GitHub, the GitHub owner is an acceptable unambiguous prefix `<github owner>-<repository name>`.
- The name of a suite to which the package belongs to: `boost-dll`.
  Only if the package is an official component of such a suite.

#### Example: Ambiguous port name

A port with the name `ip` is considered ambiguous because:

- the name is too short,
- the name is a common word, and
- the name isn't strongly associated to any singular component.

To determine if a name is ambiguous, remove the following common prefixes and suffixes used by C++ and open source components
from the port name:

- `cpp`
- `free`
- `lib`
- `open`
- numerals

For example: `ip-cpp`, `libip` and `ip5`, are ambiguous because they are reduced to the same `ip` stem.

### Limit port renames

To avoid confusion for users, a port must not be renamed more than once in any 12-month period.

### Use GitHub draft PRs

GitHub Draft PRs are a great way to get CI or human feedback on work that isn't yet ready to merge.
Most new PRs should be opened as drafts and converted to normal PRs once the CI passes.

For more information about GitHub Draft PRs, see [Introducing draft pull requests](https://github.blog/2019-02-14-introducing-draft-pull-requests/).

A vcpkg maintainer may convert your PR to draft during the review process, usually when requesting changes or leaving
comments that indicate when to mark the PR as ready for review.

#### Close inactive PRs

A vcpkg maintainer may close a PR after it has been waiting for action from the contributor for more than
60 days. The inactivity countdown runs only while contributor action is required. It doesn't run while the PR is waiting
for build lab verification, or review by a vcpkg maintainer.

A PR is in a reviewable state when it:

- Has no failing PR checks, or the contributor asks for help or justifies the reason for the failures.
- Doesn't have any pending requested changes or clarifications from a vcpkg maintainer.
- Doesn't have merge conflicts.

The countdown begins when the PR enters a state that requires contributor action. This can happen when a PR is opened
with problems the contributor needs to address, or when a vcpkg maintainer requests changes or clarification. A vcpkg
maintainer may mark the PR as a draft when requesting contributor action.

A substantive response from the contributor resets the countdown. If the response resolves all pending requests and the
PR is ready for review, the countdown stops until contributor action is required again. If the contributor doesn't
respond within 60 days while action is required, the PR is considered stale and may be closed at a vcpkg maintainer's
discretion.

## Portfiles

### Avoid deprecated helper functions

At this time, the following helpers are deprecated:

- [`vcpkg_extract_source_archive_ex()`](../maintainers/functions/vcpkg_extract_source_archive_ex.md) should be replaced by the supported overload of [`vcpkg_extract_source_archive()`](../maintainers/functions/vcpkg_extract_source_archive.md) (with `ARCHIVE`)
- The deprecated overload of `vcpkg_extract_source_archive()` without `ARCHIVE` should be replaced by the supported overload with `ARCHIVE`.
- `vcpkg_apply_patches()` should be replaced by the `PATCHES` arguments to the "extract" helpers (e.g. [`vcpkg_from_github()`](../maintainers/functions/vcpkg_from_github.md))
- `vcpkg_build_msbuild()` should be replaced by [`vcpkg_install_msbuild()`](../maintainers/functions/vcpkg_install_msbuild.md)
- `vcpkg_copy_tool_dependencies()` should be replaced by [`vcpkg_copy_tools()`](../maintainers/functions/vcpkg_copy_tools.md)
- `vcpkg_configure_cmake` should be replaced by [`vcpkg_cmake_configure()`](../maintainers/functions/vcpkg_cmake_configure.md) after removing `PREFER_NINJA`
- `vcpkg_build_cmake` should be replaced by [`vcpkg_cmake_build()`](../maintainers/functions/vcpkg_cmake_build.md)
- `vcpkg_install_cmake` should be replaced by [`vcpkg_cmake_install()`](../maintainers/functions/vcpkg_cmake_install.md)
- `vcpkg_fixup_cmake_targets` should be replaced by [`vcpkg_cmake_config_fixup`](../maintainers/functions/vcpkg_cmake_config_fixup.md)

Some of the replacement helper functions are in "tools ports" to allow consumers to pin their
behavior at specific versions, to allow locking the behavior of the helpers at a particular
version. Tools ports need to be added to your port's `"dependencies"`, like so:

```json
{
  "name": "vcpkg-cmake",
  "host": true
},
{
  "name": "vcpkg-cmake-config",
  "host": true
}
```

### Avoid excessive comments in portfiles

Ideally, portfiles should be short, simple, and as declarative as possible.
Remove any boiler plate comments introduced by the `create` command before submitting a PR.

### Use lowercase for hexadecimal digits strings

Many of the features in vcpkg rely on comparing strings of hexadecimal digits. Some examples include, but are not limited to, SHA512 hashes, Git commit IDs, and tree object hashes.


Internally, vcpkg uses lowercase normalization for comparisons of such values where the casing is irrelevant. However, tooling
built on top of vcpkg's infrastructure may not make the same considerations. For this reason, we require hexadecimal strings

to be lowercased for consistency in the following scenarios:

* The `SHA512` parameter in vcpkg helper functions.
* The `REF` parameter in vcpkg helper functions, when the value is a hexadecimal string.
* The `git-tree` object in [version database files](../concepts/registries.md#versions-files).
* The `sha512` object in the `scripts/vcpkg-tools.json` file.
* Other places where casing of the hexadecimal string is unimportant.

### Ports must not be path dependent

Ports must not change their behavior based on which ports are already installed in a form that would change which contents that port installs. For example, given:

```console
> vcpkg install a
> vcpkg install b
> vcpkg remove a
```

and

```console
> vcpkg install b
```

the files installed by `b` must be the same, regardless of influence by the previous installation of `a`. This means that ports must not try to detect whether something is provided in the installed tree by another port before taking some action. A specific and common cause of such "path dependent" behavior is described below in "When defining features, explicitly control dependencies."

### Unique port attribution rule

In the entire vcpkg system, no two ports a user is expected to use concurrently may provide the same file. If a port tries to install a file already provided by another port, installation will fail. If a port wants to use an extremely common name for a header, for example, it should place those headers in a subdirectory rather than in `include`.

This property is checked regularly by continuous integration runs which try to install all ports in the registry, which will fail with `FILE_CONFLICTS` if two ports provide the same file.

### Add CMake exports in an unofficial- namespace

A core design ideal of vcpkg is to not create "lock-in" for users. In the build system, there should be no difference between depending on a library from the system, and depending on a library from vcpkg. To that end, we avoid adding CMake exports or targets to existing libraries with "the obvious name", to allow upstreams to add their own official CMake exports without conflicting with vcpkg.

To that end, any CMake configs that the port exports, which are not in the upstream library, should have `unofficial-` as a prefix. Any additional targets should be in the `unofficial::<port>::` namespace.

This means that the user should see:

- `find_package(unofficial-<port> CONFIG)` as the way to get at the unique-to-vcpkg package
- `unofficial::<port>::<target>` as an exported target from that port.

Examples:

- [`brotli`](https://github.com/microsoft/vcpkg/blob/4f0a640e4c5b74166b759a862d7527c930eff32e/ports/brotli/install.patch) creates the `unofficial-brotli` package, producing target `unofficial::brotli::brotli`.

### Install copyright file

Each port has to provide a file named `copyright` in the folder `${CURRENT_PACKAGES_DIR}/share/${PORT}`. If a package's license content is available within its source files, this file should be created by a call to [`vcpkg_install_copyright()`](../maintainers/functions/vcpkg_install_copyright.md). `vcpkg_install_copyright` also bundles multiple copyright files if necessary.

```cmake
vcpkg_install_copyright(FILE_LIST "${SOURCE_PATH}/LICENSE")
```

An older method to manually create this file is with CMake's built in `file` command. This is discouraged in favor of `vcpkg_install_copyright` in new ports but is still allowed.

```cmake
file(INSTALL "${SOURCE_PATH}/LICENSE" DESTINATION "${CURRENT_PACKAGES_DIR}/share/${PORT}" RENAME copyright)
```

If the license content in the upstream source files is not in text form (e.g. a PDF file), `copyright` should contain an explanation as to how a user can find the license requirements. If possible, it should also include a link to the original source files indicating this, so users can check if it is up to date.

```cmake
file(WRITE "${CURRENT_PACKAGES_DIR}/share/${PORT}/copyright" [[As of 2023-07-25, according to
https://github.com/GPUOpen-LibrariesAndSDKs/display-library/blob/master/Public-Documents/README.md#end-user-license-agreement
this software is bound by the "SOFTWARE DEVELOPMENT KIT LICENSE AGREEMENT" PDF located at
https://github.com/GPUOpen-LibrariesAndSDKs/display-library/blob/master/Public-Documents/ADL%20SDK%20EULA.pdf
]])
```

### Version constraints in ports

Version constraints within ports should generally be avoided, as they can hinder the independent evolution of components. Adding such constraints is only permissible when there is a well-documented justification, such as proven incompatibility with specific earlier versions. These constraints should not be used merely to maintain parity with independent components.

### Variables in `MAYBE_UNUSED_VARIABLES` must apply to at least one configuration

When adding a new variable to [`MAYBE_UNUSED_VARIABLES`](../maintainers/functions/vcpkg_cmake_configure.md#maybe_unused_variables)
to silence a warning during the CMake configuration step, you must add a comment explaining the case when the new variable
applies. If a variable does not apply in any configuration, then it is very likely that an underlying bug exists (for example,
a misspelled variable name) and adding it has no actual effect on the build.

```cmake
vcpkg_check_features(OUT_FEATURE_OPTIONS FEATURE_OPTIONS
  FEATURES
    windowsfeature WINDOWS_OPTION
)

vcpkg_cmake_configure(
  SOURCE_PATH "${SOURCE_PATH}"
  OPTIONS
    ${FEATURE_OPTIONS}
  MAYBE_UNUSED_VARIABLES
    WINDOWS_OPTION # Applies only on Windows
)
```

## Features

### Do not use features to implement alternatives

Features must be treated as additive functionality. If `port[featureA]` installs and `port[featureB]` installs, then `port[featureA,featureB]` must install. Moreover, if a second port depends on `[featureA]` and a third port depends on `[featureB]`, installing both the second and third ports should have their dependencies satisfied.

Libraries in this situation must choose one of the available options as expressed in vcpkg, and users who want a different setting must use overlay ports at this time.

Existing examples we would not accept today retained for backwards compatibility:

- `libgit2`, `libzip`, `open62541` all have features for selecting a TLS or crypto backend. `curl` has different crypto backend options but allows selecting between them at runtime, meaning the above tenet is maintained.
- `darknet` has `opencv2`, `opencv3`, features to control which version of opencv to use for its dependencies.

### A feature may engage preview or beta functionality

Notwithstanding the above, if there is a preview branch or similar where the preview functionality has a high probability of not disrupting the non-preview functionality (for example, no API removals), a feature is acceptable to model this setting.

Examples:

- The Azure SDKs (of the form `azure-Xxx`) have a `public-preview` feature.
- `imgui` has an `experimental-docking` feature which engages their preview docking branch which uses a merge commit attached to each of their public numbered releases.

### <a name="default-features-should-enable-behaviors-not-apis"></a> Default features must not add APIs

> [!NOTE]
> A feature enabled by default by the upstream build system, does not imply that
> the feature should be added to the [`default-features`](../reference/vcpkg-json.md#dependency-default-features) 
> entries. As the intended  purpose of `default-features` is not to model the 
> decisions made by upstream but to provide convenience for [classic mode](../concepts/classic-mode.md) 
> users.

Default features are intended to ensure that a reasonably functional build of a library gets installed for customers who don't know they are using it. If they don't know they are using a library, they can't know to list features. For example, `libarchive` exposes features that enable compression algorithms to an existing generic interface; if built without any of such features, the library may have no utility.

Here, customers include both user projects that consume the package graph and downstream ports within that graph.
Both express their dependency requirements through the same feature interface.

Carefully consider whether a feature should be on by default, because disabling default features is complex.

To disable default features for a transitive dependency:

* Every customer that depends on the port must disable its default features in its dependency declaration using
  [`"default-features": false`](../reference/vcpkg-json.md#dependency-default-features).
* The top-level project must also name the transitive dependency directly, either in its manifest with
  `"default-features": false`, or on the `vcpkg install` command line with `[core]`.

Disabling default features doesn't prevent another customer from explicitly requesting any of those features.

For this policy, an API is any interface exposed to consumers, including library interfaces, executable command-line interfaces, and other binary interfaces. A feature may be enabled by default when it adds behavior behind an existing API, such as support for additional compression algorithms through an existing generic interface.

In vcpkg's curated registry, if a feature adds an API, it must be off by default. If in doubt, do not mark a feature as default.
 

### Do not use features to control alternatives in published interfaces

If a consumer of a port depends on only the core functionality of that port, with high probability they must not be broken by turning on the feature. This is even more important when the alternative is not directly controlled by the consumer, but by compiler settings like `/std:c++17` / `-std=c++17`.

Existing examples we would not accept today retained for backwards compatibility:

- `redis-plus-plus[cxx17]` controls a polyfill but does not bake the setting into the installed tree.
- `ace[wchar]` changes all APIs to accept `const wchar_t*` rather than `const char*`.

### A feature may replace polyfills with aliases provided that replacement is baked into the installed tree

Notwithstanding the above, ports may remove polyfills with a feature, as long as:

1. Turning on the feature changes the polyfills to aliases of the polyfilled entity
1. The state of the polyfill is baked into the installed headers, such that ABI mismatch "impossible" runtime errors are unlikely
1. It is possible for a consumer of the port to write code which works in both modes, for example by using a typedef which is either polyfilled or not

Example:

- `abseil[cxx17]` changes `absl::string_view` to a replacement or `std::string_view`; the [patch](https://github.com/microsoft/vcpkg/blob/981e65ce0ac1f6c86e5a5ded7824db8780173c76/ports/abseil/fix-cxx-standard.patch) implements the baking requirement.

### Recommended solutions

If it's critical to expose the underlying alternatives, we recommend providing messages at build time to instruct the user on how to copy the port into a private overlay:

```cmake
set(USING_DOG 0)
message(STATUS "This version of LibContoso uses the Kittens backend. To use the Dog backend instead, create an overlay port of this with USING_DOG set to 1 and the `kittens` dependency replaced with `dog`.")
message(STATUS "This recipe is at ${CMAKE_CURRENT_LIST_DIR}")
message(STATUS "See the overlay ports documentation at https://github.com/microsoft/vcpkg/blob/master/docs/specifications/ports-overlay.md")
```

## Build techniques

### Do not use vendored dependencies

Prefer to package dependencies separately so users can select and update them through vcpkg. Vendored dependencies
introduce several challenges:

- **Difficulty updating:** Vendored dependencies are harder to track and update, including when applying security fixes.
- **Symbol conflicts:** A vendored dependency can define the same symbols as another version of the dependency, causing
  runtime errors or undefined behavior.
- **Licensing compliance:** Vendored dependencies can obscure licensing requirements or create license compatibility
  issues.
- **Increased maintenance:** Keeping vendored dependencies synchronized with their upstream versions increases maintenance
  work and can duplicate work across ports.

For these reasons, devendor dependencies whenever practical. Vendored dependencies must not be exposed to a user's link
domain.

A link domain is the set of symbols that can resolve against one another at runtime. On Windows, each DLL generally has
its own link domain. On platforms with a process-wide symbol table, the link domain generally includes the entire process.

If a component exposes a vendored dependency to the same link domain as a dependency selected through vcpkg, both copies
can define the same symbols. This violates the One Definition Rule and can cause runtime errors or undefined behavior.
The issue might appear only when the two versions differ enough to be incompatible.

A vendored dependency is allowed only when it is fully isolated from link domains used by downstream ports and user
applications. For example, a host-only build tool can contain a statically linked dependency if downstream binaries don't
link against the tool and the dependency's symbols can't enter their link domains.

### Prefer using CMake

When multiple buildsystems are available, prefer using CMake.
Additionally, when appropriate, it can be easier and more maintainable to rewrite alternative buildsystems into CMake using `file(GLOB)` directives.

Examples: [abseil](https://github.com/Microsoft/vcpkg/tree/master/ports/abseil/portfile.cmake)

### <a name="only-static-or-shared"></a>Choose either static or shared binaries

When building CMake libraries, [`vcpkg_cmake_configure()`](../maintainers/functions/vcpkg_cmake_configure.md) will pass in the correct value for `BUILD_SHARED_LIBS` based on the user's requested variant.

You can calculate alternative configure parameters by using `string(COMPARE EQUAL "${VCPKG_LIBRARY_LINKAGE}" ...)`.

```cmake
# portfile.cmake

string(COMPARE EQUAL "${VCPKG_LIBRARY_LINKAGE}" "static" KEYSTONE_BUILD_STATIC)
string(COMPARE EQUAL "${VCPKG_LIBRARY_LINKAGE}" "dynamic" KEYSTONE_BUILD_SHARED)

vcpkg_cmake_configure(
    SOURCE_PATH ${SOURCE_PATH}
    OPTIONS
        -DKEYSTONE_BUILD_STATIC=${KEYSTONE_BUILD_STATIC}
        -DKEYSTONE_BUILD_SHARED=${KEYSTONE_BUILD_SHARED}
)
```

If a library does not offer configure options to select the build variant, the build must be patched. When patching a build, you should always attempt to maximize the future maintainability of the port. Typically this means minimizing the number of lines that need to be touched to fix the issue at hand.

#### Example: Patching a CMake library to avoid building unwanted variants

For example, when patching a CMake-based library, it may be sufficient to add [`EXCLUDE_FROM_ALL`](https://cmake.org/cmake/help/latest/prop_tgt/EXCLUDE_FROM_ALL.html) to unwanted targets and wrap the `install(TARGETS ...)` call in an `if(BUILD_SHARED_LIBS)`. This will be shorter than wrapping or deleting every line that mentions the unwanted variant.

For a project `CMakeLists.txt` with the following contents:
```cmake
add_library(contoso SHARED contoso.c)
add_library(contoso_static STATIC contoso.c)

install(TARGETS contoso contoso_static EXPORT ContosoTargets)

install(EXPORT ContosoTargets
  FILE ContosoTargets
  NAMESPACE contoso::
  DESTINATION share/contoso)
```

Replace the `install(TARGETS)` line with a conditional block that also excludes the unwanted target from the default build:
```cmake
add_library(contoso SHARED contoso.c)
add_library(contoso_static STATIC contoso.c)

if(BUILD_SHARED_LIBS)
  set_target_properties(contoso_static PROPERTIES EXCLUDE_FROM_ALL 1)
  install(TARGETS contoso EXPORT ContosoTargets)
else()
  set_target_properties(contoso PROPERTIES EXCLUDE_FROM_ALL 1)
  install(TARGETS contoso_static EXPORT ContosoTargets)
endif()

install(EXPORT ContosoTargets
  FILE ContosoTargets
  NAMESPACE contoso::
  DESTINATION share/contoso)
```

### When defining features, explicitly control dependencies

When defining a feature that captures an optional dependency, ensure that the dependency will not be used accidentally when the feature is not explicitly enabled.

```cmake
set(CMAKE_DISABLE_FIND_PACKAGE_ZLIB ON)
set(CMAKE_REQUIRE_FIND_PACKAGE_ZLIB OFF)
if ("zlib" IN_LIST FEATURES)
  set(CMAKE_DISABLE_FIND_PACKAGE_ZLIB OFF)
  set(CMAKE_REQUIRE_FIND_PACKAGE_ZLIB ON)
endif()

vcpkg_cmake_configure(
  SOURCE_PATH ${SOURCE_PATH}
  OPTIONS
    -DCMAKE_DISABLE_FIND_PACKAGE_ZLIB=${CMAKE_DISABLE_FIND_PACKAGE_ZLIB}
    -DCMAKE_REQUIRE_FIND_PACKAGE_ZLIB=${CMAKE_REQUIRE_FIND_PACKAGE_ZLIB}
)
```

The snippet below using [`vcpkg_check_features()`](../maintainers/functions/vcpkg_check_features.md) is equivalent.

```cmake
vcpkg_check_features(OUT_FEATURE_OPTIONS FEATURE_OPTIONS
  FEATURES
    "zlib"    CMAKE_REQUIRE_FIND_PACKAGE_ZLIB
  INVERTED_FEATURES
    "zlib"    CMAKE_DISABLE_FIND_PACKAGE_ZLIB
)

vcpkg_cmake_configure(
    SOURCE_PATH ${SOURCE_PATH}
    OPTIONS
      ${FEATURE_OPTIONS}
)
```

`ZLIB` in the snippet is case-sensitive. For more information, see the [`CMAKE_DISABLE_FIND_PACKAGE_<PackageName>`](https://cmake.org/cmake/help/v3.22/variable/CMAKE_DISABLE_FIND_PACKAGE_PackageName.html) and [`CMAKE_REQUIRE_FIND_PACKAGE_<PackageName>`](https://cmake.org/cmake/help/v3.22/variable/CMAKE_REQUIRE_FIND_PACKAGE_PackageName.html) documentation.

### Place conflicting libs in a `manual-link` directory

A linker input file installed in `lib` or `debug/lib`, such as a `.lib`, `.a`, `.so`, or `.dylib` file, is considered
conflicting if it does any of the following:

- Defines `main`.
- Defines `malloc`.
- Defines symbols that are also defined in other libraries.

Some linker input files intentionally provide narrowly scoped replacements for symbols that consumers would otherwise
define or obtain elsewhere. For example, Google Test's `gtest-main.lib` provides `main` for test executables whose authors
choose to use it instead of defining their own entry point.

Place conflicting linker input files in `lib/manual-link` or `debug/lib/manual-link` so build systems that automatically
link files from the library directory don't select them. Windows DLLs remain in `bin` or `debug/bin`; place their import
libraries in `manual-link` when explicit linking is required. Consumers must explicitly opt in to linking these files and
are responsible for avoiding conflicting definitions. This exception isn't a general mechanism for packaging alternative
providers of another component, and moving a linker input file into `manual-link` doesn't isolate its symbols from the
consumer's link domain.

### Installing prebuilt binaries

Ports that install prebuilt (binary-only) artifacts are allowed but strongly discouraged, provided that the artifacts
don't prevent users from selecting or updating dependencies through vcpkg. Building from source is preferred because it
respects vcpkg settings that control the compiler and flags.

We will reject ports that meet all the following conditions:

1. Install prebuilt binaries rather than building from source, and
1. Those binaries incorporate or require at runtime dependencies provided by other ports in the curated registry, and
1. The installed artifacts expose those dependencies to a link domain that can also contain versions selected through vcpkg.

This combination effectively locks the dependency graph to the versions used when the upstream prebuilt was produced.
If vcpkg updates a dependency such as `zlib` or `openssl`, the prebuilt and user-selected versions can define incompatible
symbols in the same link domain. This can cause One Definition Rule violations, ABI incompatibilities, or missed security
updates.

An artifact exposes a dependency whenever that dependency participates in a user's link domain, regardless of whether
upstream intends to expose it. This includes:

* Consumer-linkable `.lib`, `.a`, `.so`, `.dylib`, or import libraries that bring the dependency into the user's link domain.
* Installed headers that reference (directly or via inline/template code) symbols, types, or macros from other vcpkg ports.
* Installed CMake config or pkg-config files that invoke `find_dependency()` or `Requires:` on other vcpkg ports.

Allowed (but still discouraged) scenarios:

* Host-only helper tools used at build time whose outputs are consumed but which aren't linked against by dependent ports,
  provided that they isolate bundled dependencies or rely only on ubiquitous system runtime libraries.
* Prebuilt libraries whose bundled dependencies remain entirely in separate link domains and don't appear in installed
  headers or exported interfaces. Static linking alone doesn't isolate a dependency from a user's link domain.
* Data-only, firmware, or asset packages not linked into user code.

Forbidden examples:

* A prebuilt library that statically links a private copy of `zlib` and exposes its symbols to a link domain that can also
  contain the `zlib` version selected through vcpkg.
* A prebuilt `libfoo` that installs `lib/libfoo.lib` plus headers including `<zlib.h>` and was compiled against a specific `zlib` version; consumers then link against `libfoo` expecting compatibility.
* A prebuilt SDK that installs a CMake package file calling `find_dependency(OpenSSL)` while the binary was compiled against an older OpenSSL release.

Mitigations / alternatives:

* Provide a from-source build using upstream scripts or add a thin CMake wrapper.
* Ask upstream to publish a source-based release or reproducible build instructions; link the upstream issue/PR in a comment in `portfile.cmake` or `vcpkg.json`.
* Use an [overlay port](../concepts/overlay-ports.md) or a private registry for organization-specific prebuilts that cannot satisfy these rules.

## Versioning

### Follow common conventions for the `"version"` field

When creating a new port, follow the versioning convention used by the package author. When updating the port, continue to use the same convention unless upstream says otherwise. For a full explanation of our conventions, see our [versioning documentation](../users/versioning.md#version-schemes). 

If upstream has not published a release in a while, do not change the port's versioning scheme to `version-date` in order to get the latest changes. These commits may include changes that are not production ready. Instead ask the upstream repository to publish a new release.

### <a name="update-the-port-version-field-in-the-manifest-file-of-any-modified-ports"></a> Update the `"port-version"` field for every modified port

Each PR that modifies a port must produce a new version of that port. PRs are squash-merged, so update each modified
port's version once per PR, not once per commit or review revision.

If the upstream version hasn't changed, increment the `"port-version"` field. If the upstream version has changed, reset
`"port-version"` to zero by removing the field or setting it to `0`.

For Example:

- Zlib's package version is currently `1.2.1`, with no explicit `"port-version"` (equivalent to a `"port-version"` of `0`).
- You've discovered that the wrong copyright file has been deployed, and fixed that in the portfile.
- You should update the `"port-version"` field in the manifest file to `1`.

See the [versioning documentation](../users/versioning.md#port-version) for more information.

### <a name="update-the-version-files-in-versions-of-any-modified-ports"></a> Update the version files in `versions/` for every modified port

vcpkg uses a set of metadata files to power its versioning feature.
These files are located in the following locations:

- `${VCPKG_ROOT}/versions/baseline.json`, (this file is common to all ports) and
- `${VCPKG_ROOT}/versions/${first-letter-of-portname}-/${portname}.json` (one per port).

For example, for `zlib` the relevant files are:

- `${VCPKG_ROOT}/versions/baseline.json`
- `${VCPKG_ROOT}/versions/z-/zlib.json`

For each modified port in a PR, update its version files to record the final Git tree. If you make further changes during
review, run `vcpkg x-add-version <port-name> --overwrite-version` to refresh the unmerged version entry
without incrementing `"port-version"` again.

**The recommended method to update these files is to run the `x-add-version` command, e.g.:**

```console
vcpkg x-add-version zlib
```

If you're updating multiple ports at the same time, instead you can run:

```console
vcpkg x-add-version --all
```

to update the files for all modified ports at once.

For more information, see the [Versioning reference](../users/versioning.md) and [Registries](../concepts/registries.md) articles.

## Patching

vcpkg is a packaging solution, not the ultimate owners of the components that we deploy. We do need to apply patches in some cases to improve compatibility of components with platforms, or compatibility of components with each other.

* We want to avoid patches that:
  * upstream would disagree with
  * cause vulnerabilities or crashes
  * we are incapable of maintaining across upstream version updates
  * are large enough to cause license entanglement with the vcpkg repository itself

### <a name="notify-upstream-owners-for-upstream-relevant-patches"></a> Upstream notification and waiting periods

Upstream owners are the final arbiters of their component's behavior. vcpkg maintainers require upstream notification
for upstream-relevant patches and for bugs identified by AI code review tools in new port submissions.

#### Upstream-relevant patches

If a patch could possibly be useful by upstream, vcpkg maintainers require upstream to be notified of the patch's content.
Patches that apply vcpkg-specific behavior unrelated to upstream, such as devendoring a dependency, don't require notification.

To avoid situations where upstream disagrees with the patch, vcpkg maintainers wait at least 30 days after upstream
notification before applying such patches.

vcpkg maintainers skip this waiting period if they have high confidence that the change is correct. Examples of high
confidence patches include, but are not limited to:

* Upstream's acceptance as a patch (for example, backporting a specific change from a pull request upstream has merged).
* Adding missing `#include`s.
* Small and obvious product code fixes (for example, initializing an uninitialized variable).
* Disabling irrelevant-in-vcpkg components of the build such as tests or examples.

#### AI-identified bugs in new ports

The curated registry aims to avoid shipping known exploitable code, but vcpkg maintainers can't comprehensively audit
every component's source code. AI code review findings aren't, by themselves, a final determination that upstream code
is vulnerable.

When AI code review tools identify a bug during review of a new port submission, vcpkg maintainers require the bug to
be reported to upstream, following the project's security reporting process for potentially exploitable bugs. This
requirement applies even when the finding concerns upstream code that the port doesn't patch.

- If upstream determines that the reported behavior isn't a bug, vcpkg maintainers accept that determination as final
  and don't reject the new port submission on the basis of that finding.
- If upstream fixes the bug, vcpkg maintainers accept the fixed version or an upstream-approved patch, subject to the
  patching policy and the other requirements for new ports.
- If upstream doesn't respond within 30 days after the bug is reported, vcpkg maintainers default to treating the AI
  finding as an exploitable problem and close the new port submission.

The high-confidence exceptions to the patch waiting period don't waive this bug-reporting requirement or the need
for an upstream response.

### Prefer options over patching

It is preferable to set options in a call to `vcpkg_configure_xyz()` over patching the settings directly.

Common options that allow you to avoid patching:

- [MSBUILD] `<PropertyGroup>` settings inside the project file can be overridden via `/p:` parameters
- [CMAKE] Calls to `find_package(XYz)` in CMake scripts can be disabled via [`-DCMAKE_DISABLE_FIND_PACKAGE_XYz=ON`](https://cmake.org/cmake/help/v3.15/variable/CMAKE_DISABLE_FIND_PACKAGE_PackageName.html)
- [CMAKE] Cache variables (declared as `set(VAR "value" CACHE STRING "Documentation")` or `option(VAR "Documentation" "Default Value")`) can be overridden by just passing them in on the command line as `-DVAR:STRING=Foo`. One notable exception is if the `FORCE` parameter is passed to `set()`. For more information, see the [CMake `set` documentation](https://cmake.org/cmake/help/v3.15/command/set.html)

### <a name="prefer-download-patches"></a> Prefer downloading approved patches over checking them into the port

If an approved or merged patch can be obtained from an upstream commit, download and apply it instead of checking it
into the port. This process is preferred because it:

- Confirms that upstream has accepted the patch changes.
- Simplifies the review process by shifting the onus upstream.
- Reduces the vcpkg repository size for users that aren't using the patch.
- Avoids license conflicts with the vcpkg repository.

Download patches only from URLs pinned to a full commit ID, not from pull request or merge request diff URLs whose
contents can change. When downloading a commit patch from GitHub or GitLab, append `?full_index=1` to the URL. This
parameter requests full object IDs in the diff; it doesn't make a mutable URL immutable.

For example:

<https://github.com/linux-audit/audit-userspace/commit/f8e9bc5914d715cdacb2edc938ab339d5094d017.patch?full_index=1>

### Prefer patching over overriding `VCPKG_<VARIABLE>` values

Some variables prefixed with `VCPKG_<VARIABLE>` have an equivalent `CMAKE_<VARIABLE>`.
However, not all of them are passed to the internal package build [(see implementation: Windows toolchain)](https://github.com/Microsoft/vcpkg/tree/master/scripts/toolchains/windows.cmake).

Consider the following example:

```cmake
set(VCPKG_C_FLAGS "-O2 ${VCPKG_C_FLAGS}")
set(VCPKG_CXX_FLAGS "-O2 ${VCPKG_CXX_FLAGS}")
```

Using `vcpkg`'s built-in toolchains this works, because the value of `VCPKG_<LANG>_FLAGS` is forwarded to the appropriate `CMAKE_<LANG>_FLAGS` variable. But, a custom toolchain that is not aware of `vcpkg`'s variables will not forward them.

Because of this, it is preferable to patch the buildsystem directly when setting `CMAKE_<LANG>_FLAGS`.

### <a name="minimize-patches"></a>Minimize patches

When making changes to a library, strive to minimize the final diff. This means you should not reformat the upstream source code when making changes that affect a region. When disabling a conditional, it is better to add an `AND FALSE` or `&& 0` to the condition than to delete every line of the conditional. If a large region needs to be disabled, it is shorter to add an `if(0)` or `#if 0` around the region instead of deleting every line in the patch.

Don't add patches if the port is outdated and updating the port to a newer released version would solve the same issue. vcpkg prefers updating ports over patching outdated versions.

This helps to keep the size of the vcpkg repository down as well as improves the likelihood that the patch will apply to future code versions.

### Do not implement features in patches

The purpose of patching in vcpkg is to enable compatibility with compilers, libraries, and platforms. It is not to implement new features in lieu of following proper Open Source procedure (submitting an Issue/PR/etc).

### Do not build tests/docs/examples by default

When submitting a new port, check for any options like `BUILD_TESTS` or `WITH_TESTS` or `POCO_ENABLE_SAMPLES` and ensure the additional binaries are disabled. This minimizes build times and dependencies for the average user.

Optionally, you can add a `test` feature which enables building the tests, however this should not be in the `Default-Features` list.

## Enable existing users of the library to switch to vcpkg

### Do not add `CMAKE_WINDOWS_EXPORT_ALL_SYMBOLS`

Unless the author of the library is already using it, we should not use this CMake functionality because it interacts poorly with C++ templates and breaks certain compiler features. Libraries that don't provide a .def file and do not use __declspec() declarations simply do not support shared builds for Windows and should be marked as such:

```cmake
if(VCPKG_TARGET_IS_WINDOWS)
    vcpkg_check_linkage(ONLY_STATIC_LIBRARY)
endif()
```

### Do not rename binaries outside the names given by upstream

This means that if the upstream library has different names in release and debug (libx versus libxd), then the debug library should not be renamed to `libx`. Vice versa, if the upstream library has the same name in release and debug, we should not introduce a new name.

Important caveat:

- Static and shared variants often should be renamed to a common scheme. This enables consumers to use a common name and be ignorant of the downstream linkage. This is safe because we only make one at a time available.

If a library generates CMake integration files (`foo-config.cmake`), renaming must be done through patching the CMake build itself instead of simply calling `file(RENAME)` on the output archives/LIBs.

Finally, DLL files on Windows should never be renamed post-build because it breaks the generated LIBs.

### Manifests

We require that the manifest file be formatted. Use the following command to format all manifest files:

```cmd
> vcpkg format-manifest --all
```

## Triplets

We are not accepting requests to add non-community triplets at this time. Promotion from community to full triplet status is primarily based on budget for the hardware to test such triplets and will be driven by metrics submitted by vcpkg to maximize the likelihood what people actually use is fully tested.

We will add community triplets if:

* It is demonstrated that people will actually use that community triplet; and,
* we don't know that such a triplet is broken.

For example, we did not add a triplet in https://github.com/microsoft/vcpkg/pull/29034 because the author was just trying to "complete the set" rather than indicating they would actually use such a thing, and we did not add linux-dynamic until the patchelf solution to make the results relocatable was created.

## Useful implementation notes

### Portfiles are run in Script Mode

While `portfile.cmake`'s and `CMakeLists.txt`'s share a common syntax and core CMake language constructs (aka "Scripting Commands"), portfiles run in "Script Mode", whereas `CMakeLists.txt` files run in "Project Mode". The most important difference between these two modes is that "Script Mode" does not have the concepts of "Toolchain", "Language" and "Target". Any behaviors, including scripting commands, which depend on these constructs (e.g. `CMAKE_CXX_COMPILER`, `CMAKE_EXECUTABLE_SUFFIX`, `CMAKE_SYSTEM_NAME`) will not be correct.

Portfiles have direct access to variables set in the triplet file, but `CMakeLists.txt`s do not (though there is often a translation that happens -- `VCPKG_LIBRARY_LINKAGE` versus `BUILD_SHARED_LIBS`).

Portfiles and Project builds invoked by portfiles are run in different processes. Conceptually:

```
+----------------------------+       +------------------------------------+
| CMake.exe                  |       | CMake.exe                          |
+----------------------------+       +------------------------------------+
| Triplet file               | ====> | Toolchain file                     |
| (x64-windows.cmake)        |       | (scripts/buildsystems/vcpkg.cmake) |
+----------------------------+       +------------------------------------+
| Portfile                   | ====> | CMakeLists.txt                     |
| (ports/foo/portfile.cmake) |       | (buildtrees/../CMakeLists.txt)     |
+----------------------------+       +------------------------------------+
```

To determine the host in a portfile, the standard CMake variables are fine (`CMAKE_HOST_WIN32`).

To determine the target in a portfile, the vcpkg triplet variables should be used (`VCPKG_CMAKE_SYSTEM_NAME`).

See also our [triplet documentation](../users/triplets.md) for a full enumeration of possible settings.
