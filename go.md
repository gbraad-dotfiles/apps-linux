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
if [[ "${PROJECT_PATH}" != "${HOME}/Projects"* ]]; then
  echo "error: must be run from within ~/Projects/" >&2
  return 1
fi
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
One-shot ephemeral build for local arch.

```sh
devenv ${GO_FROM} ephemeral -c "cd ${PROJECT_PATH} && ${EXTRA_ARGS} go build -buildvcs=false ./..."
```

### amd-build
```sh
devenv ${GO_FROM} ephemeral -c "cd ${PROJECT_PATH} && GOARCH=amd64 GOOS=linux go build -buildvcs=false ./..."
```

### arm-build
```sh
devenv ${GO_FROM} ephemeral -c "cd ${PROJECT_PATH} && GOARCH=arm64 GOOS=linux go build -buildvcs=false ./..."
```

### windows-build
```sh
devenv ${GO_FROM} ephemeral -c "cd ${PROJECT_PATH} && GOARCH=amd64 GOOS=windows go build -buildvcs=false ./..."
```

### test
```sh evaluate
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

### make
```sh
devenv ${GO_FROM} ephemral -c "cd ${PROJECT_PATH} && make"
```

### reuse-make
```sh
if ! devenv ${GO_NAME} exists; then
  devenv ${GO_NAME} from ${GO_FROM}
fi
devenv ${GO_NAME} usercmd "cd ${PROJECT_PATH} && make"
```

### default alias run
```sh
app ${APPNAME} build
```
