# OpenShot


### vars
```sh
OPENSHOT_URL="https://github.com/OpenShot/openshot-qt/releases/download/v3.5.1/OpenShot-v3.5.1-x86_64.AppImage"
OPENSHOT_HOME="${APPSHOME}/OpenShot"
OPENSHOT_EXEC="${OPENSHOT_HOME}/OpenShot-v3.5.1-x86_64.AppImage"
```

### user-install
```sh
mkdir -p "${OPENSHOT_HOME}"
curl -L "$OPENSHOT_URL" -o "${OPENSHOT_EXEC}"
chmod +x ${OPENSHOT_EXEC}
```

### user-check
```sh
[ -x "${OPENSHOT_EXEC}" ]
```

### user-run
```sh
${OPENSHOT_EXEC}
```

### flatpak-install
```sh
flatpak install flathub org.openshot.OpenShot
```

### flatpak-run
```sh
flatpak run org.openshot.OpenShot
```

### default alias run desktop-run
```sh
if ! app ${APPNAME} check user; then
  app ${APPNAME} install user
fi
app ${APPNAME} run user
```
