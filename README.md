# QtWebEngine BaseApp

This base application provides Qt's WebEngine module, which is missing from the KDE runtime, and makes it possible to
quickly and easily package applications that depend on this module.

If you just found this base application, then you might also be interested in the [PyQt base application](https://github.com/flathub/com.riverbankcomputing.PyQt.BaseApp).

*To help improving this documentation, open a pull request against the [wiki branch](https://github.com/flathub/io.qt.qtwebengine.BaseApp/tree/wiki).*

## Supported branches

Any branch based on a non-EOL base runtime should be supported. Branches are considered end-of-life when the base branch is EOL. The current list of supported branches is:

|Baseapp branch     |Base branch   |Qt Webengine version|
|-------------------|--------------|--------------------|
|`branch/5.15-23.08`|5.15-23.08    |5.15.x-lts          |
|`branch/6.7`       |6.7           |6.7.x               |
|`branch/6.8`       |6.8           |6.8.x               |

## Features

### Included libraries

* QtWebEngine
* QtWebView
* krb5: Just the minimal needed for Kerberos authentication with Chromium
* libevent
* minizip
* pciutils: libpci without utilities
* re2
* snappy

### Dictionaries

QtWebEngine dictionaries are built from the Hunspell dictionaries and included in this base application.
When building a Flatpak application with this base application, these dictionaries will be packaged in the generated locale extensions,
will not be bundled with the application itself, and that's as long as the maintainer didn't disable locale extensions by setting the
`separate-locales` property.

Applications are expected to set the environment variable `QTWEBENGINE_DICTIONARIES_PATH` to `/app/qtwebengine_dictionaries`.

### Sandboxing support (Qt6 only)

Support for spawning sandboxed Flatpak instances is included, and will be used by QtWebEngine to sandbox Chromium's
render processes.

During build process, if an application needs a running QtWebengine
process sandboxing must be disabled:

```yaml
build-options:
  env:
    QTWEBENGINE_DISABLE_SANDBOX: '1'
  # Or alternatively
    QTWEBENGINE_CHROMIUM_FLAGS: --no-sandbox
```

*Flatpak sandboxing support for Chromium was developed by [Ryan Gonzalez](https://refi64.com/) for the [Chromium Flatpak application](https://github.com/flathub/org.chromium.Chromium).*

## QML

QML modules for Qt Pdf, Webengine and Webview are located in `/app/lib/qml`
for 6.8+ branches and `/app/qml` for branches earlier that 6.8.

Apps using branches 6.8+, do not need to set anything as that is done by
the runtime and it should work by default.

Anyone else should set `--env=QML_IMPORT_PATH=<path>` as a finish-arg
in the application manifest.

## Pkgconfig files

Qt webengine installs some pkgconfig files in `/app/lib/$(gcc --print-multiarch)/pkgconfig` which is not in the `PKG_CONFIG_PATH` set by `flatpak-builder`. If your program relies on them at build time you need to use `append-pkg-config-path` or `prepend-pkg-config-path` in the manifest.

```yaml
build-options:
  arch:
    aarch64:
      prepend-pkg-config-path: /app/lib/aarch64-linux-gnu/pkgconfig
    x86_64:
      prepend-pkg-config-path: /app/lib/x86_64-linux-gnu/pkgconfig
```

## Cleanup

Please make sure to cleanup development files from the BaseApp, in the application

```yaml
cleanup-commands:
  - /app/cleanup-BaseApp.sh
```

## Example QtWebEngine Application

```yaml
id: org.kde.QtWebEngine.SampleApplication
runtime: org.kde.Platform
runtime-version: '6.8'
sdk: org.kde.Sdk
base: io.qt.qtwebengine.BaseApp
base-version: '6.8'
command: qtwebengine-sample-application
finish-args:
  - --device=dri
  - --env=QTWEBENGINEPROCESS_PATH=/app/bin/QtWebEngineProcess
  - --share=ipc
  - --share=network
  - --socket=fallback-x11
  - --socket=pulse
  - --socket=wayland
cleanup-commands:
  - /app/cleanup-BaseApp.sh
modules:
  - name: sample-application
  ...
```

## Creating new branches

New branches are created whenever a new KDE runtime is released on
Flathub.

If it's a Qt6 based runtime, create the PR branch on top of the
latest Qt6 branch of this repo. For example if the latest branch here is
`branch/6.7`, do `git checkout -b pr-qt-6.8 branch/6.7`.

If it's a Qt5 based runtime, create the PR branch on top of the latest
`branch/5.15-xx.08`, by doing `git checkout -b pr-qt-5.15-25.08 branch/5.15-24.08`
(for example).

Now,

1. Update the [branch](https://github.com/flathub/io.qt.qtwebengine.BaseApp/blob/678880a46a9cd3a6d00bad0476a1c16f9865bcc6/io.qt.qtwebengine.BaseApp.json#L3), [runtime-version](https://github.com/flathub/io.qt.qtwebengine.BaseApp/blob/678880a46a9cd3a6d00bad0476a1c16f9865bcc6/io.qt.qtwebengine.BaseApp.json#L10) to the new runtime branch.

2. Update the [extensions](https://github.com/flathub/io.qt.qtwebengine.BaseApp/blob/678880a46a9cd3a6d00bad0476a1c16f9865bcc6/io.qt.qtwebengine.BaseApp.json#L8) and [extension path](https://github.com/flathub/io.qt.qtwebengine.BaseApp/blob/678880a46a9cd3a6d00bad0476a1c16f9865bcc6/io.qt.qtwebengine.BaseApp.json#L18) if necessary.

3. Update [qtwebengine](https://github.com/flathub/io.qt.qtwebengine.BaseApp/blob/678880a46a9cd3a6d00bad0476a1c16f9865bcc6/io.qt.qtwebengine.BaseApp.json#L42-L53), and [qtwebview](https://github.com/flathub/io.qt.qtwebengine.BaseApp/blob/678880a46a9cd3a6d00bad0476a1c16f9865bcc6/qt6-webview/qt6-webview.json#L29-L41) modules to the new Qt tag and also update their
x-checker data.

4. Qt minor version updates require rebasing the chromium sandboxing patches
in this [directory](https://github.com/flathub/io.qt.qtwebengine.BaseApp/tree/branch/6.7/patches).
These patches come from the [Chromium Flatpak](https://github.com/flathub/org.chromium.Chromium/tree/master/patches).
To rebase the patches, clone https://invent.kde.org/qt/qt/qtwebengine.git
with the chromium submodule (this is several gigabytes) and checkout the
target Qt branch. The chromium source is in `src/3rdparty/chromium`.

5. Lastly update the README in the wiki branch and add the newly created
branch to the [update workflow](https://github.com/flathub/io.qt.qtwebengine.BaseApp/blob/c89c848ed357b157e9e692f0aacefc76128e15d4/.github/workflows/update.yaml#L12)

Once that is done, open a PR with the work.

### Merging

Once a build is successful PRs for new branches needs to be **manually merged**
by a maintainer by doing

```sh
git checkout -b branch/{new_target_branch_version} branch/{old_parent_branch_version}
git merge {pr_branch}
git push
```
