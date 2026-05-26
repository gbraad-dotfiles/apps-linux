# Android tools

### fedora-install
```sh
sudo dnf install -y android-tools
```

### user-install
```sh
TMPAT=$(mktemp)
curl -fsSL https://dl.google.com/android/repository/platform-tools_r35.0.0-linux.zip -o $TMPAT
unzip -j $TMPAT -d ${LOCALBIN}
rm $TMPAT
```
