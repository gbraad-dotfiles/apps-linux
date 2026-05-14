# Macadam


### vars
```sh
GVPROXY_VERSION="v0.8.7"
GVPROXY_DOWNLOAD_BASEURL="https://github.com/containers/gvisor-tap-vsock/releases/download/${GVPROXY_VERSION}"
MACADAM_VERSION="v0.2.0"
MACADAM_DOWNLOAD_BASEURL="https://github.com/crc-org/macadam/releases/download/${MACADAM_VERSION}"
MACADAM_DOWNLOAD_MINEURL="https://github.com/gbraad-dotfiles/macadam/releases/download/v0.2.1a/"
```

### check
```sh
command -v macadam >/dev/null 2>&1
```

### install
```sh
arch=$(uname -m)
if [[ "$arch" == "x86_64" ]]; then
  GVPROXY_DOWNLOAD_URL="${GVPROXY_DOWNLOAD_BASEURL}/gvproxy-linux-amd64"
  MACADAM_DOWNLOAD_URL="${MACADAM_DOWNLOAD_MINEURL}/macadam-linux-amd64"
elif [[ "$arch" == "aarch64" ]]; then
  GVPROXY_DOWNLOAD_URL="${GVPROXY_DOWNLOAD_BASEURL}/gvproxy-linux-arm64"
  MACADAM_DOWNLOAD_URL="${MACADAM_DOWNLOAD_MINEURL}/macadam-linux-arm64"
else
  echo "::error::Unsupported architecture: $arch. Only x86_64 and aarch64 are supported."
  return 1
fi

mkdir -p /tmp/macadam-install

curl -sSL $GVPROXY_DOWNLOAD_URL -o /tmp/macadam-install/gvproxy
curl -sSL $MACADAM_DOWNLOAD_URL -o /tmp/macadam-install/macadam

chmod +x /tmp/macadam-install/gvproxy
chmod +x /tmp/macadam-install/macadam

mv /tmp/macadam-install/gvproxy ${LOCALBIN}
mv /tmp/macadam-install/macadam ${LOCALBIN}

echo "Downloaded and installed macadam + gvproxy binary"

rm -rf /tmp/macadam-install
```

