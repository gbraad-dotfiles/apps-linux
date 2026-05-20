# Antigravity

### vars
````sh
URL=https://storage.googleapis.com/antigravity-public/antigravity-hub/2.0.0-6324554176528384/linux-x64/Antigravity.tar.gz
INSTALL_DIR=${APPHSOME}/Antigravity
```

### install
```sh
mkdir -p ${INSTALL_DIR}
curl -fsSL "${URL}" -o /tmp/agy.tar.gz
tar --strip-components=1 -xzf /tmp/agy.tar.gz -C "${INSTALL_DIR}"
rm /tmp/agy.tar.gz
```

### cli-install
```sh evaluate
curl -fsSL https://antigravity.google/cli/install.sh | bash
```
