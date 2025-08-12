---
title: Dependabot support for vcpkg
description: This article describes how to use GitHub Dependabot to automatically update vcpkg dependencies in your projects.
author: JamieMagee
ms.author: jamagee
ms.date: 8/12/2025
ms.topic: concept-article
---
# Dependabot support for vcpkg

[Dependabot][dependabot] is GitHub's automated dependency management service
that helps keep your project dependencies up-to-date by automatically creating pull requests when new versions are
available. Starting in 2025, Dependabot added support for [vcpkg][vcpkg].

## How Dependabot works with vcpkg

When you enable Dependabot for your vcpkg projects, it monitors your [`vcpkg.json` manifest files][vcpkg-manifest] and automatically
creates pull requests to update the `builtin-baseline` commit hash. This ensures your C/C++ dependencies stay current
with the latest versions available in the [vcpkg port repository][vcpkg-repo].

## Setting up Dependabot for vcpkg

To enable Dependabot version updates for your vcpkg project:

1. Create or update your `.github/dependabot.yml` file in your repository's default branch:

```yaml
version: 2
updates:
  - package-ecosystem: "vcpkg"
    directory: "/" # Location of your vcpkg.json file
    schedule:
      interval: "weekly"
```

2. Customize the configuration based on your project needs:

```yaml
version: 2
updates:
  - package-ecosystem: "vcpkg"
    directory: "/"
    schedule:
      interval: "weekly"
      day: "monday"
      time: "09:00"
      timezone: "America/New_York"
    labels:
      - "dependencies"
      - "vcpkg"
    commit-message:
      prefix: "deps"
```

## Configuration options

Dependabot for vcpkg supports all standard Dependabot configuration options. The key vcpkg-specific requirements are:

- `package-ecosystem` must be set to `"vcpkg"`.
- `directory` should point to the location of your `vcpkg.json` file.

For comprehensive configuration options see the [Dependabot options reference][dependabot-options].

### vcpkg-specific considerations

- **Test thoroughly**: Baseline updates can introduce breaking changes from dependencies.
  Always test your build after applying updates.
- **Monitor vcpkg announcements**: Watch the [vcpkg repository][vcpkg-repo] for announcements
  about breaking changes or important updates.
- **Consider version pinning**: If you need some ports to stay on the same version as you move the rest forward with the
  baseline, consider [overriding][vcpkg-overrides] their versions in
  your `vcpkg.json`.

### vcpkg-specific issues

**Dependabot not creating pull requests:**

- Ensure your repository has a valid `vcpkg.json` with a `builtin-baseline` field.
- Verify the `directory` path in your configuration points to the correct location.

**Build failures after baseline updates:**

- Baseline updates may introduce breaking changes from dependencies.
- Review the vcpkg port changes included in the baseline update.
- Consider pinning specific dependency versions in your `vcpkg.json` if needed.

## Learn more

- [Dependabot version updates documentation][dependabot-version-updates]
- [Dependabot configuration options reference][dependabot-options]
- [vcpkg manifest mode documentation][vcpkg-manifest]
- [vcpkg versioning concepts][vcpkg-versioning]

[dependabot]: https://docs.github.com/code-security/dependabot
[dependabot-options]: https://docs.github.com/code-security/dependabot/working-with-dependabot/dependabot-options-reference
[dependabot-version-updates]: https://docs.github.com/code-security/dependabot/dependabot-version-updates
[vcpkg]: https://vcpkg.io
[vcpkg-manifest]: manifest-mode.md
[vcpkg-overrides]: ../users/versioning.md#overrides
[vcpkg-repo]: https://github.com/Microsoft/vcpkg
[vcpkg-versioning]: ../users/versioning.concepts.md
