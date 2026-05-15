# wmux - Web-based tmux Controller

### info

### vars
```sh
APPNAME=wmux
APPTITLE="Wmux - web-based tmux controller"
SVCNAME="dotfiles-apps-${APPNAME}"
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
/usr/local/bin/wmux --tls
```

### screen
```sh
screen app ${APPNAME} run
```

### install
```sh
ARCH=$(uname -m | sed 's/x86_64/amd64/;s/aarch64/arm64/')
curl -fsSL "https://github.com/gbraad-dotfiles/wmux/releases/latest/download/wmux-${ARCH}" \
   | sudo install -m 0755 /dev/stdin /usr/local/bin/wmux
```

### user-install
```sh
ARCH=$(uname -m | sed 's/x86_64/amd64/;s/aarch64/arm64/')
curl -fsSL "https://github.com/gbraad-dotfiles/wmux/releases/latest/download/wmux-${ARCH}" \
   | install -m 0755 /dev/stdin ${HOME}/.local/bin/wmux
```
