# Go

### config
```ini
[go]
  name=gobuild
  from=gofedora
```

### vars
```sh
PROJECT_PATH=$(pwd)
```

### start
```sh
devenv ${GO_NAME} from ${GO_FROM}
```

### stop
```sh
devenv ${GO_NAME} stop
```

### remove
```sh
devenv ${GO_NAME} remove
```

### build
One-shot ephemeral build.

```sh
devenv ${GO_FROM} ephemeral -c "cd ${PROJECT_PATH} && go build -buildvcs=false ./..."
```

### test
```sh
devenv ${GO_FROM} ephemeral -c "cd ${PROJECT_PATH} && go test ./..."
```

### fmt
```sh
devenv ${GO_FROM} ephemeral -c "cd ${PROJECT_PATH} && go fmt ./..."
```

### tidy
```sh
devenv ${GO_FROM} ephemeral -c "cd ${PROJECT_PATH} && go mod tidy"
```

### reuse-build
Reuse persistent container instead of ephemeral.

```sh
if ! devenv ${GO_NAME} exists; then
  devenv ${GO_NAME} from ${GO_FROM}
fi
devenv ${GO_NAME} usercmd "cd ${PROJECT_PATH} && go build ./..."
```

### default alias run
```sh
app ${APPNAME} build
```
