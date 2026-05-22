# Dev Containers (devpod)


### shared
```sh
_devcontainers_dir="${HOME}/.dotfiles/.devcontainer"
_devcontainers_provider="${DEVPOD_PROVIDER:-docker}"

devcontainers_commands=(
  up ssh stop delete status
)

devcontainers_prefixes() {
  for d in "${_devcontainers_dir}"/*/; do
    local name="${d%/}"
    name="${name##*/}"
    # only include dirs that have a devcontainer.json
    [[ -f "${d}devcontainer.json" ]] && echo "$name"
  done
}

devcontainers_targets() {
  devpod list --output json 2>/dev/null \
    | python3 -c "
import sys, json
try:
    items = json.load(sys.stdin)
    for w in items:
        print(w.get('id','') + '\t' + w.get('status','unknown'))
except:
    pass
" 2>/dev/null
}

devcontainers_running_targets() {
  devcontainers_targets | awk -F'\t' '$2 ~ /[Rr]unning/ {print $1 "\t" $2}'
}

devcontainers_up() {
  local name=$1
  devpod up \
    --provider "${_devcontainers_provider}" \
    --ide none \
    --source "file://${_devcontainers_dir}/${name}" \
    --id "${name}" \
    "${name}" 2>&1
}
```


### pick
```sh
devcontainers_pick() {
  local targets chosen_target target_list
  targets=$(devcontainers_targets)
  target_list=""
  declare -A known
  if [[ -n "$targets" ]]; then
    while IFS=$'\t' read -r name state; do
      target_list+="${name}\t[${state}]\n"
      known["$name"]=1
    done <<< "$targets"
  fi
  for prefix in $(devcontainers_prefixes); do
    [[ -z "${known[$prefix]}" ]] && target_list+="${prefix}\t[Create]\n"
  done
  target_list=$(printf "%b" "$target_list" | sed '/^$/d')
  chosen_target=$(printf "%b" "$target_list" | column -t -s $'\t' | fzf --prompt="Choose devcontainer> ")
  [[ -z "$chosen_target" ]] && return 1
  echo "$chosen_target" | awk '{print $1}'
}
devcontainers_pick
```


### default alias run
```sh evaluate
run_devcontainers() {
  chosen_command=$(printf "%s\n" "${devcontainers_commands[@]}" | fzf --prompt="Devcontainer command> ")
  [[ -z "$chosen_command" ]] && return

  local targets target_list chosen_target name
  declare -A known

  # Build target list based on command
  case "$chosen_command" in
    up)
      # Merge running workspaces + unstarted prefixes
      targets=$(devcontainers_targets)
      if [[ -n "$targets" ]]; then
        while IFS=$'\t' read -r n state; do
          target_list+="${n}\t[${state}]\n"
          known["$n"]=1
        done <<< "$targets"
      fi
      for prefix in $(devcontainers_prefixes); do
        [[ -z "${known[$prefix]}" ]] && target_list+="${prefix}\t[Create]\n"
      done
      ;;
    ssh)
      # Only running
      targets=$(devcontainers_running_targets)
      if [[ -z "$targets" ]]; then echo "No running devcontainers found." && return; fi
      while IFS=$'\t' read -r n state; do
        target_list+="${n}\t[${state}]\n"
      done <<< "$targets"
      ;;
    status|stop|delete)
      # All known workspaces
      targets=$(devcontainers_targets)
      if [[ -z "$targets" ]]; then echo "No devcontainers found." && return; fi
      while IFS=$'\t' read -r n state; do
        target_list+="${n}\t[${state}]\n"
      done <<< "$targets"
      ;;
  esac

  target_list=$(printf "%b" "$target_list" | sed '/^$/d')
  chosen_target=$(printf "%b" "$target_list" | column -t -s $'\t' | fzf --prompt="Choose devcontainer> ")
  [[ -z "$chosen_target" ]] && return
  name=$(echo "$chosen_target" | awk '{print $1}')

  case "$chosen_command" in
    up)
      devcontainers_up "$name"
      ;;
    ssh)
      devpod ssh --provider "${_devcontainers_provider}" "$name"
      ;;
    status)
      devpod status "$name"
      # re-prompt for follow-up action
      chosen_command=$(printf "%s\n" up ssh stop delete | fzf --prompt="Action on ${name}> ")
      [[ -z "$chosen_command" ]] && return
      case "$chosen_command" in
        up)     devcontainers_up "$name" ;;
        ssh)    devpod ssh --provider "${_devcontainers_provider}" "$name" ;;
        stop)   devpod stop "$name" ;;
        delete) devpod delete --force "$name" ;;
      esac
      ;;
    stop)
      devpod stop "$name"
      ;;
    delete)
      devpod delete --force "$name"
      ;;
  esac
}

run_devcontainers
```
