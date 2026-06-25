# Ducttape


### vars
```sh
DUCTTAPE_VERSION="v0.8"
DUCTTAPE_DOWNLOAD_BASEURL="https://github.com/ducttape-infra/ducttape/releases/download/${DUCTTAPE_VERSION}"
``

### check
```sh
command -v macadam >/dev/null 2>&1
```

### install
```sh
arch=$(uname -m)
if [[ "$arch" == "x86_64" ]]; then
  DUCTTAPE_DOWNLOAD_URL="${DUCTTAPE_DOWNLOAD_BASEURL}/ducttape-linux-amd64"
elif [[ "$arch" == "aarch64" ]]; then
  echop "Currently not supported yet"
else
  echo "::error::Unsupported architecture: $arch. Only x86_64 and aarch64 are supported."
  return 1
fi

mkdir -p /tmp/ducttape-install

curl -sSL $DUCTTAPE_DOWNLOAD_URL -o /tmp/ducttape-install/ducttape

chmod +x /tmp/ducttape-install/ducttape

mv /tmp/ducttape-install/ducttape ${LOCALBIN}

echo "Downloaded and installed"

rm -rf /tmp/ducttape-install
```

