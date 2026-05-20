# K3s

### info
**k3s** is a lightweight, production-ready Kubernetes distribution by Rancher/SUSE.
It runs as a single binary, uses SQLite (or etcd) as backing store, and is ideal for
edge, IoT, and single-node developer clusters.

- Homepage: [https://k3s.io](https://k3s.io)
- GitHub: [k3s-io/k3s](https://github.com/k3s-io/k3s)
- Kubeconfig: `/etc/rancher/k3s/k3s.yaml` (root-owned by default)

### install
```sh evaluate
curl -sfL https://get.k3s.io | sh -
```

### check
```sh
command -v k3s > /dev/null 2>&1
```

### user-kubeconfig
Copy the cluster kubeconfig to the current user so `kubectl` works without `sudo`.

```sh
mkdir -p ~/.kube
sudo cp /etc/rancher/k3s/k3s.yaml ~/.kube/config
sudo chown $(id -u):$(id -g) ~/.kube/config
chmod 600 ~/.kube/config
```

### merge-kubeconfig
Merge k3s config into an existing `~/.kube/config` (preserves other clusters).

```sh
sudo KUBECONFIG=/etc/rancher/k3s/k3s.yaml:~/.kube/config \
  kubectl config view --flatten > /tmp/kubeconfig-merged \
  && mv /tmp/kubeconfig-merged ~/.kube/config \
  && chmod 600 ~/.kube/config
```

### enable-systempods
Allow pods to run systemd as PID 1. Requires privileged securityContext and the
`/sys/fs/cgroup` hierarchy to be writable inside the pod.
K3s uses cgroupv2 by default on modern kernels — verify with:

```sh
mount | grep cgroup2
```

To allow privileged pods cluster-wide (k3s default is permissive, but ensure
PodSecurity is not blocking):

```sh
sudo mkdir -p /etc/rancher/k3s
cat <<'EOF' | sudo tee /etc/rancher/k3s/config.yaml
kube-apiserver-arg:
  - "allow-privileged=true"
EOF
sudo systemctl restart k3s
```

### install-tailscale-operator
Install the official Tailscale Kubernetes operator. Pods with Tailscale sidecars
get their own Tailscale IP — no port-forwarding needed.

```sh
KUBECONFIG=${KUBECONFIG:-${HOME}/.kube/config}
KUBECONFIG=${KUBECONFIG} kubectl apply -f https://github.com/tailscale/tailscale-kubernetes-operator/releases/latest/download/operator.yaml
```

### secret-tailscale-operator
Create the OAuth client secret the operator uses to auth new pods.
Replace CLIENT_ID and CLIENT_SECRET with values from https://login.tailscale.com/admin/settings/oauth

```sh
KUBECONFIG=${KUBECONFIG:-${HOME}/.kube/config}
KUBECONFIG=${KUBECONFIG} kubectl create secret generic tailscale-operator-oauth \
  --from-literal=client_id=CLIENT_ID \
  --from-literal=client_secret=CLIENT_SECRET \
  -n tailscale
```

### secret-tailscale-authkey
Create a k8s Secret with the Tailscale auth key for pods that run tailscale internally.
Reads from `TAILSCALE_AUTHKEY` env var or `secrets get tailscale_authkey`.

```sh
KUBECONFIG=${KUBECONFIG:-${HOME}/.kube/config}
TAILSCALE_AUTHKEY=${TAILSCALE_AUTHKEY:-$(secrets get tailscale_authkey)}
KUBECONFIG=${KUBECONFIG} kubectl create secret generic tailscale-authkey \
  --from-literal=TS_AUTHKEY="${TAILSCALE_AUTHKEY}" \
  --dry-run=client -o yaml | KUBECONFIG=${KUBECONFIG} kubectl apply -f -
```

### status
```sh
sudo systemctl status k3s
```

### nodes
```sh
KUBECONFIG=${KUBECONFIG:-${HOME}/.kube/config}
KUBECONFIG=${KUBECONFIG} kubectl get nodes -o wide
```

### start
```sh
sudo systemctl start k3s
```

### stop
```sh
sudo systemctl stop k3s
```

### restart
```sh
sudo systemctl restart k3s
```

### enable-service
```sh
sudo systemctl enable k3s
```

### disable-service
```sh
sudo systemctl disable k3s
```

### logs
```sh
sudo journalctl -u k3s -f
```

### uninstall
```sh
/usr/local/bin/k3s-uninstall.sh
```

### tls-san-tailscale
```sh
# Add Tailscale IP to k3s TLS SANs so remote kubectl works over Tailscale.
# Detects the Tailscale IP automatically via `tailscale ip`.
TS_IP=$(tailscale ip -4 2>/dev/null)
if [[ -z "$TS_IP" ]]; then
  echo "error: could not detect Tailscale IP — is tailscale running?"
  return 1
fi
echo "Tailscale IP: ${TS_IP}"

# Write tls-san config
sudo mkdir -p /etc/rancher/k3s
cat <<EOF | sudo tee /etc/rancher/k3s/config.yaml
tls-san:
  - "${TS_IP}"
EOF

# Remove old server certs so k3s regenerates them on next start
sudo rm -f /var/lib/rancher/k3s/server/tls/server-ca.crt
sudo rm -f /var/lib/rancher/k3s/server/tls/server-ca.key
sudo rm -f /var/lib/rancher/k3s/server/tls/serving-kube-apiserver.crt
sudo rm -f /var/lib/rancher/k3s/server/tls/serving-kube-apiserver.key

sudo systemctl restart k3s
echo "k3s restarted — new cert will include ${TS_IP}"
echo "Update your kubeconfig server URL to: https://${TS_IP}:6443"
```

### kubeconfig-remote
Save a kubeconfig for this k3s cluster (named after the hostname) to
`~/.kube/k3s-<hostname>.yaml`. Run on the k3s host, then copy the file to the
remote machine and merge it — or point `KUBECONFIG` at it.
On the remote, use `dev3s switch` to pick this context.

```sh
TS_IP=$(tailscale ip -4 2>/dev/null)
if [[ -z "$TS_IP" ]]; then
  echo "error: could not detect Tailscale IP — is tailscale running?"
  return 1
fi
CTX="k3s-$(hostname -s)"
OUT="${HOME}/.kube/${CTX}.yaml"
mkdir -p ~/.kube

# Build a properly named kubeconfig using kubectl
sudo KUBECONFIG=/etc/rancher/k3s/k3s.yaml kubectl config view --raw \
  | sed "s|127.0.0.1|${TS_IP}|g; s|https://localhost|https://${TS_IP}|g" \
  > /tmp/k3s-raw.yaml

KUBECONFIG=/tmp/k3s-raw.yaml kubectl config rename-context default "${CTX}" 2>/dev/null || true
KUBECONFIG=/tmp/k3s-raw.yaml kubectl config set-cluster "${CTX}" \
  --server="https://${TS_IP}:6443" &>/dev/null
KUBECONFIG=/tmp/k3s-raw.yaml kubectl config view --raw > "${OUT}"
rm -f /tmp/k3s-raw.yaml
chmod 600 "${OUT}"

echo "Saved: ${OUT}  (context: ${CTX})"
echo ""
echo "On the remote machine:"
echo "  scp $(hostname -s):${OUT} ~/.kube/${CTX}.yaml"
echo "  KUBECONFIG=~/.kube/config:~/.kube/${CTX}.yaml kubectl config view --flatten > /tmp/merged"
echo "  mv /tmp/merged ~/.kube/config && chmod 600 ~/.kube/config"
echo "  dev3s switch ${CTX}"
```

