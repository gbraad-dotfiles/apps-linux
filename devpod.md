# Devpod


### install
```sh
TMPDIR=$(mktemp -d)
curl -L -o "${TMPDIR}/devpod" "https://github.com/loft-sh/devpod/releases/latest/download/devpod-linux-${ARCH}" && sudo install -c -m 0755 "${TMPDIR}/devpod" ${LOCALBIN} && rm -rf ${TMPDIR}
```
