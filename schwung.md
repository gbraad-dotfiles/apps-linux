# Schwung for Move

### install
```sh evaluate
curl -L https://raw.githubusercontent.com/charlesvestal/schwung/main/scripts/install.sh | sh
```

### web
```sh
xdg-open http://move.local:7700
```

### ssh run default
```sh evaluate
ssh root@move.local
```

### enable-debug
```sh
ssh ableton@move.local 'touch /data/UserData/schwung/debug_log_on'
```

### disable-debug
```sh
ssh ableton@move.local 'rm -f /data/UserData/schwung/debug_log_on'
```

### view-debug
```sh evaluate
ssh ableton@move.local 'tail -f /data/UserData/schwung/debug.log'
```
