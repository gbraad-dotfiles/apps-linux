# Schwung for Move

Schwung brings an ecosystem of community-built synths, effects and tools to your Ableton Move, opening up new sounds and workflows alongside stock firmware.

  - https://schwung.dev/


### install
```sh evaluate
curl -L https://raw.githubusercontent.com/charlesvestal/schwung/main/scripts/install.sh | sh
```

### web
```sh
xdg-open http://move.local:7700
```

### dev
```sh
xdg-open http://move.local/development/
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
