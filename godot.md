
# Godot Engine


### vars
```sh
GD_URL="https://godot-releases.nbg1.your-objectstorage.com/4.6.3-stable/Godot_v4.6.3-stable_linux.x86_64.zip"
GD_HOME="${APPSHOME}/Godot"
GD_EXEC="${GD_HOME}/Godot_v4.6.3-stable_linux.x86_64"
GD_ZIP="/tmp/godot.zip"
```

### user-install
```sh
mkdir -p "${GD_HOME}"
curl -L "$GD_URL" -o "$GD_ZIP"
unzip -o "$GD_ZIP" -d "${GD_HOME}"
rm "$GD_ZIP"
```

### user-check
```sh
[ -x "${GD_EXEC}" ]
```

### user-run
```sh
${GD_EXEC}
```

### default alias run run-desktop
```sh
if ! app ${APPNAME} check user; then
  app ${APPNAME} install user
fi
app ${APPNAME} run user
```
