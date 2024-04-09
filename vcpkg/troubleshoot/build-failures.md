---
title: Troubleshoot build failures
description: Troubleshooting guide for common build failures in vcpkg ports
author: vicroms
ms.author: viromer
ms.date: 03/20/2024
ms.topic: troubleshooting-general
---

# Troubleshoot build failures in vcpkg ports

This guide is intended for users experiencing issues while [installing 
ports](../commands/install.md) using vcpkg.

## <a name="failure-logs"></a> Locate failure logs

Build failures can happen for an almost infinite number of reasons. When vcpkg
fails to build a port, the first step to diagnose the reason is to read the log files.

vcpkg generates a log file for each external process that gets invoked while
building a port. When an error occurs, vcpkg prints the location of the log file
for the last process that ran before the error ocurred. Look for the line "See
logs for more infomation:" in the vcpkg output.

Example: Log files location output

```Console
    See logs for more information:
      C:\Users\viromer\work\vcpkg\buildtrees\detect_compiler\config-x64-linux-rel-CMakeCache.txt.log
      C:\Users\viromer\work\vcpkg\buildtrees\detect_compiler\config-x64-linux-out.log
```

You can find all the produced log files by inspecting the port's `buildtrees`
directory located in `$VCPKG_ROOT/buildtrees/<port>/` (replace `<port>` with the
appropriate port name).

## Failed to download the port's assets

> [!NOTE]
> Using [asset caching](../concepts/asset-caching.md) can help you mitigate this
> kind of problems by ensuring continued availability of cached assets.

When installing a port an error occurs while downloading an asset from the
network.

```Console
CMake Error at scripts/cmake/vcpkg_download_distfile.cmake:32 (message):

  Failed to download file with error: 1
```

### Cause 1: The download URL is no longer valid
<!-- 
Steps to reproduce:
1. Modify a port's download URL
2. Attempt to install
-->

By reasons outside vcpkg's control URLs can become invalid. Broken links can be
diagnosed by using a web browser to navigate to the download URL, a broken link
will produce a 404 status code.

Steps to resolve:

1 - Modify the port to use an alternative download URL for the asset.

### Cause 2: The file's hash does not match the expected SHA512
<!-- 
Steps to reproduce:
1. Modify a port's download SHA512
2. Attempt to install
-->

```Console
error: Failed to download from mirror set
error: File does not have the expected hash:
url: https://github.com/OpenImageIO/oiio/archive/v2.4.13.0.tar.gz
File: /home/user/vcpkg/downloads/OpenImageIO-oiio-v2-9325beef.4.13.0.tar.gz.1925416.part
Expected hash: 9325beefce55b66a58fcfc2ce93e1406558ed5f6cc37cb1e8e04aee470c4f30a14483bebfb311c329f7868afb6c508a052661c6b12d819a69f707c1a30cd9549
Actual hash: 9e887c7039995ce7c41556e09a7eed940863260a522ecf7d9bec300189026ed507da560620dfa4a619deeb679be7adf42fe3a7020ff3094df777c7934c771227
```

This error occurs when the upstream file has been changed in any way by the
server but the download URL is kept the same. As a security measure, vcpkg will
reject files with SHA512 that don't match the expected SHA512 for the asset.

Steps to resolve:

1 - Verify that the downloaded file is secure
2 - Modify the port to use the new file's SHA512

## Visual Studio toolchain is not found

When installing a port vcpkg is not able to locate an appropriate toolchain:

```Console
error: in triplet arm-windows: Unable to find a valid toolchain for requested target architecture arm.
The selected Visual Studio instance is at: C:\Program Files (x86)\Microsoft Visual Studio\2019\BuildTools
The available toolchain combinations are: x86, amd64, x86_amd64, amd64_x86
```

### Cause 1: The appropriate toolchain for the target architecture is not installed
<!-- 
Steps to reproduce:
1. Uninstall a required toolchain from Visual Studio (e.g.: ARM)
2. Attempt to install for a triplet that requires that toolchain (for example: arm-windows)
-->

If the Visual Studio instance matches your desired vcpkg version and you get
this error, the most likely cause is that the appropriate toolchain is not installed.

* Open the Visual Studio Installer and install the appropriate toolchain.

### Cause 2: The selected Visual Studio version is incorrect
<!--
Steps to reproduce:
1. Uninstall a required toolchain from your latest Visual Studio version (e.g.: ARM)
2. Attempt to install for a triplet that requires that toolchain (for example: arm-windows)
-->

If the required toolchain is installed, vcpkg may have selected an incorrect
version of Visual Studio where the toolchain is not installed. See the
[documentation for the Visual Studio selection
algorithm](../users/triplets.md#VCPKG_VISUAL_STUDIO_PATH) to learn more.

Steps to resolve:

1 - Set the
  [`VCPKG_PLATFORM_TOOLSET`](../users/triplets.md#vcpkg_platform_toolset) to the
  appropriate version.
2 - Alternatively, set the
  [`VCPKG_VISUAL_STUDIO_PATH`](../users/triplets.md#VCPKG_VISUAL_STUDIO_PATH) to
  your desired Visual Studio instance installation path.

## Missing system dependencies

A required build tool or system dependency is not installed in the environment.

Example: Port requires system dependencies

```
alsa currently requires the following programs from the system package manager:
    autoconf autoheader aclocal automake libtoolize
On Debian and Ubuntu derivatives:
    sudo apt install autoconf libtool
On recent Red Hat and Fedora derivatives:
    sudo dnf install autoconf libtool
On Arch Linux and derivatives:
    sudo pacman -S autoconf automake libtool
On Alpine:
    apk add autoconf automake libtool
```

Most vcpkg ports that require system dependencies print a message during installation.
In some cases, the list of required system dependencies may be incomplete. Diagnosing
this type of issues depends on the underlying built system, [check the error 
logs](#failure-logs) to determine the cause of the failure.

Steps to resolve:

1 - Follow the appropriate steps to install the missing system dependency in your environment.

## FetchContent dependency is not found during build process

A build fails because a dependency fulfilled via
[`FetchContent`](https://cmake.org/cmake/help/latest/module/FetchContent.html)
is not available during the build process.

```cmake
include(FetchContent)

FetchContent_Declare(fmt
  GIT_REPOSITORY https://github.com/fmtlib/fmt.git
  GIT_TAG master
)
FetchContent_MakeAvailable(fmt)
```

```Console
CMake Error at CMakeLists.txt:23 (target_link_libraries):
  Target "my_sample_lib" links to:

    fmt::fmt

  but the target was not found.  Possible reasons include:

    * There is a typo in the target name.
    * A find_package call is missing for an IMPORTED target.
    * An ALIAS target is missing.
```

vcpkg disables `FetchContent` via the CMake variable
[`FETCHCONTENT_FULLY_DISCONNECTED`](https://cmake.org/cmake/help/latest/module/FetchContent.html#variable:FETCHCONTENT_FULLY_DISCONNECTED).
There are multiple motivations for disabling this feature, to name a few:

* `FetchContent` can hide vendored dependencies on ports.
* `FetchContent` does not interact well with fully disconnected scenarios.
* Dependencies acquired via `FetchContent` can change the build without
  participating in ABI hash calculation.

For port maintainers, we recommend that any package acquired via `FetchContent`
is turned into a vcpkg dependency. But for general use there are workarounds to
enable `FetchContent`.

Steps to resolve:

1 - Use
[`VCPKG_CMAKE_CONFIGURE_OPTIONS`](../users/triplets.md#vcpkg_cmake_configure_options)
to set `FETCHCONTENT_FULLY_DISCONNECTED=FALSE`


## vcpkg-installed dependencies are not linked during my project build

When building your project via manifest mode or using classic mode, your vcpkg
dependencies are not loaded and linked to your project, resulting in build
failures.

vcpkg offers automatic linkage of libraries and include directories for
[CMake](../users/buildsystems/cmake-integration.md) and
[MSBuild](../users/buildsystems/msbuild-integration.md) projects. Dependencies
installed through vcpkg should be available for consumption automatically in
your project.

### Cause 1: The required dependencies are not installed

Steps to resolve:

1 - Make sure that your project's dependencies are ready:

  If you are using a [manifest file
  (`vcpkg.json`)](../concepts/manifest-mode.md), make sure that the file is
  located in the same directory containing your `CMakeList.txt` file. Also, make
  sure that all your dependencies are listed in the
  [`"dependencies"`](../reference/vcpkg-json.md#dependencies) field in your
  manifest. vcpkg will automatically install dependencies in your manifest when
  you build your project.

  If you are using the vcpkg CLI to [install packages](../commands/install.md)
  in [classic mode](../concepts/classic-mode.md), make sure that all your
  required dependencies are pre-installed before running your project's build.
  vcpkg does not automatically install packages required by your project when
  used in classic mode.

### Cause 2: Integration for your build system is not enabled

#### CMake

Steps to resolve: 

1 - Set the [`CMAKE_TOOLCHAIN_FILE`
variable](<https://cmake.org/cmake/help/latest/variable/CMAKE_TOOLCHAIN_FILE.html>)
to the vcpkg toolchain.

The `CMAKE_TOOLCHAIN_FILE` must be set to point to the vcpkg CMake toolchain
file located in `{VCPKG_ROOT}/scripts/buildsystems/vcpkg.cmake`, make sure to
replace `{VCPKG_ROOT}` with the correct path to your vcpkg installation directory. 

You can use any of the following methods to set the toolchain file:
  
  * Using a `CMakePresets.json` file:
    * Set `toolchainFile` if you use version 3 or later.
    * Set `CMAKE_TOOLCHAIN_FILE` as a `cacheVariable` if you use version 2
      or older.
  * Pass the
    `-DCMAKE_TOOLCHAIN_FILE={VCPKG_ROOT}/scripts/buildsystems/vcpkg.cmake`
    command-line argument during your CMake call.
  * Set `CMAKE_TOOLCHAIN_FILE` in your `CMakeList.txt` file. Make sure that this
    variable is set before any call to `project()`.

If you are already using a custom toolchain file via `CMAKE_TOOLCHAIN_FILE`,
then set
[`VCPKG_CHAINLOAD_TOOLCHAIN_FILE`](../users/triplets.md#vcpkg_chainload_toolchain_file)
via a [custom triplet](../concepts/triplets.md#custom-triplets) to point to your
custom toolchain.

#### MSBuild

vcpkg provides a global integration mechanism via the [`vcpkg integrate
install`](../commands/integrate.md#vcpkg-integrate-install) command.

When the command is executed once, the vcpkg integration becomes enabled for all
projects using MSBuild, either with [manifest
mode](../concepts/manifest-mode.md) or [classic
mode](../concepts/classic-mode.md).

This integration is specific to the vcpkg instance from which the command was
run, so if you have multiple vcpkg instances, only the tool version and the
packages installed in that specific instance are available in your projects.

Read the [MSBuild
integration](../users/buildsystems/msbuild-integration.md#import-props-and-targets)
documentation to learn how to enable vcpkg integration for a single project or
solution.

Steps to resolve:

1 - Run `vcpkg integrate install` for the vcpkg instance you want to use

Run `vcpkg integrate install` to enable global integration with MSBuild. You
only need to do this once for the vcpkg instance to become enabled for all your
projects.


##  Cannot install packages using classic mode

Using the [`vcpkg install`](../commands/install.md) command fails with the
following message:

```Console
error: Could not locate a manifest (vcpkg.json) above the current working directory.
This vcpkg distribution does not have a classic mode instance.
```

### Cause 1: Using the Visual Studio embedded copy of vcpkg

Visual Studio 2022 includes a copy of vcpkg installable with the C++ workflow.
This bundled vcpkg, is installed in a read-only location, and as such, cannot be
used in [classic mode](../concepts/classic-mode.md).

You are either using a Visual Studio embedded terminal or a VS Developer
PowerShell that has a copy of the bundled vcpkg in its PATH. 

Steps to resolve:

  1 - [Create a manifest file (`vcpkg.json`) to install your project's
  dependencies](../get_started/get-started-vs.md).
  2 - If you have a standalone instance of vcpkg, you can use it instead by setting
  the `VCPKG_ROOT` variable and adding the installation path to the `PATH` variable.

  ```PowerShell
  $env:VCPKG_ROOT="path/to/standalone/vcpkg"
  $env:PATH="$env:PATH;$env:VCPKG_ROOT"
  ```

## Issue isn't listed here

If your issue isn't listed here, visit [our
repository](https://github.com/microsoft/vcpkg/issues) to create a new issue.
