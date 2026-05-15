# wmux - Web-based tmux Controller

### info


### config
```ini
[wmux]
   session=screen
```

### vars
```sh
APPNAME=wmux
APPTITLE="Wmux - web-based tmux controller"
SVCNAME="dotfiles-apps-${APPNAME}"

WMUX_CONFIG_DIR="${HOME}/.wmux"
mkdir -p "${WMUX_CONFIG_DIR}"

WMUX_CERT_FILE="${WMUX_CONFIG_DIR}/wmux.crt"
WMUX_KEY_FILE="${WMUX_CONFIG_DIR}/wmux.key"
```

### install
Checks if Tailscale is installed

```sh
if ! app ${APPNAME} check; then
  app ${APPNAME} install
fi
app ${APPNAME} service install
```

### install-service
```sh
apps_service_install ${APPNAME} ${APPTITLE}
```

### enable-service
```sh
systemctl --user enable --now ${SVCNAME}
```

### disable-service
```sh
systemctl --user disable --now ${SVCNAME}
```

### start-service
```sh
systemctl --user start ${SVCNAME}
```

### stop-service
```sh
systemctl --user stop ${SVCNAME}
```

### restart-service
```sh
systemctl --user restart ${SVCNAME}
```

### status-service
```sh
systemctl --user status ${SVCNAME}
```

### active-service
```sh
systemctl --user is-active ${SVCNAME}
```

### journal-service
```sh
journalctl --user -u ${SVCNAME} -f
```

### run-service run
```sh
${LOCALBIN}/wmux --tls
```

### screen
```sh
screen app ${APPNAME} run
```

### install
```sh
ARCH=$(uname -m | sed 's/x86_64/amd64/;s/aarch64/arm64/')
curl -fsSL "https://github.com/gbraad-dotfiles/wmux/releases/latest/download/wmux-${ARCH}" \
   | install -m 0755 /dev/stdin ${LOCALBIN}/wmux
app ${APPNAME} certgen
```

### certgen
```sh
DAYS=3650

# Get Tailscale IP if available
WMUX_IP=$(tailscale ip --4)

if [ -z "$WMUX_IP" ]; then
    echo "Warning: Tailscale IP not found, using localhost"
    WMUX_IP="127.0.0.1"
fi

openssl req -x509 -newkey rsa:4096 -nodes \
    -keyout "$WMUX_KEY_FILE" \
    -out "$WMUX_CERT_FILE" \
    -days $DAYS \
    -subj "/CN=wmux" \
    -addext "subjectAltName=IP:$WMUX_IP,IP:127.0.0.1,DNS:localhost"
```

