# QtWebEngine BaseApp

This base application provides Qt's WebEngine module, which is missing from the KDE runtime, and makes it possible to
quickly and easily package applications that depend on this module.

If you just found this base application, then you might also be interested in the [PyQt base application](https://github.com/flathub/com.riverbankcomputing.PyQt.BaseApp).

*To help improving this documentation, open a pull request against the [wiki branch](https://github.com/flathub/io.qt.qtwebengine.BaseApp/tree/wiki).*

## Supported branches

Any branch based on a non-EOL base runtime should be supported. Branches are considered end-of-life when the base branch is EOL. The current list of supported branches is:

|Baseapp branch     |Base branch   |Qt Webengine version|
|-------------------|--------------|--------------------|
|`branch/5.15-22.08`|5.15-22.08    |5.15.x-lts          |
|`branch/5.15-23.08`|5.15-23.08    |5.15.x-lts          |
|`branch/6.5`       |6.5           |6.5.x               |
|`branch/6.6`       |6.6           |6.6.x               |

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

This feature requires a D-Bus session bus socket in the Flatpak sandbox, but it doesn't need unfiltered access to the session
bus, and will work fine without any extra [xdg-dbus-proxy](https://github.com/flatpak/xdg-dbus-proxy) permissions.

Important to note that without a working session bus socket, QtWebEngine process will fail to start.

During packaging of a Flatpak application there is no session bus socket in the sandbox, so if the build process
needs running QtWebEngine, then it will fail.  
The workaround is to disable sandboxing like this:

```yaml
build-options:
  env:
    QTWEBENGINE_DISABLE_SANDBOX: '1'
  # Or alternatively
    QTWEBENGINE_CHROMIUM_FLAGS: --no-sandbox
```

*Flatpak sandboxing support for Chromium was developed by [Ryan Gonzalez](https://refi64.com/) for the [Chromium Flatpak application](https://github.com/flathub/org.chromium.Chromium).*

## QML

QML modules for Qt Pdf, Webengine and Webview are located in `/app/qml`. If you rely on those you should add `QML_IMPORT_PATH=/app/qml` as an environment variable in your application manifest.

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
runtime-version: '6.6'
sdk: org.kde.Sdk
base: io.qt.qtwebengine.BaseApp
base-version: '6.6'
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
