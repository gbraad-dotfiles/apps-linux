# GitHub Copilot

### config
```ini
[copilot]
  proxy="ndisguise"
````

### npm-install
```sh
npm install -g @github/copilot
```

### install
```sh
app ${APPNAME} install npm
```

### default alias run
```sh evaluate
${NPMGLOBAL}/bin/copilot
```

### proxied
```sh evaluate
p ${COPILOT_PROXY}
app ${APPNAME} run
```
