---
title: Install vcpkg
description: Tutorial for installing vcpkg on Windows, macOS, or Linux systems
zone_pivot_group_filename: zone-pivot-groups.json
zone_pivot_groups: operating-systems
author: JavierMatosD
ms.topic: tutorial
ms.date: 08/31/2023
ms.author: javiermat
ms.prod: vcpkg
ROBOTS: NOINDEX
#CustomerIntent: As a user, I want to install vcpkg for my operating system.
---

# Tutorial: Install vcpkg

This tutorial provides a step-by-step guide for installing vcpkg on a Windows, macOS, or Linux system. Follow this tutorial if you want to set up vcpkg on a development machine or continuous integration system. If you are a Visual Studio or Visual Studio Code user, see the following tutorials instructions to use vcpkg within the editor:

- [Use vckg from Visual Studio IDE]()
- [Use vckg from Visual Studio Code]()

## Prerequisites

- A terminal to work in.
- `Git`

## 1 - Clone the repository

The first step is to clone the vcpkg repository from GitHub. The repository contains scripts to acquire the vcpkg executable and a registry of curated open-source libraries maintained by the vcpkg community. To do this, run:

```console
git clone https://github.com/microsoft/vcpkg.git
```

The vcpkg open-source registry is a set of over 2,000 free libraries that have been tested together and optimized to work smoothly with vcpkg. While the vcpkg repo does not contain the source code for these libraries, it does contain build recipes and metadata about each library. These files will be included when vcpkg is installed to your system.

## 2 - Run the bootstrap script

Now that you have cloned the vcpkg repository, navigate to the `vcpkg` directory and execute the bootstrap script:

::: zone pivot="os-windows"

```console
cd vcpkg && ./bootstrap-vcpkg.bat
```

::: zone-end
::: zone pivot="os-linux,os-macos"

```console
cd vcpkg && ./bootstrap-vcpkg.sh
```

::: zone-end

The bootstrap script performs prerequisite checks, downloads the vcpkg executable, and manages telemetry settings.

## 3 - Enable terminal auto-completion (optional)

You'll want to integrate vcpkg into your terminal to enjoy tab-completion features. To do this, run:

::: zone pivot="os-windows"

```console
./vcpkg integrate powershell
```

::: zone-end
::: zone pivot="os-linux"

```console
./vcpkg integrate bash
```

::: zone-end
::: zone pivot="os-macos"

```console
./vcpkg integrate zsh
```

::: zone-end

The `vcpkg integrate` command sets up shell integration for vcpkg, which adds vcpkg tab-completion support.

That's it! vcpkg is set up and ready to use.
