# QtWebEngine BaseApp

This base application provides Qt's WebEngine module, which is missing from the KDE runtime, and makes it possible to
quickly and easily package applications that depend on this module.

If you just found this base application, then you might also be interested in the [PyQt base application](https://github.com/flathub/com.riverbankcomputing.PyQt.BaseApp).

*To help improving this documentation, open a pull request against the [wiki branch](https://github.com/flathub/io.qt.qtwebengine.BaseApp/tree/wiki).*

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

```
build-options:
  env:
    QTWEBENGINE_DISABLE_SANDBOX: '1'
  # Or alternatively
    QTWEBENGINE_CHROMIUM_FLAGS: --no-sandbox
```

*Flatpak sandboxing support for Chromium was developed by [Ryan Gonzalez](https://refi64.com/) for the [Chromium Flatpak application](https://github.com/flathub/org.chromium.Chromium).*


## Example QtWebEngine Application

```
app-id: org.kde.QtWebEngine.SampleApplication
runtime: org.kde.Platform
runtime-version: '6.3'
sdk: org.kde.Sdk
base: io.qt.qtwebengine.BaseApp
base-version: '6.3'
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
