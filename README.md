# setup-swift

A GitHub Action that installs a Swift toolchain through [Swiftly](https://www.swift.org/swiftly/)
on Linux and macOS runners and puts it first on `PATH`.

```yaml
- uses: actions/checkout@v6
- uses: swift-wire/setup-swift@v1
  with:
    swift-version: "6.4.0"
- run: swift build
```

With no `swift-version`, the first line of `.swift-version` in the working directory is used, so a
package that already names its toolchain needs only the one line:

```yaml
- uses: swift-wire/setup-swift@v1
```

Any form `swiftly install` accepts works: a release such as `6.4.0`, a snapshot such as
`6.4.x-snapshot-2026-09-04`, or `main-snapshot`.

## Why another one

The wire repositories are developed against whatever Swift they need — a release today, a snapshot
whenever they move ahead of one — and pin it in `.swift-version`. The action they used before this one
resolved versions from its own metadata and could not install a release its latest tag did not know
about. Swiftly resolves against swift.org directly, so a new Swift needs no change here.

Two details differ from other Swiftly-based actions, and both are deliberate:

- **It never runs `swiftly use`.** That command rewrites `.swift-version`, which dirties the tree for
  any later `git diff --exit-code` guard (a swift-format job, say). The toolchain is installed and its
  own `bin` directory is put first on `PATH` instead.
- **The requested version wins over `.swift-version`.** Swiftly's shims prefer the file in the working
  directory; putting the real toolchain on `PATH` sidesteps them. A workflow can therefore check a
  package against a toolchain other than the one the package is developed on — a release floor
  against a snapshot in `.swift-version`, for instance — without touching the file.

The toolchain's post-install script, which on Linux installs the libraries Foundation links, is run
with `sudo`, so no separate `apt-get install` step is needed.

## Inputs

| Name | Required | Description |
|---|---|---|
| `swift-version` | no | The toolchain to install. Defaults to the first line of `.swift-version`. |

## Outputs

| Name | Description |
|---|---|
| `swift-version` | The toolchain that was installed, as Swiftly names it. |
| `toolchain-path` | The installed toolchain's `bin` directory. |

## Runners

Linux (x86_64 and aarch64) and macOS. Windows runners are refused, since Swiftly does not support them.
The macOS toolchain builds against the SDK of the runner's selected Xcode, as any swift.org toolchain does.

## Licence

Apache 2.0. See [LICENSE](LICENSE).
