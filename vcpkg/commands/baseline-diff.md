---
title: vcpkg x-baseline-diff
description: Command line reference for the vcpkg x-baseline-diff command. Computes version changes to installable packages between two baselines.
ms.date: 06/14/2026
---
# vcpkg x-baseline-diff

[!INCLUDE [experimental](../../includes/experimental.md)]

## Synopsis

```console
vcpkg x-baseline-diff [options] <new-baseline>
vcpkg x-baseline-diff [options] <old-baseline> <new-baseline>
```

## Description

Compares the resolved versions of all packages that would be installed for the current
[manifest](../reference/vcpkg-json.md) between two baselines, and prints a summary of
version changes grouped by direct and transitive dependencies.

This command is useful after running [`x-update-baseline`](update-baseline.md) to review
exactly which package versions changed before committing the update.

`x-baseline-diff` requires [Manifest mode](../concepts/manifest-mode.md) — a
`vcpkg.json` must be present in the current directory or the directory specified by
[`--x-manifest-root`](common-options.md).

### Baseline arguments

The `<old-baseline>` and `<new-baseline>` arguments identify the two registry baselines to
compare. Their meaning depends on the type of the default registry:

| Registry kind | Accepted argument format |
|---|---|
| Builtin (default) or git registry pointing at `https://github.com/microsoft/vcpkg` | Full 40-character commit SHA, or any git ref (tag, branch name) that can be resolved in the local vcpkg clone |
| Git registry (other) | Full 40-character commit SHA. You can run `git rev-parse <branch name>` to get a commit SHA. |
| Filesystem registry | Named baseline string (e.g. `"v1"`, `"v2"`) as defined in `versions/baseline.json` |

### One-argument form

When only `<new-baseline>` is provided, the old baseline is determined automatically:

1. If the `vcpkg.json` contains a
   [`"builtin-baseline"`](../reference/vcpkg-json.md#builtin-baseline) field, that
   value is used as `<old-baseline>`.
2. Otherwise, if `vcpkg-configuration.json` defines a `"default-registry"` with a
   `"baseline"` field, that value is used.
3. If neither is available, the command exits with an error.

### Output

The output lists packages whose resolved version changed between the two baselines,
separated into two sections:

- **Direct dependencies** — packages listed directly in the manifest's
  [`"dependencies"`](../reference/vcpkg-json.md#dependencies) field.
- **Transitive dependencies** — packages pulled in indirectly.

Each changed package is shown as:

```
<package-name>: <old-version> -> <new-version>
```

Newly added packages (present in the new baseline but not the old) are shown as:

```
<package-name>: new: <new-version>
```

## Examples

Compare two specific release tags for a manifest that uses the builtin registry:

```console
vcpkg x-baseline-diff 2024.11.16 2024.12.16
```

Compare two commit SHAs explicitly:

```console
vcpkg x-baseline-diff 5507daa796359fe8d45418e694328e878ac2b82f 9291e3e1d8e4a3d7a9f8b2c1f0e5d6c7b8a9e0f1
```

Use the one-argument form after running `x-update-baseline` — the current
`builtin-baseline` in `vcpkg.json` is used as the old baseline:

```console
vcpkg x-update-baseline
vcpkg x-baseline-diff 2026.03.18
```

Compare two named baselines in a filesystem registry:

```console
vcpkg x-baseline-diff v1 v2
```

## Options

All vcpkg commands support a set of [common options](common-options.md).

### `--x-no-default-features`

Do not include [default features](../reference/vcpkg-json.md#default-features) of the
top-level manifest when computing the dependency graph.

### `--x-feature=<feature>`

Include the specified [feature](../reference/vcpkg-json.md#features) of the top-level
manifest when computing the dependency graph. Can be specified multiple times.

## See also

- [`x-update-baseline`](update-baseline.md) — update baselines for all configured registries
- [Versioning](../users/versioning.md) — how vcpkg resolves package versions using baselines
- [`vcpkg.json` `"builtin-baseline"`](../reference/vcpkg-json.md#builtin-baseline) — the manifest field used as the old baseline in one-argument mode
- [Registries](../users/registries.md) — how to configure custom registries with their own baselines
