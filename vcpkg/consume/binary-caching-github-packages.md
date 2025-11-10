---
title: "Tutorial: Set up a vcpkg binary cache using GitHub Packages in a GitHub Actions workflow"
description: This tutorial shows how to set up a vcpkg binary cache using a NuGet feed hosted in GitHub Packages inside a GitHub Actions workflow
author: vicroms
ms.author: viromer
ms.topic: tutorial
ms.date: 5/1/2025
#CustomerIntent: As a vcpkg user, I want to setup binary caching in my GitHub Actions workflow using GitHub Packages as the binary cache storage
zone_pivot_group_filename: zone-pivot-groups.json
zone_pivot_groups: os-runner
---
# Tutorial: Set up a vcpkg binary cache using GitHub Packages in a GitHub Actions workflow

> [!NOTE]
> This tutorial uses NuGet feeds hosted in GitHub Packages but the same instructions can be used for
> other NuGet feed providers, for example: Azure Artifacts, with minimal changes.

GitHub Packages offers a convenient repository for your NuGet binary packages produced by vcpkg.
In this tutorial, we show you how to set up a binary cache in your GitHub Actions workflow that uses
GitHub Packages as the remote storage.

In this tutorial, you'll learn how to:

> [!div class="checklist"]
>
> * [Configure authentication for GitHub Packages](#1---configure-authentication-for-github-packages)
> * [Bootstrap vcpkg](#2---bootstrap-vcpkg)
> * [Set up required environment variables](#3---set-up-required-environment-variables)
> * [Add GitHub Packages as a NuGet source](#4---add-github-packages-as-a-nuget-source)

## Prerequisites

* A code editor
* A GitHub repository using GitHub Actions
* A project using vcpkg

## 1 - Configure authentication for GitHub Packages

This tutorial uses the built-in `GITHUB_TOKEN` provided by GitHub Actions. To enable
both read and write access to GitHub Packages, add the following `permissions` block to your workflow:

```yaml
permissions:
  packages: write
```

This grants the `GITHUB_TOKEN` the necessary `packages:write` and `packages:read` permissions.
Using `GITHUB_TOKEN` has several advantages:

* No need to create or manage additional secrets
* Works automatically for pull requests from forks (with read-only access)
* Scoped to the specific repository and workflow run

> [!NOTE]
> Alternatively, you can use a classic Personal Access Token (PAT) if you need cross-repository access
> or other advanced scenarios. Follow GitHub's instructions to
> [generate a classic Personal Access Token (PAT)](<https://docs.github.com/authentication/keeping-your-account-and-data-secure/managing-your-personal-access-tokens#creating-a-personal-access-token-classic>)
> with `packages:write` and `packages:read` permissions, then add it as a
> [secret in your repository](<https://docs.github.com/actions/security-guides/using-secrets-in-github-actions>)
> and use `${{ secrets.YOUR_PAT_NAME }}` instead of `${{ secrets.GITHUB_TOKEN }}` in the examples below.

## 2 - Bootstrap vcpkg

vcpkg acquires its own copy of the `nuget.exe` executable that it uses during binary caching
operations. This tutorial uses the vcpkg-acquired `nuget.exe`.

Add a step to bootstrap vcpkg in your workflow:

::: zone pivot="windows-runner"

```YAML
- name: Bootstrap vcpkg
  shell: pwsh
  run: ${{ github.workspace }}/vcpkg/bootstrap-vcpkg.bat
```

::: zone-end

::: zone pivot="linux-runner"

```YAML
- name: Bootstrap vcpkg
  shell: bash
  run: ${{ github.workspace }}/vcpkg/bootstrap-vcpkg.sh
```

::: zone-end

You may need to replace the location of the vcpkg bootstrap script with the correct one for your
workflow, this tutorial assumes that vcpkg is located in a `vcpkg` folder in the root of the repository.

## 3 - Set up required environment variables

Add the following environment variables to your workflow file (replace `<OWNER>` with your GitHub's
username or organization's name):

```YAML
env: 
  USERNAME: <OWNER>
  VCPKG_EXE: ${{ github.workspace }}/vcpkg/vcpkg
  FEED_URL: https://nuget.pkg.github.com/<OWNER>/index.json
  VCPKG_BINARY_SOURCES: "clear;nuget,https://nuget.pkg.github.com/<OWNER>/index.json,readwrite"
```

You may need to replace the value of `VCPKG_EXE` with the location of the vcpkg executable
generated in the [bootstrap vcpkg](#2---bootstrap-vcpkg) step.

In this step you are configuring `VCPKG_BINARY_SOURCES` to use your GitHub Packages feed as a binary
caching source, read the [binary caching reference](../users/binarycaching.md) to learn more.

## 4 - Add GitHub Packages as a NuGet source

The `vcpkg fetch nuget` command outputs the location of the vcpkg-acquired `nuget.exe`, downloading
the executable if necessary.

Add the following step to your workflow file to configure the NuGet source with your `GITHUB_TOKEN`:

::: zone pivot="windows-runner"

```YAML
permissions:
  packages: write

jobs:
  build:
    runs-on: windows-latest
    steps:
    # ... other steps ...
    - name: Add NuGet sources
      shell: pwsh
      run: |
        .$(${{ env.VCPKG_EXE }} fetch nuget) `
          sources add `
          -Source "${{ env.FEED_URL }}" `
          -StorePasswordInClearText `
          -Name GitHubPackages `
          -UserName "${{ env.USERNAME }}" `
          -Password "${{ secrets.GITHUB_TOKEN }}"
        .$(${{ env.VCPKG_EXE }} fetch nuget) `
          setapikey "${{ secrets.GITHUB_TOKEN }}" `
          -Source "${{ env.FEED_URL }}"
```

::: zone-end

::: zone pivot="linux-runner"

On Linux, you need `mono` to execute `nuget.exe`. You can install `mono` using your distribution's
system package manager:

```bash
apt install mono-complete
```

Note that `ubuntu-22.04` GitHub Actions runners come with `mono` preinstalled, but starting with
`ubuntu-24.04` (which `ubuntu-latest` currently points to), `mono` no longer comes preinstalled.

```YAML
permissions:
  packages: write

jobs:
  build:
    runs-on: ubuntu-latest
    steps:
    # ... other steps ...
    - name: Add NuGet sources
      shell: bash
      env: 
        VCPKG_EXE: ${{ github.workspace }}/vcpkg/vcpkg
        USERNAME: <OWNER>
        FEED_URL: https://nuget.pkg.github.com/<OWNER>/index.json
      run: |
        mono `${{ env.VCPKG_EXE }} fetch nuget | tail -n 1` \
          sources add \
          -Source "${{ env.FEED_URL }}" \
          -StorePasswordInClearText \
          -Name GitHubPackages \
          -UserName "${{ env.USERNAME }}" \
          -Password "${{ secrets.GITHUB_TOKEN }}"
        mono `${{ env.VCPKG_EXE }} fetch nuget | tail -n 1` \
          setapikey "${{ secrets.GITHUB_TOKEN }}" \
          -Source "${{ env.FEED_URL }}"
```

::: zone-end

> [!NOTE]
> The default `GITHUB_TOKEN` provided by GitHub Actions does **not** have the required permissions to upload or download cached packages.  
> To enable package caching to GitHub Packages, use a **Personal Access Token (PAT)** instead and ensure it includes the following scopes:
>
> - `packages:read`
> - `packages:write`
>
> Store the PAT as a repository secret (for example, `VCPKG_PAT_TOKEN`) and reference it in your workflow:
>
> ```yaml
> -Password: "${{ secrets.VCPKG_PAT_TOKEN }}"
> -Source: "${{ env.FEED_URL }}"
> ```

And that's it! vcpkg will now upload or restore packages from your NuGet feed hosted on GitHub
Packages inside your GitHub Actions workflow.

## Next steps

> [!div class="nextstepaction"]
> [Authenticate to private NuGet feeds](../reference/binarycaching.md#nuget-credentials)

Here are other tasks to try next:

* [Change the default binary cache location](binary-caching-default.md)
* [Set up a local binary cache](binary-caching-local.md)
* [Set up a binary cache using a NuGet feed](binary-caching-nuget.md)
