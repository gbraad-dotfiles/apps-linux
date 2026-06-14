
# Godot Engine


### vars
```sh
GD_URL="https://godot-releases.nbg1.your-objectstorage.com/4.6.3-stable/Godot_v4.6.3-stable_linux.x86_64.zip"
GD_HOME="${APPSHOME}/Godot"
GD_EXEC="${GD_HOME}/Godot_v4.6.3-stable_linux.x86_64"
GD_ZIP="/tmp/godot.zip"
```

### install
```sh
mkdir -p "${GD_HOME}"
curl -L "$GD_URL" -o "$GD_ZIP"
unzip -o "$GD_ZIP" -d "${GD_HOME}"
rm "$GD_ZIP"
```

### check
```sh
[ -x "${GD_EXEC}" ]
```

### default run alias
```sh
if ! app ${APPNAME} check; then
  app ${APPNAME} install
fi
${GD_EXEC}
```
