# Devpod


### install
```sh
if command -v devpod &>/dev/null; then echo "devpod already installed: $(devpod version)"; return; fi
TMPDIR=$(mktemp -d)
curl -L -o "${TMPDIR}/devpod" "https://github.com/loft-sh/devpod/releases/latest/download/devpod-linux-${ARCH}" && sudo install -c -m 0755 "${TMPDIR}/devpod" ${LOCALBIN} && rm -rf ${TMPDIR}
```

### provider-docker
```sh
devpod provider add docker
devpod provider use docker
```

### provider-podman
```sh
devpod provider add podman
devpod provider use podman
```

### provider-kubernetes
```sh
devpod provider add kubernetes
devpod provider use kubernetes
```

### list
```sh
devpod list
```

### providers
```sh
devpod provider list
```
