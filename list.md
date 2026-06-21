# Application list

### show
```sh
show_apps_tree() {
  local dir="$1"
  local prefix="$2"

  # List only non-hidden entries (skip dotfiles/dirs)
  local entries=()
  while IFS= read -r entry; do
    local base=$(basename "$entry")
    [[ "$base" == .* ]] && continue         # skip hidden
    [[ "$base" == "README.md" ]] && continue # skip README.md
    entries+=("$entry")
  done < <(find "$dir" -maxdepth 1 -mindepth 1 | sort)

  local total=${#entries[@]}
  local idx=0

  for entry in "${entries[@]}"; do
    idx=$((idx+1))
    local base=$(basename "$entry")
    local connector="├──"
    [ "$idx" -eq "$total" ] && connector="└──"

    if [ -d "$entry" ]; then
      echo "${prefix}${connector} $base"
      show_apps_tree "$entry" "$prefix│   "
    elif [[ "$base" == *.md ]]; then
      if [ -L "$entry" ]; then
        target=$(readlink "$entry")
        base="${base%.md} -> ${target%.md}"
        descfile="$dir/$(basename "$target")"
      else
        base="${base%.md}"
        descfile="$entry"
      fi
      if [ -f "$descfile" ]; then
        desc=$(grep -m1 '^# ' "$descfile" | sed 's/^# //')
      else
        desc=""
      fi
      printf "%s%s %-20s -  %s\n" "$prefix" "$connector" "$base" "$desc"
    fi
  done
}

echo "App defintions in ${APPS_PATH}"
show_apps_tree ${APPS_PATH} "  "
```

### update
```sh
cd ${APPS_PATH}
git pull
cd -

echo "Generate aliases"
app list aliases
```

### switch
```sh
cd ${APPS_PATH}
git remote remove origin
git remote add origin git@github.com:gbraad-dotfiles/apps-linux
git fetch
git branch --set-upstream-to=origin/main main
cd - > /dev/null
```

### log
```sh
cd ${APPS_PATH}
git log -n 2
```

### reset
```sh
cd ${APPS_PATH}
git stash
git fetch origin
git reset --hard origin/main
cd - > /dev/null
```

### aliases
Generate aliases for application defintion that use an `alias` section

> [!NOTE]
> Needs to run in `interactive`-mode to allow the aliases to be exported

```sh evaluate
apps_aliases() {
  if [[ $(dotini launcher --get "launcher.aliases") == true ]]; then
    # Find all .md files in APPSDIR and subfolders
    find "${APPS_PATH}" -type f -name '*.md' | while read -r mdfile; do

      appname="${mdfile##*/}"
      appname="${appname%.md}"

      folder="$(dirname "${mdfile}")"
      folder="${folder##*/}"

      if [[ "${folder}" == "$(basename "${APPS_PATH}")" ]]; then
        alias_name="${appname}"
        alias_cmd="app ${appname}"   # use default
      else
        alias_name="${folder}-${appname}"
        alias_cmd="app ${folder}/${appname}"  # use default
      fi

      if grep -E -q '^###.*\balias\b' "$mdfile"; then
        alias "${alias_name}"="${alias_cmd}"
      fi
    done
  fi
}

apps_aliases
unset FILENAME
unset APPNAME
unset TITLE
```

### services
Lists all application names that have a `run-service` section

```sh
apps_list_services_and_descs() {
  local appspath="$1"
  find -L "$appspath" -type f -name '*.md' ! -name 'README.md' | sort | while IFS= read -r file; do
    # Check for 'run-service' in a level-3 heading
    if grep -Eq '^###.*\brun-service\b' "$file"; then
      relpath="${file#$appspath/}"
      relpath="${relpath%.md}"
      desc=$(grep -m1 '^# ' "$file" | sed 's/^# //')
      printf "%s\t%s\n" "$relpath" "$desc"
    fi
  done
}
apps_list_services_and_descs ${APPS_PATH}
```

### desktop
Lists all application names that have a `run-desktop` section`

```sh
apps_desktop_names_and_descs() {
  local appspath="$1"
  find -L "$appspath" -type f -name '*.md' ! -name 'README.md' | sort | while IFS= read -r file; do
    # Check for 'run-service' in a level-3 heading
    if grep -Eq '^###.*\brun-desktop\b' "$file"; then
      relpath="${file#$appspath/}"
      relpath="${relpath%.md}"
      desc=$(grep -m1 '^# ' "$file" | sed 's/^# //')
      printf "%s\t%s\n" "$relpath" "$desc"
    fi
  done
}

apps_desktop_names_and_descs ${APPS_PATH}
```

