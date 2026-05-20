# Dev Pods (k8s/k3s)


### shared
```sh
devpods_commands=(
  deploy from undeploy status logs shell exec tsconnect switch
)

devpods_prefixes() {
  local key="images"
  local output prefixes
  output=($(dotini devenv --list | grep "^${key}\." || true))
  prefixes=(${output//${key}./})
  prefixes=(${prefixes//=*/})
  printf "%s\n" "${prefixes[@]}"
}

devpods_targets() {
  KUBECONFIG=${KUBECONFIG:-${HOME}/.kube/config} kubectl get pods --no-headers 2>/dev/null \
    | awk '$1 ~ /sys$/ {sub(/sys$/, "", $1); print $1 "\t" $3 "\t" $5}'
}

devpods_running_targets() {
  KUBECONFIG=${KUBECONFIG:-${HOME}/.kube/config} kubectl get pods --no-headers 2>/dev/null \
    | awk '$1 ~ /sys$/ && $3 == "Running" {sub(/sys$/, "", $1); print $1 "\t" $3 "\t" $5}'
}
```


### pick
```sh
devpods_pick() {
    local targets chosen_target target_prefix target_list
    targets=$(devpods_targets)
    target_list=""
    declare -A known_prefixes
    if [[ -n "$targets" ]]; then
      while IFS=$'\t' read -r prefix state age; do
        target_list+="$prefix\t[$state  $age]\n"
        known_prefixes["$prefix"]=1
      done <<< "$targets"
    fi
    for prefix in $(devpods_prefixes); do
      if [[ -z "${known_prefixes[$prefix]}" ]]; then
        target_list+="$prefix\t[Deploy]\n"
      fi
    done
    target_list=$(echo -e "$target_list" | sed '/^$/d')

    chosen_target=$(echo -e "$target_list" | column -t -s $'\t' | fzf --prompt="Choose pod> ")
    [[ -z "$chosen_target" ]] && return 1
    target_prefix=$(echo "$chosen_target" | awk '{print $1}')
    echo "$target_prefix"
}
devpods_pick
```

### running-pick
```sh
devpods_pick_running() {
    local chosen_target target_prefix
    chosen_target=$(devpods_running_targets | awk '{printf "%-20s [%s  %s]\n", $1, $2, $3}' | fzf --prompt="Choose running pod> ")
    [[ -z "$chosen_target" ]] && return 1
    target_prefix=$(echo "$chosen_target" | awk '{print $1}')
    echo "$target_prefix"
}
devpods_pick_running
```

### default alias run
```sh evaluate
run_devpods() {

  chosen_command=$(printf "%s\n" "${devpods_commands[@]}" | fzf --prompt="Pod command> ")
  [[ -z "$chosen_command" ]] && return

  if [[ "$chosen_command" == "deploy" ]]; then
    target_list=""
    declare -A known_prefixes
    targets=$(devpods_targets)
    if [[ -n "$targets" ]]; then
      while IFS=$'\t' read -r prefix state age; do
        target_list+="$prefix\t[$state  $age]\n"
        known_prefixes["$prefix"]=1
      done <<< "$targets"
    fi
    for prefix in $(devpods_prefixes); do
      if [[ -z "${known_prefixes[$prefix]}" ]]; then
        target_list+="$prefix\t[Deploy]\n"
      fi
    done
    target_list=$(echo -e "$target_list" | sed '/^$/d')
    chosen_target=$(echo -e "$target_list" | column -t -s $'\t' | fzf --prompt="Choose pod> ")
    [[ -z "$chosen_target" ]] && return
    target_prefix=$(echo "$chosen_target" | awk '{print $1}')
    dev3s "$target_prefix" deploy
    return
  fi

  if [[ "$chosen_command" == "from" ]]; then
    freeform_name=$(echo | fzf --prompt="Enter pod name> " --print-query --phony | head -n1)
    [[ -z "$freeform_name" ]] && return
    source_prefix=$(devpods_prefixes | fzf --prompt="Select source image> ")
    [[ -z "$source_prefix" ]] && return
    dev3s "$freeform_name" from "$source_prefix"
    return
  fi

  if [[ "$chosen_command" == "switch" ]]; then
    ctx=$(KUBECONFIG=${KUBECONFIG:-${HOME}/.kube/config} kubectl config get-contexts --no-headers \
      | awk '{print $2}' | fzf --prompt="Select context> ")
    [[ -z "$ctx" ]] && return
    dev3s switch "$ctx"
    return
  fi

  # All other commands need a running pod
  targets=$(devpods_running_targets)
  [[ -z "$targets" ]] && echo "No running pods found." && return
  target_list=$(echo -e "$targets" | awk '{printf "%-20s [%s  %s]\n", $1, $2, $3}')
  chosen_target=$(echo -e "$target_list" | fzf --prompt="Choose pod> ")
  [[ -z "$chosen_target" ]] && return
  target_prefix=$(echo "$chosen_target" | awk '{print $1}')

  if [[ "$chosen_command" == "exec" ]]; then
    exec_cmd=$(echo | fzf --prompt="Command to exec> " --print-query --phony | head -n1)
    [[ -z "$exec_cmd" ]] && exec_cmd="bash"
    dev3s "$target_prefix" exec $exec_cmd
    return
  fi

  dev3s "$target_prefix" "$chosen_command"
  return
}

run_devpods
```
